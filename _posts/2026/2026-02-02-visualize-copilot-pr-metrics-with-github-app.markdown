---
layout: post
title: Extract and visualize PR counts using the Copilot Enterprise Metrics API
date: '2026-02-02 09:00:00'
image: /assets/images/2026/02/copilot-pr-metrics.jpg
description: >
  Using a script in my open source repo, you can download enterprise Copilot usage data and visualize PR activity by humans and Copilot.
tags:
- github
- ai
---

1. TOC
{:toc}

In this post, I’ll show you how to download and visualize Copilot PR counts (for Coding Agent and Code Review Agent) using metrics from your GitHub Enterprise using a python script. You’ll end with the JSON usage report plus a PR summary chart that visualized PRs created by humans and Copilot Coding Agent (CCA), and number of Code Reviews from humans and Copilot Code Review (CCR). This will help you keep tabs on how much usage you're getting from CCR and CCA.

> **Note:** This visualization isn't very sophisticated given the limits of the PR metrics available. It would be great to see and compare lead times for PR merges (time from open to merge) and compare those PRs with to those without CCR reviews - or even to split out by org or team! But this visualization at least gives you an idea at a coarse Enterprise level how many PRs are created by CCA and how many PRs are reviewed by CCR.

The code for this post is in [this repo](https://github.com/colindembovsky/copilot-pr-metrics).

## Prerequisites

You’ll need:

- Enterprise owner access in GitHub.
- A GitHub App installed in your enterprise (instructions in the sample repo).
- Python 3 and `pip`.

> **Assumptions:** You have permission to create and install GitHub Apps in your GitHub enterprise. If you do not, ask your enterprise admin to help. The "app" is more like a service account - there's no code at all!

## Step 1 — Create the GitHub App and capture IDs

Create a GitHub App at the enterprise level, capture the IDs and download the private key `pem` file. The full instructions are detailed in the [README file](https://github.com/colindembovsky/copilot-pr-metrics).

## Step 2 — Clone the repo and install dependencies

Use the following commands to clone the repo, activate a python virtual environment and install the dependencies.

```bash
{% raw %}
git clone https://github.com/colindembovsky/copilot-pr-metrics.git
cd copilot-pr-metrics
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
{% endraw %}
```

## Step 3 — Run the script

Run the script with your app details to fetch the latest 28-day enterprise usage report and generate PR metrics. You can enter the values as args or create a `test.env` file in the repo root like this:

```bash
{% raw %}
APP_ID=123456
INSTALLATION_ID=987654321
PRIVATE_KEY=path/to/app-key.pem
ENTERPRISE=your-enterprise-slug
API_BASE=https://api.github.com
OUTPUT=metrics-YYYY-MM-DD.json
{% endraw %}
```

Run the python command to download all the usage data as well as generate the chart of PR counts.

```bash
{% raw %}
python copilot_metrics.py \
  --app-id <GITHUB_APP_ID> \
  --private-key <PATH_TO_PRIVATE_KEY_PEM> \
  --installation-id <APP_INSTALLATION_ID> \
  --enterprise <ENTERPRISE_SLUG>
{% endraw %}
```

## Step 4 — Review the outputs

The script downloads the enterprise Copilot usage report (last 28 days) and produces a PR summary chart showing human PRs and Copilot PR activity for Copilot Coding Agent and Copilot Code Review. For an example chart, see the repository’s [sample-chart.png](https://github.com/colindembovsky/copilot-pr-metrics/blob/main/sample-chart.png).

## Alternatives and tips

- For automation, schedule the script in a CI workflow or a cron job and version the JSON output in a metrics repository or create an Issue with the chart image.
- Use the `--output` flag or the `OUTPUT` variable to keep a dated archive of metrics files.

## Conclusion

You now have a repeatable way to download Copilot usage data and visualize PR activity using a GitHub App. From here, you can automate collection and build trend reporting.

Happy measuring!