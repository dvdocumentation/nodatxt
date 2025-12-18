.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Quick start
===========================

Introductory example
--------------------------

*We will create a new project, check that it runs on the device, and create a process consisting of a single screen where two numbers are entered and the result of their sum is calculated by a button.*

First, log in at https://nmaker.pw/. I also recommend going to your profile and setting a Display Name—this is not mandatory, but it will look nice in the repository. Then we create a new configuration, give it some name and save it.

.. image:: _static/noda_config_name.png
       :scale: 55%
       :align: center

And below, in “Sections,” we add a new section. By the way, I highly recommend looking at examples of using commands in sections—it is a very handy feature. You can do certain things without entering the process/node. For now, we have just created a section.

.. image:: _static/noda_conf_section.png
       :scale: 55%
       :align: center

At this stage, we can already install the configuration on the device. An important difference is that on the client side any number of configurations can be active at the same time. They can be placed in different sections or in the same sections. They may even not be visible at all (performing some actions in the background).

Open Settings → Config repository → Add configuration and scan the QR code. There is another QR code in the menu—for rooms; we do not need it yet.

.. image:: _static/noda_repo.png
       :scale: 75%
       :align: center

As a result, the configuration should be loaded and a section should appear where there is nothing yet.

Let’s create a process that adds two numbers from input fields:

1. Go to Classes and create a class Calculator. Class type – User process. This is the analogue of a “process” in Simple. It is also a node, but one that exists in a single instance and does not need to be created—it is created when the configuration is loaded. In all other respects it is the same node as the others.

2. Classes without a cover look unpresentable. The cover, like other places where there is UI, is defined as a general “row-based” layout. We will talk about it in the next step; for now, do the following: copy the text from the hint below and edit it roughly like this: ``[[{"type":"Text","value":"Addition of numbers","bold":true}],[{"type":"Text","value":"Getting to know UI/UX"}]]``

We also specify our section.

And click "Save".

.. image:: _static/noda_section.png
       :scale: 75%
       :align: center


3. Next, add a method, call it Open, engine type – Android/Python.

.. image:: _static/noda_method.png
       :scale: 55%
       :align: center

And write the following command in it:

.. code-block:: Python

 self.Show(  [
  [{"type":"Input","input_type":"number","id":"a","caption":"A"},{"type":"Input","input_type":"number","id":"b","caption":"B"}],
  [{"type":"Button","id":"calculate","caption":"Add"}]  
  ])

We have just created a command that places two numeric input fields and a button under them.

This is the so-called main (“row-based”) layout in Noda. It is used everywhere—in screens, lists, dialogs, covers. There is also an alternative—container-based layout, but we won’t discuss it now.

The essence of the layout is as follows:

.. code-block:: Python
 
 [#vertical container
 [{object 1},{ object 2}], #row 1 by element height
 [] #row 2 by element height
 …
 ]

This is the default behavior; row and object properties (height/width/weight) can be changed, and, as I already mentioned, containers (which are also objects) can be used. Why this by default? Because it corresponds to the most common tasks.

Why did I use dynamic layout while in Simple it was static in the configurator? Here are the reasons:  
1) it is more visual to read  
2) it is easier to understand and generate for an LLM  
3) it is dynamic from the start—you draw exactly what you need. Correspondingly, screens can be built from overridable blocks. 

.. note:: By the way, about screens—there were screens in Simple, and here there are none? Yes, here they are logical. In fact, a node has a single screen on which you can output as many Show calls as you like, switching them, for example, with a button, and for convenience they can be placed in different methods. Look at the first example from the Android configuration—there are 3 screens there.

But the method is not enough. We must attach it to an event. Why was this combined in Simple, and here we need to perform an additional action? Because this is a more flexible approach. For example, the method we created can be called from another handler, and it will do the same as when called by an event—redraw the screen.
Therefore we add:

.. image:: _static/noda_event.png
       :scale: 55%
       :align: center

Now we can update the configuration (Options menu – Restart). And see what we got.

.. image:: _static/noda_result.png
       :scale: 75%
       :align: center


4. The button exists, but does not work yet. Let’s add the calculate method.

With this code:

.. code-block:: Python

 res = self._data["a"]+self._data["b"]
 toast(str(res))

Pay attention to what is happening. self is the node itself. The node has _data—this is its dynamic and at the same time stored (if saved) memory. In Simple it is a hashMap, but in Simple the hashMap is string-based, while here it is JSON-compatible. 

Numbers automatically appear there as numbers and not as strings, you do not need to convert them.

And we need to attach an event. We created a button with id=calculate, so we will create an onInput event with a filter listener=calculate.

.. image:: _static/noda_calculate.png
       :scale: 55%
       :align: center

Update the configuration and check how it works. That’s all.


.. image:: _static/noda_editor.png
       :scale: 55%
       :align: center

.. note:: For understanding how everything is arranged, go to Configuration → Android handlers. There you will see the code of our Calculator class. You can see that it inherits from Node, i.e. it can use Node methods. We also see that this is a separate python module that is loaded when the configuration is loaded, so we can place global variables and functions there and use them from different classes, while still editing from the class window. In fact, you can write code directly in the module (or in your IDE)—classes will pick it up. This is essentially like python in Simple, but with behavior similar to pythonscript.


Example of a client-server solution with offline clients
-----------------------------------------------------------------------

In the previous example there was a node, but not a regular one. It was a process node, whereas the foundation of NodaLogic solutions is a data node, and a solution is a set of such nodes. Therefore, the current example will focus specifically on regular nodes—nodes of the “Data node” type—a combination of a portion of stored data and class methods.

*The task is as follows: from some external system we will send a job document with a comment describing what the executor needs to do. The executor will need to find items, scan their barcodes (the system must check that the barcode exists in the database and display the item), enter quantity, and attach photos of the inventory process to the document (for simplicity, these will be photos common to the document and not per line). The resulting document should be sent to the server, from which it will then be fetched by the external system with a request. In terms of infrastructure, it looks like this: the external system sends job documents to the server, mobile clients communicate with the server and receive tasks, and as they complete them they send them back, while the external system then retrieves the completed documents from the server via an HTTP request. So we will quickly build a client-server system with a mobile application with local storage and synchronization.*

**Step 1.** First we will create a class into which we will receive tasks and store barcodes and other data. We do not need methods or anything else yet. As soon as you create a class, the API becomes available via which nodes (class objects) can be sent by external requests. 

Hint: see the API tab for ready-made requests. But we will use it a bit later.

.. image:: _static/ qs_class.png
       :scale: 55%
       :align: center

**Step 2.** Add the configuration to the repository on the device if it has not been added yet. 

.. image:: _static/ qs_repo.png
       :scale: 55%
       :align: center

**Step 3.** To exchange data with devices there are Rooms. Basically, this is WebSocket. Let’s create a room and connect it on the device(s).
.. attention:: In addition to scanning the room QR code, you must also enter your Login/Password for the service in the settings. They are not included in the QR code for security reasons.
.. image:: _static/ qs_settings.png
       :scale: 55%
       :align: center

**Step 4.**  Now we need to send a few test nodes to the server and then to devices in the room. We will send a request with a pair of objects (I will use Postman for the example) using the API from the tab. We will use a request that registers the record in a room (this combines two actions—sending nodes to the server and registering them in a room for devices. You can also do this with two separate requests). For me the command is as follows; yours will have its own identifiers. Also do not forget about authorization.  
POST https://nmaker.pw/api/config/348fa9a9-dd2f-4a76-8c73-b5c15bbb3ea1/node/MyDoc?room=db73145a-2d3d-4afb-954f-e158e7c86e02 *

*Here you need to specify the room_id of the created room.

Let’s create two tasks like this without line items for simplicity, just a verbal description:

.. code-block:: JSON

  [{
    "_id": "001",
    "_data":{
        "_id": "001",
        "title": "Barcode scan #001",
        "instruction": "Scan barcodes of electrical products"
    }
 },
 {
    "_id": "002",
    "_data":{
        "_id": "002",
        "title": "Barcode scan #002",
        "instruction": "Scan battery barcodes"
    }
 }
 ]

**Step 5.** Open the device and check that nodes appeared in the My documents tab.

.. image:: _static/ qs_nodes.png
       :scale: 55%
       :align: center

**Step 6.** In addition to documents, we also need to send an item reference with barcodes. To do this, we need to create a goods Dataset. We will configure its fields for search and hash indexes and send the “item reference.”

.. image:: _static/ qs_dataset.png
       :scale: 55%
       :align: center
We take the request text from the hint, substitute the dataset name and send several items with real barcodes so there is something to scan.

``https://nmaker.pw/api/config/348fa9a9-dd2f-4a76-8c73-b5c15bbb3ea1/dataset/<dataset_name>/items``

Here is the JSON:

.. code-block:: JSON

  [
   {"_id": "1", "name": "WIFI wall switch 3 line", "barcode": "1000075755897"},
   {"_id": "2", "name": "EKF MRVA", "barcode": "4690216127392"},
   {"_id": "3", "name": "Smurtbuy AAA 10x", "barcode": "4690626023178"},
   {"_id": "4", "name": "Duracell CR2032", "barcode": "6911332373226"}
 ]

Now everything is ready to build the client side.

**Step 7.** Our nodes look unattractive. Let’s change the cover. Our MyDoc nodes have title and instruction fields. Let’s display them and the node ID as well. We place 3 “rows” with id, title, and instruction. You can read more about layout rules in the Mobile client section.

.. image:: _static/ qs_cover3.png
       :scale: 70%
       :align: center

**Step 8.** Time to draw the node form. We will place the same information as on the cover plus a table of scanned items. For brevity we will not define a custom layout for the table records—it will generate it automatically. In the rows we will place the barcode, item name, and quantity. 

The table data—list_scanned—will be taken from self._data["scanned"], where we will store it as scanning proceeds.

.. code-block:: Python

 if "scanned" in self._data:
   list_scanned = self._data["scanned"]
 else:
  list_scanned = []

 self.Show([
      [{"type":"Text","italic":true,"value":"@_id"}],
      [{"type":"Text","bold":true,"value":"@title"}],
      [{"type":"Table","id":"t1","value":list_scanned }]
    ])

Add handling for the onShow event and check the result. You don’t see the table because there are no records.

**Step 9.** We need to add scanning and quantity input.

First, connect the camera scanner (hardware scanner is connected similarly) using the PlugIn command:  
``self.PlugIn([{"type":"CameraBarcodeScannerButton", "id":"barcode_cam"}])``
We will write a method that will process the barcode—search the reference and, if found, prompt for the quantity in a dialog. This can be done differently (through screens), but I want to demonstrate dialog usage. To do this, we search the dataset; if we find the record, we take the item name and show a dialog. The barcode and name are temporarily stored in _data—they will come in handy. This is not required to be stored this way—you can define variables in the handler module; I am just writing in the online constructor and this way is more convenient for me.

Here is the resulting text:

.. code-block:: Python
 
 barcode = self._data["barcode_cam"]
 goods= GetDataSet("goods")
 res = goods.getStr("barcode",barcode)
 if res == None:
   speak("Product not found")
 else:
   result = json.loads(res)
 self._data["barcode"]  = barcode
 self._data["sku"]  = result.get("name")
 
 Dialog("dlg_qty","Enter  quantity","Ok",None,[[{"type":"Input","id":"qty_dialog","caption":"Quantity","input_type":"number"}]])  
  
We attach our method to the barcode event.

And we immediately implement the dialog quantity input event handler. Note that the listener filter has the _positive suffix—this means the user confirmed the input.

.. image:: _static/ qs_process.png
       :scale: 70%
       :align: center

Once we have quantity and the previously obtained item name and barcode, we just add this to the table data and refresh the form (we could refresh only the table, but I will do it the simple way). We save the node and refresh the screen.

.. code-block:: Python
 
 qty = self._data.get("qty_dialog")

 if "scanned" in self._data:
   scanned = self._data["scanned"]
 else:
   scanned = []
 scanned.append({"barcode":self._data["barcode"],"name":self._data["name"],"qty":qty})    
 self._data["scanned"] = scanned
 #save node
 self._save()
 self.Open()

We test scanning.

**Step 10.** Let’s add photos with a gallery. The previous step was monstrously complex, so with photos we will keep things simple. In fact, to add a photo from the camera and a gallery, it is enough to connect them via PlugIn and it will work. Check it.

.. code-block:: Python

 self.PlugIn([{"type":"CameraBarcodeScannerButton", "id":"barcode_cam"},
  {"type":"PhotoButton", "id":"capture_photo"},
  {"type":"MediaGallery", "id":"pic_files"}
 ])

But we have a task not just to accumulate photos on the device but to fetch them in the external system, while the gallery only stores file paths, which do not give us useful information.

The correct way, in fact, is to perform a background conversion outside the node form, on a schedule. But we will convert photos as they are added, right from the screen, also in the background. The idea is that we hang the long procedure (conversion to base64) in the background, without blocking the interface. At the same time, the image is automatically added to the gallery (the gallery variable). Documentation describes a slightly different approach—fully replacing adding to the gallery—but we will use a hybrid approach: we add to the gallery automatically, and write to base64 in a handler.

We create a method and attach it to the camera event:

.. code-block:: Python

 if "photos_base64" in self._data:
   photos_base64= self._data["photos_base64"]
 else:
   photos_base64= []
 base64 = getBase64FromImageFile(self._data["result_file"],50,50) #does not lead anywhere, we just got a cropped base64
 photos_base64.append(self._data["result_file"])
 self._data["photos_base64"] = photos_base64
 self._save()
 toast("Saved to photos")  

Here is what our “document” looks like now:

.. image:: _static/ qs_doc_result.png
       :scale: 70%
       :align: center


**Step 11.** Sending to the server. We will also create a button to upload the document to the server.

Let’s add a nice button to the toolbar:

``{"type":"ToolbarButton","id":"btn_upload","caption":"Upload","svg":svg,"svg_size":24,"svg_color":"#FFFFFF"}``

And create a method that will upload the node. We define an event handler. We also need a default server set in Servers. To upload to the server we simply use the _upload method, which overwrites the object’s _data on the server. There are many possible implementation approaches. For example, you can create a server method accept_data on the server node and pass the required data as parameters, and it will be processed there. But here we will keep it simple.

.. code-block:: Python
 
 status,error = self._upload()
 if status:
   message("Object uploaded")
 else:
   toast(error)  

**Step 12.** Now from the external system we need to retrieve what we did on the device. We will use the API. 

This is the end result: we have built a server, an offline solution for collecting items and photos, and we can retrieve the devices’ work results from the server. 

.. image:: _static/ qs_doc_result.png
       :scale: 70%
       :align: center


.. hint:: It is important to understand that the demonstrated “nodes + datasets” stack is NOT the only option! You can organize storage of both user tasks and item references in SQLite (in python in onLaunch initialize the database in the app folder and read/write to it). There is Pelican and key-value storage—a NoSQL-style approach for the same purposes, as another option. 

You can download this example here: https://disk.yandex.ru/d/Gtpq4nfO7oYskA

There are also larger examples covering all Android client features: https://disk.yandex.ru/d/rMqYvPEe8DH2ZQ and also for data nodes: https://disk.yandex.ru/d/sO8e2Cqii0Fhjg
