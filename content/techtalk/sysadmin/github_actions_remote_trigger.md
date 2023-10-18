+++
title =  "Github Actions Remote Trigger"
description = "Github Actions Remote Trigger"
date = "2023-10-18T14:06:39-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

[GitHub actions](https://github.com/features/actions) is a popular option to automate all common software workflows such as test, build and deploy. [Jenkins](https://www.jenkins.io/) was the most popular option for doing the same tasks; with that said, a challenge to connect two platforms will be: How to invoke GitHub action workflows from a Jenkins pipeline. In this technical post, we will go over this approach and we will describe what we need to do. The first step is to create a [fine-grained token](https://github.blog/2022-10-18-introducing-fine-grained-personal-access-tokens-for-github/) to get access with some privileges to the target GitHub repository. You can do this in Settings > Developer settings > Personal access tokens > Fine-grained tokens > Generate new token with these permissions:

## Repository Permission
Read access to:

- Actions variables
- Deployments
- Environments
- Metadata
- Secrets

Read and Write access to:

- Action
- Code (Contents)
- Workflows

Then, you will need to add a `repository_dispatch` event as you can see in this example:

```yaml
name: Playwright Tests
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  repository_dispatch:
  schedule:
    - cron:  '0 21 * * *'
env:
  APPLITOOLS_API_KEY: ${{ secrets.APPLITOOLS_API_KEY }}
  USERNAME: ${{ secrets.USERNAME }}
  PASSWORD: ${{ secrets.PASSWORD }}
jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: 18
    - name: Install dependencies
      run: npm ci
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    - name: Run Playwright visual tests
      run: npx playwright test visual --project chromium
    - name: Run Playwright e2e tests
      run: npx playwright test e2e
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

The last step is to create a HTTP request

```bash
curl --request POST \
  --url 'https://api.github.com/repos/josdem/${YOUR_REPOSITORY}/dispatches' \
  --header 'Accept: application/vnd.github+json' \
  --header 'Authorization: Bearer ${TOKEN}' \
  --data '{"event_type: "Called from API request"}'
```

where:
- `${YOUR_REPOSITORY}` Repository with workflow you want to run
- `${TOKEN}` Your fine-grained generated token


To browse a project with this implementation go [here](https://github.com/josdem/playwright-vetlog), to download the project:

```bash
git clone git@github.com:josdem/playwright-vetlog.git
```

[Return to the main article](/techtalk/sysadmin)
