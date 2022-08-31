---
layout: post
title: 'Fine Tuning CodeQL Scans using Query Filters'
date: '2022-08-30 01:22:01'
image: /assets/images/2022/08/fine-tune/target.jpg
description: >
  CodeQL is a fantastic Static Analysis Scanning Tool (SAST). It can be enabled quickly using Actions, but it can be hard to figure out how to fine-tune which queries are run. In this post I'll cover using Query Filters to fine-tune your CodeQL scans.
tags:
- security
- actions
---

1. TOC
{:toc}

> Image by [Mauro Gigli](https://unsplash.com/@maurogigliphoto?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/target?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

CodeQL scanning involves four phases:

1. **Initialize** - where an empty database is created and hooks are configured into the compiler for compiled languages
1. **Build** - where the database is populated from the code-base
1. **Query** - where queries are executed against the database - results are output to a SARIF file
1. **Upload** - where the SARIF file is uploaded to the GitHub repo

> **Note:** The default [`analyze` Action](https://github.com/github/codeql-action/blob/main/analyze/action.yml) will query and upload in a single step.

In the initialize phase, you specify which of the [supported languages](https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/) you want to analyze. You can also (optionally) specify the set of queries you want to run.

## Query Organization

Queries are the lowest level artifact in CodeQL scans. These are T-SQL like in syntax (with `from`, `where` and `select` clauses), but also have very powerful abstractions like `predicate`, `class` and `override`.

Queries are typically grouped into _suites_. CodeQL _packs_ can contain queries and suites. Additionally, you can [_filter_ queries](https://codeql.github.com/docs/codeql-cli/creating-codeql-query-suites/#reusing-existing-query-suite-definitions) - which we'll get to shortly!

Before we move on, one more concept we need to understand is that queries have _[metadata](https://codeql.github.com/docs/writing-codeql-queries/metadata-for-codeql-queries/)_ associated with them. The metadata are more than just a way to describe the query - they are also critical for filtering.

Let's look at the metadata from a [query](https://github.com/github/codeql/blob/main/csharp/ql/src/Security%20Features/CWE-359/ExposureOfPrivateInformation.ql) in the [CodeQL](https://github.com/github/codeql) repo to examine some of the metadata:

~~~
{% raw %}
/**
 * @name Exposure of private information
 * @description If private information is written to an external location, it may be accessible by
 *              unauthorized persons.
 * @kind path-problem
 * @problem.severity error
 * @security-severity 6.5
 * @precision high
 * @id cs/exposure-of-sensitive-information
 * @tags security
 *       external/cwe/cwe-359
 */
{% endraw %}
~~~

A typical CodeQL metadata example.
{:.figcaption}

We'll use some of these metadata properties to filter - notably the `kind`, `security-severity`, `precision` and `tags`.

## Why filter?

If you do not specify a suite in the [CodeQL Action](https://github.com/github/codeql-action/blob/main/init/action.yml), then you'll get a default set of queries for the language you're scanning. However, the default set is a subset of all the queries. There are some queries that have higher or lower severity or different levels of "precision" (we'll discuss what that is later). Rather than give you _all_ the queries, the default setting _filters out_ some queries. [This file](https://github.com/github/codeql/blob/main/misc/suite-helpers/code-scanning-selectors.yml) contains the default set of filters.

> The default set of queries is called the `code-scanning` suite. Each language has a `.qls` (query suite) file that specifies the list of queries and applies the `code-scanning-selectors.yml` selector. For example, [this file](https://github.com/github/codeql/blob/main/csharp/ql/src/codeql-suites/csharp-code-scanning.qls) is the default code scanning suite for `csharp`.

You can also customize the [query suite](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-code-scanning#using-queries-in-ql-packs) by specifying other "standard" selectors: either `security-extended` or `security-and-quality`, which change the filter criteria by adding in additional queries that are excluded in the default selection.

Let's examine a couple of selectors and how they are specified, and then a couple of use-cases where we use selectors to specify a different set of queries to execute during the Analyse phase.

## Standard Selectors

If you look at the `includes` from the [standard selectors](https://github.com/github/codeql/tree/main/misc/suite-helpers) you'll see that [security-extended-selectors.yml](https://github.com/github/codeql/blob/main/misc/suite-helpers/security-extended-selectors.yml) selects queries that contain the `security` `tag`:

~~~yml
{% raw %}
- description: Selectors for selecting the security-extended queries for a language
- include:
    kind:
    - problem
    - path-problem
    precision:
    - high
    - very-high
    tags contain:
    - security
    ...
{% endraw %}
~~~

Selectors in the `security-extended-selectors.yml` file.
{:.figcaption}

By contrast, the [security-and-quality-selectors.yml](https://github.com/github/codeql/blob/main/misc/suite-helpers/security-and-quality-selectors.yml) file does **not** filter by that `tag`:

~~~yml
{% raw %}
- description: Selectors for selecting the security-extended queries for a language
- include:
    kind:
    - problem
    - path-problem
    precision:
    - high
    - very-high
    ...
{% endraw %}
~~~

Selectors in the `security-and-quality-selectors.yml` file.
{:.figcaption}

This means that the `security-extended` suite will only include queries that have `security` in their `tags` metadata, while the `security-and-quality` suite will include additional queries that do not contain this `tag`.

However, we can also filter on other properties - such as `kind`, `security-severity` or `precision`.

## Filtering by Security Severity

Last week I heard of a company using CodeQL that were hitting upper limits on the upload size of the SARIF file. They are scanning a large mono-repo and are getting a large number of results in the scan. Arguably, there are other issues at play here, but the team did not want to refactor their build or their codebase.

In this case, neither of the default suites works. Perhaps we need to focus just on the most critical alerts first - so we are going to want to filter by `security-severity`.

### Security Severity Levels

When you see a CodeQL alert, it is marked with `low`, `medium`, `high` or `critical` severity:

![CodeQL Alerts showing security severity](/assets/images/2022/08/fine-tune/codeql-alerts.png){: .center-image }

CodeQL Alerts showing security severity.
{:.figcaption}

However, if you look at the query metadata, these levels don't appear. That's because there is a [table](https://github.blog/changelog/2021-07-19-codeql-code-scanning-new-severity-levels-for-security-alerts/#about-security-severity-levels) that shows how GitHub calculates the level based on the `security-severity` number:

Severity|Score Range
:--:|:--:
None|0.0
Low|0.1 - 3.9
Medium|4.0 - 6.9
High|7.0 - 8.9
Critical|9.0 - 10.0
{:.stretch-table}

The mapping of severity to `security-severity` score.
{:.figcaption}

So how do we filter on security level?

## Query Filters

You can filter queries using _query filters_ in a configuration file. Then you just point the `init` action to the config file, and you're done! I'll use code from [this repo](https://github.com/colindembovsky/dotnet-webapi-boilerplate/) for the examples.

Here's and example of an `init` action that specifies a custom config:

~~~yml
{% raw %}
# file: '.github/workflows/codeql-high-severity.yml'
- name: Initialize CodeQL
  uses: github/codeql-action/init@v2
  with:
    languages: csharp
    config-file: ./.github/codeql/high-severity.yml
{% endraw %}
~~~

Specifying a custom config file for CodeQL.
{:.figcaption}

Let's then look at the custom config file:

~~~yml
{% raw %}
# file: '.github/codeql/high-severity.yml'
name: "Custom CodeQL Config for high/very high severity only"
disable-default-queries: true
queries:
  - uses: security-extended
query-filters:
  - include:
      precision:
      - high
      - very-high
      tags contain: security
      security-severity: /([7-9]|10)\.(\d)+/
{% endraw %}
~~~

A custom configuration to only include queries with `security-severity` >= 7.
{:.figcaption}

Notes:
1. First we specify a `name`.
1. We then disable the default queries.
1. We bring in the default `security-extended` queries.
1. We then apply a `query-filter`
1. The filter selects only queries that have a `high` or `very-high` precision and a `security` tag.
1. Finally, we use `regex` to include only queries that contain a numeric value >= 7

## Precision

Before we go on, what exactly is `precision`? This is a measure of how many false positives are likely to be returned by the query. Queries with higher precision will return fewer false positives, while queries with lower precision tend to yield more false positives.

When security professionals are analyzing code-bases or writing queries, they may want to dial down precision. However, teams that want to make security remediation _actionable_ should default to higher precision queries. The default setting for the out-the-box suites is `high` and `very-high` precision to ensure very few false positives.

> **Note:** Who decides on the precision? While the [CodeQL repo](https://github.com/github/codeql) is open-source and accepts community contributions, it is maintained by GitHub. Queries are rigorously tested and vetted, so the precision metadata is accurate.

## Widening the Filter

The filter above narrowed the number of queries that will be executed in the analysis phase. But we can go the other way too! Here's a snippet from the configuration for a set of lower precision queries that teams can use if they understand that they are going to get more false positives with this setting:

~~~yml
{% raw %}
# file: '.github/codeql/high-severity.yml'
name: "Custom CodeQL Config for lower precision"
disable-default-queries: true
queries:
  - uses: security-extended
  - uses: security-and-quality
query-filters:
- include:
    kind:
    - problem
    - path-problem
    - alert
    - path-alert
    precision:
    - low
    - medium
    - high
    - very-high
    tags contain:
    - security
    - correctness
    - maintainability
    - readability
- include:
    kind:
    - problem
    - path-problem
    precision:
    - medium
    problem.severity:
    - error
    - warning
    - recommendation
    tags contain:
    - security
...
{% endraw %}
~~~

A custom configuration to include more queries.
{:.figcaption}

Notes:
1. First we specify a `name`.
1. We then disable the default queries.
1. We bring in the both the default `security-extended` and `security-and-quality` queries.
1. We then apply a couple of `query-filter`s
1. The first filter `includes` every type of `kind`, `precision` and `tag`
1. The next filter `includes` queries with a security tag and all types of `problem.severity` (different from `security-severity`).
1. The remainder of the file is the same as the default selectors from the CodeQL repo

## Testing the Configurations

We can compare and contrast three scenarios:

Name|Description|Branch|Actions File|Config file
--|--|--|--|--
Default|A default scan (no custom config)|`main`|`.github/workflows/codeql-analysis.yml`|None
High Severity|A high-severity config to only include high and critical security queries|`high-severity`|`.github/workflows/codeql-high-severity.yml`|`.github/codeql/high-severity.yml`
Low Precision|A "low-precision" config to include more queries with lower precision and severity|`low-precision`|`.github/workflows/codeql-low-precision.yml`|`.github/codeql/low-precision.yml`

Three scenarios for CodeQL configuration.
{:.figcaption}

The code on all 3 branches is identical - the only reason I created them was for filtering the results in the Security tab.

### Adding Debug to the `init` Action

For the purposes of our exploration, I wanted to be able to analyze the SARIF results file after each scan run. To do this, I just added `debug: true` to the `init` action just below the `config-file`. This will zip up the scanning database and the results file as artifacts that can be downloaded - I am really only interested in the results file since we can compare results, but also because the results file includes the list of the queries that are executed during a scan!

### Executing the Scans

I've added a `workflow_dispatch` trigger to the workflow files - so you have to navigate to the Actions tab of the repo and queue a run. After queueing a run for each scenario (and selecting the corresponding branch) I downloaded the SARIF results files for comparison.
 
To count the number of results in the SARIF, I crafted a quick `jq` query:

~~~bash
{% raw %}
cat default-results.sarif | jq '.runs[0].results | length'
{% endraw %}
~~~

We can also figure out the count of queries. The language for this repo is `csharp` so we look for the `codeql/csharp-queries` tools extension in the file for the list of all the queries (`rules`) that were included in the analysis:

~~~bash
{% raw %}
cat default-results.sarif | jq '.runs[0].tool.extensions[] | select(.name == "codeql/csharp-queries") | .rules | length'
{% endraw %}
~~~

When we do the comparison, we get the following results:

Scenario|Rule Count|Result Count
:--:|:--:|:--:
Default|47|6
High Severity|35|5
Low Precision|159|74
{:.stretch-table}

The result and rule count for each scan.
{:.figcaption}

We can also see the counts in the Code Scanning tab in the repo. Just change the branch filter to see the different result counts:

![Default count](/assets/images/2022/08/fine-tune/results-default.png){: .center-image }

![High severity count](/assets/images/2022/08/fine-tune/results-high.png){: .center-image }

![Low precision count](/assets/images/2022/08/fine-tune/results-low.png){: .center-image }

CodeQL Alert counts for each scenario.
{:.figcaption}

# Conclusion

CodeQL is incredibly powerful - but there are times when you want to fine-tune the set of queries for analysis. Using Query Filters we can easily tweak exactly what we want to scan.

Happy scanning!
