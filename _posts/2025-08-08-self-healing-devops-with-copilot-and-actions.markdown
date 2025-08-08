---
layout: post
title: Self-Healing DevOps with Copilot and Actions
date: '2025-08-08 09:00:00'
image: /assets/images/2025/08/failed-image.png
description: >
  Turn flaky or failing builds into a self-healing loop with GitHub Actions, GitHub Models, and Copilot. Automatically analyze failures, classify root cause, and open a remediation issue when it is not transient.
tags:
- ai
- actions
---

1. TOC
{:toc}

> Photo generated in ChatGPT

## Watch it in action

Here’s a short video showing the self-healing loop analyzing a failed run, classifying the issue, and opening a remediation issue which Copilot Coding Agent immediately fixes!

<div style="position:relative;padding-bottom:56.25%;height:0;overflow:hidden;max-width:100%;">
  <iframe class="center-image" src="https://www.youtube-nocookie.com/embed/N53ddmqmABg?rel=0&modestbranding=1" title="Self-healing DevOps demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" loading="lazy"></iframe>
</div>
Watch the self-healing loop analyze a failed run, open a remediation issue and fix it.
{:.figcaption}

## Why this matters

Build failures can be expensive. They interrupt flow, burn time on triage, and often hide what actually needs fixing. Most failures fall into a few categories: transient network issues, dependency problems, broken code, configuration issues, or test failures.

What if your pipeline could diagnose itself - and even propose a fix?

In this post, I’ll show you how to wire up a self-healing loop in GitHub Actions using GitHub Copilot (via the models API wrapped in https://github.com/actions/ai-inference) to:

- Analyze failed workflow runs automatically
- Classify the failure with a short summary and a clear plan
- Skip transient failures
- Open a labeled remediation issue for non-transient failures
- Assign the issue to Copilot Coding Agent (CCA) if code work is needed

The goal is simple: shrink time-to-understanding and time-to-action automatically.

## Prerequisites

- Workflows built with GitHub Actions
- A Personal Access Token (PAT) with the following permissions:
  - Account-level: Models (read)
  - Org-level: Issues (read/write), Actions (read)
  - This PAT should be saved as a repo secret like `AUTO_REMEDIATION_PAT`

> Note: The workflow requires the `models: read` permission and uses the repo `GITHUB_TOKEN` for model inference, plus a PAT for any assignment actions that the default token cannot perform.

## How it works

At the heart of the solution are two pieces:

1. A reusable prompt that tells the model exactly how to analyze logs and specifies a JSON schema for a structured output.1. A workflow that triggers on failed runs, calls the inference task, parses the JSON, and then creates or updates a remediation issue with labels. For certain categories, it assigns the issue to Copilot.

### Categories and decisions

The prompt (outlined below) constrains responses to a strict JSON schema and a small set of categories like code, test, config, dependency, infrastructure or quality. The category can be used to guide the next steps.

If the analysis marks `transient: true`, we log and stop. If `false`, we open or update a remediation issue with labels like `auto-remediation`, `workflow:<name>`, and `category:<type>`.

## Auto Analyze Build Failures Workflow

This workflow triggers on every completed run and filters to failed conclusions (excluding itself). It then:

1. Calls `actions/ai-inference@v1` with the prompt file and GitHub MCP enabled so the model can fetch failed job logs using the `github-mcp-server`.
2. Parses the JSON result (category, summary, plan, transient).
3. Stops if the failure is deemed to be transient. Otherwise, ensures labels exist, creates a remediation issue, and assigns Copilot when appropriate.

```yml
{% raw %}
name: Auto Analyze Build Failures

on:
  workflow_run:
    workflows: ["*"]
    types: [completed]

permissions:
  contents: read
  actions: write
  issues: write
  pull-requests: read
  models: read

jobs:
  analyze-failure:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'failure' && github.event.workflow_run.name != 'Auto Analyze Build Failures' }}
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Analyze build failure
      id: analyze
      uses: actions/ai-inference@v1
      with:
        prompt-file: '.github/models/failed-run-analyze.prompt.yml'
        enable-github-mcp: true
        token: ${{ secrets.GITHUB_TOKEN }}
        github-mcp-token: ${{ secrets.AUTO_REMEDIATION_PAT }}
        max-tokens: 10000
        input: |
          repo: ${{ github.event.repository.name }}
          owner: ${{ github.event.repository.owner.login }}
          workflow_run_id: ${{ github.event.workflow_run.id }}

    - name: Parse results
      id: parse
      uses: actions/github-script@v7
      env:
        RESPONSE_JSON: ${{ steps.analyze.outputs.response }}
      with:
        script: |
          const responseString = process.env.RESPONSE_JSON;
          if (!responseString) {
            core.setFailed('No response received from analysis step');
            return;
          }
          const responseJSON = JSON.parse(responseString);
          core.setOutput('category', responseJSON.category || '');
          core.setOutput('summary', responseJSON.summary || '');
          core.setOutput('plan', responseJSON.plan || '');
          core.setOutput('transient', responseJSON.transient || 'false');

    - name: Check for existing remediation issue
      if: ${{ steps.parse.outputs.transient == 'false' }}
      id: check-issue
      run: |
        workflow_name="${{ github.event.workflow_run.name }}"
        existing_issue=$(gh issue list \
          --repo "${{ github.repository }}" \
          --state open \
          --label "workflow:$workflow_name" \
          --label "auto-remediation" \
          --json number \
          --jq '.[0].number')
        echo "existing_issue=$existing_issue" >> $GITHUB_OUTPUT
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - name: Create remediation issue
      id: create-issue
      if: ${{ steps.parse.outputs.transient == 'false' }}
      run: |
        workflow_name="${{ github.event.workflow_run.name }}"
        workflow_url="${{ github.event.workflow_run.html_url }}"
        category="${{ steps.parse.outputs.category }}"
        existing_issue="${{ steps.check-issue.outputs.existing_issue }}"
        if [[ -n "$existing_issue" ]]; then
          echo "Skipping issue creation - existing issue #$existing_issue found"
          exit 0
        fi

        issue_body=$(cat << EOF
        ## Build Failure Analysis
        
        **Workflow:** [$workflow_name]($workflow_url)
        **Run ID:** ${{ github.event.workflow_run.id }}
        **Category:** $category
        **Branch:** ${{ github.event.workflow_run.head_branch }}
        **Commit:** ${{ github.event.workflow_run.head_sha }}
        
        ### Summary
        ${{ steps.parse.outputs.summary }}
        
        ### Remediation Plan
        ${{ steps.parse.outputs.plan }}
        
        ### Links
        - [Failed Workflow Run]($workflow_url)
        - [Repository](${{ github.event.repository.html_url }})
        
        ---
        *This issue was automatically created by the build failure analysis system.*
        EOF
        )
        
        # Ensure required labels exist (idempotent)
        gh label create "auto-remediation" \
          --description "Issues automatically created by build failure analysis" \
          --color "FF6B6B" \
          --repo "${{ github.repository }}" || true
        gh label create "workflow:$workflow_name" \
          --description "Issues related to $workflow_name workflow" \
          --color "0052CC" \
          --repo "${{ github.repository }}" || true
        gh label create "category:$category" \
          --description "Issues categorized as $category" \
          --color "7057ff" \
          --repo "${{ github.repository }}" || true
        
        issue_url=$(gh issue create \
          --repo "${{ github.repository }}" \
          --title "🔧 Auto-Remediation: $workflow_name Build Failure" \
          --body "$issue_body" \
          --label "auto-remediation" \
          --label "workflow:$workflow_name" \
          --label "category:$category")
        issue_number=$(echo "$issue_url" | sed 's/.*\/issues\///')
        echo "issue_number=$issue_number" >> $GITHUB_OUTPUT
      env:
        GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    - name: Assign issue to Copilot (optional)
      if: ${{ steps.parse.outputs.transient == 'false' && steps.create-issue.outputs.issue_number != '' }}
      run: |
        category="${{ steps.parse.outputs.category }}"
        issue_number="${{ steps.create-issue.outputs.issue_number }}"
        if [[ "$category" == "code" || "$category" == "test" || "$category" == "config" ]]; then
          echo "Assigning issue #$issue_number to Copilot"
          # Query suggested actors to find Copilot agent id
          copilot_query='query {
            repository(owner: "${{ github.event.repository.owner.login }}", name: "${{ github.event.repository.name }}") {
              suggestedActors(capabilities: [CAN_BE_ASSIGNED], first: 100) {
                nodes { login __typename ... on Bot { id } }
              }
            }
          }'
          copilot_response=$(gh api graphql -f query="$copilot_query")
          copilot_id=$(echo "$copilot_response" | jq -r '.data.repository.suggestedActors.nodes[] | select(.login == "copilot-swe-agent") | .id')
          if [[ -n "$copilot_id" && "$copilot_id" != "null" ]]; then
            issue_query='query { repository(owner: "${{ github.event.repository.owner.login }}", name: "${{ github.event.repository.name }}") { issue(number: '$issue_number') { id } } }'
            issue_response=$(gh api graphql -f query="$issue_query")
            issue_id=$(echo "$issue_response" | jq -r '.data.repository.issue.id')
            assign_mutation='mutation { replaceActorsForAssignable(input: {assignableId: "'$issue_id'", actorIds: ["'$copilot_id'"]}) { assignable { ... on Issue { id } } } }'
            gh api graphql -f query="$assign_mutation" >/dev/null 2>&1 || echo "Copilot assignment failed"
          else
            echo "Copilot agent not available in this repository"
          fi
         else
           echo "Category '$category' does not require Copilot assignment"
         fi
      env:
        GH_TOKEN: ${{ secrets.AUTO_REMEDIATION_PAT }}
{% endraw %}
```

Notes:
- Line 1: Sets a clear workflow name to reference in labels and filters.
- Line 4-7: Triggers on all completed workflow runs; we’ll filter to failures in the job-level condition.
- Line 9-14: Grants least-privilege permissions; `models: read` is required for inference, `issues: write` for creating remediation issues.
- Line 17-18: Uses an `if` to only run when a workflow failed and to avoid analyzing itself.
- Line 21-22: Checks out the repository so the prompt file is available in the workspace.
- Line 25-35: Calls actions/ai-inference with the prompt, enabling GitHub MCP so the model can fetch logs; passes both `GITHUB_TOKEN` and a PAT for MCP actions that need expanded scopes. Passes in `repo`, `owner`, and `workflow_run_id` as inputs. The MCP integration in the ai-inference Action handles fetching failed job logs for analysis.
- Line 38-51: Parses the JSON response into outputs (`category`, `summary`, `plan`, `transient`) for downstream steps; fails early if there’s no response.
- Line 54-66: Uses gh CLI to detect an existing open remediation issue by labels to prevent duplicates; gated on `transient == 'false'`.
- Line 69-115: Creates labels idempotently, opens a structured remediation issue, and captures the new issue number.
- Line 118-146: Optionally assigns the issue to the Copilot Coding Agent for code-like categories by querying GraphQL for the agent id and replacing assignees.

## The analysis prompt

The prompt is a small contract: constrain the task, provide examples, and require a strict JSON response with a known schema. That keeps downstream steps simple and reliable:

```yaml
{% raw %}
name: Failed Run Autofix
model: openai/gpt-4.1
messages:
  - role: system
    content: >
      You are an expert DevOps engineer and build master with deep knowledge of
      CI/CD pipelines,  build systems, and software development workflows. You
      specialize in analyzing build failures  and providing actionable
      remediation plans for the OctoCAT Supply Chain Management application.

      Your task is to analyze failed CI/CD pipeline runs and determine:
      1. Whether the failure is transient (network issues, temporary service
      outages, etc.)
      2. The root cause and category of the failure
      3. A detailed plan for fixing the issue

      When analyzing logs, look for:
      - Compilation errors (TypeScript, build tool issues)
      - Test failures (unit tests, integration tests)
      - Dependency issues (missing packages, version conflicts)
      - Configuration problems (environment variables, config files)
      - Infrastructure issues (network timeouts, service unavailability)
      - Code quality issues (linting, formatting)

      Provide your analysis in the specified JSON format with clear, actionable
      recommendations.
  - role: user
    content: >
      ## CI/CD Pipeline Failure Logs

      Use the GitHub MCP `get_job_logs` tool to retrieve failed job logs from
      the specified workflow run.
        - Repo: {{repo}}
        - Owner: {{owner}}
        - Workflow Run ID: {{workflow_run_id}}

      Analyze the failure logs and provide a comprehensive assessment including
      whether this is a  transient failure, or if it requires code  changes.
      If code changes are needed, provide a detailed remediation plan. Keep the
      summary short and DO NOT include the entire build log.
responseFormat: json_schema
jsonSchema: |
  {
    "name": "failure_analysis",
    "strict": true,
    "schema": {
      "type": "object",
      "properties": {
        "transient": { "type": "boolean" },
        "summary": { "type": "string" },
        "plan": { "type": "string" },
        "category": {
          "type": "string",
          "enum": ["code","test","config","dependency","infrastructure","quality","repeat-transient"]
        }
      },
      "required": ["transient","summary","plan","category"],
      "additionalProperties": false
    }
  }
{% endraw %}
```

Notes:
- Line 1: Human-friendly prompt name for traceability in logs.
- Line 2: Sets the target model; pick a capable, response-format compliant model.
- Line 3-33: System message establishes expertise, scope, and expected behavior focusing on CI/CD analysis.
- Line 35-49: User message instructs the model to use GitHub MCP to fetch logs and defines inputs the workflow passes.
- Line 50: Forces a strict JSON response to simplify parsing downstream.
- Line 51-74: Defines a compact JSON schema with `transient`, `summary`, `plan`, and `category` with an enum to keep 
categories consistent.

In general, you'll want to keep the entire prompt and the `jsonSchema` small and strict. Your downstream logic gets much simpler when the response is tightly constrained. The ai-inference task is also going to error out if there are too many steps or too much content, so watch for this.

## Conclusion

You can turn noisy failures into crisp, actionable work in minutes. The combination of GitHub Actions, Copilot API, and Copilot Coding Agent gives you a lightweight, reliable self-healing loop: automatic analysis, clear categorization, and immediate remediation tracking. Happy healing!