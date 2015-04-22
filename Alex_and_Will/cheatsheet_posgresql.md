# Change a SQLite Rails app to Postgres!

* First step : add your Ruby version to your gemfile at the top

* Second step: while you're here, add gem  'pg' and delete gem 'sqlite3'

* Third step: go to your database.yml and change your adapter from sqlite to postgresql

* Fourth step: bundle in your command line

* Fifth step: rake db:create and rake db:migrate

# That's it!
