---
layout: post
title: How to create a new blog post
tags: [lighthorse, new blog post]
excerpt_separator: <!--more-->
---

This post will walk you through creating a new Team Lighthorse blog entry.

<!--more-->

### Set Up Environment

Install Jekyll by following the instructions [here](https://jekyllrb.com/docs/installation/windows/). You will be installing the Ruby programming language and a runtime environment. Follow the instructions and it should have you install Jekyll and Bundler once the Ruby installer has finished. All-in-all this will set up the environment you'll need to build and run the blog site.

Once these steps have been completed, follow these instructions to build and run the TeamLighthorse blog.

- Open a terminal window inside VSCode
- type `git clone https://github.com/TeamLighthorse/teamlighthorse.github.io.git`
- Open the newly created folder inside the editor
- In your terminal window, type `bundler install`
- This should install all the dependencies for the site
- Once this finished, type `jekyll build`. This will build the blog site.
- Once finished, type `jekyll serve`. This will serve up the site on localhost port 4000.

### Creating a new blog post
- Create new markdown file inside the folder "__posts_"
--File name should be in the format: _yyyy-MM-dd-title-separated-by-dashes.md_
- Type up your content using markdown syntax
- Add any metadata that you may want

Example
```
---
layout: post
title: Welcome to Lighthorse!
tags: [lighthorse, introduction, neversayneigh]
excerpt_separator: <!--more-->
---
```
- Note the usage of "_excerpt_separator_". This will control what text shows up on the blog post excerpt in the blog post list.

### Testing Locally
- If the site is currently running locally via `jekyll serve`, any time you save your markdown file, the site will be rebuilt and you should be able to refresh your browser and see your changes.
- If the site isn't running, you should simply be able to run `jekyll serve` and the site will build and the local server will start up again.

### Publishing
Since we're using a Git repo and GitHub Pages, all we need to do is commit our changes and push them to the remote. GitHub Pages will then rebuild our site and our new content should be available almost immediately upon pushing.

Ensure you're on the master branch, then...
`git add -A`
`git commit -m "Your very helpful commit message."`
`git push`

### More Info
- More on blog post metadata [here](http://jovandeginste.github.io/2016/05/18/add-metadata-tags-to-jekyll-blog-posts.html).
- Jekyll markdown info [here](https://www.markdownguide.org/tools/jekyll/)