.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov 5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Classes
==============

A node class is, on the one hand, a data storage aspect (a structure where class instances will be stored and an API for accessing the class and nodes). On the other hand, it's like a classic OOP class: it encapsulates methods and inherits methods from its parent (the Node class). The class also describes the behavior of objects—which section they belong to, what events are generated, their skin, etc. Client-specific class properties are discussed in the Mobile Client section.

This section describes working with class methods separately for the client and server, as well as working with the server via a "remote class." As in standard OOP, there are class methods (object creation, object search, get all objects, etc.) and methods of the class instance itself (in NL, this is called a "node"), inherited from the Node object.


Method execution and handlers
-----------------------------------
The method can be executed:

* Locally on the server (and web client) and on the mobile client
* In the mobile client, you can create a RemoteClass and get a "doppelganger" of the server with methods and data. Under the hood, these are HTTP requests.
* An external system can execute a method on the server via REST API
* An external system can execute a method even on the client via Room, by sending a request and then another one to get the result

Executing node events
-----------------------------

.. image::_static/c_events.png
       :scale: 70%
       :align: center

Events occur on the client (Events tab). Each event has a type and can also have a clarifying listener property. For example, if you have two buttons with id=button_1 and id=button_2, when clicked, both will trigger an onInput event, but with different listeners: button_1/button_2. You can assign a single handler to the onInput event without specifying a Listener, in which case both buttons will execute this handler. Or you can create two event handlers with a specified listener, so the listener is a filter.

.. image::_static/c_actions.png
       :scale: 70%
       :align: center

You can subscribe an array of handlers to one event – ​​they will be executed one after another.

The action property is responsible for how the event will be executed in the system - **run** - synchronously, **runasync** - asynchronously (it makes sense to fill in the postExecuteMethod property - execution of a callback after execution of an asynchronous method) and **runprogress** - essentially the same as runasync but with interface blocking

It's worth noting here that in NodaLogic with Python/Android handlers, you can, in principle, always use run, and launch async and progress bar display using client methods, which allows for a flexible approach (for example, you can make a progress bar only on a button).

We consider Python as the execution engine everywhere.

In the event, we specify the methods that need to be defined for the class. This can be done either in the configurator mode, on the Methods tab, or simply in the handler code in the class body—the system will pull the methods from the code. Incidentally, you can also create classes from code—they will be automatically pulled into the configuration by the designer.

An example of a method as it looks in a constructor (if working with a method from a constructor):

.. code-block:: Python

self.Show(
  [
    [{"type":"Input","id":"input1","caption":"input 1","value":"@input1"}],
    [{"type":"Button","id":"button1","caption":"Get result"}]
  ]
)

return True,{}

And this is what the whole class looks like:

.. code-block:: Python

class MyClass(Node):
    def __init__(self, modules, jNode, modulename, uid, _data):
        super().__init__(modules, jNode, modulename, uid, _data)

    """Class MyClass"""

    def Open(self, input_data=None):
        self.Show(
          [
            [{"type":"Input","id":"input1","caption":"input 1","value":"@input1"}],
            [{"type":"Button","id":"button1","caption":"Get result"}]
          ]
        )

        return True,{}
    def Input(self, input_data=None):
        toast(self._data["input1"])
        
        return True,{}


In the method itself we have parameters:

``self`` – a reference to a class object. You can access both the class's own methods and the parent's methods (more details here).

It is especially worth noting the property of any node ``self._data`` - the node's data storage

The method can also accept input_data, a dictionary containing input data. UI events don't pass anything to it, but if you call a method from another method, for example, you can pass data there as a dictionary.

The method's result is also returned via a True/False, output_data tuple. The first part of the tuple can influence the execution of an array of event handlers (if False, the remaining handlers will not be executed), or it can be used at your discretion.

From the handler, you can work not only with the current node, but also with other nodes through the class object and methods of these nodes.

Node Event Types
---------------------

This section lists events related to a specific node instance. Other events (not related to a node) are called "General Events" and are described in the corresponding section.

**onShow/onShowWeb** - launch the node form on the mobile/web client when opened in any way (from the list, using the _open method, etc.)

**onResume** - only for the mobile client, an event when returning to the form (for example, when returning from another node form)

**onInput/onInputWeb** - the primary input event for mobile/web clients. This event handles both user input and interception of events, such as scanner events.

**onAcceptServer** is an event before data is written. It is executed only on the server, regardless of where the writing is initiated (clients or API requests). This is the primary event for building server business logic. The handler's input_data is passed the _saved_state variable, which stores the state of _data before the change, while _data stores the current state before writing. The developer can influence the data in _data before writing—appending or changing data. Writing can also be aborted by returning False in the tuple. For example: ``return False,{"message":"wrong data"}``



Executing methods on the server
------------------------------------

The server executes web client events, the onAcceptServer event for persisted data, and will eventually include other common events, such as scheduled tasks. Any node can also be accessed via the API (which is automatically generated when the class is created in the configuration) and any method can be executed.

You can access their methods using the REST command ``POST /node/<class_name> /<node_id>/<method_name>`` , which is generated for each class (in the API tab of the class). You can pass parameters (they will go into the method's input_data) and get the result.

The class also has other API commands (described in the Synchronization section) that allow you to:

* create/update a node of the selected class
* register the node in the selected room for transmission to devices
* get all nodes
* access a specific node to get/update/delete

Working with a node on a server via RemoteClass
-------------------------------------------------

Despite the presence of REST-API, it is more convenient to work in the Python handler through a wrapper in the form of a RemoteClass object
Through it, you can get a class on the server, create/update nodes on the server, and execute their methods.

To work, you first need to get a remote class object: GetRemoteClass(class_name,server_url="") , where you specify the class name and optionally specify a server alias. The server URL and alias (as well as the default server) should be specified in the Servers section of the configuration. At least one server must be the default. For example, here's how to access a node class on a server:

``Warehouse = GetRemoteClass("Warehouse")``

The remote class (RemoteClass object) has methods:

* **get(self, uid)** – returns the remote node by id
* **create(self, data=None)** – creates a node on the server, you can pass _data for initialization
* **all(self)** – returns a list of all nodes of the class

Example:

.. code-block:: Python

# Get the deleted class
Warehouse = GetRemoteClass("Warehouse")
# Getting the object
wh1 = Warehouse.get("4503")
if wh1:
     toast("Warehouse found")
     # Call the income method
     result = wh1.income({"qty": 1})
     message("Method result:"+str(result))
     # Data access
     #message("Quantity:"+ str(wh1._data["qty"]))
     #message("Name:"+str(wh1._data["name"]))

     # Data update
     wh1._data["name"] = "Updated warehouse #1"
     wh1._save()
     toast("Updated name:"+wh1._data["name"])

# Creating a new object
new_wh = Warehouse.create({
    "name": "New warehouse",
    "qty": 50,
    "_id":"2"
})
message("New warehouse ID:"+ new_wh._data["_id"])

# Getting all objects
all_warehouses = Warehouse.all()
"""for wh in all_warehouses:
    print(f"Warehouse {wh['_id']}: {wh['name']} - {wh['qty']} units")"""

The remote node (RemoteObject) has public methods and also user methods and _data

* **_save(self)** - saves data to disk
* **_register(self, room_uid)** – registers a node in a room so that other devices can receive it via Room

Executing Node Methods via Room
----------------------------------------

Finally, it's possible to directly access a device node from an external system (remember, the device is connected via WebSocket). In this case, we access the room mechanism via HTTP and transmit the request via WebSocket.

In response to the request, we receive a request_id by which later (when the remote node responds) we can read the execution status and result


Methods of classes and nodes on the client.
---------------------------------------

In the solution, user classes are inheritors of Node and, in addition to their own methods, inherit the class and object methods described in this section.
Here is an example showing the principle of working with nodes.

.. code-block:: Python

class Note(Node):
    def __init__(self, modules, jNode, modulename, uid, _data): #required to be added by the constructor
        super().__init__(modules, jNode, modulename, uid, _data)
    def MyMethod(self, input_data=None): #user method
        pass

#...
class_obj = Note() #Custom class object
new_note = class_obj.create() #New node
new_note._data["some_variable"]= 1
new_note.MyMethod({"a":5,"b":2})
new_note._save() #New node


Mobile Client class methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **create(uid=None, data=None)** – creates a node and returns the node object. You can pass a uid
* **get(uid)** – get an object by UID
* **get_all()** – get all nodes of a class
* **sort( key=None, reverse=False)** – returns objects sorted by key. And also auxiliary ones: ``sort_by_field(field_name, reverse=False), sort_by_numeric_field(field_name, reverse=False), ort_by_date_field(field_name, reverse=False, date_format='%Y-%m-%d')``
* **find(condition)** – search for objects by condition in the form of a lambda expression of the type lambda node: node['price'] > 100
* **count()** - returns the total number of objects of the class
* **_upload_all(cls, server_url=None, config_uid=None, condition=None)** – uploads all class nodes to the default server or to a specific server
* **register_all(cls, room_uid, server_url=None, config_uid=None, condition=None)** – registers all class nodes in the room for other devices


Mobile client node methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **_save()** – writes the node to disk. You can also check the Autosave box in the class, and input from input elements will be written automatically. Important! When using Autosave, if you don't want certain variables written, their names must begin with "!"
* **delete()** - deletes a node and all its child nodes
* **_upload(self, server_alias=None, config_uid=None)** - uploads/updates the node to the server.
* **_delete_from_server(self, server_alias=None, config_uid=None)** - deletes an object on the server
* **_register(self, room_uid, server_alias=None, config_uid=None)** – registers an object in a room for other devices

* **AddChild(parent,_class,uid=None,_data=None)** – adds a child node of the selected class. ``new_line = self.AddChild("OrderLine")``
* **RemoveChild(parent,uid)** – removes a child and all its descendants recursively
* **GetChildren(self, level=None)** – gets all child nodes and their children. You can choose the level to which to slice.

Helper functions for working with nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **to_uid(nodes_list)** – converts a list of nodes into a list of UIDs
* **from_uid(uids_list)** – converts a list of UIDs into a list of nodes

Server and web client class and node methods

-------------------------------------------

Server and Web Client Class Methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **create(uid=None, data=None)** – creates a node and returns the node object. You can pass a uid and an initial _data value.
* **get(uid)** – get an object by UID
* **get_all()** – get all nodes of a class
* **sort( key=None, reverse=False)** – returns objects sorted by key. Also helpful: sort_by_field(field_name, reverse=False), sort_by_numeric_field(field_name, reverse=False), ort_by_date_field(field_name, reverse=False, date_format='%Y-%m-%d')
* **find(condition)** – search for objects by condition in the form of a lambda expression of the type lambda node: node['price'] > 100

Server and Web Client Node Methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **_save()** – writes the node to disk.
* **delete()** - deletes a node and all its child nodes
 
* **AddChild(parent,_class,uid=None,_data=None)** – adds a child node of the selected class. ``new_line = self.AddChild("OrderLine")``
* **RemoveChild(parent,uid)** – removes a child and all its descendants recursively
* **GetChildren(self, level=None)** – gets all child nodes and their children. You can choose the level to which to slice.

Helper functions on the client for working with nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **to_uid(nodes_list)** – converts a list of nodes into a list of UIDs
* **from_uid(uids_list)** – converts a list of UIDs into a list of nodes

Transactions
--------------

Common methods for both the client and server allow you to organize accounting in the node (in _data), maintain a secure transaction log, and calculate totals by analytics. For example, you can track products by package in the "Finished Goods Warehouse" node, and track the latest prices in the "Product" node. The purpose of transactions is to:

* **always cumulative** (transactions are always added)
* inextricably linked to the total (the remainder of the indicator is always the result of the last transaction, and its remainder is the result of the previous one, etc.)
* **protected** (transactions form a chain protected by a hash)
* This approach provides the highest performance – to calculate the balance, you don't need to perform a sum on the table; just take the last transaction and add the new value to it. To get the balance, you just need to take the latest transactions.

In general, it is possible to create an accounting system purely on nodes (without transactions), but transactions are a mechanism tailored for accounting purposes.

There are two types of results:

* **Sum totals** (for sum transactions). This is when the total is simply calculated—i.e., the new value is added to the previous one. For example, the remaining inventory is income +5, expenses -1. Balance - 4
* **Slice of the latest values**, by transaction status. The new value replaces the previous one. For example, the price is 100, the new price is 105. Balance is 105.

Transactions and balances are separated by schemas. This is a simple string identifier that allows you to differentiate balance types. For example, you can simultaneously manage product balances by "item_unit" (schema _sku_unit) and "whole product" (schema _sku). The schema must match the transaction keys—an array of keys.

Methods:
 
1. _sum_transaction(self, scheme_name, period=None, keys=None, values=None, meta=None)

* scheme_name – scheme identifier,
 
* period – date/time of the transaction,
 
* keys – array of keys
 
* values ​​– array of values
 
* meta – dictionary of arbitrary data

2. _get_balance(self, scheme_name) – gets the balance for total transactions

3._get_sum_transactions(self, scheme_name) – gets an array of transaction sums

4. _state_transaction, _get_state_balance, _get_state_transactions – are completely analogous to amount transactions, only the principle of calculating the balance differs.

 

Example

.. code-block:: Python

WClass = GetRemoteClass("Warehouse") #get the class on the server on the client
wh = WClass.get(self._data.get("_id"))
res = wh._sum_transaction(
    "sku_balance", #scheme name
    "2025-09-05", #transaction date
    [self._data.get("sku")], #analytics keys
    [5], #values ​​(quantity of goods)
    meta={"comment": "Receipt note #1"}
)

balance = wh._get_balance("sku_balance") #getting balances



