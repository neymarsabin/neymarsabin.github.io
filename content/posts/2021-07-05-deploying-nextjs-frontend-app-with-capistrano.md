+++
layout = 'post'
title = 'Deploying NextJS App with Capistrano and PM2 Server'
date = 2021-07-05T12:01:00+05:45
+++

Capistrano can be used to deploy a NextJS app into a linux server as well. We can take advantages of capistrano rake tasks to automate the whole process of building and restarting `pm2` server. Start with adding gemfile to the root of the project.
```ruby
# Gemfile
source 'https://rubygems.org'

gem 'capistrano'
gem 'capistrano-npm'
gem 'dotenv' 
gem 'httparty' # these will be used to send honeybadger notification after deployment is complement
```
and bundle: 
```sh
bundle install
```

Now we can intialize config for capistrano and add some configs.
```sh
bundle exec cap install
```
This commnd creates multiple files required by capistrano to read configuration and work accordingly. we have following files: 
```sh
config/deploy.rb
config/deploy/production.rb
config/deploy/staging.rb
Capfile
```
The `production.rb` and `staging.rb` are environment specific configuration files. For eg: server's ip or git branch to build is different for production and staging environments that can be mentioned in those configuration files.
Before modifying those files let's first modify the `Capfile`.

```
# Capfile
# Load DSL and set up stages
require "capistrano/setup"
require "capistrano/deploy"
require "capistrano/scm/git"
require "capistrano/npm"

install_plugin Capistrano::SCM::Git
# Load custom tasks from `lib/capistrano/tasks` if you have any defined
Dir.glob("lib/capistrano/tasks/*.rake").each { |r| import r }
```
we are requiring capistrano DSLs(basic predefined classes from capistrano and other packages) for git and npm managment. We are also requiring custom rake tasks. We haven't written the rake tasks yet.
We will also modify `deploy.rb` to add generalized configurations for both of our environments and environment specific files for environment specific configurations.

```sh
# config/deploy/production.rb
role :web, %w{xxx.xxx.xxx.xxx}
```

```sh
# config/deploy.rb
# config valid for current version and patch releases of Capistrano
lock "~> 3.11.0"

set :application, "my-app-name"
set :repo_url, "git@github.com:my-app/my-app-frontend.git"
set :remote_user, "deploy"
set :application_folder, "my-app-folder"
set :npm_flags, ""
set :deploy_to, "/home/#{fetch(:remote_user)}/#{fetch(:application_folder)}"
set :keep_releases, 3
set :honeybadger_env, fetch(:stage)
set :honeybadger_server, -> { roles(:web).first.hostname }

append :linked_files, ".env"

set :ssh_options, {
  user: fetch(:remote_user), # overrides user setting above
  forward_agent: true,
  auth_methods: %w(publickey),
  port: 3939
}

# Default branch is :master
ask :branch, `git rev-parse --abbrev-ref HEAD`.chomp

after "npm:install", "npm:build"
after "deploy:publishing", "pm2:reload"
after "deploy:finished", "deploy:notify_honeybadger"
```

Every after hook is a rake task, some of them are custom written, let's write them.

```sh
# lib/capistrano/tasks/capistrano_npm.rake
namespace :npm do
  desc "Build Web application"
  task :build do
    on roles(:web) do
      within fetch(:npm_target_path, release_path) do
        with fetch(:npm_env_variables, {}) do
          execute :npm, "run build:#{fetch(:stage)}"
        end
      end
    end
  end
end
```

```sh
# lib/capistrano/tasks/post_tasks.rake
namespace :pm2 do
  desc "Reload PM2 instance"
  task :reload do
    on roles(:web) do
      within fetch(:release_path) do
        execute :pm2, "reload application.json --env #{fetch(:stage)}"
      end
    end
  end
end

namespace :deploy do
  desc "Notifies Honeybadger locally using HTTParity"
  task :notify_honeybadger do
    require "httparty"
    require "dotenv/load"

    honeybadger_api_url = "https://api.honeybadger.io/v1/deploys"
    options = {
      body: {
        deploy: {
          environment: fetch(:stage),
          revision: fetch(:revision),
          repository: fetch(:repo_url),
          local_username: ENV["USER"],
        },
        api_key: ENV["HONEYBADGER_API_KEY"],
      },
    }
    result = HTTParty.post(honeybadger_api_url, options)

    if result&.parsed_response["status"] == "OK"
      puts "Honeybadger Notification Complete."
    else
      puts "Honeybadger Notification Failed: #{result.parsed_response}"
    end
  end
end
```
The above rake tasks are for reloading a pm2 instance and sending out http request for honeybadger notification after everything from capistrano is done.
