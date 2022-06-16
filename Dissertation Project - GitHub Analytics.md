# Dissertation Project - GitHub Analytics

## Links

### GitHub

- Project folder: https://github.com/anthoor/gh-webhook-handler

## Notes for Outline Discussion

- The main objective of this project can be categorised into 3 different parts

- And the scope of this project is to make use of the available GitHub Enterprise Server APIs and WebHook handlers as an access point to gather to the available GitHub data

- GitHub raises event for all of the activities occurring on the platform such as:

  - PING

  - PUSH

  - COMMIT_COMMENT

  - CREATE

  - DELETE

  - ISSUE_COMMENT

  - ISSUES

  - PULL_REQUEST

  - PULL_REQUEST_REVIEW

- These events can be used to extract useful information from the GitHub

- After collecting the data next step is to perform analysis on the collected data to generate meaningful insights like:

  - state of open PRs
    - aged pull requests
    - number of pull requests opened and merged daily
    - lines of code per PR
  - inactive branches in a repository
  - quality of the PR reviews
  - ownership of different modules
  - stability of the pipeline
  - Modules in a repository which are more error pron

- Existing process and limitation

- This data collection can be passive using WebHooks

- In my project itself we have around 5-6 repositories which are active at all the time

  - The data flow to this repos will be huge in quantity

- This information is more oriented towards management - people, managing of products & project. Who can take some actions based on the information





































## GitHub REST API

## Open Points

- Obtaining `OAuth2` tokens using the [web application flow](https://docs.github.com/en/developers/apps/authorizing-oauth-apps#web-application-flow) for production applications
- [GraphQL](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#graphql-global-node-ids)
- [Authorising apps on your behalf](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#requests-from-user-accounts)
- [GitHub Java libraries](https://docs.github.com/en/rest/overview/libraries#java)
- Trunk based development
  - Grafana
- Extracting useful information from webhook payloads 
- Converting webhook payload using JPA libraries
  - Creating notmalized tables
- How does number of lines changed and number of files changed affects the quality of the PR?































## GitHub Webhooks

## Common Objects

- What is the action?
- Who triggered the action?
- In which repository did the action take place?
- In which organisation did the action take place?

## Delivery Headers

- What kind of event triggered the hook?

## Event Type Specific Payloads

### code_scanning_alert

### deploy_key

### deployment

### fork

### github_app_authorization

### gollum

### installation

### installation_repositories

### label

### marketplace_purchase

### membership

### meta



Available Event Types

- [check_run](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#check_run) - --PRQuality (force merge)

- [check_suite](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#check_suite)

- [commit_comment](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#commit_comment) - excluding the auto rebase bot commit & commit comments (for active PR)

- [create](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#create) - number of available branches ++

- [delete](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#delete) - number of available branch --

- [discussion](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#discussion) - low prio

- [discussion_comment](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#discussion_comment)

- [fork](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#fork) - external contributers / interrest in a repo (without access) - low prio

- [issue_comment](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#issue_comment) - low prio

- [issues](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#issues) - low prio

- [member](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#member) -low prio
- Push - number of lines changed can be retrieved using the pull_request event

- [milestone](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#milestone) - low prio

- [pull_request](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request)

- [pull_request_review](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request_review) - for future use - compare with master build status event (PR review quality)

- [pull_request_review_comment](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#pull_request_review_comment) - compare with master build status event (PR review quality)

- [push](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#push)

- [release](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#release)

- [repository](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#repository)

- [status](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#status)

## Open points

- do not update the quality of any variable from more that one event
- repository onboarding
  - query the initial state
- define web hook scope for the github app
  - configure the app in the organization
  - analyse the payload received in `setup URl` & `callback URL`
- Creating Table for GitHub events from the opensource library
- Where can we store the aggregated result ?
- Storing the possible aggregated results
- keeping only the live values which are necessory
- fetching the required values using query / view at run time
- clearing the unused data (e.g. when a PR gets merged)