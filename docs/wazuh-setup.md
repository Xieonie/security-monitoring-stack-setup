## Wazuh Setup Guide

Wazuh is a powerful open-source security platform that provides XDR and SIEM capabilities. It consists of a Wazuh manager (server), Wazuh indexer (for storing and indexing data, based on OpenSearch), Wazuh dashboard (for visualization, based on OpenSearch Dashboards), and Wazuh agents installed on monitored endpoints.

This guide provides a high-level overview. For detailed instructions, **always refer to the official [Wazuh documentation](https://documentation.wazuh.com/)**. Wazuh offers various installation methods, including an all-in-one deployment script, step-by-step installation, and Docker containers.

## Recommended Installation Method: Docker (for testing/small environments)

Wazuh provides official Docker images and a `docker-compose.yml` for deploying the full stack. This is often the easiest way to get started.

1.  **Clone the Wazuh Docker repository:**
    ```bash
    git clone [https://github.com/wazuh/wazuh-docker.git](https://github.com/wazuh/wazuh-docker.git) -b v4.x.x # Replace v4.x.x with the desired Wazuh version branch
    cd wazuh-docker
    ```
2.  **Review and Configure:**
    * The repository contains different `docker-compose.yml` files for single-node and multi-node setups. For a basic setup, you might look at `single-node/docker-compose.yml`.
    * You might need to adjust environment variables, persistent volume paths, and exposed ports as needed.
    * Pay attention to resource allocation (RAM, CPU) for the Docker containers, especially for the Wazuh indexer.
3.  **Generate Certificates (for internal communication):**
    The Wazuh Docker setup often includes scripts to generate the necessary SSL certificates for secure communication between components.
    ```bash
    # Example command, refer to the specific Wazuh Docker branch documentation
    # docker-compose -f single-node/docker-compose.yml run --rm wazuh-certs-generator
    ```
4.  **Start the Wazuh Stack:**
    ```bash
    docker-compose -f single-node/docker-compose.yml up -d
    ```
5.  **Access Wazuh Dashboard:**
    * Typically available at `https://<your_server_ip>` (port 443).
    * Default credentials are often `admin` / `SecretPassword` (or similar - **CHANGE THESE IMMEDIATELY!** Refer to the Wazuh Docker documentation for the specific version).

An example `docker-compose/wazuh-compose.yml` is provided in this repository, but it's **highly recommended to use the official and version-specific `docker-compose.yml` from the Wazuh GitHub repository** as it's actively maintained and tested.

## Wazuh Agent Installation (on monitored endpoints)

Once the Wazuh server components are running, you need to install Wazuh agents on the hosts you want to monitor.

1.  **From Wazuh Dashboard:**
    * The Wazuh dashboard usually provides an "Agents" section where you can get pre-configured deployment commands for various operating systems.
    * This often includes setting the `WAZUH_MANAGER` IP address.
2.  **Manual Installation:**
    * Download the agent package for your OS from the [Wazuh documentation](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html).
    * During installation or by editing `/var/ossec/etc/ossec.conf` on the agent, configure the IP address of your Wazuh manager:
      ```xml
      <client>
        <server>
          <address>YOUR_WAZUH_MANAGER_IP</address>
          ...
        </server>
      </client>
      ```
    * Start the agent service (e.g., `sudo systemctl start wazuh-agent`).

## Agent Registration/Authentication

Agents need to be registered and authenticated with the Wazuh manager.
* **Password-based:** Simple, but less secure for large deployments.
* **Certificate-based (default):** More secure. Agents will attempt to register automatically if manager allows it, or you can use `agent-auth` or manage through the API/dashboard.

## Key Wazuh Configuration Files

* **Manager:** `/var/ossec/etc/ossec.conf` (see `wazuh-config/ossec.conf.manager.example`)
    * Global settings, rule locations, decoders, integrations (e.g., email alerts, Slack).
* **Agent:** `/var/ossec/etc/ossec.conf` (see `wazuh-config/ossec.conf.agent.example`)
    * Manager IP, log collection settings, syscheck (FIM), local rule configurations.
* **Rules & Decoders:** Located in `/var/ossec/ruleset/` on the manager.
    * Custom rules can be added to `/var/ossec/etc/rules/local_rules.xml` (see `wazuh-config/custom-rules/local_rules.xml`).
    * Custom decoders to `/var/ossec/etc/decoders/local_decoders.xml`.

## Basic Wazuh Usage

* **Dashboard:** Explore events, security alerts, agent status, FIM changes, vulnerability assessments.
* **Ruleset:** Understand existing rules and consider creating custom rules for your specific environment and security policies.
* **Integrations:** Configure integrations for alerting (e.g., email, Slack) or to forward Wazuh alerts to other systems like Splunk/OpenSearch if needed.

## Important Considerations

* **Resource Requirements:** Wazuh, especially the indexer component, can be resource-intensive (RAM, disk space). Plan accordingly.
* **Updates:** Keep your Wazuh components (server and agents) updated.
* **Tuning:** Wazuh rules can generate a lot of noise initially. You'll need to tune them (e.g., by creating whitelists or modifying rule levels) to reduce false positives.
* **Backups:** Regularly back up your Wazuh manager configuration and the Wazuh indexer data.

This guide provides a starting point. The official Wazuh documentation is your primary resource for in-depth information.