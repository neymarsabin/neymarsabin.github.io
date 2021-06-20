---
layout: post
title: Capistrano Rake Task for Sidekiq Process
type: blog
author: neymarsabin
tags:
- Capistrano
- Deployment
- Rails
---

Capistrano supports writing rake tasks(custom hooks) before/after deployments. These rake task can include commands that you want to perform before/after every deployment like clearing out cache, management(stop and restart) of sidekiq processes, building client side assets(capistrano can be used with pm2 for client side app deployments) etc. Sidekiq processes can be managed using linux systemd(systemctl) process monitor. So, before/after deployment we can write cusom hooks to restart sidekiq processes. 

file path: `capistrano/tasks/capistrano_sidekiq.rake`

```
namespace :sidekiq do
  desc 'Start sidekiq workers'
  task :start do
    on roles(:worker) do
      execute 'systemctl --user start sidekiq_some_task.service'
      execute 'systemctl --user start sidekiq_second_task.service'
    end
    on roles(:api_worker) do
      execute 'systemctl --user start sidekiq_another_task.service'
    end
  end

  desc 'Stop sidekiq workers'
  task :stop do
    on roles(:worker) do
      execute 'systemctl --user stop sidekiq_some_task.service'
      execute 'systemctl --user stop sidekiq_second_task.service'
    end
    on roles(:api_worker) do
      execute 'systemctl --user stop sidekiq_another_task.service'
    end
  end

  desc 'Quiet Sidekiq workers and stop accepting jobs'
  task :quiet do
    on roles(:worker) do
      execute 'systemctl --user kill -s TSTP sidekiq_some_task.service', raise_on_non_zero_exit: false
      execute 'systemctl --user kill -s TSTP sidekiq_second_task.service', raise_on_non_zero_exit: false
    end
    on roles(:api_worker) do
      execute 'systemctl --user kill -s TSTP sidekiq_another_task.service', raise_on_non_zero_exit: false
    end
  end

  desc 'Restart sidekiq workers (stop->start)'
  task :restart do
    invoke! 'sidekiq:stop'
    invoke! 'sidekiq:start'
  end
end
```

The task can be invoked in `deploy.rb` as it is recognized by capistrano.
```
after 'deploy:updated', 'sidekiq:stop'
before 'deploy:migrating', 'secrets:setup'
after 'deploy:published', 'sidekiq:start'
after 'deploy:failed', 'sidekiq:restart'
```
The start task will run after publish task of capistrano is completed. And `restart` task will run after deploy fails for some reason, sidekiq processes are restarted for original state.
