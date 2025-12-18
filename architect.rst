.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Platform Architecture
===========================

NodaLogic is a distributed system that can be either centralized with a server and mobile client devices, or decentralized, in the sense that the logic of “nodes,” the main building blocks of the system, can run both on the server and on client devices; there may be many servers, and each client can in turn also act as a server.

The core of solutions is the “node.” It is an object that is simultaneously a data storage and has executable methods. It is a self-contained microservice that stores data and is ready to interact with other nodes. Frontend and backend solutions are built out of nodes (the boundary between front/back is blurred in NodaLogic, since “the server is wherever the node is being executed at the moment”). In some sense, it is like a neuron that has input/output and weights.

A node can represent entities such as a “task,” “document,” “document row,” any other business entity—warehouse, item, bin. And also virtual entities—for example, “item balance in a bin.”

.. image:: _static/node.png
       :scale: 55%
       :align: center

Node data is stored in _data — this is both operational and persistent memory. _data is a JSON-compatible data structure. A node has a reference to a class. The class defines the behavior of the node, its envelope, describes events (the mapping between events and methods).

Data from the interface and handlers goes into _data, and conversely, data stored in _data is reflected in the interface. Essentially, this is a regular JSON-oriented NoSQL.

On a mobile device, when the user enters something, an onInput event occurs, and if there is any method-handler for it, it processes this data, saves it, may call other methods or send it to other nodes (by calling their methods). In the picture below the situation is: the user pressed a button — an onInput event triggered, and there is a subscription to it in the form of an Input method (python) — the python method (on the mobile device) shows a toast (message) with the variable input1, which stores the value entered in the input field.

.. image:: _static/event.png
       :scale: 55%
       :align: center

On the server (if the node has server methods), the node can be viewed as a microservice. When you create a class, the REST API for that class is automatically created, allowing you to access both the class and its objects—nodes—via HTTP requests. If the node has server methods, they can also be executed remotely both from external systems and from mobile clients.

.. image:: _static/api.png
       :scale: 55%
       :align: center

Thus, formally, **a node is _data + a class (describing methods and behavior of the node in the system)**. On one hand, a node is a data unit, like a record in NoSQL; on the other hand, it is a microservice.

Here is an example of a node class in python (for the mobile client). You can see that it inherits from Node and has some of its own methods. The Open method is responsible for rendering, and Input — for handling input.

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

At the moment, two types of nodes are available:

 * A regular node (data node) with the behavior described above
 * A user process — a node that exists in a single instance, created in the client interface to perform certain user tasks. Unlike a node, it does not need to be created or transferred; it is created when the configuration is loaded

Classes, in turn, are stored in the configuration — a JSON structure or file. Thus, a configuration is a set of node classes. But not only that — the configuration also defines the solution settings as a whole — general events, sections, and so on. That is, the configuration is a JSON storage of node classes, handlers (in the form of a base64-encoded python file), and global settings. More about the structure of this file in the section "Configuration Structure"

.. image:: _static/repo.png
       :scale: 55%
       :align: center

A client device can have any number of configurations loaded simultaneously into the “configuration repository,” and they are all active at the same time, both for the user and for each other. Classes simply tend to be placed in their own sections and do not interfere with one another. The server follows the same principle — each configuration exists independently, each class has its own API, and each node operates independently.

A node can operate on the server and on the client or both simultaneously. As a rule, it works locally — on the machine where it is executed, but it may access a node on the server, which will be described further.

Here is the scenario of *passive node use in an offline solution* (as a data object):

 1. Nodes are pushed to the server via an HTTP request. A class in the server configuration automatically has its own REST API. It can be used to create nodes, request data, and execute methods.
 2. Nodes are sent to clients through the Rooms mechanism. Devices are grouped into rooms via WebSocket and are always ready to receive updates about nodes, just like messengers receive messages.
 3. A node on the device operates independently offline. No connection to the server is required. The client runs UI/UX and accumulates data.
 4. When necessary, data is sent to server nodes by calling their methods, or simply the _data of the node on the server is replaced by _data of the node from the client.
 5. The external system retrieves data through the same REST API.

The scenario where the server plays its role is almost the same but has an important difference — business logic is executed on the server. Let’s look at this using a WMS solution example:

 1. The accounting system sends orders to the NodaLogic server — let’s say these are some types of documents — customer orders, supplier invoices.
 2. Nodes on the server that are task generators create task-nodes for client devices and register them in a Room; they appear on the device.
 3. On the device, the user completes tasks; the data about actual completion is immediately sent to the server (essentially an online mode). Accounting processes are executed on the server, such as calculating item balances, for example.
 4. After completion, either the server sends an HTTP request to the external system, or the system requests completed tasks from the task-generator node (which are effectively actual results). It may also contact the accounting node to request, for example, item balances in bins.

There can also be scenarios *without a server at all*, where a mobile solution is simply created, in which nodes are sent to the server through rooms or even through files. Or node data is not used at all — and this is just a mobile solution.  
Thus, the system can simply serve as a mobile client builder — a frontend builder.

Summarizing all of the above, one can say that a node is a self-contained object, a solution is a set of classes, and a system — whether a server or a client — is a set of objects that can interact with each other. And a client application is a “player” of nodes, while nodes are delivered to it in a messenger-like manner (or, for example, simply through files via email or other messengers), generated by the user or by other nodes. This is a kind of evolution of the idea of suip-files in SimpleUI.
