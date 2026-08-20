Quant Ledger
============

``quant_ledger`` is an optional server-side SQL module for NodaLogic that stores additive movements and current balances. It is useful for stock, money, reservations, location load, and similar cases where values change by ``+/-`` deltas and the resulting balance must be updated reliably and atomically.

The module has no separate Designer UI and is intended only for server-side Python handlers. When the ``quant_ledger`` directory is present in the project, the required SQL tables are created at application startup. The current implementation supports SQLite.

Import
------

Normally, import only the functions used by the handler:

.. code-block:: Python

 from quant_ledger.api import (
     quant, move, transaction,
     balance, balances, movements, statement,
     NegativeBalanceError,
 )

Core concepts
-------------

**space** is an application-defined ledger namespace. For example, detailed stock may live in ``"stock.detail"`` while location load is kept separately in ``"location.load"``.

**quant** is the complete analytical key. The combination of ``space + quant`` identifies one balance row.

For warehouse stock, a full quant may contain warehouse, product, lot, and location:

.. code-block:: Python

 full_quant = quant(warehouse_id, product_id, lot_id, location_id)

**selector_quant** is a second, coarser key used to select a group of balances efficiently. For example, all lots and locations of one product in one warehouse:

.. code-block:: Python

 selector = quant(warehouse_id, product_id)

``selector_quant`` is not part of the unique balance key. It is stored in a dedicated indexed column for group lookups. Once a full ``quant`` exists, its ``selector_quant`` must not be changed.

**resources** is an ordered list of additive numeric values. Up to 16 resources are supported. In a stock example, resource ``0`` is normally quantity and resource ``1`` may be amount:

.. code-block:: Python

 [quantity, amount]

Public values are returned as ``Decimal``. Internally, SQL stores them as fixed-point integers with 6 decimal digits of precision.

Posting one movement
--------------------

``move()`` writes a movement and updates the corresponding balance atomically. Either both changes are committed or neither is committed.

Receipt example:

.. code-block:: Python

 result = move(
     "stock.detail",
     quant(warehouse_id, product_id, lot_id, location_id),
     document_date,
     f"{document_id}:receipt",
     {
         "document": document_id,
         "employee": employee_id,
         "comment": "Goods receipt",
     },
     [quantity, amount],
     selector_quant=quant(warehouse_id, product_id),
     allow_negative=False,
 )

Arguments:

 * **space** — ledger namespace;
 * **quant** — complete analytical key;
 * **period** — movement date/time;
 * **operation_id** — stable identifier of one logical movement;
 * **details** — arbitrary movement metadata for history and filtering;
 * **resources** — resource deltas;
 * **selector_quant** — fast group-selection key;
 * **allow_negative** — whether resource ``0`` may become negative.

An expense is the same operation with negative deltas:

.. code-block:: Python

 move(
     "stock.detail",
     full_quant,
     document_date,
     f"{document_id}:expense",
     {"document": document_id},
     [-quantity, -amount],
     selector_quant=selector,
     allow_negative=False,
 )

By default, resource ``0`` may not become negative. If the balance is insufficient, ``NegativeBalanceError`` is raised:

.. code-block:: Python

 try:
     move(...)
 except NegativeBalanceError as exc:
     Message("Insufficient balance: " + str(exc.attempted))

To protect several resources from becoming negative, specify their indexes explicitly:

.. code-block:: Python

 move(..., nonnegative_resources=[0, 1])

Reposting
---------

``operation_id`` must stay stable for one logical movement. Calling ``move()`` again with the same ``operation_id`` does not duplicate or ignore the movement. The old movement is reversed and replaced with the new one in one SQL transaction.

.. code-block:: Python

 result = move(
     "stock.detail",
     full_quant,
     document_date,
     f"{document_id}:receipt",
     {"document": document_id},
     [new_quantity, new_amount],
     selector_quant=selector,
 )

 if result.reposted:
     Message("Document reposted")

During reposting, the period, full ``quant``, ``selector_quant``, ``details``, and resources may change. If the replacement fails validation, for example because it would create a negative balance, the whole repost is rolled back and the previous movement remains unchanged.

Grouping several movements
--------------------------

When one business operation consists of several movements, wrap them in ``transaction()``. A typical example is a transfer: the source expense and target receipt must succeed together.

.. code-block:: Python

 source_quant = quant(warehouse_id, product_id, lot_id, source_location_id)
 target_quant = quant(warehouse_id, product_id, lot_id, target_location_id)
 selector = quant(warehouse_id, product_id)

 with transaction() as tx:
     tx.move(
         "stock.detail",
         source_quant,
         document_date,
         f"{document_id}:source",
         {"document": document_id},
         [-quantity],
         selector_quant=selector,
         allow_negative=False,
     )

     tx.move(
         "stock.detail",
         target_quant,
         document_date,
         f"{document_id}:target",
         {"document": document_id},
         [quantity],
         selector_quant=selector,
         allow_negative=False,
     )

If either movement fails, the entire transaction is rolled back. Different logical movements in the same document must use different ``operation_id`` values, such as ``:source`` and ``:target``.

Reading current balances
------------------------

**balance(space, quant)** returns one exact balance:

.. code-block:: Python

 row = balance("stock.detail", full_quant)
 qty = row.resources[0]
 amount = row.resources[1]

If no balance exists yet, a row with zero-valued resources is returned.

**balances(...)** selects several balances. The preferred efficient pattern is selection by ``selector_quant``:

.. code-block:: Python

 rows = balances(
     "stock.detail",
     selector_quant=quant(warehouse_id, product_id),
     positive_resource=0,
 )

 for row in rows:
     warehouse, product, lot, location = row.parts
     qty = row.resources[0]

Useful ``balances()`` arguments:

 * **selector_quant** — indexed group selection;
 * **quants** — a set of exact full quants;
 * **nonzero_resource** — keep rows where the selected resource is non-zero;
 * **positive_resource** — keep rows where the selected resource is positive;
 * **limit** — limit the number of returned rows.

Movement history
----------------

**movements()** returns movement history and can filter by period, ``quant``, ``selector_quant``, ``operation_id``, and fields stored in ``details``.

.. code-block:: Python

 rows = movements(
     "stock.detail",
     period_from=date_from,
     period_to=date_to,
     details={"document": document_id},
 )

 for row in rows:
     print(row.period, row.resources, row.details)

``details`` is intended for audit data and exact movement filtering. It is a convenient place to store document UID, employee UID, operation type, or comment.

Period statement
----------------

Use ``statement()`` for reports such as Opening / Income / Expense / Closing. There is no need to load every movement and calculate these totals manually.

.. code-block:: Python

 rows = statement(
     "stock.detail",
     period_from=date_from,
     period_to=date_to,
     selector_quant=quant(warehouse_id, product_id),
 )

 for row in rows:
     opening = row.opening[0]
     income = row.income[0]
     expense = row.expense[0]
     closing = row.closing[0]

Each ``StatementRow`` represents one full ``quant``. If the report needs a higher aggregation level, for example warehouse + product, aggregate the returned rows in Python after the SQL selection.

FEFO and other business sorting
-------------------------------

A ``quant`` stores keys, not fields of linked nodes. Therefore SQL cannot sort lots by ``Lot.expire_date`` when the quant contains only the lot UID.

The correct FEFO pattern is:

.. code-block:: Python

 rows = balances(
     "stock.detail",
     selector_quant=quant(warehouse_id, product_id),
     positive_resource=0,
 )

 lot_ids = {str(row.parts[2]) for row in rows}
 lots = Lot.get_many_data(lot_ids)

 rows.sort(
     key=lambda row: str(
         (lots.get(str(row.parts[2])) or {}).get("expire_date") or "9999-12-31"
     )
 )

SQL first narrows the candidate balances using ``selector_quant``. Linked nodes are then loaded in bulk and business-specific sorting is performed in Python.

Working with quant values
-------------------------

``quant()`` creates a canonical and reversible key. Do not split the resulting string manually by ``|``.

.. code-block:: Python

 q = quant(warehouse_id, product_id, lot_id, location_id)
 parts = parse_quant(q)
 lot_id = quant_part(q, 2)

Import ``parse_quant`` and ``quant_part`` when direct quant parsing is needed.

If the application uses ``"~"`` as a special empty dimension, it is still an exact value, not a wildcard:

.. code-block:: Python

 q = quant(warehouse_id, product_id, "~", location_id)

Verification and rebuild
------------------------

**verify_space(space)** compares stored balances with the sum of movements:

.. code-block:: Python

 from quant_ledger.api import verify_space

 result = verify_space("stock.detail")
 if not result.valid:
     print(result.errors)

**rebuild_balances(space)** rebuilds all balances in one space from movement history and then verifies the result:

.. code-block:: Python

 from quant_ledger.api import rebuild_balances

 result = rebuild_balances("stock.detail")

These functions are intended for diagnostics and recovery, not normal document posting.

Configuration scope
-------------------

Inside a NodaLogic server handler, the current configuration ``scope`` is resolved automatically, so it normally does not need to be passed explicitly.

Outside a server handler, pass ``scope`` explicitly:

.. code-block:: Python

 row = balance("stock.detail", full_quant, scope=config_uid)

Rows from different configurations remain isolated even when they use the same ``space`` and ``quant`` values.

Practical rules
---------------

 * Never update SQL balance rows directly; post deltas through ``move()``.
 * Do not implement balance updates as “read current value + calculate new value + save”.
 * Use stable ``operation_id`` values so documents can be reposted safely.
 * Use ``transaction()`` for all-or-nothing business operations.
 * Design ``selector_quant`` in advance for efficient group reads.
 * Do not change ``selector_quant`` for an already existing full ``quant``.
 * Use ``balance()``/``balances()`` for current state, ``movements()`` for history, and ``statement()`` for period reports.
 * Do not call ``quant_ledger`` from Android handlers; the module is server-only.
