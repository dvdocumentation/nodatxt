.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov  5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Mobile client (Android)
=============================

Interface functions
----------------------

General structure of the interface and events
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All visual nodes (data nodes and user processes) are placed in sections. 

.. image:: _static/sections.png
       :scale: 55%
       :align: center

Sections are defined in the Sections part of the configuration as an ID and a title. Commands can also be specified for a section; they will be shown as buttons at the bottom of the section in the form of a comma-separated list <command caption>|<command id>. When a button is pressed, a global event onStartMenuCommand is generated, and the command ID is passed as a parameter.
A node class, in turn, is linked to a section via the Section Code property, where you must select an existing section.

All nodes in sections can have a cover or so-called default cover. A cover is defined in the form of standard markup (see below) in the node class, but it can also be defined in _data (the "_cover" property)—thus nodes of the same class can have different markup. It can also be overridden for a specific node by the SetCover method.

.. image:: _static/cover.png
       :scale: 55%
       :align: center

When you tap a node in a section, the node form opens and the onShow event is generated, where you can define a handler for drawing the screen, otherwise it may appear empty. The user can leave this screen and then return (for example, by opening another node on another screen), so in such cases, when returning to the screen, an onResume event is generated, where you can define the same logic as in onShow or something special.

Principles of screen and other visual form layout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The system uses two alternative approaches to layout that can be combined. By default (we will call this the main approach) a “by rows” layout principle is used, i.e. a vertical list of “rows,” each row being an array of elements horizontally (each element is a visual object of some type type). Obviously, this approach is not exhaustive, but it is sufficient for most business applications. The height and behavior of both rows and elements can be adjusted if needed. An alternative to this can be container-based layout—vertical, horizontal, and vertical/horizontal scroll containers. Both approaches can be mixed; for example, you can simply add a container into the first row and build the entire screen using containers (you cannot completely bypass row layout—the layout must still have at least one row).  

The same approach is used for screen layout, node cover, dialog, and list elements, in the form of a JSON layout string (in python method parameters this corresponds to internal list/dict types) of the following form:

.. code-block:: Python

 [ #common vertical list top to bottom
 [{"type":…},{"type":…}], #horizontal row of elements
 [{"type":…},{"type"":…}], #horizontal row of elements
 ...
 ]


Example:

.. code-block:: Python
 
 Layout1 =[ 
 [{"type":"Table","id":"l1","value":table_data,"layout":table_layout }], 
 [{"type":"Button","id":"btn_repl","caption":"@caption"}]
 ]

Visual elements are placed in “rows.” Each visual element has the following properties:
 * **"type"** (string) – element type, 
 * **"id"** (string) – element identifier. For active elements (those the user interacts with), as a rule, an event is generated with a listener property equal to the element id. You can also refer to the element by id and set its properties.
 * **"visible"** (int) – element visibility: 1 – visible, 0 – invisible (but takes space on the screen), -1 – invisible (and does not take space on the screen)
 * **"w"** – element weight. The element’s weight horizontally/vertically (depending on the parent container) relative to other elements. By default w=1. Example:  ``[{"type":"Button","id":"back","caption":"Back"},{"type":"Button","id":"next","caption":"Forward","w":2}] #the next button is twice as wide because it has w=2 (back has w=1 by default).`` In combination with height/width, you can control layout. It depends on the behavior of the parent container. If the container is “wrap content,” then weight does not make sense; it only makes sense when “match parent” or 0.

 * **"width"** – width. Can be set both as a relative numeric size and as absolute values: -1 – full container width, -2 – by element height. Depends on the root container behavior. If the container is measured “by element width,” then “full width” makes no sense.

 * **"height"** – height. Can be set both as a relative numeric size and as absolute values: -1 – full container height, -2 – by element height. Depends on the root container behavior. If the container is measured “by element width,” then “full width” makes no sense.

 * **padding** - internal padding

Default rules are as follows:

 * The root vertical list is stretched to fill the entire screen without scrolling. 
 * All elements in a row by default have height “by element height,” and the row itself is stretched by width “to the full container” (i.e. horizontally from one side of the screen to the other) and has no weight.  
 * But you can change the row properties by adding a "Parameters" element with properties "w", "width" and "height"—that is, for the horizontal row (which is a container) you set weight, width, and height. This is important when a more complex layout is required. 
 
For example, in this layout, the bottom row takes all remaining height because it has weight = 1; then it makes sense to set height = -1 (full height) for the Input element in this row:

.. code-block:: Python

 [ 
  [{"type":"Input","id":"title","caption":"Subject","value":"@title"}],
  [{"type":"Parameters","w":1},{"type":"Input","height":-1,"id":"body","caption":"Text","value":"@body","input_type":"multiline"}]
 ]

The horizontal row itself is a container, but you can also place other containers inside it. In principle, you can completely give up row layout and output only a single row with height equal to full screen, and place all elements inside it as containers.

Containers are used both for layout and for grouping several elements together to control them as a group (for example, visibility).

.. warning::  It is extremely important to remember that if you use containers, you must specify width and height for elements. Usually, containers themselves also need these. And you may also need Parameters on the “row.”

Containers have the following properties:

 * Common properties (type, weight, width, height)
 * The value property – the list of elements. For Card this is the general “by rows” layout (an array of arrays) – [[]], and for other containers—a flat list/array ( [] ) of elements. These are then laid out horizontally or vertically depending on the container type.

The following container variants exist:

 * **"VerticalLayout"** – vertical non-visual container without scrolling
 * **"HorizontalLayout"** – horizontal non-visual container without scrolling
 * **"VerticalScroll"** – vertical scrolling. Keep in mind that scrolling “destroys” the “full height” property of an element.
 * **"HorizontalScroll"** – horizontal scrolling
 * **"Card"** – visual “card” container used to visually format both list elements and groups of elements on a screen.

Types of visual objects and their unique properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Label
"""""""""

Displays a text string.

"type" : "Text"
 
"value": a string constant or a reference to a variable with @ prefix. You can use HTML markup (as with all other labels).
 
"text_color" – text color in HEX format (e.g. #F54927)

"radius" - background rounding (number)

"stroke" - line outline (number)

"background" – background color in HEX format (e.g. #F54927)
 
"size" – size (integer)

Picture
"""""""""""

.. image:: _static/picture.png
       :scale: 55%
       :align: center

Displays an image.

"type" : "Picture"

"value" – absolute file path. A constant or a reference to a variable in _data with @ prefix.

Example:

.. code-block:: Python

 text_samples = [
 [{"type":"Text","value":"@my_text"}], #text from my_text key in _data
 [{"type":"Text","value":"Hello <u>world</u>"}] #text constant with html
 ]

HTML field
"""""""""""""

Displays an HTML document.

"type" : "HTML"

"value" – a string in HTML format

Button
"""""""""

Displays a button. When the screen button is pressed, an event with listener=<element id> is generated. For use in lists, see the section Active list elements.

"type" : "Button"

"caption" – button label

"background" – button background color in HEX format

Example:
``{"type":"Button","id":"btn_update","caption":"Simple button"}``

Bottom button bar
"""""""""""""""""""""""""

.. image:: _static/bottom_buttons.png
       :scale: 55%
       :align: center

Displays a horizontal array of buttons at the bottom of the screen. Only for placement on the screen.

"type" : "BottomButtons"

"value" – a list of buttons with ids. Each button’s parameters are the same as for Button.

Example:
``{"type":"BottomButtons","id":"bottom","value":[{"type":"Button","id":"back","background":"#F54927","caption":"Back"},{"type":"Button","id":"next","background":"#25a018","caption":"Forward","w":2}]}``

Switch
"""""""""""""

Toggle element. Generates an event and writes the current switch value into a variable (element id).

"type": "Switch"

"caption" – (string) element caption

"value" – (boolean) initial value of the element—either a constant or a reference to a variable. Usually a reference to the id is used here.

Example:

``{"type":"Switch","caption":"Setting 1","id":"sw1","value":"@sw1"}``

Checkbox
"""""""""""

Toggle

Toggle element. Generates an event and writes the current switch value into a variable (element id).

"type" : "CheckBox"

"caption" – (string) element caption

"value" – (boolean) initial value of the element—either a constant or a reference to a variable. Usually a reference to the id is used here.

Example:

``{"type":"CheckBox","caption":"Setting 2","id":"cb1","value":"@cb1"}``

Lists
"""""""""""""

.. image:: _static/tables.png
       :scale: 55%
       :align: center

Displays various lists of elements. As content you can use either a list created in a handler or a dataset or a list of nodes of a class. Active elements may be placed inside list elements. For nicer formatting list elements can be wrapped in a Card. By default, the item layout is auto-generated, but it can be overridden both for the entire list and for any specific element.

"type": "Table"

"value" – the data source of the list. It can be defined simply as a “list of dictionaries” in python, which makes it well compatible with NoSQL such as Pelican, for example.

Example of such a definition:

.. code-block:: Python

 table_data = [{"name": "element 1", "key":"el1"},{"name": "element 2", "key":"el2"}]
 {"type":"Table","id":"l1","value":table_data}

Or (if datasets are used) a reference to a dataset can be specified:

``{"type":"Table","id":"l2","value":"goods" }``

Or, if the nodes_source property is used, you can display a list of nodes (in this case, tapping an item will open the node). This can be a search (filter) of nodes or just some arbitrary list. Nodes are listed as UIDs. You can convert a list of objects into a list of UIDs using the to_uid function.

Example (displaying all nodes of the Note class):

``{"type":"Table","id":"t1","nodes_source":True,"value":to_uid(Note.get_all())}``

"layout" – list layout as a whole in the standard “by rows” format. 

Example:

.. code-block:: Python


 table_layout = [
        [{"type":"Text","value":"@_id","bold":True}],
        [{"type":"Text","value":"@name"},{"type":"Text","value":"@barcode"}]
        ]
 {"type":"Table","id":"l1","value":table_data, "layout": table_layout }
 
*_layout* in a list item does the same as layout but only for a specific list element. This property is simply added to the data source:

``table_data = [{"name": "element 1", "key":"el1","_layout":layout2},{"name": "element 2", "key":"el2"}]``

*_background* in a list element colors the background with the specified HEX color.

``table_data = [{"name": "element 1", "key":"el1","_background":"#701705" },{"name": "element 2", "key":"el2"}]``
 
"columns_count" – number of table columns. Displays the list in several columns.

"horizontal" – horizontal list orientation.

"search_enabled" – enables search mode.

For a dataset there is its own search settings section dataset_search, which contains method (search method), keys (fields to search by) and min_length (optional) —minimum length from which search starts.

Method can be:

 * text – for normal substring search
 * levenshtein – for fuzzy search by Levenshtein distance (results are shown in descending order of accuracy, with a threshold >75; the accuracy itself is added to records in the _confidence field)

Pagination is always enabled for the dataset (invisibly), but the page size can be configured via the page_size property.

Example:

.. code-block:: Python

 dataset_search = {"dataset":"goods","keys":"name","method":"text","min_length":2}
 self.Show([ 
                    [ {"type":"Table","id":"l2","value":"goods" ,"search_enabled":True,"dataset_search":dataset_search}]
        ])

List of child nodes
"""""""""""""""""""""""""

.. image:: _static/children.png
       :scale: 55%
       :align: center

Displays a hierarchical list of child nodes and their descendants.

"type" : "NodeChildren"

No parameters.

Input fields
"""""""""""""

.. image:: _static/inputs.png
       :scale: 55%
       :align: center

The entire palette of input fields (except dataset fields, which are configured similarly) have type Input—regular fields, multiline, dropdown lists, etc. All entered characters are immediately written into the node’s _data. If autosave is enabled for the node class, then the node is immediately saved as well.

"type" : "Input"

"input_type" – input value type (by default just text, input_type can be omitted):
 
 * NUMBER – numeric values. Numbers written into a variable are automatically recognized as float/integer depending on the presence of a fractional part.
 * PASSWORD – password input (obscured by asterisks)
 * MULTILINE – multiline text
 * DATE – date. When a date is chosen, two values are written into _data—human-readable date (into the variable=id) and into _d<element id>—the date in ISO format.

"events" – a flag enabling event generation when typing in the input field.

"value" – default displayed value.

"caption" – field caption.

"spinner_mode" – dropdown list mode. In this case, you must set the source property value—a list of values as a string with ";" as delimiter.

"autocomplete" – mode of suggestion by first characters. source must also be set.

Dataset input fields
"""""""""""""""""""""""""

Used to select a specific dataset value on a form—a reference element, document, etc. The convenience is that a reference to an object is selected, and you can get either the full object or its representation by it. *In the screenshot in the Input fields section, Production is a dataset field, which displays the product name and code—this is how the template is defined.*

The dataset object representation is defined in the dataset’s Record Template field. HTML can also be used there.

"type" : "DatasetField"

"dataset" – dataset name

"spinner" – mode of selection from a list (mutually exclusive with autocomplete mode)

"autocomplete" – autocomplete mode.

"caption" – field caption

"value" – default value. Set as a reference to a dataset element <dataset name>$<dataset item id>.


Bare labels (without object)
"""""""""""""""""""""""""""""

In markup you can simply specify a string instead of an object with type=Text, and then the string is rendered, or if the @ prefix is given, the value from _data is used, e.g. ``["Hello world"]``.


Alternatively you can use the string construction <title>|<value>, and then the string value will also be rendered but as a kind of Card with a title: ``["title|Hello world"]``


Such constructions can be used for simplification instead of Text or Card.

Active list elements (in Table)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the markup of list elements, you can use not only labels and images but also some active elements, such as buttons. 
When interacting with them, the node receives slightly different (extended) data compared to placement on a screen. The developer gets the list name, active element name, position, and value (for value input elements).

For example, a Button. 
Variables for the press event (onInput):

listener – is given in the format ``<table id>_input<element id>``

``<table id>_input_position``  – the position of the list element where the press occurred is returned here.

If _data has key, then the key is also returned:

``<table id>_input_key``  – the value of the element's key.

For input fields – CheckBox/Switch/Input – the entered value itself is additionally placed into _data in the variable ``<table id>_input<element id>``.

Available active elements:

 * Button
 * Input fields (Input)
 * Checkbox
 * Switch 

UI/UX functions/methods of the mobile client
----------------------------------------------

Node methods (on the device)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Screen drawing method Show
""""""""""""""""""""""""""""""""

**Show(layout)** – draws the node screen. This method “cleans the canvas” and renders the layout whose principles are described in the “Screen layout” section. At a minimum, it must be specified in the onShow handler so that something appears. It can also be called in other handlers to update the screen with new data, redraw it. Although there is a more economical UpdateView command for that, sometimes redrawing the entire screen is simply easier.

.. code-block:: Python

 self.Show([
  [{"type":"Input","id":"title","caption":"Subject","value":"@title"}],
  [{"type":"Parameters","w":1},{"type":"Input","height":-1,"id":"body","caption":"Text","value":"@body","input_type":"multiline"}]
 ])


Screen mechanisms connection method PlugIn
""""""""""""""""""""""""""""""""""""""""""""

**PlugIn(elements)** connects several elements to the screen. These include hardware mechanisms, such as a hardware scanner, and visual elements outside layout (otherwise they would have been output via Show). The command parameter is a one-dimensional array of objects (a Python list) with objects of the form {"type":"type_of_element","id":"element_id"}. 

.. attention:: The command clears all elements before adding them (self.PlugIn([]) will clear everything), so you must list all elements at once.

The following object types (type key) exist:

**CameraBarcodeScannerButton** – on-screen barcode scan element. This is a button on the screen that opens the camera for scanning. When a code is scanned, an event with listener=<element id> is generated and the barcode is placed into _data[<element id>].

Example:

``self.PlugIn([{"type":"CameraBarcodeScannerButton", "id":"barcode_cam"}])``

**BarcodeScanner** – connects hardware barcode scan intercept (for data collection terminals). The hardware scanner must be configured in Settings, in the Hardware Scanner Setup section. Subscription to hardware scanner events must be enabled, and the data sent via Intent by your scanner software must be specified. The scanner must also be configured to send barcodes via broadcast Intent instead of to the keyboard. When a code is scanned, an event with listener=<element id> is generated, and the barcode is placed into _data[<element id>].

**FloatingButton** – adds a floating button (there can be several) in the lower right corner of the screen. This element, besides id (by which, as for all elements, events are generated), has a caption field for button text and also fields to configure an svg icon (see the “svg icons” section for details). 

Example:

``self.PlugIn([{"type":"FloatingButton","id":"add_child","caption":"Add <b>line</b>"}])``

**ToolbarButton** – adds a button in the toolbar. This element, besides id (by which, as for all elements, events are generated), has a caption field for button text and also fields to configure an svg icon (see the “svg icons” section for details). 

Example:

``self.PlugIn([{"type":"ToolbarButton","id":"pin","caption":"Save","svg":svg2,"svg_size":24,"svg_color":"#FFFFFF"} ]])``

**PhotoButton** – opens the camera for taking a picture. The image is saved to a file, and the file path is stored in _data[<element id>]. If MediaGallery is connected, the picture is automatically added to the gallery. Automatic addition can be disabled by placing the OFFAutoupdateMediaGallery flag in the node’s _data. In that case, the photo will be intercepted in a handler, and you can do some processing before adding it to the gallery.

Example of manual processing 

.. code-block:: Python
 
 def CaptureImage(self, input_data=None):
        gallery_array = self._data["pic_files"]
        base64 = getBase64FromImageFile(this._data["result_file"],50,50) #does not lead anywhere, we just got a cropped base64
        gallery_array.append(this._data["result_file"])
        toast(self._data["result_file"])
        UpdateMediaGallery(gallery_array )

        return True,{}


**GalleryButton** – opens the gallery to attach a media file. The selected file is saved, and the file path is stored in _data[<element id>]. Similar to PhotoButton.

**MediaGallery** – a gallery of an array of media files at the bottom of the screen with open/delete capabilities. By the key equal to element id, an array of file paths is stored and displayed in the gallery. By default, PhotoButton and GalleryButton work with it automatically, but this array can also be edited in a handler (for example, to add a photo there). User deletion is also available. After deletion, an event <variable_name>_delete is generated, where you can, for example, read the updated gallery file array.

.. image:: _static/inputs.png
       :scale: 90%
       :align: center


Node opening method in the interface _open()
"""""""""""""""""""""""""""""""""""""""""""

**_open(method=None)** – opens the node form as if it were opened by the user. By default, onShow and the method specified in the configuration will run, but this can be overridden in the method parameter.

Screen element update method UpdateView
"""""""""""""""""""""""""""""""""""""""""""""

**UpdateView(id,element=None)** – command for selectively updating elements. It is recommended for high-load interfaces (such as ActiveCV).

It works in several modes:

 1. Simply redraws the element by id. This makes sense if the element’s value is specified via a variable (using @) and not a constant. Example: ``self.UpdateView("btn_repl",None)``.
 2. Changes element properties. Example: ``self.UpdateView("btn_repl",{"background":"#C82909"})``.
 3. Replaces an element with another element. For example, you can do this with containers. Example: ``self.UpdateView("btn_repl",{"type":"Input","id":"inp1","caption":"New input---"})``.

General UI/UX methods for the mobile client
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**SetTitle(title)** – sets the node window title.

**RefreshTab()** – refreshes the current tab (section) from which the node was opened. Makes sense when a node is added/changed and you are returning to the list.

**CloseNode()** – closes the node form.

**RunGPS/ StopGPS** – commands to start/stop the GPS. May require user permission. Once started, GPS reads device data at intervals, and the first data will not appear instantly; accuracy may also vary, so to read data via GetLocation it is better to use a timer.

**GetLocation** – obtains GPS data as a JSON object serialized to a string with fields:

 *altitude*  – altitude
 *latitude* – latitude
 *longitude* – longitude
 *accuracy* – accuracy in meters
 *provider* – data provider

**ScanBarcode(listener)** – starts camera-based scanning as if the user pressed a scanner button connected via PlugIn. The result is returned as an event specified in the parameter.

**Dialog(id,title,yes_caption="",no_caption="",layout=None)** – method for displaying both simple dialogs and dialogs with layout. At a minimum—only a title; you can also override the two buttons and create any layout according to common layout rules.

.. image:: _static/dialog.png
       :scale: 90%
       :align: center

Returns an event with ``listener=<id>_positive/<id>_negative`` depending on which button the user pressed. If the dialog has layout and there are input fields in it, the data is written directly into _data.

Examples:

.. code-block:: Python
 
 def simple_dialog(self, input_data=None):
        Dialog("dlg1","To be or not to be?","To be","Not to be")
        
        return True,{}
        

 def layout_dialog(self, input_data=None):
        Dialog("dlg2","Enter quantity","Ok",None,[[{"type":"Input","id":"qty_dialog","caption":"Quantity","input_type":"number"}]])
        
        return True,{}
        

 def input(self, input_data=None):
        if self._data.get("listener") == "dlg1_positive":
          toast("To be")
        elif self._data.get("listener") == "dlg1_negative":
          toast("Not to be")  
        elif self._data.get("listener") == "dlg2_positive":
          toast(str(self._data.get("qty_dialog") ))

**AddTimer(key, period)/ StopTimer(key)** – starts/stops a timer with the given key. 

Example:

``AddTimer("my_timer",5)``

**ShowProgressButton/ HideProgressButton** – draws a progress wheel on the button and disables it until HideProgressButton is executed.

Example:

.. code-block:: Python
 
 def worker():
    time.sleep(2)
    HideProgressButton("button1")

 ShowProgressButton("button1")
 t = threading.Thread(target=worker)
 t.start()

**ShowProgressGlobal/ HideProgressGlobal** – shows a global (full-screen) progress bar / hides it. 

**SetCover(node,layout)** – sets a new layout for a node. This is exactly a layout, not data. If layout values are set via @, they will be updated anyway.

**UpdateMediaGallery()** – refreshes the gallery. Must be called after any operations with the gallery composition.

android module functions
~~~~~~~~~~~~~~~~~~~~~~~~~

Interface commands:

 * toast(String toast) – show Android toast
 * message(String text) – show message
 * speak(String text) – speak text (TTS engine)
 * listen() – start speech recognition listening
 * vibrate() and vibrate(int duration) – vibration and vibration for a specified duration
 * beep()/beep(int tone)/ beep(int tone,int beep_duration,int beep_volume) – sound signal, including the ability to choose tone (1 to 99), duration, and volume (default 100)
 * notification(String message)/ notification (String message,String title)/ notification(String message,String title,int number) – notification in the notification shade. number is the notification ID to later access it for removal or updating.
 * notification_progress(String message,String title,int number,int progress) – notification with a progress bar (0 to 100) notification_cancel(int number) – hide notification.


Some common element properties
-------------------------------

HTML text in labels
~~~~~~~~~~~~~~~~~~~~~~~~~

Everywhere in the interface where text is present you can use HTML tags for text formatting. For example: Привет <b>мир</b>.

svg icons
~~~~~~~~~~~~~~~

.. image:: _static/svg.png
       :scale: 100%
       :align: center

For elements that support icons, you can use svg icons. The approach is:

 1. Download and save the svg file. 
 2. Open the file in a text editor and copy the text into a variable.
 3. You can use this variable in the svg property of the element (you can also set size and color with svg_size and svg_color).

.. code-block:: Python
 
 svg1 = '<svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960" width="24px" fill="#1f1f1f"><path d="m720-120 160-160-56-56-64 64v-167h-80v167l-64-64-56 56 160 160ZM560 0v-80h320V0H560ZM240-160q-33  0-56.5-23.5T160-240v-560q0-33 23.5-56.5T240-880h280l240 240v121h-80v-81H480v-200H240v560h240v80H240Zm0-80v-560 560Z"/></svg>'
 self.PlugIn([
          {"type":"FloatingButton","id":"add","svg":svg1,"svg_size":48,"svg_color":"#000000"}
    ])

Functions for image and gallery conversion
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**getBase64FromImageFile(path_to_image)** – returns a base64 string from a file.

**convertImageFilesToBase64Array(paths_to_images_array)** – convenient for converting a “Gallery” into a single string: converts an array of paths into an array of base64 strings.

**saveBase64ToFile(base64_string)** – saves a base64 string to a temporary file and returns its path.

**convertBase64ArrayToFilePaths** – saves an array of base64 strings to an array of paths (temporary files) suitable for use in the Gallery.


Global events of the mobile client. Application event handling.
-------------------------------------------------------------------

Some events occur not in nodes but in the application as a whole, for example the event triggered when the platform starts, when scanning a barcode outside a node, and so on. They are handled in the same way as node events—that is, an event and a handler are assigned, but not in the node structure; instead they are configured in the configuration structure on the Global Events tab (in the configuration, this is the CommonEvents section).

Python handlers for such events are defined not in node classes, but simply as functions in the handler code. Some events also contain data, which is passed in input_data in the corresponding key.

Example:

.. code-block:: Python
 
 def onBarcode(input_data):    
    toast(input_data.get("barcode"))    
    return True,{}

They must have the input_data parameter as a dictionary. 

List of global event types:

 * **onLaunch** – event when the configuration is loaded (or on restart). No parameters.
 * **onTimer** – timer event. input_data contains the “timer_key” key—the key of the triggered timer.
 * **onJSONFile** – event when a JSON file is opened by the app (via Open or Share). input_data contains the "content" key—the file contents as a string.
 * **onTextFile** – event when a text file is opened by the app (via Open or Share). input_data contains the "content" key—the file contents as a string.
 * **onBarcode** – barcode event, scanned via ScanBarcode, i.e. outside a node, from the main menu.
 * **onStartMenuCommand** – a command click event from the configuration section (those commands added via Configuration Sections).
 * **onDialogResult** – dialog event triggered from the main menu. Since the dialog is called with a specific identifier, result will be either result_positive or result_negative, and result_data contains data from the dialog’s input elements, if any.

Searching, Sorting, and Visibility of Nodes (Client)
------------------------------------------------------

The NodaLogic mobile client has built-in mechanisms for searching, sorting, and hiding nodes that operate on the client side and require no additional code or API.

These mechanisms are controlled through special service fields in the node's _data.

Searching by Node
~~~~~~~~~~~~~~~~~~~

Default Behavior

If a node doesn't specify a special search field, the application searches:

by all keys of the _data dictionary

search is performed by the string representation of the values

_search_index Field

To explicitly control the search and improve performance, it is recommended to use the field:

``_search_index``

If specified:

* search is performed only by this field

* search becomes faster and more controllable

Example

.. code-block:: Python

self._data["_search_index"] = (
self._data.get("number", "") + " " +
self._data.get("customer_name", "")
)

``_search_index`` must be a regular string

Sorting Nodes
~~~~~~~~~~~~~~~~~~~

Sorting node lists is also performed on the client side.

Special fields are used for this:

``_sort_string`` — sorting in ascending order

``_sort_string_desc`` — sorting in descending order

Sorting example

Sorting by date (ascending):

``self._data["_sort_string"] = self._data.get("plan_date")``

Sorting by date (descending):

``self._data["_sort_string_desc"] = self._data.get("created_at")``

Hiding Nodes
~~~~~~~~~~~~~~~~~~~

A node can be hidden from the interface without physically deleting it.

The field used for this is:

_hidden

If set to True:

The node is not displayed in lists.

The node remains accessible through code and the API.

Example of hiding a node

``self._data["_hidden"] = True``
