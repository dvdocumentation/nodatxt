.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Node synchronization
=======================

Transferring nodes to the server via API
----------------------------------------

A class declared in the configuration automatically has an API whose description can be taken from the constructor (there it is generated with the actual identifiers), and in general form it looks like this:

Adding/updating nodes on the server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


``POST /api/config/<config_uid> /node/<class_name>``

.. code-block:: JSON 

 [{
    "_id": "node_id",
    "field1": "value1",
    "field2": "value2"
 }]

Adding/updating nodes on the server with registration for devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``POST /api/config/<config_uid>/node/<class_name>?room=<room_id>``

Return all nodes of a class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``GET /api/config/<config_uid>/node/<class_name>``

Register all nodes in a room
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``POST /api/config/<config_uid>/node/<class_name>/register/<room_uid>``

Register specific nodes in a room
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``POST /api/config/<config_uid>/node/<class_name>/register/<room_uid>``

.. code-block:: JSON 

 ["node_id_1","node_id_2"]

Working with specific node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Get specific node : ``GET    /api/config/<config_uid>/node/<class_name>/<node_id>``

Update _data in specific node : ``PUT /api/config/<config_uid>/ /node/<class_name>/<node_id>``

Delete specific node : ``DELETE /api/config/<config_uid>/node/<class_name>/<node_id>``

As you can see, when transferring data via API, the class and configuration are defined in the request, i.e. it is assumed that at the time of the API call the configuration with classes already exists on the server. The same applies to the device—register will mark nodes for transfer to devices, but the device must have the class of these nodes, otherwise the nodes will not be accepted. That is, the configuration with the corresponding classes must be installed on the device.
Data transfer formats via files include both the node data itself and a reference to the class which can be downloaded from somewhere. As an option, if there is no Internet at all, you can first install the configuration into the repository and then receive files with nodes. 


Data transfer between server and devices via Rooms
-----------------------------------------------------------------------

In order to receive data from the server in real time, mobile clients are grouped into so-called “rooms”—a persistent WebSocket connection. To do this, you must create a regular room on the server and connect devices to it by scanning the room QR code.

The following works through this mechanism:

 * node transfer to the server
 * method execution

The API is described in the previous section. For devices to receive objects, these objects must be “registered.”


Transferring nodes to a device via files
---------------------------------------------

If there is no Internet connection or you do not want to use rooms, you can transfer files to the device in any way and either “Open” them with or “Share” them to NodaLogic. 

This must be a *.nl file of a specific format, necessarily containing a reference to the class in the format ``<configuration uid>$<class name>`` and _data.

Example of such a format:

.. code-block:: Python
 
 [{
 "_id":"1010",
 "_class":"885a12de-2bb5-4222-a671-9a7286902938$MyOrder",  
 "_data":{
 "order_number":"00-5000",
 "_cover":[["@order_number"]]  
 }
 }
 ]




