+++
title =  "Playwright Reports Deployment"
description = "Playwright_reports_deployment"
date = "2023-06-18T15:46:49-04:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "playwright", "githubactions", "nginx"]
categories = ["techtalk", "code", "ui", "automation"]
+++

[Playwright reports](https://playwright.dev/docs/test-reporters) are essential in a project since they provide a quick analysis about pass/fail ratio, scenarios, browsers behaviour, logs, etc. In this technical post, we will go over two approaches to exporting reports: static HTML pages and [NGINX server](https://www.nginx.com/). **NOTE:** If you need to know how to setup [Playwright](https://playwright.dev/), please refer my previous post: [Playwright Getting Started](/techtalk/ux/cypress_getting_started/). Then, let's create a file named `static.yml` in your `${PROJECT_HOME}/.github/workflows` directory with this content:

```yaml
name: Deploy static content to Pages
on:
  push:
    branches: ["main"]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: "pages"
  cancel-in-progress: false
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Pages
        uses: actions/configure-pages@v3
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: "playwright-report/"
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```
That is right; using [upload-pages-artifact](https://github.com/actions/upload-pages-artifact) GitHub actions, we export our Playwright build-in reporter into GitHub pages. If you want to know more about how to deploy to GitHub Pages using custom workflows, go [here](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow). Whenever there is a push to the main branch of the repository, we will trigger a new exporting process; how cool is that? Here you can see the result from this repository [playwright-workshop](https://josdem.github.io/playwright-workshop/)

## Publishing Reports using NGINX

This approach uses an [NGINX server](https://www.nginx.com/). In order to do that, we will use another GitHub actions project: [scp-action](https://github.com/appleboy/scp-action). For authentication, we can use an [SSH key](https://www.ssh.com/academy/ssh-keys) which you can store inside a [repository secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets); also we need to specify `HOST`, `PORT`, and `USERNAME`. This is how our `.github/workflows/main.yml` in deploys section looks like:

```yaml
- name: Deploy using SSH key
        uses: appleboy/scp-action@v0.1.4
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          port: ${{ secrets.PORT }}
          key: ${{ secrets.SSH_KEY }}
          source: "playwright-report/index.html"
          target: "/usr/share/nginx/html"
```

That's it, whenever there is a push to the main branch of the repository, we will trigger a new deployment process to an external NGINX server. Here you can see the result from this repository [playwright-workshop](https://josdem.io/playwright-report/). To browse the code go [here](https://github.com/josdem/playwright-workshop), to download the project:

```bash
git clone git@github.com:josdem/playwright-workshop.git
```

[Return to the main article](/techtalk/ux)