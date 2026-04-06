---
name: post-references
description: 'Create or update blog post references in this site. Use when linking to another post, adding companion or previous post links, fixing series references, or converting plain text mentions into Jekyll post_url links in _posts.'
argument-hint: 'Describe the post reference you want to add or update'
---

# Post References

Use this skill when you need to link one blog post to another in this repository.

## What This Skill Does

- finds the correct target post in `_posts`
- uses the site-standard `post_url` pattern instead of hard-coded URLs
- keeps link text readable in the surrounding sentence
- helps avoid broken or deprecated post reference formats

## Procedure

1. Search `_posts` by title keywords, series name, or slug fragments.
2. Confirm the exact target file and post title before adding the link.
3. Write the link in markdown using Jekyll `post_url`.
4. In this repo, use the year-prefixed format inside the tag: `{% post_url YYYY/YYYY-MM-DD-slug %}`.
5. Prefer descriptive anchor text such as `previous post`, `companion post`, `episode one`, or the actual post title, depending on the sentence.
6. If you edited a post, run `bundle exec jekyll build` to verify the Liquid tag resolves cleanly.

## Examples

- `[previous post]({% post_url 2025/2025-08-11-eight-principles-agentic-software-delivery %})`
- `[episode one]({% post_url 2026/2026-04-03-from-sprints-to-swarms-part-1-ai-made-code-cheap %})`

## Avoid

- hard-coded generated URLs
- links to `_site`
- bare `_posts/...` paths in prose
- vague link text like `here`
- `post_url` tags that omit the year prefix used by this repo