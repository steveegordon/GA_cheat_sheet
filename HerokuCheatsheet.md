How to deploy on Heroku

 If you are not logged in to heroku input these commands in terminal

1.  heroku login
    -#- then input your username and password
2.  heroku keys:add 
    -#- if you are a new user and did not use gui you have to state your keys 

 Now you will go into your apps home directory and input these commands

3. heroku create chosen_name
    -#- chosen_name is optional, heroku will create a random one for you otherwise

 Go to your gemfile and make these changes to prepare your app for production

4. At the bottom of your gemfile you have to make a production group insert this:

  group :production do
    gem 'pg'
    gem 'thin'
    gem 'rails_12factor'
  end
      -#- save these changes, if you have already placed gem 'pg' at the top of your gemfile and developed your app with postgres it is already universal and does not need to be added here 

In your terminal you should now input these commands 

5.  bundle install --without production 

6.  bundle update 
      -#- These commands make sure all your gems are installed and ready for the push to heroku 

7. git push heroku master 
      -#- This sends all your files to your heroku html and database for deploy 

-optional- 8.  heroku open
      -#- This opens your app in the browser for testing

-optional- 9.  heroku logs
      -#- If your app is failing this will give you the logs for debugging if this does not work you can debug on a local production server by entering:

      rails s -e production 

