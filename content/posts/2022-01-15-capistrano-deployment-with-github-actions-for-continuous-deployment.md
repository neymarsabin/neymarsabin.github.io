## Capistrano Deployment with Rails
[capistranorb](https://capistranorb.com/) is an awesome tool for automation and deploying applications across multiple servers. Rails applications can also be easily deployed using `capistrano`. This post assumes you already have capistrano deployment up and running for rails application. Rails supports multiple environments and capistrano can deploy to multiple environments too. To deploy to environments `staging` or `production` following capistrano commands can be used:

```shell
bundle exec cap staging deploy:check # to run a dry run or test run of the deployment
bundle exec cap staging deploy    # to actually deploy 
```

## Deploying with Github Actions
Github Actions offers features to trigger workflows with a variety of events. You should be familiar with the syntax for [Github Actions Syntaxes](https://docs.github.com/en/actions/learn-github-actions). You can use variety of events to trigger deployment workflow like `pull_request`, `push`, and `workflow_dispatch`. We will configure `push` event to deploy application for two branches `staging` and `master`. `staging` branch is for `staging` environment and `master` branch is for `production` environment. We will have following steps configured:

- find environment to be used from branches
- send deployment started slack notification
- add key to ssh agent
- checkout to current branch
- setup ruby
- deploy check using capistrano
- deploy using capistrano
- success notification in slack [ conditional ]
- failure notification in slack [ conditional ]
- send honeybadger deploy event

#### Find Environment
We will have environment value used across our github actions configuration. This step will determine what is the environment to which the current branch is being deployed to and sets the environment value as the output of the step. Github Actions support setting output of one step to be used to another step. We can do that with some shell command like below:
```yaml
      # find and set environment based on current github branch name
      # set-oputput is used to set some variable as output of a step, that can be used later
      # for more info: https://docs.github.com/en/actions/learn-github-actions/workflow-commands-for-github-actions
      - name: Find Environment
        id: find_env
        run: |
          if [ ${{ github.ref_name }} == "master" ]; then
            echo "::set-output name=environment::production"
          else
            echo "::set-output name=environment::staging"
          fi
```
The environment value can be used with `${{ steps.find_env.outputs.environment }}` in the github actions workflow. 

#### Send Deployment Started Slack Notification
We will use `voxmedia/github-action-slack-notify-build@v1` action to send notifications to slack.

```yaml
      - name: Send Deployment Started Slack Notification
        id: slack
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy STARTED by ${{ github.actor }}
          color: warning
```
`${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}` expects you have stored `SLACK_NOTIFICATIONS_BOT_TOKEN` in the repository or organization level secrets. And similary for `SLACK_NOTIFICATIONS_CHANNEL_NAME`. 

#### Add key to ssh-agent
Deploy keys is needed to execute commands in our remote servers where the application is being hosted. Store the deploy keys in the repository secret in `KEYS_FOR_DEPLOYMENTS` secret. 
```yaml
      - name: Add key to ssh agent
        uses: webfactory/ssh-agent@v0.5.4
        with:
          ssh-private-key: ${{ secrets.KEYS_FOR_DEPLOYMENTS }}
```

#### Checkout to current branch
```yaml
      - uses: actions/checkout@v2

```

#### Setup Ruby
```yaml
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
```
This step uses `ruby/setup-ruby@v1` action. This action reads the ruby version to be installed from the file `.ruby-version` inside the repository. 

#### Deploy Check
```yaml
      - name: "Deploy check ${{ steps.find_env.outputs.environment }}"
        run: |
          bundle exec cap ${{ steps.find_env.outputs.environment }} deploy:check
```

#### Deploy
```yaml
      - name: "Deploy ${{ steps.find_env.outputs.environment }}"
        run: |
          bundle exec cap ${{ steps.find_env.outputs.environment }} deploy

```

#### Success Notification in Slack
```yaml
      - name: Notify slack success
        if: success()
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy SUCCESS by ${{ github.actor }}
          color: good
```
the `if: success()` statement will check if the above steps passed or failed. If passed will send notification to slack.

#### Failure Notification in Slack
```yaml
      - name: Notify slack fail
        if: failure()
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy FAILED by ${{ github.actor }}
          color: danger

```

#### Honeybadger Notification
```yaml
      - name: Send Honeybadger Notification
        uses: honeybadger-io/github-notify-deploy-action@v1
        with:
          api_key: ${{ secrets.HONEYBADGER_API_KEY }}
          environment: ${{ steps.find_env.outputs.environment }}
```

The final github actions workflow file looks like this: 
```yaml
name: Deploy API to Server

on:
  push:
    branches: [ staging, master ]

jobs:
  deploy:
    name: "Deploy ${{ github.ref_name }}"
    runs-on: [ self-hosted, "${{ github.ref_name }}", api ]


    steps:
      # find and set environment based on current github branch name
      # set-oputput is used to set some variable as output of a step, that can be used later
      # for more info: https://docs.github.com/en/actions/learn-github-actions/workflow-commands-for-github-actions
      - name: Find Environment
        id: find_env
        run: |
          if [ ${{ github.ref_name }} == "master" ]; then
            echo "::set-output name=environment::production"
          else
            echo "::set-output name=environment::staging"
          fi
      - name: Send Deployment Started Slack Notification
        if: success()
       n id: slack
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy STARTED by ${{ github.actor }}
          color: warning

      - name: Add key to ssh agent
        uses: webfactory/ssh-agent@v0.5.4
        with:
          ssh-private-key: ${{ secrets.KEYS_FOR_DEPLOYMENTS }}

      - uses: actions/checkout@v2

      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true

      - name: "Deploy check ${{ steps.find_env.outputs.environment }}"
        run: |
          bundle exec cap ${{ steps.find_env.outputs.environment }} deploy:check
      - name: "Deploy ${{ steps.find_env.outputs.environment }}"
        run: |
          bundle exec cap ${{ steps.find_env.outputs.environment }} deploy
      - name: Notify slack success
        if: success()
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy SUCCESS by ${{ github.actor }}
          color: good

      - name: Notify slack fail
        if: failure()
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_NOTIFICATIONS_BOT_TOKEN }}
        uses: voxmedia/github-action-slack-notify-build@v1
        with:
          channel: ${{ secrets.SLACK_NOTIFICATIONS_CHANNEL_NAME }}
          status: Deploy FAILED by ${{ github.actor }}
          color: danger

      - name: Send Honeybadger Notification
        uses: honeybadger-io/github-notify-deploy-action@v1
        with:
          api_key: ${{ secrets.HONEYBADGER_API_KEY }}
          environment: ${{ steps.find_env.outputs.environment }}
```

Because I am running this workflow from a `self-hosted` runner, I have added the labels `self-hosted, "${{ github.ref_name }}", api` to the `runs-on` key. 
```yaml
runs-on:  [ self-hosted, "${{ github.ref_name }}", api ]
```

`github.ref_name` returns the current branch. This `yml` file must be located inside `.github/workflows/<your_file_name>.yml`, so that github actions runners can pick this up and run the workflow.
