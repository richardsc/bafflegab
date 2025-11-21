+++
title = "Notes for setting up a simple hugo website from scratch"
date = 2025-11-18
draft = false
tags = []
+++

## Introduction

These notes grew out of a discussion within the DODG group at Dalhousie, around how to create a simple website that a student can use for promoting themselves and their work. While I have used [hugo](gohugo.io) for sometime to maintain my own [personal website](clarkrichards.org) and blog, because I don't update or build new sites frequently the process to get started was a little new. The below notes are meant to permit someone to create a simple and to teach enough about hugo to be able to customize it. Thanks are due to Github's copilot AI, for both making the process easier and harder at the same time (e.g. by generating content that I then had to correct).

The goal here is to create a page that could be used by an individual or research group, that contains: a landing page, sub-pages for things like "About" and "Projects", and a blog functionality that permits regular contributions/posts.

Hugo works by converting [markdown]()-formatted text files into html pages and organizes them and the relative style files using a "theme". To first order, creating a hugo site involves simply creating/editing markdown files (pages). Sub-pages (e.g. page "children") are created by creating directories and putting pages within them.


Things to include:

* how to resolve _root_ vs _subdirectory_ hosting on github pages (and how to make sure relative links, like images, work). This matters if the site is being deployed at e.g. `richardsc.github.io/` vs `richardsc.github.io/subsite/`
* if using a submodule to install the theme, make sure to update it after cloning with: `git submodule update --init --recursive`

## 1. Install hugo

I won't go into details on this, but on a mac this is most easily done with:

```bash
brew install hugo
```

after installing [homebrew](https://brew.sh/) of course. On linux, you can do the same (if you're using homebrew/linuxbrew as I do), or you can do:

```bash
sudo apt update
sudo apt install hugo
```

Note that using `apt install` will often install an older version of hugo. You can check your version with:

```bash
hugo version
```

## 2. Create a new site and initialize a git repository

```bash
hugo new site mysite
cd mysite
git init
```

## 3. Add a theme

Hugo works by making use of _themes_, which control most of the look and functionality of a site, and keep most of the actual content (i.e. the markdown files) separate from the code that controls how the site looks. There are many themes available at [themes.gohugo.io](https://themes.gohugo.io/). For this example, we'll use the [ananke](https://themes.gohugo.io/themes/hugo-theme-ananke/) theme, which is a simple and popular theme.

The theme files get installed in the `themes/` folder, and be added directly (e.g. by downloading and unzipping the theme files there), or by using git submodules. The latter is the preferred way, as it makes it easier to update the theme later. For this example we'll use the [ananke](https://github.com/theNewDynamic/gohugo-theme-ananke) theme. To add ananke as a submodule, do:

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```

## 4. Configure the site

To tell hugo which theme to use, and to set some basic site parameters, you need to edit the `config.toml` file in the root of your site folder. A minimal example for using ananke is below, where I have configured two menu items (About and Projects; we'll add content to those later).

```toml
baseURL = "https://example.com/"
languageCode = "en-us"
title = "My Hugo Site"
theme = "ananke"
enableRobotsTXT = true
publishDir = "docs"

[menu]
  [[menu.main]]
    identifier = "about"
    name = "About"
    url = "/about/"
    weight = 10

  [[menu.main]]
    identifier = "projects"
    name = "Projects"
    url = "/projects/"
    weight = 20
 ```

 Note also some of the required parameters, such as the `baseURL` (which controls the creation of links for the deployed site), and the `publishDir = "docs"`. The latter ensures that when the site is built, the rendered files all get put in the `docs/` folder of the site project. This is relevant later for how we will use Github pages to host the site for us.

## 5. Add content

### Home page content

First, to add content to the home page, you can create a file called `content/_index.md`, with something like:

```markdown
+++
title = "Home"
date = 2025-11-17T00:00:00Z
draft = false
+++

Welcome — this is the site homepage. Add a short intro and links to your blog and pages.
```

### "About" page content

To create content for the "About" page, which we have already linked from the main page based on the configuration in the `config.toml`, we simply need to create a file called `content/about.md`. An example of such a file is:

```markdown
+++
title = "About"
date = 2025-11-17T00:00:00Z
draft = false
+++

Write a short about blurb here. This is a normal Markdown file.

I want to write stuff here!
```

Note the header option `draft = false`, which signifies that the page _should_ be built by hugo (i.e. it is not a _draft_ page). Using `draft = true` is helpful when working on pages that aren't ready for building, e.g. so you can keep the source contained in git while working.

### "Projects" page content

For the "Projects" pages, it would be nice to be able to put each project in its own page, with the "Projects" page collecting the listing of the projects as links. To do this, create the folder `content/projects/`, and within it place a file called `_index.md` with content like:

```markdown
+++
title = "Projects"
date = 2025-11-17T00:00:00Z
draft = false
+++

## Projects

Below are project pages:

- [Project One]({{< relref "projects/project-one.md" >}})  <!-- preferred: works well if permalinks change -->
- [Project Two]({{< relref "projects/project-two.md" >}})  <!-- preferred: works well if permalinks change -->

You can also show summaries automatically if your theme supports section lists; otherwise add manual links like above.
```

Note the use of the `{{...}}` constructs (which is a hugo feature) along with the `relref` functionality, that allows the site to be able to create the correct link without having to specify it explicitly. Then, you can create individual pages for each project (e.g. `projects/project-one.md`), e.g.:

```markdown
+++
title = "Project One"
date = 2025-11-17T02:33:31Z
draft = false
+++

## Project One

Short description of Project One. Add details, images, links, etc.

- Feature A
- Feature B

You can reference other pages with the relref shortcode:
[Back to Projects]({{< relref "projects/_index.md" >}})
```

Create a `projects/project-two.md` similarly.

### Blog/posts

To make use of the "post" or blog functionality in hugo, we just have to create a markdown file in `content/posts/`. An easy way to do this is to use the archetype from the theme (or a custom one you create). To create a new post, do:

```bash
hugo new posts/my-first-post.md
```

This will create the above markdown file in `content/posts/`, based on the archetype of the theme, which will look like:

```markdown
---
title: "My First Post"
date: 2025-11-17T19:08:07-04:00
tags: []
featured_image: ""
description: ""
---
```

Note that this theme (ananke) uses yaml for the header information. Hugo can use both yaml and toml. To override the theme archetype, we can create a new one in `archetype/default.md`, e.g.:

```markdown
+++
title = "{{ replace .Name "-" " " | title }}"
date = {{ .Date }}
draft = true
tags = []
+++

Your post content goes here.
```

which if we use to create a new post (e.g. by doing `hugo new posts/my-first-post.md`) will give:

```markdown
+++
title = "My First Post"
date = 2025-11-17T19:14:56-04:00
draft = true
tags = []
+++

Your post content goes here.
```

Note the default setting of `draft = true`, which much be changed to `false` in order for hugo to build the page.

## Deploying to Github pages

While working on your site locally, you can use the `hugo server` command to build (and rebuild) any changes. To actually host the site on the internet, the easiest method is to use the Github pages service.

There are two ways to host your site on Github pages:

1. By building your site on your local machine to the `docs/` folder in your site repository, and then telling Github through the repository settings to look there for a Github pages site, and

2. By using Github Actions to build the site _on_ Github (e.g. after every push to the repository), and then deploying the built site to the `gh-pages` branch of the repository.

For this guide, I will only describe option 1, as it is the simplest to set up. However, the option 2 is preferable for sites that will be updated frequently, as it avoids the need to build and push the built files manually.

### Deploying to `docs/`

To deploy your site to the `docs/` folder, make sure the following option is set in your `config.toml`:

```toml
publishDir = "docs"
```

The next step is to configure Github to serve the site from the `docs/` folder. To do this, go to your repository on Github, then go to "Settings" -> "Pages", and under "Source", select "Deploy from a branch", then select the branch (e.g. `main` or `master`) and the folder (`/docs`). Save the settings.

To view your github-hosted site, go to `https://<your-github-username>.github.io/<your-repo-name>/`. If the site is contained in a repo named e.g. `<your-github-username>.github.io` then it will be hosted as the "root" site on Github pages.

## Some pitfalls to avoid

1. If using git submodules to install the theme, make sure to update it after cloning to a new machine with: `git submodule update --init --recursive`

2. There are some subtleties I haven't totally figured out when you are using a baseURL that isn't itself a _root_ URL. For example, for this site, the `baseURL` is `https://richardsc.github.io/bafflegab/`, which means that viewing the locally-built version has to be done at `https://localhost:1313/bafflegab/`, and it can make some of the relative linking (e.g. to images or other content) a little more complicated. One example is in the ["Projects"]({{< relref "projects/_index.md" >}})) page of this site, which uses the `relref` functionality from hugo.
