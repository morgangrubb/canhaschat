UPDATE:
http://yeti-factory.org/2008/2/28/sleeping-mongrel-handlers
May be of interest to people who want to further research the effect of a bulk of sleeping mongrel handlers on their server.  In testing, the Rails app showed to still be responsive while the threads were sleeping (granted, the trivial code).  More anecdotal production use notes are still required, tho.  Also:  in the testing, the server showed itself to eventually reach an error condition with "too many open files" caused by the number of sleeping threads.   This was at the 200 simultaneous threads point.


can has chat is a plugin for Ruby on Rails that allows you to easily plugin messaging functionality to your app using XMPP4R.  The plugin uses a separate dRuby server to receive chat events, and can optionally use a provided Mongrel handler script to use pseudo-server-push style communication that eases the load that server polling (also supported) would cause.

can has chat requires the JSON and XMPP4R gems, and a working Jabber server (http://www.ejabberd.im/, http://www.igniterealtime.org/projects/openfire/index.jsp).  Use of the server-push script requires Mongrel.  Chats with users of non-Jabber transports requires your Jabber server to be configured with transports for those networks, as well as a valid username and password for each chatter on that network.  If you wish to use anonymous jabber authentication, you'll need to use the trunk version of XMPP4R.


To install:
```
script/plugin install  http://canhaschat.googlecode.com/svn/trunk/canhaschat 
```


Example code:

Model:
```
class ChattyObject < ActiveRecord::Base
  can_has_chat :namespace => "My Multi-IM App",
                 :transports => {
                    :aim => :get_aim_nick,
                    :icq => [:get_icq_nick],
                    :yahoo => [:get_yahoo_nick, :get_yahoo_password]
                 }
   # methods
end
```

View file:
```
<%= wait_for_messages(:url=>"/mychats/push?chat_id=#{@chat_id}") %>
<% for_new_messages do |msg| %>
    <li><%= msg.timestamp %>: [<%= msg.from %>] <%= msg.message %></li>
<% end %>
```

Controller:
```
class ChatController < Application

   def chat()
      @chat_user = ChattyObject.find_by_id(params[:id])
      hash_of_transport_passwords_not_in_db = {
           :aim => "someaimpassword",
           :icq => "someicqpassword"
      }
      @chat_id = @chat_user.start_chat("my_jabber_password", hash_of_transport_passwords_not_in_db)
   end
end
```

More usage examples and documentation are available in the project README.



can has chat was initially created by the Indianapolis Star (http://www.indystar.com/, http://www.indy.com/) and has been released under the MIT License.