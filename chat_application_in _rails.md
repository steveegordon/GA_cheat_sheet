Gmail like chat application in ruby on rails


$ rails g model Conversation sender_id:integer recipient_id:integer

It's a good idea to add an index to both sender_id and recipient_id as we will use this fields while searching.
create_conversation.rb file

class CreateConversations < ActiveRecord::Migration
  def change
    create_table :conversations do |t|
      t.integer :sender_id
      t.integer :recipient_id

      t.timestamps
    end

    add_index :conversations, :sender_id
    add_index :conversations, :recipient_id
  end
end
-------------------------------------------------------------------------

After migrating the database, lets update our user model.
Since we don't have the user_id column in our conversation table, we must explicitly tell rails which foreign key to use.
user.rb file

class User < ActiveRecord::Base

  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  has_many :conversations, :foreign_key => :sender_id
end
------------------------------------------------------------------------------
Next, lets edit our conversation model.
A conversation will belong to both a sender and a recipient all of which are instances of a user.

conversation.rb file

class Conversation < ActiveRecord::Base
  belongs_to :sender, :foreign_key => :sender_id, class_name: 'User'
  belongs_to :recipient, :foreign_key => :recipient_id, class_name: 'User'

  has_many :messages, dependent: :destroy

  validates_uniqueness_of :sender_id, :scope => :recipient_id

  scope :involving, -> (user) do
    where("conversations.sender_id =? OR conversations.recipient_id =?",user.id,user.id)
  end

  scope :between, -> (sender_id,recipient_id) do
    where("(conversations.sender_id = ? AND conversations.recipient_id =?) OR (conversations.sender_id = ? AND conversations.recipient_id =?)", sender_id,recipient_id, recipient_id, sender_id)
  end
end
---------------------------------------------------------------------------------------

Now that we have our conversation model,
 lets proceed and create our message model. Run the following command on your terminal and run rake db:migrate.
 $ rails g model Message body:text conversation:references user:references

 Our messages table will have a body, conversation_id, and user_id columns. The conversation_id column will keep track of which conversation a message belongs to and the user_id column will keep track of the user who sent the message during chat.
 Adding some validations
 message.rb file
 class Message < ActiveRecord::Base
  belongs_to :conversation
  belongs_to :user

  validates_presence_of :body, :conversation_id, :user_id
end
-----------------------------------------------------------------------------------------

ADDING THE MESSAGE BUTTON

index.html.erb

<% @users.each_with_index do |user, index| %>
    <tr>
      <td><%= index +=1 %></td>
      <td><%= user.name %></td>
      <td>
        <%= link_to "Send Message", "#", class: "btn btn-success btn-xs start-conversation",
                    "data-sid" => current_user.id, "data-rip" => user.id %>
      </td>
    </tr>
<% end %>

---------------------------------------------------------------------------------------

Lets create a file chat.js inside our javascripts folder
user.js file

var ready = function () {

    /**
     * When the send message link on our home page is clicked
     * send an ajax request to our rails app with the sender_id and
     * recipient_id
     */

    $('.start-conversation').click(function (e) {
        e.preventDefault();

        var sender_id = $(this).data('sid');
        var recipient_id = $(this).data('rip');

        $.post("/conversations", { sender_id: sender_id, recipient_id: recipient_id }, function (data) {
            chatBox.chatWith(data.conversation_id);
        });
    });

    /**
     * Used to minimize the chatbox
     */

    $(document).on('click', '.toggleChatBox', function (e) {
        e.preventDefault();

        var id = $(this).data('cid');
        chatBox.toggleChatBoxGrowth(id);
    });

    /**
     * Used to close the chatbox
     */

    $(document).on('click', '.closeChat', function (e) {
        e.preventDefault();

        var id = $(this).data('cid');
        chatBox.close(id);
    });


    /**
     * Listen on keypress' in our chat textarea and call the
     * chatInputKey in chat.js for inspection
     */

    $(document).on('keydown', '.chatboxtextarea', function (event) {

        var id = $(this).data('cid');
        chatBox.checkInputKey(event, $(this), id);
    });

    /**
     * When a conversation link is clicked show up the respective
     * conversation chatbox
     */

    $('a.conversation').click(function (e) {
        e.preventDefault();

        var conversation_id = $(this).data('cid');
        chatBox.chatWith(conversation_id);
    });


}

$(document).ready(ready);
$(document).on("page:load", ready);
-------------------------------------------------------------------------

CREATING CONTROLLERS

We now have our javascript files logic ready all is left is piecing up our conversations and messages controllers to handle requests from our javascript file. Let's start by our conversations controller

$ rails g controller conversations

class ConversationsController < ApplicationController
  before_filter :authenticate_user!

  layout false

  def create
    if Conversation.between(params[:sender_id],params[:recipient_id]).present?
      @conversation = Conversation.between(params[:sender_id],params[:recipient_id]).first
    else
      @conversation = Conversation.create!(conversation_params)
    end

    render json: { conversation_id: @conversation.id }
  end

  def show
    @conversation = Conversation.find(params[:id])
    @reciever = interlocutor(@conversation)
    @messages = @conversation.messages
    @message = Message.new
  end

  private
  def conversation_params
    params.permit(:sender_id, :recipient_id)
  end

  def interlocutor(conversation)
    current_user == conversation.recipient ? conversation.sender : conversation.recipient
  end
end
-------------------------------------------------------------------

go to gemfile

gem 'private_pub'
gem 'thin'

rails g private_pub:install


We can now start up that Rack server by running this command on a separate terminal

rackup private_pub.ru -s thin -E production

This command starts up Faye using the Thin server in the production environment which is necessary to get Faye working. The last step is to add private_pub to the application’s JavaScript manifest file.

//= require private_pub

With private_pub installed, we can now proceed to make our first view that uses private_pubs functionality. This will be the show view of our conversation's controller

show.html.erb
<div class="chatboxhead">
  <div class="chatboxtitle">
    <i class="fa fa-comments"></i>

    <h1><%= @reciever.name %> </h1>
  </div>
  <div class="chatboxoptions">
    <%= link_to "<i class='fa  fa-minus'></i> ".html_safe, "#", class: "toggleChatBox", "data-cid" => @conversation.id %>
    &nbsp;&nbsp;
    <%= link_to "<i class='fa  fa-times'></i> ".html_safe, "#", class: "closeChat", "data-cid" => @conversation.id %>
  </div>
  <br clear="all"/>
</div>
<div class="chatboxcontent">
  <% if @messages.any? %>
      <%= render @messages %>
  <% end %>
</div>
<div class="chatboxinput">
  <%= form_for([@conversation, @message], :remote => true, :html => {id: "conversation_form_#{@conversation.id}"}) do |f| %>
      <%= f.text_area :body, class: "chatboxtextarea", "data-cid" => @conversation.id %>
  <% end %>
</div>

<%= subscribe_to conversation_path(@conversation) %>
----------------------------------------------------------------------

Our last controller is the messages controller.

class MessagesController < ApplicationController
  before_filter :authenticate_user!

  def create
    @conversation = Conversation.find(params[:conversation_id])
    @message = @conversation.messages.build(message_params)
    @message.user_id = current_user.id
    @message.save!

    @path = conversation_path(@conversation)
  end

  private

  def message_params
    params.require(:message).permit(:body)
  end
end
-------------------------------------------

We only have the create action for the messages controller. We store the conversation's path in the @path instance variable. We will use this path to publish push notifications to our view. Remember we suscribed to the same url in our conversation's show view. This will always be unique for any two users in our app.

Our create action will render a javascript template. Let's go ahead and create that. In our messages folder create this file

create.js.erb

<% publish_to @path do %>
    var id = "<%= @conversation.id %>";
    var chatbox = $("#chatbox_" + id + " .chatboxcontent");
    var sender_id = "<%= @message.user.id %>";
    var reciever_id = $('meta[name=user-id]').attr("content");

    chatbox.append("<%= j render( partial: @message ) %>");
    chatbox.scrollTop(chatbox[0].scrollHeight);

    if(sender_id != reciever_id){
        chatBox.chatWith(id);
        chatbox.children().last().removeClass("self").addClass("other");
        chatbox.scrollTop(chatbox[0].scrollHeight);
        chatBox.notify();
    }
<% end %>

------------------------------------------------------

Here we publish to the same path we suscribed to in our view. We also asign a number to variables from our rails app to variables in our javascript file. You are probably wondering what the following line does?

var reciever_id = $('meta[name=user-id]').attr("content");

We need a way to identity who is the recipient when publishing the notifications. A simple way around this is to create a meta tag in our application.html layout file that store's the id of the currently logged in user. In the head section of our layout file add


<meta content='<%= user_signed_in? ? current_user.id : "" %>' name='user-id'/>

message.html.erb file

<li class="<%=  self_or_other(message) %>">
  <div class="avatar">
    <img src="http://placehold.it/50x50" />
  </div>
  <div class="chatboxmessagecontent">
    <p><%= message.body %></p>
    <time datetime="<%= message.created_at %>" title="<%= message.created_at.strftime("%d %b  %Y at %I:%M%p") %>">
      <%= message_interlocutor(message).name %> • <%= message.created_at.strftime("%H:%M %p") %>
    </time>
  </div>
</li>
--------------------------------------------------


module MessagesHelper
  def self_or_other(message)
    message.user == current_user ? "self" : "other"
  end

  def message_interlocutor(message)
    message.user == message.conversation.sender ? message.conversation.sender : message.conversation.recipient
  end
end
---------------
routes.rb

Rails.application.routes.draw do

  devise_for :users

  authenticated :user do
    root 'users#index'
  end

  unauthenticated :user do
    devise_scope :user do
      get "/" => "devise/sessions#new"
    end
  end

  resources :conversations do
    resources :messages
  end
end
----------
application.html.rb

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="description" content="">
  <meta name="author" content="">
  <meta content='<%= user_signed_in? ? current_user.id : "" %>' name='user-id'/>

  <title>Chatty</title>
  <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track' => true %>
  <%= stylesheet_link_tag '//maxcdn.bootstrapcdn.com/font-awesome/4.1.0/css/font-awesome.min.css' %>
  <%= javascript_include_tag 'application', 'data-turbolinks-track' => true %>
  <%= csrf_meta_tags %>

  <!-- shiv here -->

</head>

<body>

<%= render 'layouts/nav' %>

<div class="container">
  <!-- flash messages here -->
  <%= yield %>
</div>
<audio id="chatAudio"><source src="/sounds/notification.mp3" type="audio/mpeg"></audio>
</body>

</html>
