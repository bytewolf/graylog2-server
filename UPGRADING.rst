**************************
Upgrading to Graylog 2.2.x
**************************

.. _upgrade-from-21-to-22:

This file only contains the upgrade note for the upcoming release.
Please see `our documentation <http://docs.graylog.org/en/latest/pages/upgrade.html>`_
for the complete upgrade notes.

Email Alarm Callback
====================

In previous Graylog versions, creating an alert condition on a stream and adding an alert receiver meant that, if no alarm callback existed for that stream, Graylog would use an Email alarm callback by default.

Due to the extensive rework done in alerting, this behaviour has been modified to be explicit, and more consistent with other entities within Graylog: from now on **there will not be a default alarm callback**.

To easy the transition to people relying on this behaviour, we have added a migration step that will create an Email alarm callback for each stream that has alert conditions, has alert receivers, but has no associated alarm callbacks.

Alert Notifications (previously known as Alarm Callbacks)
=========================================================

Graylog 2.2.0 introduces some changes in alerting. Alerts have now states, helping you to know in an easier way if something requires your attention.

These changes also affect the way we send notifications: Starting in Graylog 2.2.0, alert notifications are only executed **once**, just when a new alert is triggered. As long as the alert is unresolved or in grace period, **Graylog will not send further notifications**. This will help you reducing the noise and annoyance of getting notified way too often when a problem persists for a while.

If you are using Graylog for alerting, please take a moment to ensure this change will not break any of your processes when an alert occurs.

Default stream/Index Sets
=========================

With the introduction of index sets, and the ability to change a stream's write target, the default stream needs additional information, which is calculated when starting a new Graylog 2.2 master node.

It requires recalculation of the index ranges of the default stream's index set, which when updating from pre-2.2 versions is stored in the `graylog_` index. This is potentially expensive, because it has to calculate three aggregations across every open index to detect which streams are stored in which index.

Please be advised that this necessary migration can put additional load on your cluster.

.. warning:: Make sure that all rotation and retention strategy plugins you had installed in 2.1 are updated to a version that is compatible with 2.2 before you start the Graylog 2.2 version for the first time. (e.g. Graylog Enterprise) This is needed so the required data migrations will run without problems.

.. warning:: The option to remove a message from the default stream is currently not available when using the pipeline function `route_to_stream`. This will be fixed in a subsequent bug fix release. Please see `the corresponding Github issue <https://github.com/Graylog2/graylog-plugin-pipeline-processor/issues/117>`_.

RotationStrategy & RetentionStrategy Interfaces
===============================================

The Java interfaces for ``RetentionStrategy`` and ``RotationStrategy`` changed in 2.2. The ``#rotate()`` and ``#retain()`` methods are now getting an ``IndexSet`` as first parameter.

This only affects you if you are using custom rotation or retention strategies.

Changes in Exposed Configuration
================================

The exposed configuration settings on the ``/system/configuration`` resource of the Graylog REST API doesn't contain the following (deprecated) Elasticsearch-related settings anymore:

* ``elasticsearch_shards``
* ``elasticsearch_replicas``
* ``index_optimization_max_num_segments``
* ``disable_index_optimization``

Changes in Split & Count Converter
==================================

The behavior of the split & count converter has been changed to that it resembles typical ``split()`` functions.

Previously, the split & count converter returned 0, if the split pattern didn't occur in the string. Now it will return 1.

Examples:

+-------------+---------------+------------+------------+
| String      | Split Pattern | Old Result | New Result |
+=============+===============+===========+=============+
| <empty>     | ``-``         | 0         | 0           |
+-------------+---------------+-----------+-------------+
| ``foo``     | ``-``         | 0         | 1           |
+-------------+---------------+-----------+-------------+
| ``foo-bar`` | ``-``         | 2         | 2           |
+-------------+---------------+-----------+-------------+

Graylog REST API
================

Streams API
-----------

Due to the introduction of index sets, the payload for creating, updating and cloning of streams now requires the ``index_set_id`` field. The value for this needs to be the ID of an existing index set.

Affected endpoints:

* ``POST /streams``
* ``PUT  /streams/{streamId}``
* ``POST /streams/{streamId}/clone``
