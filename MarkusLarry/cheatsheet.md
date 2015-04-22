#Google Api Cheatsheet

create new rails app
add gem ‘figaro’
figaro install

edit application.yml
remove comments
enter GOOGLE_MAPS:	‘server_key’

create welcome controller with maps action
create local variable url
url = "https://maps.googleapis.com/maps/api/js?key=“
create local variable key
key = ENV[‘GOOGLE_MAPS’]
concatenate with instance variable @endpoint
@endpoint = url + key