.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Classes
==============

A node class is, on the one hand, a data-storage aspect (the structure of a node’s data and the rules for working with it, so it can be stored and have an API to access the class and its nodes), and, on the other hand, a set of methods that implement the node’s behavior in the system. In a class, you define handlers (subscriptions) for node events, and you describe which methods should be executed when events occur. Client-related topics are described in the **Mobile client** section.

This section describes working with class methods separately from node UI/UX: class methods (in NL these are called “nodes”) are inherited from the base ``Node`` object.

Execution of methods and handlers
-----------------------------------

A method can be executed:

 * Locally on the server (and in the web client) and on the mobile client
 * In the mobile client you can create a ``RemoteClass`` and obtain a remote “copy” of a server class with its methods and data. Under the hood this uses HTTP requests.
 * An external system can execute a method on the server via REST API
 * An external system can execute a method even on the client via Room, by sending a request and then another one to fetch the result

Executing node events
---------------------

On the client, events are generated (the **Events** tab). Each event has a ``listener`` field — effectively, a filter. A handler is executed only when its ``listener`` matches the event’s listener.

Multiple handlers can be subscribed to a single event — they will be executed sequentially.

The ``action`` property defines how the event is executed. Available actions:

 * ``run`` — run a method synchronously
 * ``runasync`` — run a method asynchronously
 * ``runasyncmodal`` — similar to ``runasync`` but with UI blocking (modal state)

Note: in NodaLogic, Python handlers follow the same approach across clients and server. This allows a unified style (e.g., you can show a progress bar only on a specific button).

In all examples below, Python is used as the execution engine.

In an event you specify method names that must be implemented in the node class. If methods are missing, they will be pulled into the configuration automatically by the constructor.

Handler input data
------------------

When a handler runs, the node has access to:

 * ``self._data`` — the node data (JSON-like)
 * ``listener`` — the listener of the event
 * ``input_data`` — a dictionary with additional input data (optional)

If a method is called from another method, you can pass data as a dictionary via ``input_data``.

Method results
--------------

A method may return a tuple ``True/False, data``. ``True`` means success; ``False`` means failure (subsequent handlers will not run). You can also use this mechanism for your own purposes, e.g. returning a message:

``return False,{"message":"wrong data"}``

From a handler you can work not only with the current node, but also with other nodes via the class object and their methods.

Node event types
----------------

This section lists events related to a specific node. Global events are called **Common events** and are described in the corresponding section.

**onShow/onShowWeb** — open the node form in the mobile/web client; fired when opened by any means (from a list, via ``_open`` method, etc.).

**onResume** — mobile only; fired when returning to the form (e.g. returning from another node form).

**onInput/onInputWeb** — main input event for mobile/web; includes both normal input and intercepted events such as scanner events.

**onAcceptServer** — server-side event before writing data. Must return ``True`` to allow saving. If it returns ``False``, saving is cancelled.

Executing methods on the server
------------------------------------

On the server, web-client events and ``onAcceptServer`` are executed. You can also access any class and execute any method via the REST API (the API is created automatically when a class is defined in the configuration).

You can call methods using a REST command ``POST /api/<class_name>/<node_uid>/<method_name>`` and pass parameters (they will appear in ``input_data``) and get the result.

The class API also includes other commands (see **Synchronization**) that allow you to:

 * create/update a node of the selected class
 * retrieve a node by UID
 * search nodes by filters (depending on the server implementation)

Working with a node on the server via RemoteClass
-------------------------------------------------

In the mobile client (and in the web client when connected to the server), you can create a ``RemoteClass`` object and work with server nodes as if they were local.

Typical flow:

 * Get a remote class: ``WClass = GetRemoteClass("Warehouse")``
 * Get/create nodes using class methods
 * Call node methods remotely; under the hood it is an HTTP request

Executing node methods via Room
----------------------------------------

Room allows sending commands to a device (client). This enables a scenario where an external system sends a request into a room, the client executes a method and then the external system polls for the result.

Class and node methods on the client
------------------------------------

Client-side methods are split into mobile and web/server sets.

Mobile client class methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **Create(uid=None,_data=None)** — create a new node object locally
 * **Get(uid)** — get a node by UID from local storage
 * **All()** — return all nodes of the class
 * **Find(condition)** — search nodes using a lambda condition

Mobile client node methods
~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **_save()** — save the node to local storage
 * **delete()** — delete the node and all its child nodes
 * **AddChild(_class,uid=None,_data=None)** — add a child node of the given class
 * **RemoveChild(uid)** — remove a child (recursively with its descendants)
 * **GetChildren(level=None)** — get child nodes (optionally limited by depth)

Helper functions for working with nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **to_uid(nodes_list)** — convert a list of nodes into a list of UIDs

Class methods for server and web client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **create(uid=None,_data=None)** — create a node on the server
 * **get(uid)** — get a node by UID
 * **get_all()** — get all nodes of the class
 * **sort(key=None, reverse=False)** — return sorted objects
 * **find(condition)** — search objects using a lambda condition like ``lambda node: node['price'] > 100``

Node methods for server and web client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **_save()** — save node to disk
 * **delete()** — delete node and its descendants
 * **AddChild(parent,_class,uid=None,_data=None)** — add a child node of the selected class. Example: ``new_line = self.AddChild("OrderLine")``
 * **RemoveChild(parent,uid)** — remove a descendant recursively
 * **GetChildren(self, level=None)** — get all descendants; optionally limit depth

Client-side helper functions for nodes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 * **to_uid(nodes_list)** — convert a list of nodes into a list of UIDs

Transactions
------------

There are 2 transaction types:

1. **Sum transactions** — simple “add/subtract” transactions with analytics keys.

Transaction fields:

 * ``period`` — transaction date/time
 * ``keys`` — array of analytic keys
 * ``values`` — array of values
 * ``meta`` — a dictionary of arbitrary metadata

Methods:

2. ``_get_balance(self, scheme_name)`` — get balance for sum transactions  
3. ``_get_sum_transactions(self, scheme_name)`` — get an array of sum transactions  
4. ``_state_transaction``, ``_get_state_balance``, ``_get_state_transactions`` — stateful transactions; differ only by the balance calculation principle.

Example
~~~~~~~

.. code-block:: python

   WClass = GetRemoteClass("Warehouse") # get server class in the client
   wh = WClass.get("wh1")
   wh._sum_transaction(
       "sku_balance", # scheme name
       "2025-09-05",  # transaction date
       [self._data.get("sku")], # analytic keys
       [5], # values (quantity)
       meta={"comment": "Inbound delivery note #1"}
   )
   balance = wh._get_balance("sku_balance") # get balance
