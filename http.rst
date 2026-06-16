.. NodaLogic documentation master file, created by
   sphinx-quickstart on Wed Nov 5 07:29:33 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.
Online HTTPRequest event handler (mobile client)
=============================================================


HTTPRequest is an event handler that sends a POST request to an external backend.
The backend receives JSON with the event data, executes its logic and returns JSON.
The response can return new data and/or commands for the UI.

In the event, select HTTPRequest and specify the request method in the function name(method) field. The platform will take the URL base from the application settings (Online Handlers section), append /<function_name>, and send it as a POST request.


In this case, the request sends JSON of this type

.. code-block:: JSON
 
 
{
   "_data": {
    "...": "current node data"
  },
  "input_data": {
    "...": "method arguments (in some cases)"
  }
}


In case of general events - only _data

.. code-block:: JSON
 
 
{
  "_data": {
    "...": "event data"
  }
}


On the server side, you need to prepare a response. You can influence the node's _data by adding or deleting something in _data; it will be overwritten. You can also include commands that the client should execute when the response is returned.


Return example - added status and commands:

.. code-block:: JSON
 
 
{
  "_data": {
    "_id": "config_uid$Goods$123",
    "name": "Item 001",
    "barcode": "4601234567890",
    "qty": 1,
    "status": "checked"
  },
  "_commands": [
    {
      "command": "SetTitle",
      "argument": ["Product verified"]
    },
    {
      "command": "Refresh"
    },
    {
      "command": "message",
      "argument": ["Barcode accepted"]
    }
  ]
}

Important:
- if you return _data in a node event, the client will replace node._data;
- if class autosaved=true, the node will be saved;
- if _data is not returned, the node data does not change;
- _commands are executed after receiving a response;
- It is better to always pass the argument as a JSON array.


Command format

.. code-block:: JSON
 
 
{
  "command": "CommandName",
  "argument": ["argument1", "argument2"]
}

If there are no arguments:

{
  "command": "Refresh"
}

Recommendations:

* Always return a JSON object.
* For business errors, it is better to return a message rather than HTTP 500.
* Return _data only if you really need to change the data.
* argument must be passed as an array even for one argument.
* Command names are case sensitive.


6. HTTPRequest UI Commands
-------------------------

Show
~~~~~~
Purpose: show the layout of the current node.
Argument format: ``[ layout_array ]``
Example:
.. code-block:: JSON
 
 
{
  "command": "Show",
  "argument": [
    {"type":"Text", "text":"Done"}
  ]
}

PlugIn
~~~~~~~~~

Purpose: Substitute layout/plugin into the current node.
Argument format: [ layout_array ]
Example:
.. code-block:: JSON
 
 
{
  "command": "PlugIn",
  "argument": [
    {"type":"Text", "text":"Additional block"}
  ]
}

UpdateView
~~~~~~~~~~~~

Purpose: to update an interface element.
The argument format for the current node is: [view_id, value_json]
Argument format with node_id: [node_id, view_id, value_json]
Example:

.. code-block:: JSON
 
{
  "command": "UpdateView",
  "argument": ["status_text", {"text":"Done"}]
}

SetTitle
~~~~~~~~~~~

Purpose: to change the screen title.
Argument format: [title]
Example:
{
  "command": "SetTitle",
  "argument": ["Acceptance"]
}

Refresh
~~~~~~~~~~~~~
Purpose: To update the current shape of the node.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "Refresh"
}

RefreshTab
~~~~~~~~~~~~
Purpose: refresh tab.
argument format: [] or [tab_id]
Example:

.. code-block:: JSON
 
 
{
  "command": "RefreshTab",
  "argument": ["goods"]
}

CloseNode
~~~~~~~~~~
Purpose: Close the current node form.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "CloseNode"
}

ScanBarcode
~~~~~~~~~~~~
Purpose: to start barcode scanning.
Argument format: [listener, event]
Example:
.. code-block:: JSON
 
 
{
  "command": "ScanBarcode",
  "argument": ["onBarcode", "scan"]
}

Dialogue
~~~~~~~~~~~
Purpose: to show a dialog.
Argument format: [title, text, ok_button, cancel_button]
Extended format: [title, text, ok_button, cancel_button, layout_json]
Example:
.. code-block:: JSON
 
 
{
  "command": "Dialog",
  "argument": ["Done", "Operation completed", "OK", ""]
}

AddTimer
~~~~~~~~~~~
Purpose: add a client timer.
Argument format: [timer_key, seconds]
Example:
.. code-block:: JSON
 
 
{
  "command": "AddTimer",
  "argument": ["refresh_status", 5]
}

StopTimer
~~~~~~~~~~
Purpose: stop the client timer.
Argument format: [timer_key]
Example:
.. code-block:: JSON
 
 
{
  "command": "StopTimer",
  "argument": ["refresh_status"]
}

StopAllTimers
~~~~~~~~~~~~~~~
Purpose: Stop all client timers.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "StopAllTimers"
}

ShowProgressButton
~~~~~~~~~~~~~~~~~~~
Purpose: to show progress on a button.
Argument format: [button_id]
Example:
.. code-block:: JSON
 
 
{
  "command": "ShowProgressButton",
  "argument": ["save_button"]
}

ShowProgressGlobal
~~~~~~~~~~~~~~~~~~~~~~
Purpose: to show a global progress indicator.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "ShowProgressGlobal"
}

HideProgressGlobal
~~~~~~~~~~~~~~~~~~~
Purpose: Hide the global progress indicator.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "HideProgressGlobal"
}

RunPython
~~~~~~~~~~
Purpose: Run a method of the current node. The old, incorrect name RunPyhon is also supported.
Argument format: [method_name]
Example:
.. code-block:: JSON
 
 
{
  "command": "RunPython",
  "argument": ["onAccept"]
}

RunEvent
~~~~~~~~~~~

Purpose: to trigger an event.
For the argument node: [event, listener]
For common argument: [listener, event, parameter2]
Example for node:
.. code-block:: JSON
 
 
 
{
  "command": "RunEvent",
  "argument": ["onInput", "after_http"]
}

Save
~~~~~~~~~

Purpose: save the current node.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "Save"
}

Upload
~~~~~~~~
Purpose: upload the current node to the server.
Argument format: []
Example:
.. code-block:: JSON
 
 
{
  "command": "Upload"
}


General UI commands
~~~~~~~~~~~~~~~~~~~~~

**toast**
Purpose: short system toast message.
Argument format: [text]
Example:
.. code-block:: JSON
 
  {
   "command": "toast",
   "argument": ["Saved"]
  }

**message**
Purpose: snackbar message at the bottom of the screen.
Argument format: [text]
Example:
.. code-block:: JSON
 
{
   "command": "message",
   "argument": ["Operation completed"]
}

**vibrate**
Purpose: vibration.
Argument format: [] or [milliseconds]
Example:
.. code-block:: JSON
 
  {
   "command": "vibrate",
   "argument": [300]
  }

**SendNotification**
Purpose: Show Android notification.
Argument formats:
- [message]
- [message, title]
- [message, title, number, progress]
Example:
.. code-block:: JSON
 
{
   "command": "SendNotification",
   "argument": ["Task completed", "NodaLogic"]
}

**SendProgressNotification**
Purpose: Show/refresh progress notification.
Argument format: [message, title, number, progress]
Example:
.. code-block:: JSON
 
  {
   "command": "SendProgressNotification",
   "argument": ["Loading", "NodaLogic", 1001, 45]
}

**beep**
Purpose: sound signal.
Argument formats:
- []
- [tone]
- [tone, duration_ms, volume]
Example:
.. code-block:: JSON
 
  {
   "command": "beep"
  }

**speak**
Purpose: to pronounce text via Text-To-Speech.
Argument format: [text]
Example:
.. code-block:: JSON
 
{
   "command": "speak",
   "argument": ["Done"]
}

**listen**
Purpose: Run speech recognition if RECORD_AUDIO permission is present.
Argument format: []
Example:
.. code-block:: JSON
 
  {
   "command": "listen"
  }

**share_text**

Purpose: Open the system share dialog for text.
Argument format: [text]
Example:
.. code-block:: JSON
 
  {
   "command": "share_text",
   "argument": ["Text to send"]
}

_SendNotificationProgress
Purpose: Internal version of progress notification. SendProgressNotification is generally preferred.
Argument format: [message, title, number, progress]
Example:
.. code-block:: JSON
 
  {
   "command": "_SendNotificationProgress",
   "argument": ["Loading", "NodaLogic", 1001, 70]
}
