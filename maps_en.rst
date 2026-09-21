Maps on Android: a unified API and map providers
================================================

NodaLogic provides a unified set of mapping tools for the Android client.
Application handlers use functions such as ``MapCenter``, ``MapPoint``,
``MapPolyline`` and ``MapRoute`` without calling the mapping SDK directly.

The dynamic-layout element selects the map engine. For example, ``YandexMap``
creates a map powered by Yandex MapKit. Commands are routed to a specific map
instance by its ``id``.

A unified API **does not imply identical capabilities** across providers.
Displaying a supplied polyline is different from geocoding an address or
calculating a route. Such operations may depend on the provider, its license,
and network availability. See :ref:`map-capabilities` for a capability matrix.

.. note::

   The built-in mapping layer described here is available in the Android
   client. A web map can be implemented separately with HTML and JavaScript.
   ``YandexMap`` is not a web-layout element.

Architecture
------------

Mapping involves three layers:

#. **The NodaLogic solution** stores coordinates, places, areas and routes
   received from external systems as ordinary business data.
#. **The unified map API** accepts ``Map*`` commands, dispatches them to the
   map with the specified ``id``, and reports events back to solution handlers.
#. **The provider** renders the map and performs supported operations such as
   moving the camera, drawing objects, searching or calculating routes.

When a server has already calculated a route, the solution can display its
coordinates using ``MapPolyline`` without invoking the provider's routing
service. Search, reverse geocoding and route calculation are distinct
capabilities that must not be assumed to exist on every provider.

.. _map-capabilities:

Provider capability matrix
--------------------------

The table covers the API described in this article. Additional provider
columns can be added as other integrations are documented.

.. list-table:: Supported capabilities
   :header-rows: 1
   :widths: 44 56

   * - Unified API capability
     - YandexMap
   * - Map display and camera control
     - Yes; powered by Yandex MapKit.
   * - GPS and current device location
     - Yes, subject to Android permission and location availability.
   * - Points, SVG markers and labels
     - Yes.
   * - Polylines, polygons and circles
     - Yes.
   * - Server-calculated route displayed using ``MapPolyline``
     - Yes; no provider routing request is required.
   * - Automatic car-route calculation using ``MapRoute``
     - Yes; asynchronous, subject to MapKit operating conditions.
   * - Search and reverse geocoding
     - Yes; asynchronous, through MapKit.
   * - Tap, search-result and routing events
     - Yes.
   * - Offline display of a previously downloaded region
     - Yes, with an appropriate paid MapKit license.
   * - Offline region management
     - Through the Android client's map manager and MapKit facilities.
   * - Offline search and car routing
     - Supported by MapKit for downloaded regions with a paid license; verify
       each scenario with the actual API key and routing mode.

.. note::

   The matrix distinguishes features of the **NodaLogic unified API** from
   features of the **provider SDK**. For instance, MapKit's support for
   offline search does not guarantee that every query issued by an application
   will be resolved locally. Do not assume that ``MapReverseGeocode`` works
   offline without testing that specific scenario.

Shared Python API
-----------------

Import the API from ``nodesclient`` in an Android handler:

.. code-block:: python

    from nodesclient import (
        MapCall, MapState, MapCamera, MapCenter,
        MapPoint, MapPolyline, MapPolygon, MapCircle,
        MapRemove, MapClear, MapRoute,
        MapSearch, MapReverseGeocode,
        MapUserLocation, MapCenterGPS, MapFit,
    )

The first argument to most functions is the map instance's layout ``id``.
A handler can therefore address the intended map even when a screen contains
multiple map elements.

The low-level entry point allows less common actions and parameters:

.. code-block:: python

    MapCall("map", "action", {...})

A generic entry point does not make every action available on every
provider. Prefer the dedicated ``Map*`` functions in ordinary application
code.

Camera and GPS
--------------

``MapCenter`` accepts coordinates from any source: node fields, an external
system, search results or GPS.

.. code-block:: python

    MapCenter("map", [55.75, 37.62], zoom=14)

    MapCenter(
        "map",
        {"latitude": 55.75, "longitude": 37.62},
    )

The current zoom is preserved when ``zoom`` is omitted.

For additional camera options:

.. code-block:: python

    MapCamera(
        "map", 55.75, 37.62,
        zoom=15, azimuth=0, tilt=0, animated=True,
    )

Show the Android device's location and center the map on it:

.. code-block:: python

    MapUserLocation("map", visible=True)
    MapCenterGPS("map", zoom=16)

If location permission has not been granted, the map component initiates the
standard Android permission request. Application handlers do not need to
request it again. For coordinates already supplied by another system, use
``MapCenter`` rather than ``MapCenterGPS``.

Map objects
-----------

Add a point with its own object identifier:

.. code-block:: python

    MapPoint(
        "map", "client_1", 55.75, 37.62,
        text="Customer", color="#3f7353",
    )

The first argument identifies the map, while the second identifies the map
object. Calling the function again with the same object ID replaces the point.

You can use a custom SVG marker:

.. code-block:: python

    FLAG_SVG = '<svg ...>...</svg>'

    MapPoint(
        "map", "warehouse_1", 55.76, 37.64,
        text="Warehouse 1", svg=FLAG_SVG,
        svg_size=42, svg_color="#1E88E5", number="1",
        text_placement="bottom", text_offset=5,
    )

Other parameters include ``size``, ``draggable``, ``z``,
``number_background``, ``number_color``, ``text_size`` and
``text_offset_from_icon``. The resulting appearance depends on the selected
provider's marker implementation.

Remove an object:

.. code-block:: python

    MapRemove("map", "warehouse_1")

A supplied route or track
-------------------------

When a server or external system has already returned route geometry, pass
its coordinates to ``MapPolyline``. The map displays the line immediately;
it does not need to calculate the route again.

.. code-block:: python

    external_route = [
        [55.7510, 37.6100],
        [55.7542, 37.6160],
        [55.7580, 37.6210],
    ]

    MapPolyline(
        "map", "external_route", external_route,
        color="#1565C0", width=7,
    )

Two points form a simple segment:

.. code-block:: python

    MapPolyline(
        "map", "segment_1",
        [[55.75, 37.61], [55.76, 37.64]],
        color="#db3030", width=5,
    )

Additional parameters ``dash_length`` and ``gap_length`` control dashed
lines. Supply coordinates in route order: a polyline does not discover roads
between arbitrary waypoints.

Polygons and circles
--------------------

.. code-block:: python

    MapPolygon(
        "map", "zone_1",
        [[55.75, 37.62], [55.75, 37.64], [55.77, 37.64]],
        stroke="#3f7353", fill="#403f7353", width=3,
    )

    MapCircle(
        "map", "radius_1", 55.75, 37.62, 250,
        stroke="#1565C0", fill="#401565C0", width=3,
    )

Map events
----------

With ``events=true``, the most recent event is written to:

.. code-block:: python

    self._data["map_event"]
    self._data["<map_id>_event"]

For a map whose ID is ``delivery_map``, the second field is named
``delivery_map_event``. If a screen contains several maps and their events
must be kept separate, read the ID-specific field.

The map triggers the standard ``onInput`` event with ``listener`` equal to
its ``id``. For a map with ``id="map"``, configure this handler:

.. code-block:: json

    {
      "event": "onInput",
      "listener": "map",
      "actions": [
        {
          "action": "run",
          "source": "internal",
          "server": "internal",
          "method": "MapEvent",
          "order": 1
        }
      ]
    }

Example Python handler:

.. code-block:: python

    def MapEvent(self, input_data=None):
        event = self._data.get("map_event") or {}
        kind = event.get("event")
        data = event.get("data") or {}

        if kind == "tap":
            lat = data.get("latitude")
            lon = data.get("longitude")

        elif kind == "object_tap":
            object_id = data.get("object_id")
            lat = data.get("latitude")
            lon = data.get("longitude")

        elif kind == "search_ready":
            results = data.get("results") or []

        elif kind == "route_ready":
            route_count = data.get("count")

        elif kind in ("route_error", "search_error", "location_error"):
            error = data.get("error")

        return True, self._data

Possible events include ``tap``, ``long_tap``, ``object_tap``,
``route_ready``, ``route_error``, ``search_ready``, ``search_error``,
``location_permission_required``, ``location_permission_granted``,
``location_permission_denied`` and ``location_error``. Some events only
occur when the selected provider implements the corresponding capability.

Setting up a map in onShow
--------------------------

It is convenient to define the map declaratively and add its objects in
``onShow``:

.. code-block:: python

    def InitMap(self, input_data=None):
        MapCenter("map", [55.80, 37.59], zoom=10, animated=False)
        MapUserLocation("map", visible=True)
        MapPoint("map", "office", 55.75, 37.62, "Office")
        return True, self._data

``onShow`` may run before Android has finished constructing the View tree.
Short commands targeting a not-yet-mounted map are queued until its element
has been registered. The process layout must still contain the map element:
a later ``Show()`` with an empty layout may replace an already rendered
screen.

Storing your own places
-----------------------

The map is responsible for display, not for persistent business-data
storage. Keep places as ordinary dictionaries in ``_data``:

.. code-block:: python

    self._data["my_locations"] = [
        {
            "id": "client_1",
            "name": "Customer",
            "address": "Moscow, ...",
            "latitude": 55.75,
            "longitude": 37.62,
        }
    ]

The same array can be displayed in a standard ``Table`` and rendered on the
map using ``MapPoint``. Address is optional: a label and coordinates can be
saved without a geocoding request. When the user selects a table row, pass
that item's coordinates to ``MapCenter``.

Opening a location in an external navigation app
------------------------------------------------

.. code-block:: python

    from android import share_location, open_location

    share_location(55.75, 37.62, "Customer")
    open_location(55.75, 37.62, "Customer")

These functions use Android's standard ``geo:`` intent and do not depend on
any particular installed navigation app.

NodaScript
----------

The following unified mapping commands are available on Android:

.. code-block:: text

    map_call(map_id, action, params)
    map_state(map_id)
    map_camera(map_id, latitude, longitude, zoom)
    map_center(map_id, latitude, longitude, optional_zoom)
    map_point(map_id, object_id, latitude, longitude, text)
    map_route(map_id, params)
    map_search(map_id, query)
    map_center_gps(map_id, zoom)
    map_clear(map_id)
    share_location(latitude, longitude, label)
    open_location(latitude, longitude, label)

Use ``map_call`` for extended parameters:

.. code-block:: text

    map_call(
        'map',
        'add_polyline',
        {
            'id':'route_1',
            'points':[[55.75,37.61],[55.76,37.64]],
            'color':'#1565C0',
            'width':7
        }
    );

Clearing objects, fitting the view and inspecting state
-------------------------------------------------------

.. code-block:: python

    MapClear("map")                # Remove ordinary map objects
    MapClear("map", routes=True)   # Also remove calculated routes

    MapFit(
        "map", [[55.75, 37.62], [55.77, 37.64]],
        max_zoom=14,
    )

    state = MapState("map")

The returned state contains the camera position and zoom, provider, object
count and diagnostic fields. Some fields are SDK-specific, such as the API
key source or routing-session status.

.. _map-providers:

Providers
---------

.. _map-provider-yandex:

YandexMap (Yandex MapKit)
~~~~~~~~~~~~~~~~~~~~~~~~~

``YandexMap`` is a dynamic-layout element powered by Yandex MapKit.
Rendering and interaction use the shared ``Map*`` API; the provider-specific
search and routing features are described below.

The YandexMap element
^^^^^^^^^^^^^^^^^^^^^

Minimal layout:

.. code-block:: json

    {
      "type": "YandexMap",
      "id": "map",
      "height": 420,
      "center": [55.751225, 37.62954],
      "zoom": 13,
      "events": true
    }

``id`` is required. Use a different ``id`` for each map on the same screen.

Element parameters:

* ``center`` — initial coordinates ``[latitude, longitude]``;
* ``zoom`` — initial zoom level;
* ``azimuth`` and ``tilt`` — camera bearing and tilt;
* ``show_user_location`` — show the device's location;
* ``center_gps`` — center on the device's position when opened;
* ``heading`` — device heading, if supported by the SDK;
* ``night`` — night mode;
* ``style`` — custom MapKit style;
* ``route_color`` and ``route_width`` — color and width of calculated routes;
* ``events`` — emit NodaLogic events; defaults to ``true``;
* ``objects`` — initial ``point``, ``polyline``, ``polygon`` and ``circle``
  objects;
* ``route_debug`` — diagnostic routing Toast messages; keep ``false`` in
  normal use.

If ``height`` is omitted, the Android component uses its default height.
Normal NodaLogic dynamic-layout parameters also apply.

MapKit API key
^^^^^^^^^^^^^^

The user can supply a key in the Android client's settings:

``Settings -> Maps -> Yandex MapKit API key``.

If the application build provides a default platform key, leaving this field
empty uses that key. A user-provided key takes precedence. MapKit reads the
key on its first initialization in the application process, so fully restart
the app after changing the key.

Do not store the key in solution configuration or Python handlers. Commercial
capabilities, including offline regions, require a key with the appropriate
permissions and licensing terms.

Automatic route calculation
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current integration supports car routes through MapKit:

.. code-block:: python

    MapRoute(
        "map",
        [
            [55.9292, 37.5207],
            [55.76594, 37.68549],
            [55.70306, 37.53028],
        ],
        routes=1, clear=True, fit=True,
        max_zoom=12, router_type="online",
    )

To start at the device's current location:

.. code-block:: python

    MapRoute(
        "map",
        [[55.9292, 37.5207], [55.70306, 37.53028]],
        from_gps=True, fit=True, router_type="online",
    )

``from_gps=True`` uses the shared GPS mechanism and requests Android
permission when necessary.

The request is asynchronous: its immediate response typically contains
``pending=true``, followed by ``route_ready`` or ``route_error``. The
``route_ready`` event provides the number of routes and geometry points,
but not the complete coordinate array. Exposing the MapKit-computed geometry
as an array to application handlers would require a separate extension to
the platform API.

.. important::

   ``MapRoute`` asks MapKit to **calculate** a route. ``MapPolyline`` simply
   **draws** supplied geometry and is appropriate for routes calculated on a
   server or obtained from another system.

Search and reverse geocoding
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

    MapSearch("map", "Gorky Park", limit=10)
    MapReverseGeocode("map", 55.75, 37.62)

Unless an explicit search area is supplied through ``MapCall``, the visible
map area is used. Results arrive in ``search_ready`` and may contain
``name``, ``description``, ``address`` and ``points``.

For reverse geocoding, the event's ``query`` field is
``reverse_geocode``. Errors arrive in ``search_error``. Search results
can be stored as ordinary solution data; the map does not persist them
automatically.

Yandex offline maps
^^^^^^^^^^^^^^^^^^^

MapKit can download regions in advance and use their data without an
internet connection. The Android client provides a ``Yandex`` tab in its
``Offline maps`` screen for managing these downloads (when enabled in the
application build). MapKit supplies the region catalog and map data: this
is **not** an import of arbitrary map files from device storage.

The tab lets users browse regions, start downloads, follow progress, pause
and resume downloads, update stored data and delete downloaded regions.
MapKit provides its own network and offline-cache storage controls.
Downloaded regions are used automatically by the SDK; a single "active
region" does not need to be selected for ``YandexMap``.

.. important::

   MapKit offline regions are available **only with a paid Yandex license**.
   The presence of the tab in the Android client does not itself grant the
   right or ability to download regions. Download the regions while a network
   connection is available, and verify the key and license terms before
   operating a device offline.

For downloaded regions, MapKit documents map display, offline search and car
routing without traffic information. However, do not assume that
``router_type="online"`` automatically switches to local calculation:
verify the mode and availability of the particular request. Likewise, do
not guarantee that ``MapReverseGeocode`` is resolved offline without testing
that scenario.

Official documentation: `Offline maps in MapKit for Android
<https://yandex.com/maps-api/docs/mapkit/android/generated/tutorials/map_offline.html>`_.

Practical recommendations
-------------------------

* Use the shared ``Map*`` API for application actions, but check the
  capability matrix before relying on provider-specific features.
* Keep business places and supplied routes in solution data. The map is
  responsible for display and user interaction.
* Use ``MapPolyline`` for a supplied route; use ``MapRoute`` to request an
  SDK-calculated route when the selected provider supports it.
* ``MapRoute``, ``MapSearch`` and ``MapReverseGeocode`` are asynchronous:
  handle their final results through map events.
* Do not put the MapKit API key in solution configuration.
* Use ``MapCenter`` when coordinates are known; ``MapCenterGPS`` refers to
  the device's own location.
* Disable ``route_debug`` in normal interfaces and store large SVG definitions
  in handler constants.
