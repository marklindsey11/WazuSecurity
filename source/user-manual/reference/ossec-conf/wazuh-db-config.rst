.. Copyright (C) 2022 Wazuh, Inc.

.. meta::
	:description: Check out this section about the local configuration of Wazuh and learn about the configuration options of the Wazuh-DB daemon.


.. _wazuh-db-config:

wazuh-db
========

.. topic:: XML section name

	.. code-block:: xml

		<wdb>
		</wdb>

Configuration options for the :ref:`wazuh-db <wazuh-db>` daemon.

Options
-------

- `backup`_

backup
^^^^^^

Configuration block to define the wazuh-db databases backup behavior.


+--------------------+-----------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Allowed tags**   | database  | Defines the database to backup.                                                                                                                                    |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Allowed values**             | global                                                                                                                            |
+--------------------+-----------+--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
| **Allowed values** | enabled   | Enables the regular backup of this particular database.                                                                                                            |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Default value**              | yes                                                                                                                               |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Allowed values**             | yes, no                                                                                                                           |
|                    +-----------+--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    | interval  | How often the backup is created.                                                                                                                                   |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Default value**              | 1d                                                                                                                                |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Allowed values**             | A positive number that should contain a suffix character indicating a time unit: s (seconds), m (minutes), h (hours) or d (days). |
|                    +-----------+--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    | max_files | The maximum number of simultaneous backups to store before deleting the oldest ones. The pre-restore snapshot, if present, is not considered in this limit.        |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Default value**              | 3                                                                                                                                 |
|                    |           +--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+
|                    |           | **Allowed values**             | A positive number.                                                                                                                |
+--------------------+-----------+--------------------------------+-----------------------------------------------------------------------------------------------------------------------------------+

Example of configuration
------------------------

The following configuration will generate a backup of the **global.db** every day, with a maximum of 3 simultaneous files

.. code-block:: xml

    <wdb>
        <backup database='global'>
            <enabled>yes</enabled>
            <interval>1d</interval>
            <max_files>3</max_files>
        </backup>
    </wdb>
