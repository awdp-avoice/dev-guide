# AWDPI website development guide

_Last updated: 2025-12-23_

## Prerequisites

- You need to know the basics of git, GitHub, HTML, CSS, Sass, JavaScript, Node.js, and npm.
- It's also good to learn the basics of [Hugo](https://gohugo.io/) if you've never used it before.
- If you don't understand anything or get stuck anywhere, AI tools likely can answer most of your questions.

## First steps

- You need to be a member of the GitHub organization (https://github.com/awdp-avoice) to access the repos. An owner can add you as a member or owner of the organization. [Ava Wang 王硕雅](https://github.com/avawang2024) is currently an owner. If she isn't available, here are some other owners whom I don't know:
  - [Amy-RZ](https://github.com/Amy-RZ)
  - [awdp-admin](https://github.com/awdp-admin)
  - [Nancy](https://github.com/jwhhh)
  - [Cara Qing Ge](https://github.com/qg2125)
  - [Yarhanry](https://github.com/Yarhanry)

## Developing the website

- The website is deployed at https://awdpi.org/.
- The main repo is https://github.com/awdp-avoice/avoice-site-hugo.
- It's based on [Hugo](https://gohugo.io/). You need to install it on your computer such that the `hugo` command is available on your command line.
  - If you use [Nix](https://nixos.org/), check out `/flake.nix`. You might only need to run `nix develop`.
- Clone the repo to your computer. You might want to add `--recurse-submodules` because `/themes/coHub` is a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules).
- In the repo, run `hugo server` to start the local development server. Then you can view the built website in your browser (default address http://localhost:1313/).
- When you push changes to the `main` branch (directly or via a pull request), the [Deploy to GitHub Pages](https://github.com/awdp-avoice/avoice-site-hugo/actions/workflows/gh-pages.yml) GitHub Action will run, which will build the website and push it to the [awdp-avoice.github.io](https://github.com/awdp-avoice/awdp-avoice.github.io) repo, which will trigger [another GitHub Action](https://github.com/awdp-avoice/awdp-avoice.github.io/actions/workflows/pages/pages-build-deployment) to publish it to https://awdpi.org/.

### Adding WeChat articles

- Adding WeChat articles (公众号文章) is a relatively new feature. A small batch of articles has been added [here](https://awdpi.org/zh-cn/stories/). We need to add the remaining articles, and periodically add the latest articles.
- Ask 池一诺 to export the articles as HTML + assets.
- The `index.html` is bloated and many of the assets are huge, so we need to process them with the `process-wechat.js` script. It's good to have a basic understanding of the code before using it. You need to install [ImageMagick](https://github.com/ImageMagick/ImageMagick) and [pnpm](https://pnpm.io/) (unless you use Nix) and run `pnpm install` before running the script.
- Before running the script, check some of the `index.html` files in the browser, and copy the images you don't want in the output (like the 10MB gif) from `assets` to `/known-images-to-remove` in the repo (also check out `README.md` there).
- Put the articles you want to add in the `/temp` directory in the repo. Don't put too many at once, because it will make it harder to debug or rerun the script. Run `pnpm process`. The script will:
  - Strip unnecessary elements from `index.html` and create `index-clean.html` next to it, which you can open in the browser and review.
  - Reduce the size of the images actually used with ImageMagick and put them in `/static/images/illustrations/stories/[article-id]`, where `[article-id]` is the article publish date + a short hash of the article name (to distinguish articles published on the same day).
  - Extract the `<body>` element from the HTML, minify it, add some Markdown header, and put it `/content/chinese/stories/[article-id].md`.
- After processing, check the size of each new directory in `/static/images/illustrations/stories`. The average size should be about 100KB. If you see any directory that's too big, there are probably some unnecessary images that aren't removed. You need to add them to `/known-images-to-remove` and rerun the script, probably as `pnpm process [dir-name]`, where `[dir-name]` is the name of the directory containing the only article you want to reprocess. It's important to keep the article sizes down because we have hundreds of articles.

### Problems with the Hugo site

- Hugo is fast, but it's not very flexible or extensible. You need awkward template syntax for complex logic. It's unnatural to add JavaScript or TypeScript. It doesn't play well with modern UI frameworks like React or Svelte.
- The site uses [Bootstrap](https://getbootstrap.com/), which is largely deprecated by [Tailwind CSS](https://tailwindcss.com/). Worse still, it uses the old Bootstrap v4 because the [coHub theme](https://github.com/StaticMania/hugo-coHub/tree/8a28ed94dbf32fb8f89ad4be91c807bf77d8104f) is stuck at that version.

### Astro site

- Ideally we will migrate the website to [Astro](https://astro.build/) in [this repo](https://github.com/awdp-avoice/awdpi-site-astro). Astro solves all the problems above and is also very fast. You need to know the basics of TypeScript before learning/using it. The repo already contains some basic setup, including Tailwind CSS. Run `pnpm install` and then `pnpm dev` to start the development server.

## To-dos

- Add remaining WeChat articles.
- Remove the large "exit" button in the top-right corner of the website.
- Translate all Chinese content to English, and ideally more languages. Exactly how to do this requires some discovery and discussion. There are a few axes of variations in the potential approach:
  - Do we only translate to English or also other languages? Which languages?
  - Do we commit the translated content in the repo? Or do we do something like automatic whole-page translations in the browser using Google Translation service?
  - If we commit the translated content in the repo, we probably want to use LLMs to do the translations, but which LLM, and how to feed the content? Do we copy-paste manually or use an API? How much will it cost? How to deal with the deeply nested content in the WeChat article HTML files?
- Migrate to Astro, ideally with new designs from "kylan" (this is his/her WeChat name).