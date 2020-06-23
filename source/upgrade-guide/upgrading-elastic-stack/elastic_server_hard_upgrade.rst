.. Copyright (C) 2020 Wazuh, Inc.

.. _elastic_server_hard_upgrade:

Upgrading Elastic Stack from 6.x to 6.8
=======================================

This section guides through the upgrade process of Elastic Stack components including Elasticsearch, Logstash, Filebeat and Kibana for the Elastic distribution.

Prepare the Elastic Stack
-------------------------

#. Stop the services:

    .. code-block:: console

      # systemctl stop logstash
      # systemctl stop filebeat
      # systemctl stop kibana

#. In case of having disabled the repository for Elastic Stack 6.x it can be enabled using:

    .. tabs::

      .. group-tab:: YUM

        .. code-block:: console

          # sed -i "s/^enabled=0/enabled=1/" /etc/yum.repos.d/elastic.repo

      .. group-tab:: APT

        .. code-block:: console

          # sed -i "s/#deb/deb/" /etc/apt/sources.list.d/elastic-6.x.list
          # apt-get update

      .. group-tab:: ZYpp

        .. code-block:: console

          # sed -i "s/^enabled=0/enabled=1/" /etc/zypp/repos.d/elastic.repo


Upgrade Elasticsearch
---------------------

#. Disable shard allocation:

    .. code-block:: bash

      curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
      {
        "persistent": {
          "cluster.routing.allocation.enable": "primaries"
        }
      }
      '

#. Stop non-essential indexing and perform a synced flush (optional):

    .. code-block:: bash

      curl -X POST "localhost:9200/_flush/synced"

#. Shut down a single node:

    .. code-block:: console

      # systemctl stop elasticsearch

#. Upgrade the node you shut down:

    .. tabs::

      .. group-tab:: YUM

        .. code-block:: console

          # yum install elasticsearch-|ELASTIC_6_LATEST|

      .. group-tab:: APT

        .. code-block:: console

          # apt-get install elasticsearch=|ELASTIC_6_LATEST|
          # systemctl restart elasticsearch


#. Restart the service:

    .. code-block:: console

      # systemctl daemon-reload
      # systemctl restart elasticsearch

#. Start the newly-upgraded node and confirm that it joins the cluster by checking the log file or by submitting a ``_cat/nodes`` request:

    .. code-block:: bash

      curl -X GET "localhost:9200/_cat/nodes"

#. Reenable shard allocation:

    .. code-block:: bash

      curl -X PUT "localhost:9200/_cluster/settings" -H 'Content-Type: application/json' -d'
      {
        "persistent": {
          "cluster.routing.allocation.enable": null
        }
      }
      '

#. Before upgrading the next node, wait for the cluster to finish shard allocation:

    .. code-block:: bash

      curl -X GET "localhost:9200/_cat/health?v"

#. Repeat the steps for every Elasticsearch node.

#. Load the Wazuh template for Elasticsearch:

    .. code-block:: console

      # curl https://raw.githubusercontent.com/wazuh/wazuh/v|WAZUH_LATEST|/extensions/elasticsearch/6.x/wazuh-template.json | curl -X PUT "http://localhost:9200/_template/wazuh" -H 'Content-Type: application/json' -d @-


Upgrade Logstash
----------------

#. Upgrade the ``logstash`` package:

    .. tabs::

      .. group-tab:: YUM

        .. code-block:: console

          # yum install logstash-|ELASTIC_6_LATEST|

      .. group-tab:: APT

        .. code-block:: console

          # apt-get install logstash=1:|ELASTIC_6_LATEST|-1

#. Download and set the Wazuh configuration for Logstash:

    .. tabs::

      .. group-tab:: Local configuration

        .. code-block:: console

          # cp /etc/logstash/conf.d/01-wazuh.conf /backup_directory/01-wazuh.conf.bak
          # curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/v|WAZUH_LATEST|/extensions/logstash/6.x/01-wazuh-local.conf
          # usermod -a -G ossec logstash

      .. group-tab:: Remote configuration

        .. code-block:: console

          # cp /etc/logstash/conf.d/01-wazuh.conf /backup_directory/01-wazuh.conf.bak
          # curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/v|WAZUH_LATEST|/extensions/logstash/6.x/01-wazuh-remote.conf


#. Start the Logstash service:

    .. code-block:: console

      # systemctl daemon-reload
      # systemctl start logstash.service


Upgrade Filebeat
----------------

#. Upgrade Filebeat:

    .. tabs::

      .. group-tab:: YUM

        .. code-block:: console

          # yum install filebeat-|ELASTIC_6_LATEST|

      .. group-tab:: APT

        .. code-block:: console

          # apt-get install filebeat=|ELASTIC_6_LATEST|

#. Update the configuration file:

    .. code-block:: console

      # cp /etc/filebeat/filebeat.yml /backup/filebeat.yml.backup
      # curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/v|WAZUH_LATEST|/extensions/filebeat/6.x/filebeat.yml
      # chmod go+r /etc/filebeat/filebeat.yml

#. Restart Filebeat:

    .. code-block:: console

      # systemctl daemon-reload
      # systemctl restart filebeat


Upgrade Kibana
--------------

#. Upgrade the Kibana package:

    .. tabs::

      .. group-tab:: YUM

        .. code-block:: console

          # yum install kibana-|ELASTIC_6_LATEST|

      .. group-tab:: APT

        .. code-block:: console

          # apt-get install kibana=|ELASTIC_6_LATEST|

#. Uninstall the Wazuh Kibana plugin:

    a)  Update file permissions. This will prevent errors when generating new bundles or updating the Wazuh Kibana plugin:

      .. code-block:: console

        # chown -R kibana:kibana /usr/share/kibana/optimize
        # chown -R kibana:kibana /usr/share/kibana/plugins

    b) Remove the Wazuh Kibana plugin:

      .. code-block:: console

        # cd /usr/share/kibana/
        # sudo -u kibana bin/kibana-plugin remove wazuh

#. Upgrade the Wazuh Kibana plugin:

    .. tabs::

      .. group-tab:: Install from URL

        .. code-block:: console

          # cd /usr/share/kibana/
          # rm -rf optimize/bundles
          # sudo -u kibana NODE_OPTIONS="--max-old-space-size=3072" bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-|WAZUH_LATEST|_|ELASTIC_6_LATEST|.zip

      .. group-tab:: Install from the package

        .. code-block:: console

          # cd /usr/share/kibana/
          # rm -rf optimize/bundles
          # sudo -u kibana NODE_OPTIONS="--max-old-space-size=3072" bin/kibana-plugin install file:///path/wazuhapp-|WAZUH_LATEST|_7.6.0.zip

    .. note::

      The Wazuh Kibana plugin installation process may take several minutes.

#. Start the Kibana service:

    .. code-block:: console

      # systemctl daemon-reload
      # systemctl enable kibana.service
      # systemctl start kibana.service

Disabling repositories
----------------------

    .. tabs::

      .. group-tab:: YUM

          .. code-block:: console

            # sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/elastic.repo

      .. group-tab:: APT

        .. code-block:: console

          # sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-6.x.list
          # apt-get update

        Alternatively, you can set the package state to ``hold``, which will stop updates (although you can still upgrade it manually using ``apt-get install``):

        .. code-block:: console

          # echo "elasticsearch hold" | sudo dpkg --set-selections
          # echo "kibana hold" | sudo dpkg --set-selections

      .. group-tab:: ZYpp

        .. code-block:: console

          # sed -i "s/^enabled=1/enabled=0/" /etc/zypp/repos.d/elastic.repo