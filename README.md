# odinprojectnotes
Notes on web development

## Table of contents

* [Rails](#rails)
  * [APIs and building your own](#apis-and-building-your-own)
    * [How Rails knows which type of file you are expecting back when you make an HTTP request.](#1)
    * [What is the purpose of the #respond_to method?](#2)
    * [How do you return a User object but specify that you don't want to include certain attributes](#3)
    * [How do you tell a controller action to render nothing but an error message?](#4)
    * [How do you build your own custom error messages?](#5)
    * [Why can't you use session-based controller authentication methods if you want people to access your API programmatically?](#6)
    * [What is "Service Oriented Architecture?"](#7)

## Rails

### APIs and building your own

#### <a name=1></a>How Rails knows which type of file you are expecting back when you make an HTTP request. 
  * Add the file extension to the html request `GET /users.json`


#### <a name=2></a>What is the purpose of the #respond_to method?
  * Render the object appropriately for the given html request
  
  ``` Ruby
  def index 
    @users = User.all
    respond_to |format| do
      format.html # renders users/index.html.erb by default
      format.json { render :json => @users } # due to #render @users.to_json will be called automatically
    end
  end
  ``` 
  
  
#### <a name=3></a>How do you return a User object but specify that you don't want to include certain attributes?
  * `#to_json` is broken up into two methods `as_json` which returns a hash of attributes to render and `ActiveSupport::json.encode` to       render the hash into JSON. Naturally we want to target `as_json` then
  ``` Ruby
  class User < ActiveRecord::Base
    # In this example we only want to return the names of users 
    # option 1
    def as_json(options={})
      {:name => self.name}
    end
    
    # option 2
    def as_json(options={})
      super(only: [:name])
    end
  end
  ```
  
  
#### <a name=4></a>How do you tell a controller action to render nothing but an error message?
  ```
  class UsersController < ApplicationController
    # render nothing and set status to 404 not found when users#index is called 
    def index
      render nothing: true, status: 404 # 
    end
  end
  ```
  
  
#### <a name=5></a>How do you build your own custom error messages?
  ``` Ruby
  #config/application.rb
  config.exceptions_app = self.routes
  ```
  ``` Ruby
  #config/routes.rb
  %w(404 422 500) do |code|
    get code, :to => 'errors#show', status_code: code
  end
  ```
  ``` Ruby
  #app/controllers/errors
  def show
    render status_code.to_s, status: status_code
  end
  
  protected
    def status_code
      params[status_code] || 500
    end
  ```
  Add the respective views into `/app/views/errors/` making sure to name it as its status code e.g. `404.html.erb`
  
  
#### <a name=6></a>Why can't you use session-based controller authentication methods if you want people to access your API programmatically?
  * Browser cookies will not work because API is accessed usually by non-browser e.g. CLI
  
  
#### <a name=7></a>What is Service Oriented Architecture?
  * Each component of the application provides a *service* to other components of the application
  * Beneficial because each compnent can be independent thus it is easy to: isolate issuses, update/upgrade the service
  * Allows different programming languages to communicate e.g. python program and ruby program

#### <a name='public-method'></a>public/private/protected methods
[refer to this resource](https://tenderlovemaking.com/2012/09/07/protected-methods-and-ruby-2-0.html)

# Working with external apis
* What's the best way to locate an API's docs?
  * Google "companyX API doc". First link should be most applicable 
* What are the steps you'll almost always have to go through when setting up your app to use an API?
  * Setup the app with resource provider 
* What is an API key?
  * A unique random string used to indentify your app

How do you avoid including an API's secret token in your Github repo (e.g. hard coding it)?
 * Use figaro gem, follow the readme to learn how to use it
    
What (basically) is OAuth?
 * A protocol used to authorize & authenticate your app with the user's resource provider

What is OmniAuth and why does it save you tons of time/pain?
 * Implements OAuth protocol for you 

``` bash
rails console --sandbox # changes will be rolled back when you exit
```

#### <a name='mailers'><a> Mailers
Analogous to a controller but handles the flow of emails. Put another way controller handles HTTP and mailer handles SMTP. 
Why do we need both html and text version of your mails?
* Some people don't like receiving html 
Why can't you use `*_path` link helpers in mailer views?
* Because the mail could be opened anywhere thus relative links (i.e. `*_path`) don't work, absolute urls are needed (i.e. `*_url`)
