# odinprojectnotes
Notes on web development

## Table of contents

* [Rails](#rails)
  * [APIs and building your own](#apis-and-building-your-own)
    *[How Rails knows which type of file you are expecting back when you make an HTTP request.](#How Rails knows which type of file you are expecting back when you make an HTTP request.)
    *[What is the purpose of the #respond_to method?](#What is the purpose of the #respond_to method?)
    *[ How do you return a User object but specify that you don't want to include certain attributes](#How do you return a User object but specify that you don't want to include certain attributes)
    *[How do you return a User object but specify that you don't want to include certain attributes?](#How do you return a User object but specify that you don't want to include certain attributes?)
    *[How do you tell a controller action to render nothing but an error message?](#How do you tell a controller action to render nothing but an error message?)
    *[How do you build your own custom error messages?](#How do you build your own custom error messages?)
    *[Why can't you use session-based controller authentication methods if you want people to access your API programmatically?](#Why can't you use session-based controller authentication methods if you want people to access your API programmatically?)
    *[What is "Service Oriented Architecture?"](#What is Service Oriented Architecture?)

## Rails

### APIs and building your own

#### How Rails knows which type of file you are expecting back when you make an HTTP request.
  * Add the file extension to the html request `GET /users.json`


#### What is the purpose of the #respond_to method?
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
  
  
#### How do you return a User object but specify that you don't want to include certain attributes?
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
  
  
#### How do you tell a controller action to render nothing but an error message?
  ```
  class UsersController < ApplicationController
    # render nothing and set status to 404 not found when users#index is called 
    def index
      render nothing: true, status: 404 # 
    end
  end
  ```
  
  
#### How do you build your own custom error messages?
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
  
  
#### Why can't you use session-based controller authentication methods if you want people to access your API programmatically?
  * Browser cookies will not work because API is accessed usually by non-browser e.g. CLI
  
  
#### What is Service Oriented Architecture?
  * Each component of the application provides a *service* to other components of the application
  * Beneficial because each compnent can be independent thus it is easy to: isolate issuses, update/upgrade the service
  * Allows different programming languages to communicate e.g. python program and ruby program
