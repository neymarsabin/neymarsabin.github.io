---
layout: post
title: Datadog Environment Specific Configuration for Rails Applications
type: blog
author: neymarsabin
tags: 
- datadog
- rails
- environment
---

### Installing Datadog Agents
Datadog Agents collect logs, metrics, traces from the Rails application and forward it. The servers need to have this agent installed to forward the logs. Generate an [API Key](https://app.datadoghq.com/account/settings#api) and run the following command being ubuntu user:
```
DD_API_KEY=<API_KEY> DD_AGENT_MAJOR_VERSION=7 bash -c "$(curl -L <https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)">
```

Checking the status of datadog-agent:
```
sudo datadog-agent status
```
Starting/Restarting/Enabling/disabling datadog-agent:
```
sudo systemctl <status|restart|disable|enable> datadog-agent
```

#### Configure Agent to collect request logs and traces
Datadog agents APM and logs tracing is disabled by default. For all of the servers where `datadog-agent` is installed, copy the configuration file `/etc/datadog-agent/datadog.yaml` to `/etc/datadog-agent/datadog.yaml.sample` and edit `datadog.yaml` file.
```
api_key: <API_KEY>
site: datadoghq.com
env: <staging | production>
logs_enabled: true # this is for logs management, will need later
apm_config: 
  enabled: true
  env: <staging | production | test | pre-prod>
```

```
sudo systemctl restart datadog-agent
```

#### Rails Initializers Configuration
Datadog uses the [ddtrace](https://docs.datadoghq.com/tracing/setup_overview/setup/ruby/) gem and a Rails initializer file for instrumenting your application. Add the following gem to your Gemfile:
```
gem 'ddtrace'
```

then bundle install:
```
bundle install
```

Next, create a `datadog.rb` file in your application's `config/initializers` directory:
```
# config/initializers/datadog.rb
require 'ddtrace'

app_name = 'your-app-name'
configured_environments = %w(staging production preprod)
environment = ENV['RAILS_ENV']

if configured_environments.include?(environment)
  Datadog.configure do |c|
    c.tracer enabled: true
    c.env = environment

    # <https://github.com/DataDog/dd-trace-rb/blob/master/docs/GettingStarted.md#rails>
    c.use :rails,
        service_name: app_name + '-app',
        database_service: app_name + '-postgres',
        controller_service: app_name + '-controller',
        distributed_tracing: true,
        middleware_names: true

    c.use :sidekiq, client_service_name: "#{app_name}-app", service_name: "#{app_name}-sidekiq", tag_args: true

    # <https://github.com/DataDog/dd-trace-rb/blob/master/docs/GettingStarted.md#aws>
    c.use :aws, service_name: "#{app_name}-aws"
    c.use :redis, service_name: "#{app_name}-redis"

    # <https://github.com/DataDog/dd-trace-rb/blob/master/docs/GettingStarted.md#nethttp>
    c.use :http, split_by_domain: true
  end
else
  Datadog.configure do |c|
    c.tracer enabled: false
  end
end
```
`c.env = environment` is required to categorize/scope traces/logs based on the environment.

### Sending Deployment Metrics to Datadog
There is a tag called `version` that is read by datadog to find out changes between codes and compare deployments. If you use `Capistrano` application uses capistrano for deployments and capistrano creates a file called `REVISION` in the remote folder. The file contains the latest commit hash `git rev-parse HEAD` which can be sent to datadog as a new version. Let's write a module to extract that version and send it to datadog.
```
module Configuration
  module Version
    module_function

    REVISION_FILE_NAME = 'REVISION'

    # find latest commit hash from REVISION file inside current folder created by capistrano in servers
    def git_head
      File.read(Rails.root.join(REVISION_FILE_NAME)).strip
    end
  end
end
```
The method `git_head` can be called in `datadog.rb` like this.
```
...
  Datadog.configure do |c|
    ... 
    c.version = Configuration::Version.git_head
    ...
  end
...
```
