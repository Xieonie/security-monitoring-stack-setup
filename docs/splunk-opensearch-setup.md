# Splunk / OpenSearch Setup Guide for Log Aggregation

This document provides a high-level overview of setting up Splunk or OpenSearch as a central log aggregation and analysis platform, complementing Wazuh or acting as a standalone SIEM.

**Choice:**
* **Splunk:** Powerful, feature-rich commercial product. Free tier available with limitations (e.g., 500MB/day indexing).
* **OpenSearch (with OpenSearch Dashboards):** Open-source fork of Elasticsearch and Kibana. Powerful, scalable, and no licensing costs. Often used as the backend for Wazuh Indexer/Dashboard.

## Splunk Setup (Conceptual)

Refer to the [official Splunk documentation](https://docs.splunk.com/Documentation/Splunk) for detailed installation and configuration.

### 1. Splunk Enterprise Installation

* Download Splunk Enterprise from the Splunk website.
* Install it on a dedicated server (or a powerful VM).
* During setup, an admin user and password will be created.

### 2. Splunk Universal Forwarder (UF) Installation (on data sources)

* Install UFs on all hosts from which you want to collect logs.
* Configure the UF (`outputs.conf`) to point to your Splunk Indexer(s).
* Configure inputs on the UF (`inputs.conf`) to specify which files, ports, or scripts to monitor. (See `splunk-config/inputs.conf.example`)

### 3. Splunk Configuration (on Splunk Server)

* **Receiving Data:** Configure Splunk to listen for data from UFs (typically on port 9997). This is usually enabled by default.
* **Indexes:** Create indexes to store different types of data (e.g., `os_logs`, `app_logs`, `firewall_logs`).
* **Source Types:** Define source types to help Splunk parse and categorize data correctly. Often auto-detected, but can be customized (`props.conf`). (See `splunk-config/props.conf.example`)
* **Apps & Add-ons:** Install Splunk apps and add-ons for specific technologies (e.g., Splunk Add-on for Unix and Linux, Splunk App for Wazuh if integrating).

### 4. Accessing Splunk

* Web interface is typically `http://<your_splunk_server_ip>:8000`.

## OpenSearch Setup (Conceptual)

Refer to the [official OpenSearch documentation](https://opensearch.org/docs/latest/) for detailed installation. OpenSearch can be deployed via Docker, tarball, or as part of managed services.

### 1. OpenSearch Cluster Installation

* **Single Node (for testing/small setups):**
    * Can be run via Docker.
    * `docker run -p 9200:9200 -p 9600:9600 -e "discovery.type=single-node" --name opensearch-node -d opensearchproject/opensearch:latest`
* **Multi-Node Cluster (for production):** Requires careful planning for discovery, master nodes, data nodes, etc.

### 2. OpenSearch Dashboards Installation

* This is the visualization layer (equivalent to Kibana).
* Can also be run via Docker, configured to connect to your OpenSearch cluster.
    * `docker run -p 5601:5601 -e "OPENSEARCH_HOSTS=http://<your_opensearch_node_ip>:9200" --name opensearch-dashboards -d opensearchproject/opensearch-dashboards:latest`

### 3. Data Ingestion (Log Shippers)

You'll need agents to send logs to OpenSearch:
* **Filebeat:** Lightweight shipper for log files. Part of the Elastic Stack, but can be configured to output to OpenSearch.
* **Fluentd / Fluent Bit:** Flexible data collectors.
* **Logstash:** More powerful data processing pipeline before sending to OpenSearch.

**Configuration Example (Filebeat to OpenSearch):**
Edit `filebeat.yml` on the agent:
```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    - /var/log/syslog
    # Add other paths

output.opensearch:
  hosts: ["http://<your_opensearch_node_ip>:9200"]
  # Optional: username: "user"
  # Optional: password: "password"
  # Optional: index: "filebeat-%{+yyyy.MM.dd}" # Index pattern
  # Optional: ssl.verification_mode: "none" # If using self-signed certs (not recommended for prod)
  ```

### 4. OpenSearch Configuration

   **Index Templates/Mappings:** Define how data is stored and indexed for optimal searching and analysis.

   **Index Lifecycle Management (ILM):** Configure policies for managing indexes (e.g., rollover, shrink, delete).

   **Security Plugin:** OpenSearch comes with a security plugin. Configure users, roles, and access controls.


### 5. Accessing OpenSearch Dashboards

  Web interface is typically ```http://<your_opensearch_dashboards_ip>:5601```.
  Use "Index Patterns" to define how Dashboards access data in OpenSearch indexes.


### Integrating with Wazuh

  If you are using Wazuh's Docker deployment or its standard installation, it often comes with its own pre-configured OpenSearch (Wazuh Indexer) and OpenSearch Dashboards (Wazuh Dashboard) stack.

  You can configure Wazuh to forward alerts to an external Splunk or OpenSearch instance if you want to centralize logs further or use features from your primary SIEM. This is typically done via Wazuh's ```ossec.conf``` using ```syslog_output``` or specific integration blocks.


### Key Considerations

**Storage:** Log data can grow very large. Plan for adequate, fast storage.

**Resources (CPU/RAM):** Both Splunk and OpenSearch can be resource-intensive, especially during indexing and searching.

**Parsing:** Ensure your logs are parsed correctly (timestamp extraction, field extraction) for effective analysis. This is managed by source types in Splunk and ingest pipelines or Filebeat processors in OpenSearch.

**Security:** Secure access to your Splunk/OpenSearch instances and dashboards. Use HTTPS. Implement access controls.

**Licensing (Splunk):** Be mindful of Splunk's licensing model if you exceed the free tier limits.

This guide is a starting point. Both Splunk and OpenSearch are vast platforms with extensive documentation and community support.


