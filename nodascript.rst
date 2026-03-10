.. NodaLogic documentation master file, created by
sphinx-quickstart on Wed Nov 5 07:29:33 2025.
You can adapt this file completely to your needs, but it should at least
contain the root `toctree` directive.

NodaScript
==============

NodaScript can be used directly in markup and can also be used in the event system by specifying it as an event handler (as an alternative to Python handlers). It is a high-performance native interpreter for its own language that works directly and is always available. It can be used in forms without regard for performance. It runs directly in the front-end in Android and in the JS code of HTML pages in the web client.

The goal is to reduce the codebase, making solutions more lightweight and readable. You can offload all interface processing to it, leaving business logic, integrations, and any complex algorithms in Python.

Using Element Values
-------------------------------------

In any visual element value, similar to ``@key``, you can specify ``#<script return value>``. This can be a script of any complexity, including loops and conditions. The main thing is that there is at least one return value.

Example: ``#return ?(_date.balance<0,'--',_date.balance)``

Example of an element in a 1C-like implementation:

.. code-block:: JavaScript

 {"type":"Text","value":"#if _data.qty!=null and _data.price!=null {return _data.qty*_data.price} else {return '--'}", "gravity":"right"}

Using events directly in markup
-----------------------------------------------

For active elements, you can assign a handler directly to the element's property in the markup:

``{"type":"Button","id":"btn_update","caption":"Simple button","onClick":"message('btn_update')"}``

List of element events:

* **Button** : ``onClickParent``(for the list owner), ``onClick``
* **Switch** : ``onClickParent``(for the list owner), ``onClick``
* **CheckBox** : ``onClickParent``(for the list owner), ``onClick``
* **Spinner** : ``onClickParent``(for the list owner), ``onClick``
* **Input** : ``onClickParent``(for the list owner), ``onClick``
* **NodeInput** : ``onSelect``
* **DatasetField** : ``onSelect``
* **Table(list click event)** : ``onClick``

Using as an event handler in Events
----------------------------------------------------------

Just like Python handlers, you can create NodaScript handlers as event subscriptions (in the Class Events section). This makes the configuration even more compact. To do this, select NodaScript as the method in the event and enter the code in the code window.

Language Description
-------------------

Data
~~~~~~~~~~~

All scripts work with the root JSON object _data and modify it in place:

.. code-block:: JavaScript

_data.Sum = 0;
line = _data.lines[_data.tab1_input_position];
line.qty = line.qty + 1;

General
~~~~~~~

* Case-insensitive (If/IF/if).
* Statement separator: `;` (the final `;` is optional before `}` or the end of the script).
* Comments: // ...
* Strings: 'single' or "double"

Value types (strict)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Numbers: int and float
* String: string
* Boolean: bool (True/False)
* Date: date (epochMillis, long)
* Array: array
* Object: object
* null

Boolean expressions
~~~~~~~~~~~~~~~~~~~~~~~~~

Comparison operators:

`` == != > >= < <=``

Support: numbers, strings, date (by epochMillis). For array/object, comparison by reference (identity).

Logical operators (bool only):

AND / AND / && — logical AND

OR / OR / || — logical OR

NOT / NOT / ! — logical NOT

Instance check:

  x in arr
 
  "key" in obj      // checks object key existence

Examples:

.. code-block:: JavaScript

// comparison
_data.qty > 0 AND _data.price >= 10;

// parentheses and NOT

NOT (_data.isClosed OR _data.isDeleted);

// in (inclusion)
5 IN [1,2,3,5];

"name" IN _data;``

Control flow
~~~~~~~~~~~~~~
  
.. code-block:: JavaScript

  //IF:
  if cond { ... } else { ... };

  //WHILE:
  while cond { ... };

  //FOR (C-style):
  for i = 0; i < 10; i = i + 1 { ... };

  //FOR-IN:
  for item in _data.lines { ... };

  //Ternary:
  x = cond ? a : b;

  //also:
  x = ?(cond, a, b);


Abort / Continue
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: JavaScript

 Abort;
 Continue;

Exceptions
~~~~~~~~~~~~~~~~~~~

.. code-block:: JavaScript

 try { ... } except err { ... }
 throw expr;

Return (for #-expressions)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: JavaScript

 Return expression;
 Return; // null

Built-ins (core)
~~~~~~~~~~~~~~~~~~~~

Now()
ParseDate(text[, pattern])
FormatDate(date, pattern)

AddDays(date, days)
AddMonths(date, months)
AddYears(date, years)

StartOfDay(date)    EndOfDay(date)
StartOfMonth(date)  EndOfMonth(date)
StartOfYear(date)   EndOfYear(date)

String(x)

Length(x)   // strings/arrays
Substring(str, start, len)
IndexOf(str, sub)

HasProperty(obj, key)

NewArray()
NewObject()
NewStructure(key1, val1, key2, val2, ...)

Literals
~~~~~~~~~~~~
.. code-block:: JavaScript
  
 //Object literal:
  o = {a: 1, "b": 2};

 //Array literal:
  a = [1, 2, 3];

Array methods
~~~~~~~~~~~~~~~~~

.. code-block:: JavaScript
  
  arr.add(x)
  arr.clear()
  arr.contains(x)

Dates: notes
~~~~~~~~~~~~~~~~~~
  
 * date is stored as epochMillis (long).
 * ParseDate accepts:
  - digits-only string => epochMillis
  - otherwise parses using SimpleDateFormat
*  Default ParseDate patterns (when pattern omitted) include:
  yyyy-MM-dd'T'HH:mm:ss.SSSZ
  yyyy-MM-dd'T'HH:mm:ss.SSS
  yyyy-MM-dd'T'HH:mm:ss
  yyyy-MM-dd
* ParseDate / FormatDate and period boundaries use engine timezone (rt.tz / Engine.setTimeZone).

HTTP bridge (optional, synchronous)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

 In script:
  
.. code-block:: JavaScript
  

  r = HTTPRequest("GET", "https://example.com/api/items", {"q":"abc"}, null);
  // with headers:
  r = HTTPRequest("GET", "https://example.com/api/items", {"q":"abc"}, null, {"X-Token":"123"});
  // with basic auth:
  r = HTTPRequest("GET", "https://example.com/api/items", null, null, null, "user:pass");
  // or auth as object:
  r = HTTPRequest("GET", "https://example.com/api/items", null, null, null, {"user":"u","pass":"p"});

  //Result:
  
 {
    "ok": true/false,
    "status": <http code or 0>,
    "headers": { ... },
    "text": "<response body>",
    "json": <JSONObject/JSONArray or null>
  }

 HTTPRequest is blocking — call scripts on a background thread (not UI thread).
