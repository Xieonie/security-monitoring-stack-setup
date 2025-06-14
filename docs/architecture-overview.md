# Security Monitoring Stack: Architecture Overview

This document outlines the proposed architecture for the comprehensive security monitoring system. The goal is to create a layered approach, leveraging the strengths of different tools for metrics, logs, and security event management.

## Core Components

1.  **Endpoints (Monitored Systems):**
    * Servers, workstations, network devices, applications.
    * These systems will run agents to collect data.

2.  **Agents:**
    * **Prometheus Exporters:** (e.g., `node_exporter` for system metrics, `blackbox_exporter` for endpoint probing, application-specific exporters). They expose metrics in a format Prometheus can scrape.
    * **Wazuh Agents:** Installed on endpoints to collect logs, monitor file integrity, detect intrusions, check configurations, and forward data to the Wazuh Manager.
    * **Log Shippers (for Splunk/OpenSearch):**
        * **Splunk Universal Forwarder (UF):** If using Splunk, UFs collect and forward logs.
        * **Beats (Filebeat, Metricbeat, etc.) / Fluentd / Fluent Bit:** If using OpenSearch/Elasticsearch, these agents collect and forward logs and metrics.

3.  **Metrics Collection & Alerting Layer:**
    * **Prometheus Server:** Scrapes metrics from configured exporters. Stores time-series data. Evaluates alerting rules.
    * **Alertmanager:** Receives alerts from Prometheus. Handles deduplication, grouping, silencing, and routing of alerts to notification channels (email, Slack, PagerDuty, etc.).

4.  **Log Management & SIEM Layer:**
    * **Wazuh Manager:**
        * Receives data from Wazuh agents.
        * Analyzes data using decoders and rules.
        * Generates alerts for security events.
        * Provides an API for querying data.
    * **Wazuh Indexer (based on OpenSearch):** Stores and indexes alerts and event data generated by the Wazuh manager.
    * **Wazuh Dashboard (based on OpenSearch Dashboards):** Provides a UI for visualizing Wazuh data, managing agents, and reviewing alerts.
    * **Splunk (Alternative/Complementary):**
        * **Splunk Indexers:** Store and index all collected logs (from UFs, syslog, etc.).
        * **Splunk Search Heads:** Provide the interface for searching, analyzing, and visualizing data.
        * **Splunk Apps & Add-ons:** Extend functionality for specific data sources and use cases (e.g., Splunk App for Wazuh).
    * **OpenSearch/Elasticsearch & OpenSearch Dashboards/Kibana (Alternative to Splunk):**
        * Similar functionality to Splunk for indexing, searching, and visualizing log data. Often used as the backend for Wazuh Indexer/Dashboard.

5.  **Visualization & Analysis Layer:**
    * **Grafana:** Connects to Prometheus (for metrics) and potentially OpenSearch/Elasticsearch/Splunk (for logs and SIEM data via plugins) to create unified dashboards.
    * **Wazuh Dashboard:** For Wazuh-specific analysis and management.
    * **Splunk Search & Reporting / OpenSearch Dashboards:** For deep log analysis, ad-hoc queries, and SIEM dashboards.

## Data Flow Example

1.  **Metrics:**
    * `node_exporter` on Server A exposes system metrics.
    * Prometheus scrapes `/metrics` endpoint of `node_exporter` on Server A.
    * Metrics are stored in Prometheus TSDB.
    * Grafana queries Prometheus to display system metrics for Server A.
    * Prometheus evaluates rules; if an alert fires (e.g., high CPU), it sends to Alertmanager.
    * Alertmanager sends a notification (e.g., email).

2.  **Logs & Security Events (Wazuh + Splunk/OpenSearch):**
    * Wazuh agent on Server A detects a failed SSH login attempt.
    * Agent sends event data to Wazuh Manager.
    * Wazuh Manager analyzes the event, matches it against rules, and generates a Wazuh alert.
    * The Wazuh alert is stored in the Wazuh Indexer (OpenSearch).
    * The alert can be viewed in the Wazuh Dashboard.
    * **Option A (Wazuh to Splunk/OpenSearch):** Wazuh alerts/logs can be forwarded from the Wazuh Manager or directly from the Wazuh Indexer to a central Splunk/OpenSearch instance for broader correlation.
    * **Option B (Direct to Splunk/OpenSearch):** A Splunk UF or Filebeat on Server A also collects `/var/log/auth.log` and sends it directly to Splunk/OpenSearch.
    * Splunk/OpenSearch indexes these logs.
    * Analysts use Splunk Search/OpenSearch Dashboards to query logs, create dashboards, and correlate events from multiple sources (including Wazuh data if forwarded).

## Network Considerations

* Ensure agents can reach their respective managers/servers (Prometheus, Wazuh Manager, Splunk Indexers, etc.).
* Secure communication channels (TLS) between all components.
* Plan for network bandwidth, especially for log shipping.

## Deployment Strategy (High-Level)

* **Containerization (Recommended where possible):**
    * Prometheus, Alertmanager, Grafana can be easily run in Docker.
    * Wazuh offers Docker deployments for its manager, indexer, and dashboard.
    * Splunk/OpenSearch can also be run in Docker for smaller setups, but for production, dedicated instances might be preferred.
* **Dedicated Hosts/VMs:** For larger deployments, core components like Splunk Indexers, OpenSearch clusters, or Wazuh Managers might require dedicated resources.

This architecture provides a flexible and powerful foundation for security monitoring. The specific implementation details will depend on the scale of your environment and your available resources.