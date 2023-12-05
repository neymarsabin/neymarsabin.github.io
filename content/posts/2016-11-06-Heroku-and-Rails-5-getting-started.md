---
layout: post
type: blog
author: neymarsabin
title: Heroku and Rails 5 Getting Started
tags: 
- rails 
- heroku
--- 


If you are developing some kind of apps related to these platforms (Ruby,NodeJs,Go,scala,PHP,python,Java) then one of the 
best place to deploy them is **Heroku**.
<!--more-->

> Heroku is a cloud platform that lets companies build,deliver,monitor and scale apps.

Not only companies.
If you haven't used heroku, then take a look at [heroku website](https://www.heroku.com/what).Don't forget to register.


# Installation

Let's install Heroku CLI client for your system.Heroku's cli let's you easily manage and deploy your apps in heroku.
If you are a ruby developer and i am sure u know about **rubygems**.

    gem install heroku 

For other system specific installation instructions take a look at [here](https://devcenter.heroku.com/articles/heroku-command-line). 


# Heroku Login

After installation, you can use **heroku** command from command prompt or terminal. 
Now login to heroku account using the following command: 

    heroku login

It will ask for your heroku username and password.
If you want to know more about heroku's command line, you can always ask for help.

    heroku help


# Rails 5

If you haven't already setup your rails environment:- 

    gem install rails 

Let's create a new application called **ImagePick**. 

    rails new ImagePick --database=postgresql

Move into the project and do **git init**. 

    cd ./ImagePick
    git init

Let's add some functionality to the rails application :

    rails generate controller Pages index about 

And edit the **app/views/pages/index.html.erb** 

    <h2>Hello World</h2>
    <p>
    The time is now: <%= Time.now %>
    </p>

Edit the **config/routes.rb** and on line 2 add: 

    root 'pages#index' 

Now, start the rails server and test your application

    rails server 

Navigate to <http://localhost:3000> in a browser.
Let's commit the changes and start using heroku.

    git add . 
    git commit -m "Pages controller add with home and index"


# Heroku create and deploy

Make sure you are in the directory that contains your Rails app, then create an app on heroku.

    heroku create

If the above command succeds without any errors, you are ready for one last command to deploy your application to production: 
Now to deploy do the app 

    git push heroku master 

Your application will be live on the internet.Let's migrate your database to **heroku's**. 

    heroku run rake db:migrate

Your app will be on the internet. To open your app you can do

    heroku open
