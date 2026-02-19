.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov 5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Quick start
===========================

Reminder on the transfer of decisions
--------------------------------


After transferring a solution from one server to another or loading a configuration from a file, you may need to perform several actions:

1. Change the current_module_name in the file. You can take it, for example, from the URL string of the same configurator.
2. If you use _upload or RemoteClass to upload from a mobile client to a server, you must specify a default server (a different server may be specified when uploading from a file).
3. If you are using migration to mobile devices, you need to go to the Rooms section of the configuration (in the configuration itself) and specify the correspondence between nicknames and actual rooms.

Example configurations for all NodaLogic capabilities are available in the Samples folder on GitHub: https://github.com/dvdocumentation/nodalogic


Introductory example (mobile platform)
--------------------------------------

*Let's create a new project, check that it runs on the device, and create a process consisting of one screen where two numbers will be entered and the result of their addition will be calculated by pressing a button.*

First, log in to https://nmaker.pw/. I also recommend logging into your profile and setting a Display Name—this is optional, but it will look nice in the repository. Next, create a new configuration, give it a name, and save it.

.. image::_static/noda_config_name.png
       :scale: 55%
       :align: center

And below, under "Sections," we'll add a new section. By the way, I highly recommend looking at the examples of using commands in sections—it's a very handy feature. You can do things without going into a process or node. For now, we've simply created a section.

.. image::_static/noda_conf_section.png
       :scale: 55%
       :align: center

At this point, we can deploy the configuration to the device. An important difference is that the client can have any number of active configurations at once. They can be located in different sections, or in the same sections. They may even be invisible (performing some actions in the background).

Go to Settings - Configuration Repo - Add Configuration and scan the QR code. There's another QR code in the menu, this one for rooms, but we won't need it for now.

.. image::_static/noda_repo.png
       :scale: 75%
       :align: center

As a result, the configuration should load and a section should appear in which there is nothing.

Let's create a process that will add two numbers in input fields:

1. Go to Classes and create a Calculator class. The class type is User Process. This is exactly the same as the "process" in Simple. It's also a node, but it exists only once and doesn't need to be created—it's created when the configuration is loaded. In all other respects, it's the same node as any other node.

2. Without a cover page, classes look unpresentable. The cover page, like other UI elements, is defined using generic "string" markup. We'll cover that in the next step, but for now, copy the text from the tooltip below and edit it to look something like this:

.. code-block:: Python

[[{"type":"Text","value":"Adding Numbers","bold":true}],[{"type":"Text","value":"Getting to Know UI/UX"}]]``

We will also indicate our section

And you need to click "Save"

.. image::_static/noda_section.png
       :scale: 75%
       :align: center


3. Next, we'll add a method, call it Open, and set the engine type to Android/Python.

.. image::_static/noda_method.png
       :scale: 55%
       :align: center

And we will write the following command in it

.. code-block:: Python

self.Show( [
  [{"type":"Input","input_type":"number","id":"a","caption":"A"},{"type":"Input","input_type":"number","id":"b","caption":"B"}],
  [{"type":"Button","id":"calculate","caption":"Add"}]  
  ])

We just made a command that places 2 number input fields and a button below them.

This is what's called primary ("line") markup in Noda. It's everywhere—in screens, lists, dialogs, and covers. There's also an alternative—container markup—but we won't discuss that now.

The essence of the markup is as follows:

.. code-block:: Python
 
[#vertical container
[{object 1},{object 2}], #line 1 by element height
[] #line 2 by element height
…
]

This means that by default, row and object properties (height/width/weight) can be changed, plus, as I already mentioned, containers (also objects) can be used. Why is this the default? This suits the most common tasks.

Why did I use dynamic markup instead of static markup in Simple's configurator? Here are the reasons: 1) it's easier to read, 2) it's easier to understand and generate with LLM, and 3) it's dynamic right out of the box, so you can draw whatever you need. Consequently, screens can be assembled from redefinable blocks.

.. note:: Speaking of screens – Simple had them, but here they're gone? Yes, they're logical. Essentially, a node has one screen on which you can display as many Shows as you want, toggled, for example, by a button, and for convenience, they can be hidden using different methods. Look at the first example from the Android configuration – it has three screens.

But the method isn't everything. We need to attach it to an event. Why did Simple do this together, but here we need to perform an additional action? Because it's a more flexible approach. For example, the method we created can be called from another handler's code, and it will do the same thing as an event—redraw the screen.
Therefore, let's add

.. image::_static/noda_event.png
       :scale: 55%
       :align: center

Now you can refresh the configuration (Options menu - Restart). And see what happens.

.. image::_static/noda_result.png
       :scale: 75%
       :align: center


4. The button exists, but it doesn't work. Let's add a calculate method.

With this code:

.. code-block:: Python

res = self._data["a"]+self._data["b"]
toast(str(res))

Note what's happening. self is the node itself. The node has _data, which is its dynamic and persistent (if persisted) memory. In Simple, this is a hashMap, but in Simple, the hashMap was a string map, while here it's a JSON-compatible one.

Numbers are automatically stored there as numbers, not as strings, and do not need to be converted.

And we need to hook up an event. We created a button with id=calculate, so we'll create an onInput event with a filter by listener=calculate.

.. image::_static/noda_calculate.png
       :scale: 55%
       :align: center

Let's update the configuration and check how it works. That's all.


.. image::_static/noda_editor.png
       :scale: 55%
       :align: center

.. note:: But to understand how everything works, go to Configuration - Android Handlers. There you'll see the code for our Calculator class. You can see that it inherits Node, meaning it can use Node methods. We also see that this is a separate Python module that loads when the configuration file loads, meaning we can place global variables and functions there and use them from different classes. But we can still edit them from the class window. In fact, you can write code directly in the module (or in your IDE) – the classes will pick it up. It's essentially like Python in Simple, but with the behavior of Pythonscript.


Example of a storage server solution with offline clients
-----------------------------------------------------------------------

The previous example involved a node, but not a typical one. It was a process node, while the foundation of NodaLogic solutions is a data node, and a solution is a collection of such nodes. Therefore, the current example will focus on typical nodes—nodes of the "Data Node" type—a collection of stored data and class methods.

*The task is as follows: we'll send a task document with a comment from an external system to the performer. The performer will need to find the products, scan their barcodes (the system will check that the barcode is in the database and highlight the product), enter the quantity, and attach photos of the inventory process to the document (for simplicity, these will be general photos for the document, not photos for each line). The resulting document will be sent to the server, where it will then be retrieved via a request from the external system. Infrastructure-wise, it looks like this: the external system sends the task documents to the server, mobile clients communicate with the server and receive the tasks, sending them back when ready, and the external system then retrieves the completed documents from the server, also via an HTTP request. In other words, we'll quickly build a client-server system with a mobile app featuring local storage and synchronization.*

**Step 1.** First, let's create a simple class to which we'll send tasks and collect barcodes and other data. We don't need methods or anything else yet. Once you've created the class, an API will be immediately available through which nodes (class objects) can be passed as external requests.

Hint: via API with ready-made requests in the API tab. But we'll use that a little later.

.. image::_static/qs_class.png
       :scale: 55%
       :align: center

**Step 2.** Add the configuration to the repository on the device if it has not been added yet.

.. image::_static/qs_repo.png
       :scale: 55%
       :align: center

Step 3. To communicate with devices, use Rooms. This is essentially a WebSocket. Let's create a room and connect it to the device(s).
.. attention:: In addition to scanning the room's QR code, you must also enter your login/password for the service in the settings. This is not transmitted in the QR code for security reasons.
.. image::_static/qs_settings.png
       :scale: 55%
       :align: center

**Step 4.** Now we need to transfer several test nodes to the server and then to the devices in the room. Let's send a request with a couple of objects (I'm using Postman for this example) using the API from the bookmark. We'll use a request with room registration (this combines two actions - transferring nodes to the server and registering devices in the room. You can also do this with two requests). For me, this is the command; you will have your own identifiers. Also, don't forget about authorization. POST https://nmaker.pw/api/config/348fa9a9-dd2f-4a76-8c73-b5c15bbb3ea1/node/MyDoc?room=db73145a-2d3d-4afb-954f-e158e7c86e02 *

*Here you need to specify the room_id of the created room.

Let's make two of these tasks without the tables for simplicity, but just a verbal description.

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

**Step 5.** Let's go to the device and check that nodes have appeared on the My documents tab.

.. image::_static/qs_nodes.png
       :scale: 55%
       :align: center

**Step 6.** In addition to the documents, we also need to transfer a product catalog with barcodes. To do this, we need to create a "goods" dataset. We'll configure its search fields and hash indexes. Then, we'll transfer the "product catalog."

.. image::_static/qs_dataset.png
       :scale: 55%
       :align: center
Let's take the query text from the hint, substitute the dataset name there, and pass in several products with real barcodes so that there is something to scan.

``https://nmaker.pw/api/config/348fa9a9-dd2f-4a76-8c73-b5c15bbb3ea1/dataset/<dataset_name>/items``

This is the JSON

.. code-block:: JSON

  [
   {"_id": "1", "name": "WIFI wall switch 3 line", "barcode": "1000075755897"},
   {"_id": "2", "name": "EKF MRVA", "barcode": "4690216127392"},
   {"_id": "3", "name": "Smurtbuy AAA 10x", "barcode": "4690626023178"},
   {"_id": "4", "name": "Duracell CR2032", "barcode": "6911332373226"}
]

Now everything is ready to make the client part.

**Step 7.** Our nodes look ugly. Let's change the skin. Our MyDoc nodes have a title and instruction. Let's display it and the node ID at the same time. We'll place three "rows" with the ID, title, and instruction. You can read more about the markup rules in the Mobile Client section.

.. image:: _static/ qs_cover3.png
       :scale: 70%
       :align: center

**Step 8.** Now it's time to draw the node shape. We'll place the same information as in the cover, plus a table of scanned items. For brevity, we won't create a record layout in the table—it will generate one automatically. We'll place the barcode, product name, and quantity in the rows.

The table data – list_scanned, will be taken from self._data["scanned"] where we will place them during scanning

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

Let's add an onShow event handler and check the result. The table isn't visible because there are no records.

**Step 9.** We need to add scanning and quantity input.

First, let's connect the camera scanner (a hardware scanner is connected in a similar way) using the PlugIn command ``self.PlugIn([{"type":"CameraBarcodeScannerButton", "id":"barcode_cam"}])``
Let's write a method that will process a barcode, search it in the reference book, and if found, request the quantity in a dialog. This could be done differently (using screens), but I want to demonstrate how to work with a dialog. To do this, we search the dataset, and if found, we take the product name and display the dialog. For now, we'll simply store the barcode and name in _data – we'll need them later. It doesn't have to be stored like this – you can create variables in the handler module. I just use the online designer and find it more convenient.

This is the text that turned out

.. code-block:: Python
 
barcode = self._data["barcode_cam"]
goods = GetDataSet("goods")
res = goods.getStr("barcode",barcode)
if res == None:
   speak("Product not found")
else:
   result = json.loads(res)
self._data["barcode"] = barcode
self._data["sku"] = result.get("name")
 
Dialog("dlg_qty","Enter quantity","Ok",None,[[{"type":"Input","id":"qty_dialog","caption":"Quantity","input_type":"number"}]])  
  
We attach our method to the barcode event.

And let's immediately handle the event for the quantity input dialog. Note that the listener filter has the _positive suffix, indicating the user has confirmed the input.

.. image::_static/qs_process.png
       :scale: 70%
       :align: center

Having finally received the quantity and the previously obtained product name and barcode, all we need to do is add it to the data table and update the form. (You can update just the table, but I do it the simple way.) We save the node and update the screen.

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

Checking the scan

Step 10. Let's add photos and a gallery. The previous step was incredibly complicated, so with photos, everything will be simple. To add photos from your camera roll and a gallery, simply connect them via PlugIn, and it will work. Check it out.

.. code-block:: Python

self.PlugIn([{"type":"CameraBarcodeScannerButton", "id":"barcode_cam"},
  {"type":"PhotoButton", "id":"capture_photo"},
  {"type":"MediaGallery", "id":"pic_files"}
])

But we need to do more than just store photos on our device; we also need to transfer them to an external system, and the gallery—paths to files that won't give us any information.

Here, it's more appropriate to actually implement background conversion outside the node form, on a schedule. However, we'll convert photos as they're added directly on the screen, also in the background. The idea is that we'll run the lengthy process (converting to bas64) in the background without slowing down the interface. Meanwhile, our photo is automatically added to the gallery (a gallery variable). The documentation describes a slightly different approach—completely replacing the gallery conversion—but we'll use a hybrid approach: we'll add photos to the gallery automatically and have a handler handle the base64 conversion.

We create a method and attach it to the camera event

.. code-block:: Python

if "photos_base64" in self._data:
   photos_base64= self._data["photos_base64"]
else:
   photos_base64= []
base64 = getBase64FromImageFile(self._data["result_file"],50,50) #doesn't lead to anything, just got a crop in base64
photos_base64.append(self._data["result_file"])
self._data["photos_base64"] = photos_base64
self._save()
toast("Saved to photos")  

This is what our “document” looks like now.

.. image:: _static/ qs_doc_result.png
       :scale: 70%
       :align: center


**Step 11.** Sending to the server. We'll also create a button to upload the document to the server.

Let's add a nice button to the toolbar

``{"type":"ToolbarButton","id":"btn_upload","caption":"Upload","svg":svg,"svg_size":24,"svg_color":"#FFFFFF"}``

Let's create a method for uploading. We'll write an event handler. We also need a default server in Servers. To upload to the server, we simply use the _upload method, which overwrites the _data object on the server. There are many implementation options here. For example, you could create a server-side accept_data method in the server node, passing the required data as parameters, and it will be processed. But here's a simple one.

.. code-block:: Python
 
status,error = self._upload()
if status:
   message("Object uploaded")
else:
   toast(error)  

Step 12. Now we need to retrieve what we've done on the device from the external system. Let's use the API.

This is the end result: we've created a server, an offline solution for collecting products and photos, and can now retrieve the results of device work from the server.

.. image:: _static/ qs_doc_result.png
       :scale: 70%
       :align: center


.. hint:: It's important to understand that the demonstrated "nodes + datasets" stack is NOT the only solution! You can organize storage for both user tasks and the product catalog in SQLite (in Python, initialize the database in the application folder in onLaunch and write/read to it). Pelican and key-value storage—a NoSQL approach for the same purpose—are also an option.

This example can be downloaded here: https://disk.yandex.ru/d/Gtpq4nfO7oYskA or in the Samples section on GitHub


Another example of a client-server application with web and mobile clients, working with photos in different ways. Organizing settings.
-----------------------------------------------------------------------------------------------------------------------------------------------------

.. image::_static/qs_photo_notes.png
       :scale: 50%
       :align: center

*This example implements notes (Subject/Text) with attached photos, with a full-featured web client, in addition to a standalone mobile client. The mobile client sends photos to the server. Two methods for working with photos are shown: with base64 and with plain files. The "Settings" node pattern for solution constants is also shown.*

This is not a step-by-step example, just what needs to be done.

Class for taking notes with photos for mobile and web clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We'll create a class to store notes and photos for them – PhotoNote.

We will create the form not through the onShow handler, but simply in the Init screen class property.

.. image::_static/qs_photo_notes_class.png
       :scale: 70%
       :align: center

But we still need to create an onShow handler since PlugIn doesn't provide one yet.

.. code-block:: Python
 
self.PlugIn([
  {"type":"PhotoButton", "id":"capture_photo"},
  {"type":"MediaGallery", "id":"pic_files"}
])

We're making the same handler for the web client, but there's no camera, but you can connect a file gallery if needed.

.. code-block:: Python
 
  self.PlugIn([
  {"type":"MediaGallery", "id":"pic_files_web"},
  {"type":"FileGallery", "id":"files"}
])

Before moving on, we need to organize the storage of a base64 constant—a checkbox that stores the method for storing and transferring files. In this example, we'll implement two approaches.

How to organize settings?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let's create a Settings class of the custom_process type. It's perfect for storing constants and settings. The class doesn't need any methods, just a Cover and Init screen.

Cover:

.. code-block:: Python

[[{"type":"Text","value":"Client-server image exchange settings"}],["Use base64|@base64"]]

Screen:

.. code-block:: Python

[[{"type":"Switch","caption":"Use base64 compressing","id":"base64","value":"@base64"}]]

We'll need a save, so let's enable "Use standard commands." That's all there is to it.

I'll show you right away how to **get** the settings:

In the mobile client: find the settings node and retrieve its value. To access the node using the formula <configuration uid>$<class name> (this is the uid for custom_process), we use the constants current_module_name
        base64opt =False
        settings = Settings.get(current_module_name+"$Settings")
        if settings:
          base64opt = settings.get("base64",False)

In the web client: there is no such constant, but you can calculate it from the node's _id, it is always there:
        #get instance UID
        s = self._data.get("_id")
        i = s.find("$")
        current_config_uid = s[:i] if i > 0 else ""
        #getSettingsnode
        settings = Settings.get(current_config_uid+"$Settings")

Transferring images to the server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, we need to organize file sending based on the settings. We'll do this in the same way as in the previous example – in an asynchronous handler.

Sending images via bas64 is similar to the previous example. This method has the advantage of encapsulating the data within the node, storing it directly in _data and traveling with it. This means that if you plan to access the node via API from an external program, everything will already be there. With files, conversion will be required. The disadvantage is that conversion operations are quite expensive.

Sending images "via files" is more efficient. It uses regular requests.post. Images are sent to the NodaLogic route /api/userfiles/<configuration_uid>/images , where they are saved to a folder and then stored simply by filename. Here's what we need to send an image:

1. We have the file path. The path is written to self._data["result_file"] after saving the photo.
2. Short file name - it can be calculated using the string method
3. Server URL. Here you'll need getServerUrl, which reads the actual URL using the alias (this is described in the cheat sheet in this section). This is a "common solution" pattern; if you're doing it for yourself, you can simply specify the URL in the handler body—hardcode it.
4. instance uid - current_module_name

Total for the entire processor

.. code-block:: Python
 
base64opt =False
settings = Settings.get(current_module_name+"$Settings")
if settings:
  base64opt = settings.get("base64",False)


if base64opt:
  if "photos_base64" in self._data:
    photos_base64= self._data["photos_base64"]
  else:
    photos_base64= []

  base64 = getBase64FromImageFile(self._data["result_file"],50,50) #doesn't lead to anything, just got a crop in base64
  photos_base64.append(base64)

  self._data["photos_base64"] = photos_base64
  self._save()
  self.upload_to_server()
else:
  import requests
  from com.dv.noda import NodesCore as ncore
  
  server_url = ncore.getServerUrl(current_module_name,"main")

  path = self._data["result_file"]
  filename = path.split('/')[-1]
  url = f"{server_url}/api/userfiles/{current_module_name}/images"

  files = [
      ("files", (filename , open(path , "rb"), "image/png"))
  ]

  r = requests.post(url, files=files, data={"overwrite": "0"})
  if r.status_code==200:
    if "pic_files_web" in self._data:
      pic_array = self._data.get("pic_files_web")
    else:
      pic_array = []
    pic_array.append(filename)      
    self._data["pic_files_web"]=pic_array
    self._save()
    self.upload_to_server()
    
  toast(r.status_code)  

You can also add an ImageSlider to the cover for aesthetics. In this example, it simply receives the path to the file array as its value. You can use it to display captions—some meaningful descriptions; I just used the file names. I also use the cover property to "fit" the element to its size with padding. You can also create thumbnails for this purpose when processing the file.

.. image::_static/qs_photo_notes_imageslider.png
       :scale: 70%
       :align: center


There's nothing else you need to do. You can view the photo gallery in a note on both mobile and web.

The example can be downloaded from Samples on GitHub https://github.com/dvdocumentation/nodalogic - config_PhotoNotes.

A large client-server document-oriented example, with integration with 1C
---------------------------------------------------------------------------------

An example of an accounting solution for warehouse accounting in a document-oriented format is here: https://infostart.ru/1c/articles/2614496/








