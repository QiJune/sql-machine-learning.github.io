## SQLFlow.org

This repository holds the content of [SQLFlow.org](https://sql-machine-learning.github.io/). We rely on the [Github Pages](https://pages.github.com/) to convert Markdown files in this repo into HTML files. Github pages calls the converter [Jekyll](https://jekyllrb.com/docs/) , which churns through layout and other decorations.  In addition, we use Jekyll to call [Just-the-Doc](https://pmarsceill.github.io/just-the-docs/) to generate the corresponding documents.

## Repo Walkthrough

We see the following files and directories at the root of this repository:

- `Gemfile` lists Ruby dependencies required by Jekyll to build this repository into a Web site.
- `index.md`, `_config.yml`, `_data`, `_layouts` are [what Jekyll needs](https://jekyllrb.com/docs/structure/) to build the Web site.
- `pages` holds Jekyll [Pages](https://jekyllrb.com/docs/pages/).
- `assets` includes figures, images, and Javascript files.
- `doc_index` holds template files of categories of documents generated by Jekyll and Just-the-Doc.
- `gohive`, `gomaxcompute`, `pysqlflow`, `sqlflow` are git-submodules to other repositories in the same organization.  Jekyll and Just-the-Doc convert Markdown files in these git-submodules and listed in `/_config.yml` into the documentation.  Also, `index.html` files in these subdirectories redirect URLs like https://sqlflow.org/gohive to https://github.com/sql-machine-learning/gohive

## Instructions for serving a local SQLFlow website 

To run SQLFlow website we need to install Jekyll and its dependencies. However, there are different versions of Jekyll, making it hard to reproduce runtime errors. Therefore, we recommend using dockerized Jekyll, which includes a specifici version (3.8) of Jekyll and the dependencies in the image. Note for Mac users, please install [Docker for Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac); other tools like Docker Toolbox cannot expose Jekyll port in the local host.

**Step One**: Clone SQLFlow repo to your local machine. For potential contributors, please fork the repo and then make a clone.

```bash
git clone https://github.com/sql-machine-learning/sql-machine-learning.github.io
```

**Step Two**: Update markdown files. Most of the document markdown files and redirect `index.html` files can be found in the adjacent repos like SQLFlow, GoHive, PySQLFlow. To include them into the website, we create git submodules like `/sqlflow` then point them to the right repo. If you are going through the instruction for the first time, please use below command to include the markdown files before running Jekyll serve. Whenever there are updates to markdown file in other repo, please run the command again without --init to update the content. Otherwise, the updated content won't show up. 

```bash
git submodule update --init --recursive --remote
```

**Step Three**: Fire up docker run under the repo root. Note the command will pull the image from the remote automatically. Also, make sure to replace `b90ffea2` with your personal access token. It takes a few seconds to generate the token by following [this document](https://help.github.com/en/articles/creating-a-personal-access-token-for-the-command-line). Now you can access SQLFlow.org from http://localhost:4000 with exact the same content.

```bash
cd sql-machine-learning.github.io
docker run --rm -it \
   -v $PWD:/srv/jekyll \
   -e JEKYLL_GITHUB_TOKEN=b90ffea2 \
   -p 4000:4000 \
   jekyll/jekyll:3.8 \
   jekyll serve --incremental
```

## Jekyll Pages, Documents, and Redirects

The generated website serves the four kinds of contents below:

1. Jekyll [Pages](https://jekyllrb.com/docs/pages/).  

   Each source file is a Markdown file in directories like `/` and `/pages`.  In the [front matter](https://jekyllrb.com/docs/front-matter/) of the Markdown file, the `layout` tag refers to a layout template file. For example, in `/index.md` there is a line:

   ```yaml
   layout: index
   ```

   which tells Jekyll to use `/_layouts/index.html` as the layout template when rendering `/index.md` into `/index.html`.


1. Redirect Files

   Some `index.html` files, for example, `/gohive/index.html` and `/sqlflow/index.html`, are handwritten or generated by [`goredirects`](https://github.com/bramp/goredirects).  These files contain HTML directives that redirect https://sqlflow.org/gohive to https://github.com/sql-machine-learning/gohive.  Some `index.html` files also contain the `go-import` meta tags, so when users write `import "sqlflow.org/gohive` in their Go source files, they import `github.com/sql-machine-learning/gohive` indeed.


1. Documents

   Each document page is generated from a Markdown file in git-submodules like `/sqlflow`, which points to https://sql-machine-learning.github.io/sqlflow/.  We organize these document pages into a tree structure defined in `/_config.yml`.
   
   Jekyll allows document authors to specify the layout template by adding a [front matter](https://jekyllrb.com/docs/front-matter/) in their document Markdown files.  However, most document authors are developers who don't know anything about the front matter and follow the plain Markdown syntax.  So, in `/_config.yml` we write the following snippet

```yaml
# in _config.yml, using defaults to set default value of md files
defaults:
  - scope:
      path: "sqlflow"
    values:
      layout: "doc"
      nav_exclude: true # hide all files from nav
      grand_parent: Document
```

where `layout: doc"` specifies the default layout template file to be `/_layouts/doc.md`.

Another essential role of `/_config.yml` is to pick some Markdown files as document page source files.


1. The Navigation Bar

   In `_data/navigation.yml`, we define the navigation bar on the homepage.  Currently, there are three links there.  If we want more than three, we would need to edit the style of `_layout/index.html`.


## How to Contribute

When we make change to this repo, we can run Jekyll locally to create a Web site for a quick preview of our change.  If everything looks good, we can git-push our change to our fork repository, say, https://github.com/somebody/sql-machine-learning, and create a pull request.  Reviewers could go to https://somebody.github.io/sql-machine-learning.github.io/ for a preview.  (To enable the preview of your fork, you need to change the settings of your fork as described [here](https://github.com/sql-machine-learning/sql-machine-learning.github.io/issues/46)).

## Trouble Shooting

- Jekyll liquid syntax error
  
  Jekyll allows Markdown file authors to use the liquid braces `{{}}` to specify the front matter.  To avoid ambiguity, when we write double curly braces in our Markdown files, we must wrap them up into `{% raw %}` and `{% endraw %}` or we can try just don't use double curly braces.