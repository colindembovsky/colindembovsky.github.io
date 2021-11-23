---
layout: post
title: 'Displaying Help for Custom CodeQL Queries'
date: '2021-11-23 01:22:01'
image: /assets/images/2021/11/custom-codeql-help/color-interrogation-symbols.jpg
description: >
  The latest release of CodeQL CLI now includes the ability to display help files for custom queries. In this post I walk through how to get your custom help files to display.
tags:
- security
---

1. TOC
{:toc}

> Image from dashu83 on [www.freepik.com](https://www.freepik.com/photos/background)

A few months ago, I created a [repo](https://github.com/colindembovsky/custom-codeql-python) for experimenting with a custom CodeQL query. I [blogged]({% post_url 2021-02-09-custom-codeql %}) about how I went about that. One of the limitations I encountered was that custom query help files were not displayed, even when you created them.

Fortunately, the [latest release of CodeQL tools](https://github.blog/changelog/2021-11-23-display-help-text-for-your-custom-codeql-queries-in-code-scanning/) includes the ability to display custom query help files!

# Displaying Custom Query Help

To display custom query help files, you have to:

1. Create markdown for the help files
1. Update your code scanning Action to ensure you use CodeQL >= 2.7.1

## Creating Markdown Help Files

[Query Help Files](https://codeql.github.com/docs/writing-codeql-queries/query-help-files/) are XML files with the `.qhelp` extension. This is enough for the standard queries to display help information. However, for custom help files to be displayed, you must create the help files in markdown format.

You can do this by hand, but you can also use the `codeql cli`. You have to create the markdown file with the same file name as the `.qhelp` file. In my case, my help file is in the path `/.github/codeql/custom-queries/python/rmtree.qhelp`.

> **Note**: If you're writing the markdown by hand, then the style guide is [here](https://github.com/github/codeql/blob/main/docs/query-help-style-guide.md).

So how do I then run the command to convert the `.qhelp` to `.md`? I can install the `codeql cli` on my machine - or I can use a CodeSpace!

### Rabbit Trail: CodeSpace with CodeQL

I love [CodeSpaces](https://github.com/features/codespaces) since they eliminate some of the "setup tax" you have to pay to experiment. Here was a perfect opportunity to do some CodeQL in a CodeSpace container.

I opened up my repo in a CodeSpace, then opened the command palette and selected "Add Dev Container config file". In the wizard, I selected Python as my base environment (since the code in the repo is Python code) and then made sure to select Java as an installed utility (since CodeQL requires Java to run). I always install Git and the GitHub CLI in my containers, so selected those too.

VSCode generated the `devcontainer.json` file as well as the dev container `Dockerfile`. I added these lines to the `Dockerfile`:

~~~dockerfile
{% raw %}
RUN curl -OL https://github.com/github/codeql-cli-binaries/releases/download/v2.7.2/codeql-linux64.zip && \
    unzip codeql-linux64.zip && \
    mv codeql /opt
ENV PATH="/opt/codeql:${PATH}"
{% endraw %}
~~~

Adding commands to install the `codeql cli` into the container.
{:.figcaption}

The first line downloads the `codeql cli` release, unzips it and moves the unzipped files to `/opt/codeql`. I then update the `PATH`, prepending `/opt/codeql` so that you can just run `codeql` from anywhere.

Finally, I used the command palette to rebuild the dev container.

### Creating Markdown From QHelp

Now that I have a container with the `codeql cli` I am ready to convert to markdown. I just `cd` to the folder with the `qhelp` file, and run `codeql generate query-help --format markdown rmtree.qhelp > rmtree.md`. This dumped the markdown into a new file called `rmtree.md`. Then I added and commit the new file to my repo.

## Updating the Code Scan Action

Next, I had to open up and edit the code scanning workflow. Here are the original steps for CodeQL:

~~~yaml
{% raw %}
# Initializes the CodeQL tools for scanning.
- name: Initialize CodeQL
  uses: github/codeql-action/init@v1
  with:
    languages: ${{ matrix.language }}
    config-file: ./.github/codeql/codeql-config.yml

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v1
{% endraw %}
~~~

Original code scan steps.
{:.figcaption}

All I had to change was the tag for the Actions and adding `tools: latest` to the `init` step:

~~~yaml
{% raw %}
# Initializes the CodeQL tools for scanning.
- name: Initialize CodeQL
  uses: github/codeql-action/init@v1.0.24
  with:
    languages: ${{ matrix.language }}
    config-file: ./.github/codeql/codeql-config.yml
    tools: latest

- name: Perform CodeQL Analysis
  uses: github/codeql-action/analyze@v1.0.24
{% endraw %}
~~~

Udpated code scan steps.
{:.figcaption}

> **Note**: I had to add `tools: latest` to get the latest version of the `codeql cli` since the default tool version without that argument is `2.7.0` which does not include the logic for displaying custom query help files.

### Results

Now when the code scan runs and a match is found for my custom query, I get the full help file displayed in the Security alert!

![Custom help for a security alert](/assets/images/2021/11/custom-codeql-help/custom-help.png){: .center-image }

Custom help for a security alert.
{:.figcaption}

# Conclusion
CodeQL is about _developer-centric security_. If developers can understand what code is vulnerable, and how to fix it, then they'll write more secure code! Custom queries allows teams to add their own queries, and now that queries can display custom help files, the developer experience is even better.

Happy securing!