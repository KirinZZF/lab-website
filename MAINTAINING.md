# Maintaining the lab website

This guide is for whoever looks after the site next. You do not need to
know Jekyll, Ruby or web design. Almost every change is an edit to one
text file, made in the browser on GitHub.

## How the site works

GitHub Pages turns the files in this repository into the website every
time a change is pushed to the `main` branch. It takes about a minute.
Watch progress under the repository's **Actions** tab; a green tick means
the new version is live.

Content and appearance are kept apart:

| To change… | Edit this file |
|---|---|
| Lab name, address, phone, email, demo link, site address | `_config.yml` |
| Menu items | `_data/navigation.yml` |
| People (join, leave, photos, bios) | `_data/people.yml` |
| Publications | `_data/publications.yml` |
| News | `_data/news.yml` |
| Research theme cards and Research page | `_data/research.yml` |
| Home page text | `index.html` (text between the tags) |
| TurbineCore project page | `research/turbinecore.md` |
| Contact page | `contact.html` |
| Colours, fonts, spacing | `assets/css/main.css` (variables at the top) |
| Header, footer, layouts | `_includes/`, `_layouts/` (rarely needed) |

## Editing on GitHub (no installation)

1. Open the file on github.com and click the pencil icon.
2. Make the change. Keep the indentation exactly as in the lines around it:
   YAML files use **two spaces**, never tabs.
3. Click **Commit changes**. Wait a minute and reload the site.

If the site stops updating, open **Actions**, click the failed (red) run,
and read the error. It is almost always a YAML indentation mistake or a
missing quote in the file you just edited. Titles containing a colon
(`:`) must be wrapped in double quotes.

## Common tasks

### Add a publication

Copy an existing block in `_data/publications.yml` and change it:

```yaml
- title: "Title of the paper"
  authors: [First Author, Second Author, Xiandong Ma]
  venue: "Journal name"
  type: journal          # journal | conference | preprint | thesis
  year: 2027
  details: "vol. 12, 104567"
  doi: 10.1016/j.xxxx.2027.104567
```

Lab members are printed in bold automatically when their name matches
`pub_name` in `people.yml`. Only list papers that are accepted or
published, never ones under review. An accepted paper can carry
`status: "Accepted / In press"`, shown as a badge and left out of the
home page list; remove the line once it has a volume and DOI.

### Figures

Only put figures from **published** papers, or result plots the authors
are happy to make public, in `assets/img/results/`. Do not post method or
architecture diagrams of unpublished work.

### Add a person

Add a block under the right group in `_data/people.yml`. For a photo,
upload a square image (about 600 × 600 px, under 300 KB) to
`assets/img/people/` and put its file name in `photo:`. Without a photo
the card shows the person's initials.

When someone leaves, move their block to the `Alumni` group and add a
`now:` line, for example `now: "Lecturer, University of X"`.

### Add news

Add a block at the **top** of `_data/news.yml`. The newest three appear
on the home page automatically.

### Add a page

Create a file such as `teaching.md`:

```markdown
---
layout: page
title: Teaching
permalink: /teaching/
---

Text in Markdown here.
```

Then add it to `_data/navigation.yml`.

### Show the TurbineCore live demo

Set `turbinecore_demo_url` in `_config.yml` to the demo's public address.
The "Live demo" buttons appear on the home and TurbineCore pages. Leave it
empty while a related paper is under double-blind review.

## Previewing locally (optional)

Only needed for large changes. With Ruby installed:

```bash
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000/lab-website/>.

## Handing over

* Give the next maintainer write access: repository **Settings →
  Collaborators**.
* Better, transfer the repository to a GitHub organisation owned by the
  group leader (**Settings → Transfer**), so the site does not depend on
  any one student's account. After a transfer, update `url`, `baseurl`
  and `repository` in `_config.yml`; GitHub redirects the old repository
  link, but not the old `github.io` address.
* Add yourself to the Alumni list on the way out.
