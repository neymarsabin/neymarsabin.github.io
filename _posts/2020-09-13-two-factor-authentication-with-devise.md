---
layout: post
title: Two-Factor Authentication :> Rails API :> devise_token_auth + devise-two-factor
type: blog
author: neymarsabin
tags: 
- devise
- two-factor
- rails-api
---

# Two-Factor Authentication Schemes
There are multiple strategies through which you can setup two-factor authentication in your Rails API application. I am going to cover these strategies in this post:
- Email
- SMS
- Google Authenticator

# Setup
I have a Rails API application with `devise_token_auth`. This gem use `devise` underneath allowing API to generate/share tokens to the clients. With Devise as our authentication engine, there are lots of gems which will provide fairly simple two-factor authentication integration. But, it is kind of tricky to integrate in API application. I found this barebone gem `devise-two-factor` which provide the core to generate `otp_codes` within a certain time interval. This gem provides all the methods to validate the `otp_codes`, expire old ones, integrates well with Rails model to provide helpers to enable/disable two-factor authentication for that model.Let's build a two-factor authentication system using this gem. In your Gemfile:
```ruby
gem 'devise-two-factor' # you may want to version lock
gem 'rqrcode' # generate qr-code uri's for easy scan
gem 'twilio-ruby' # send sms with the code to the users
```



