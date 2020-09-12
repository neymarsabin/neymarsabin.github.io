---
layout: post
type: blog
author: neymarsabin
title: Rails:> Pushing Jobs to faktory:> Part 1
---

### Faktory || Sidekiq
Sidekiq is pretty well known to most Rails developers. It is an awesome library to execute asynchronous operations from Rails. When we need to perform some operations like sending email to thousands of customers, processing a large dataset we need asynchronous behaviour instead of doing everything in a single API call or in a main thread synchronously. Sidekiq is a mature library by Mike Perham, one of the best. There are Professional and Enterprise versions of sidekiq which you have to pay for. Likewise, Faktory is also a Job server which is written in Go language. The main advantage of Faktory over Sidekiq is we can write clients for faktory in any programming language. Sidekiq only workers written in Ruby. Faktory can easily run the jobs, distribute them among multiple servers. Clients can use Faktory API to fetch jobs from queue(jobs are pushed in Redis) and run them. Here's what we will cover:

(part 1)
- Faktory Server Installation
- Use faktory ruby client/gem to push jobs to Faktory server

(part 2)
- Write faktory client in Golang to fetch the jobs pushed by Ruby client and execute them

### Faktory Server Installation
Everyone use docker nowadays, let's use official image from faktory and run a faktory container. You can find more options on this installation page: [faktory installation guide](https://github.com/contribsys/faktory/wiki/Installation)

- pull the image
  
	  docker pull contribsys/faktory
	  
- run the container/expose ports(not using persistent volume for this example)

	  docker run --rm -it -p 127.0.0.1:7419:7419 -p 127.0.0.1:7420:7420 contribsys/faktory:latest
	  
  Here we are exposing two ports from the container to the host machine. `7419` serves as the server whereas `7420` port is for the Web UI to view the stats. Not visit [localhost:7420](http://localhost:7420) to view the UI. You should add the `username/password` for using faktory in production.

### Use faktory ruby client/gem to push jobs to Faktory server

I asume you already have a ruby on rails application for this purpose. Clients can be any ruby application if you have the library installed. Let's start with installing the library. Add it to Gemfile:

	gem 'faktory_worker_ruby'
	
And do a quick bundle:
	
	bundle
	
Now, let's push some jobs to Faktory Server. The job I am going to write will take in a file path as argument, the job will be written in Golang which will read the file from the path containing around 30k rows of data and convert it to json by generating another file. I am going to write this ruby class which will push jobs to faktory. This will push unique job for each of the files. Instead of writing it in a json file, your application might insert it into a database. I am not going to use database here.

	class PushJobs
	  FOLDER_PATH = Rails.root.to_s + "/tmp/csvs/*"
		
	  def self.load
	    # faktory_client using url to push jobs
	    faktory_client = Faktory::Client.new(url: ENV['FAKTORY_SERVER_URL'])
			
        # every job must have a unique id, generated md5 digest
        # args must always be an array
        # job_type is important, we will register worker in golang using this job_type
        # queue is the queue we will maintain for these jobs in redis
			
	    Dir[FOLDER_PATH].each do |file_path|
	      job = { 
		    jid: Digest::MD5.hexdigest(i),
		    args: [file_path],
		    queue: 'default',
		    job_type: 'csv_to_json'
		  }
		  faktory_client.push(job)
	    end
	  end
	end

I have place around 14 CSV files in the folder each containing around 30k of rows. I will describe how the data looks like and how Go worker will read through the csv files and push rows into JSON file later. Let's first test whether jobs will be pushed into faktory or not. Run the method from the rails console: 

	:> PushJobs.load
	
If everything goes well, you will see jobs enqueued in [web UI](http://localhost:7420) of faktory server. These enqueued jobs are waiting for some worker to fetch and run them. I will write about running these enqueued jobs in the next part. 
