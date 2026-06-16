.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov 5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Synchronization of nodes
=======================

This section describes various channels and methods for synchronizing data (nodes, datasets) between elements of the NodaLogic system.

* Exchange with external systems
* Delivery of documents and reference books to mobile clients, including batch delivery
* Delivery of messages at the user level (messaging) in different directions, including between devices, bypassing the server
* Uploading from clients to the server
* Direct interaction with the mobile client from the outside

For ease of understanding, the text is divided into scenarios.

Scenario: "Delivery of directories and documents from an external system to the server and mobile clients, where they will be stored locally"
-----------------------------------------------------------------------------------------------------------------------------

This scenario needs to be broken down into 2 parts:

1. Delivery to the server (nodes and datasets)
2. Transfer to mobile clients

Transferring nodes to the server via API
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A class declared in a configuration automatically has an API, the description of which can be found in the constructor (it is generated there taking into account the identifiers), and in general it is as follows:

Adding/updating nodes on the server:

.. sourcecode::text

      POST /api/config/<config_uid> /node/<class_name>

   [{
      "_id": "node_id",
      "field1": "value1",
      "field2": "value2"
   }]

Return all nodes of a class:

.. sourcecode::text

      GET /api/config/<config_uid>/node/<class_name>

Transferring documents and reference books to mobile devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are several synchronization methods to choose from:

1. Synchronization of nodes with devices via the Rooms mechanism. It functions as a broadcast subscription via either WebSocket or FCM and is primarily aimed at fast delivery. Nodes (obtained via the API or their own) register in a room and broadcast messages to devices connected to the room. There can be any number of rooms. Access is via aliases. Registration is via the _register command. One device is connected to the first group. A pending mechanism and message storage are available.
2. Node synchronization via the messaging system (https://habr.com/ru/articles/1034202/) – both in human-readable chats and hidden in chats (handler-to-handler). You can send p2p messages, group messages, and general configuration messages. This is also aimed at the fastest possible delivery of one or more nodes. Routing is tied to users (and all user devices), as is storage and all guaranteed delivery mechanisms.
3. Dataset synchronization. This mechanism essentially synchronizes an entire immutable set of external data. Simply put, a directory is exported as JSON and loaded onto the device upon startup. Entirely, without change tracking. To keep things moving, indexes are used. Datasets are tied to the configuration. Incidentally, when a configuration is deleted, everything associated with it is cleared, including datasets, and contracts are not tied to the configuration. Previously, the task of transferring directories was generally solved through datasets; now there is a choice of which mechanism to use.
4. Contract. Unlike points 1 and 2, this is a batch transfer of nodes, oriented towards big data. However, it's not lightning fast. Furthermore, a change tracking system is built right in. Unlike point 3, these are nodes, not just JSON, meaning they can be changed, they have handlers, an interface, etc. In other words, datasets are immutable data (for reference only), while here we have regular nodes.

Synchronization via the Rooms mechanism
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is a fast, broadcast delivery of nodes from the server to the client via WebSocket or in two cycles – push via FireBase messaging + API request. It operates on the principle of broadcasting to a group of devices. This delivery channel is more focused on speed than on pumping large amounts of data, and it's more suitable for "documents" than "reference books."

To do this, you need to create a Room of the required type on the server, and on clients, you need to join this room by scanning a QR code (in the "Join Room" menu). The room displays the devices connected to the room.
But that's not all. The server needs to know which nodes to send. To do this, the node needs to be registered. This can be done from the handler code, manually by the user, or via the API.

Note: Specific rooms are objects specific to a specific user instance, while solutions (configurations) are created to be deployed to any client, so they operate with aliases rather than specific UIDs or URLs. The configuration has a "Rooms" section where you need to assign an alias to a specific room.
After registration, if the device is connected, the node will be delivered immediately and will immediately appear in the sections (if it is displayed in the sections)

Via API
"""""""""""""

Register all nodes in a room. These commands register either all objects of a class or selected objects to a specific room UID (not an alias).

.. sourcecode::text

      POST /api/config/<config_uid>/node/<class_name>/register/<room_uid>

Register specific nodes in a room

.. sourcecode::text

      POST /api/config/<config_uid>/node/<class_name>/register/<room_uid>
      
["node_id_1","node_id_2"]

Through server handlers
"""""""""""""""""""""""""""

The node has a method ``_register(room_alias)`` - registers a specific node for sending to a room by alias (see above for the correspondence between aliases and real rooms)

And the class method ``Register(cls, uids: list, room_alias: str, config_uid: str = None)`` registers class objects in a group way

Via a custom command or auto-registration upon saving
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

In the class, on the Migration tab, you can specify a room nickname and enable the "Register Command"; then, if Standard Commands are enabled, the Register button will appear, which will register the node in the room.

.. note:: **Important point** This checkbox enables the "Upload" button in the mobile form of the node.
You can also enable "Register on Save" in the class on the Migration tab, which will automatically register the class when saving. Important! In the mobile client, this will also upload the class to the server when saving. Use this option with caution.

Synchronization via Contracts mechanism
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is a batch synchronization tool for nodes, designed for large downloads using files with resumable downloads. Contracts also monitor changes on devices through a confirmation mechanism, meaning they only deliver what has not yet been delivered (changed) to a specific device.

The external system dumps data to the URL and doesn't care about anything, and the server distributes it to devices, accepts acks from devices, and only doesn't send those nodes that haven't been accepted yet or contain changes.

This works on request from the client via files. That is, the client subscribes to receive data for one or more contacts and downloads it at startup, on a schedule, or manually.

Contracts are not tied to a specific configuration; the same data can be used in different configurations.

The contract can accept data for specific classes, or it can accept data directly with the packaged class (nodes without configurations or "standalone nodes"). The client simply submits an array of objects of the form [{"_id":<internal id>,…}], and the system automatically normalizes them and converts them into digestible system documents.

In order to then organize a contract and start accepting data into it, you need to:

1. Go to Contracts, create a new contract.
2. Select the classes to which it will deliver data (Class Source = class). You can also choose no classes in the contract (Class Source = external_only), in which case you should send objects with the class already packaged (the _class parameter should be a JSON class object, not a class reference).
3. On your device, go to the Contracts menu, add a contract, and scan the QR code.
4. The contract is ready for receiving and synchronization. Its API can be copied from the card and used in your solution.

The contract receives data from an external system via a PUSH request to the URL from the contract card on the route ``api/contracts/<contract uid>/push``

The request body contains a JSON array of the form [{“_id”:<internal id>, other object fields…}] . The system must convert this data into nodes, so it will take the _id and normalize it into the format <configuration UID>$<class>$<id> and create nodes. If multiple classes are selected in a contact, multiple nodes will be created for each object, and they will then be maintained independently.

You need to understand that these are regular nodes (with a class, handlers, etc.) and further work with them is like with regular nodes, and the Contract is a method for delivering and tracking changes.

After the contract is downloaded, the ``onContractReceived`` event occurs, to which, if necessary, a handler can be attached. In the handler's _data, you can get a list of node _ids as an array in the ``nodes`` key, the ``contract_uid`` key is the contract's uid.

You can view nodes received through a contract by simply clicking on it in Contracts. These are generally standard nodes. They can be placed in interface sections or not (but only used in documents as links or for searching).

Contracts, if configured, are downloaded upon logging into the app, either manually (from the contract card), or you can set a download timer in the app settings. Since it's on the worker, it's no more than once every 15 minutes (but it will download even when the app isn't running).


Various scenarios for transmitting nodes and just data in arbitrary directions: Messaging system.
--------------------------------------------------------------------------------------------------------------

Like Rooms, messaging-based exchange allows for instant and guaranteed node delivery, as it's focused on chats, like any regular messenger. However, unlike Rooms, messaging allows for more flexible addressing management. Addressing is targeted not only to devices but also to users.
Users register on nmaker.pw, log in, and can use this login to send messages. Messages can be sent to both users and groups. nmaker.pw stores device tokens, and delivery to the user (or group member) takes these into account and delivers messages to all of the user's devices.

A more detailed review can be found here: https://habr.com/ru/articles/1034202/

Messaging in NodaLogic encompasses user chats, exchanges between handlers, and everything in between: chat-to-chat, chat-to-handler, handler-to-chat, handler-to-handler. This means that both hidden (outside of chats) exchanges and process notifications addressed to users and groups are possible using this mechanism.

HTTP API
~~~~~~~~~~
There is a REST API for external systems to host nodes and conduct discussions in them.

``POST /api/user/<username or group>/nodes-message`` – sending an all-in-one node – the node is deployed, i.e., receives its URL, and then the message is sent to the user or group. For a group, the format is group:<group_id>

``POST /api/node-discussion/by-node/<path:node_id>/messages`` sends a message to the node discussion.

``GET /api/node-discussion/by-node/<path:node_id>/messages`` gets all messages in the discussion by node.

Client-level functions and event handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Functions for display messages
"""""""""""""""""""""""""""""""""""""""

From handlers, you can write “human” messages - i.e. messages that are displayed both in chats and in discussions on the node

In the recipients, you can specify either the recipient's login (p2p chats) or a group in the format group:<group_id>

``sendTextMessage(String target,String text)`` sending a text message

``sendImageMessage(String target, String text, String filename)`` sends an image or an image with a caption. The image is a link to S3.

``sendNodeMessage(target,id)`` – sends the node itself to the chat

``sendTextToNodeDiscussion(NodeEx node,String text)`` – sending a message to a discussion on a node

``sendImageToNodeDiscussion(NodeEx node,String text, String filename)`` – sends an image to a discussion on a node

Node-level handlers on the client (in event/listener format)
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

``onInput/onDiscussionMessage`` – the event of a message appearing in a discussion on a node. The following variables are stored in _data:
"_message_text" – message text
"_message_image_url" - image, if any
"_message_sender_user" – sender
"_message_sender_user_display_name" – sender, display name

In Python handlers, the node object also has the following available:

``self._deploy(self)`` - publish/update a node in "self-node" mode
``self._send(self,target)`` - publish and send the node to the recipient

Data messages
""""""""""""""""""

``sendDataToNode`` – sends arbitrary data (json) to the node on the receiving end. The message is not displayed in chats; it triggers the node's onInput event with listener = onDataMessage, which receives the data in the ``message_payload`` variable, which receives the data specified in the payload.
``payload= {"action":"I clicked the button"};``
``sendDataToNode("_server",_data.my_node,payload)``

``sendDataToConfiguration(String target, String configurationUid, JSONObject payload)`` – sending arbitrary data not to a node, but to an entire configuration. It is assumed that the recipient has a configuration that will accept messages (usually the same as the sender, but not necessarily) and that it has a common onDataMessage event defined. Its input will be the payload sent by the sender. As a reminder, Python handlers and NodaScript handlers are structured slightly differently: for NS, this will simply be data, while for Python, it will be the inputdata function parameter.
 
Server-level functions and handlers. Chatbots.
"""""""""""""""""""""""""""""""""""""""""""""""

In the sendDataToNode message, you can specify the predefined "_server" target, and then it won't go beyond the server. Actually, if the node is stored on the server, you can access it directly, without using the messaging system, although there might be a connection issue, for example... But that's not the point. The point is that the node might not be on the server. Remember from the very beginning—a node can be shared by an external system; it's essentially JSON with an access URL and nothing more. How can it receive and process the message?

It turns out it can be done just like with the node from the configuration. But there are some nuances: since this node isn't deployed, the handler Python modules aren't deployed either. For this case, handlers need to be written in a special portable Python script, called PythonScript in the system. What's the point? You write it as a handler script.
in the configurator window, then it's saved as an S3 link (it's accessible via this link, as it's public). The script link is packaged in the class, and when it needs to be executed, it's retrieved from S3. Naturally, it's cached, like the node itself, in download_url to avoid downloading it again each time.

As a result, the node receives ``onInputServer/onDataMessage`` already on the server

The server can also intercept messages in a node's discussion (using the same scheme). Essentially, a server node can act as a kind of chatbot, intercepting all correspondence and writing to chats as needed.

The server can subscribe to an event for each message in the node correspondence (node-level event) onInputServer/onDiscussionMessage and, accordingly, the server sees what you wrote to it through message_text.

At the same time, the node on the server itself can write in chats:

``sendTextMessage(target: str, text: str)``

``sendImageMessage(target: str, text: str, filename: str)``

``sendTextToNodeDiscussion(node, text: str)``

``sendImageToNodeDiscussion(node, text: str, filename: str)``

Some technical information
"""""""""""""""""""""""""""""""""""""""

The server component is written in Python and is available on Github (links at the end of the article), meaning you can download and deploy it yourself—along with all the other core NodaLogic functionality—the node server, web clients, and configurator. Otherwise, everything goes through nmaker.pw. Then you'll have all the data—nodes and messages—yourself. However, Firebase Messaging support is available only on nmaker's side, as that's where the account is stored. So, the only thing unavailable is sending push notifications. With the "server on your side" approach, you'll simply need to tell nmaker to send the push notification.  
This scheme also makes extensive use of S3 storage—for images and handlers. For security, this is wrapped in a server API, meaning the client doesn't store S3 write access keys (it's assumed that reading keys aren't required—it's public. If not, you need to add read authorization as well). The client receives a temporary token, writes data with it, and receives a link. nmaker has its own storage. If you're deploying your own NL server, you'll also need S3 storage (if you're using PythonScript handlers or images), and you'll need to enter your account credentials in boto3.client.


Scenario "Uploading nodes from a mobile client to the server."
---------------------------------------------------------------

This is the delivery of new nodes or updating previously uploaded nodes with new data. When designing client-server solutions, you can go beyond this synchronization (which, when uploading, essentially simply updates the server's _data and the client's node's _data) and transfer information through calls to server methods. You can also upload nodes (via RomoteClass). However, for simple situations, you can simply use the upload options described below.

Note: A specific server is a specific instance of your solution (it can be deployed anywhere), and configurations are released for all instances. Therefore, solutions operate with server aliases rather than specific URLs. There must be at least one primary server alias. This is configured in the Servers section of the configuration. In most cases, there is only one server, and simply creating one server with the "Primary Server" checkbox is sufficient.

Unloading can be interactive or via a method handler.

To unload through a handler, you need to use methods

``_upload(server_alias=None)`` is an object (node) method. Uploads to the default server or to the specified alias.

``_delete_from_server(server_alias=None)`` - an object (node) method. Deletes from the default server or to the specified alias.

``_register(room_uid)`` is an object (node) method. It registers a room.

``_upload_all()`` is a class method. Uploads everything

``register_all(room_uid)`` is a class method. Registers everything



Interactive is configured on the Migration tab, just like Rooms registration. For the server node, it's Rooms registration, while for the mobile client, it's server upload. Be especially careful when checking the "Register when saving" box, as the client has autosave and other features, such as



Scenario "Retrieve nodes from the server or perform other actions with them"
---------------------------------------------------------------------------

Return all nodes of a class

``GET /api/config/<config_uid>/node/<class_name>``

Get specific node : ``GET /api/config/<config_uid>/node/<class_name>/<node_id>``

Update _data in specific node : ``PUT /api/config/<config_uid>/ /node/<class_name>/<node_id>``

Delete specific node : ``DELETE /api/config/<config_uid>/node/<class_name>/<node_id>``

Scenario "Transferring nodes to a device via files"
----------------------------------------------------------

If you don't have internet access or don't want to use Rooms, you can transfer files to your device using any method and either "Open" or "Share" them with NodaLogic.

This must be a *.nl file of a special format, which must contain a reference to the class in the format <configuration uid>$<class name> and _data.

An example of this format:

.. sourcecode:: JSON

     [{
   "_id":"1010",
   "_class":"885a12de-2bb5-4222-a671-9a7286902938$MyOrder",
   "_data":{
   "order_number":"00-5000",
   "_cover":[["@order_number"]]
   }
   }
   ]
