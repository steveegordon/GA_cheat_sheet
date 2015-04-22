#Facebook OAuthorization

Objs
Understand OAuth Flow
Create FB Login

open -a google chrome
Facebook developer console
After logging in, sign up for a developer account (register as a developer) .
Then select My Apps - Website.
Create a new app - call it GA_test (or whatever)
Then create new Facebook app id - then for category chose education.
Next got to the site URL - http://localhost:3000
select quick start - Gotten an app ID an an app secret
Then rails new Facebook_login -T in terminal
then cd and subl . -
now getting gems for use:
gem ‘omniauth’
gem ‘omniauth-facebook’,
gem ‘figaro’
afterward bundle
then generate the yml for figaro with 'figaro install' in the terminal.
then go to the application.yml file.
Type in FACEBOOK_APP_ID: ‘app id number’
next type in FACEBOOK_SECRET: ’secret integer’

go to the following folders: config> initializers, Then create a new file called. omniauth.rb (also for carrier wave).
When in the Omniauth.rb file type:
OmniAuth.config.logger = Rails.logger

Rails.application.config.middleware.use Omniauth:: Builder do
    provider :Facebook, ENV[‘FACEBOOK_APP_ID’], ENV ['
        FACEBOOK_SECRET’]
end

Now we are going to route to our application - Go inside the app > views and create a new folder called applications. Then go into the application folder and create an index.html.erb file.

Next go to routes.rb. Then type
root ‘application#index’
match ‘auth/:provider/callback’ to: ‘sessions#create’, via: [:get, :post]
match ‘auth/failure’,to: redirect(‘/‘), via: [:get, :post]
match ‘signout’, to: ‘sessions#destroy’, as: ‘signout’ , via: [:get, :post]

end

next create a sessions controller in the console: rails g controller sessions
Now we create the model rails g model User provider hid name oauth_token oauth_expires_at:date time (this token persist as long as the person logs in)
then rake db:migrate
Then go back to sublime SessionsController to create actions and routes

def create
    user = User.from_omniauth(request.env[‘omniauth.auth’])
    session [:user_id] = user.id
    redirect_to root_url
end

def destroy
    session[:user_id] = nil
    redirect_to root_url
end
end

next go to User.rb and type the following

def self.from_omniauth(auth)
        where({:provider => auth['provider'], :uid => auth ['uid']}).first_or_initialize.tap do |user|
            user.provider = auth['provider']
            user.uid = auth ['uid']
            user.name = auth ['info']['name']
            user.oauth_token = auth ['credentials']['token']
            user.oauth_expires_at = Time.at(auth['credentials']['expires_at'])
            user.save!
        end
    end
end

Then go into the Application_helper.rb and type:
 def current_user
        @current_user ||= User.find(session[:user_id]) if session[:user_id]
    end
end

After wards go to application controller.rb and type
include Application Helper
def index
end

Then to to index.html.erb

div
<% if current_user %>
 welcome <strong> <%= current_user.name %> </strong>!
<%= link_to ‘Signout’ , signout_path %>
<% else %>
<%= link_to “Sign in with Facebook”, “/auth/facebook” %>
<% end %>
div

then start rails s - go to localhost 3000 an boom?

