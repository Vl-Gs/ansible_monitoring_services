
# Automated Service Configuration and Monitoring System

## Project Overview

This project enables automated installation, configuration, and monitoring of multiple services across different servers. By leveraging Ansible playbooks and roles, it ensures that specified services are installed, running, and providing metrics that are collected for monitoring via Prometheus and visualized in Grafana. The monitoring stack uses Dockprom, running on a dedicated monitoring server with Docker Compose.

## Prerequisites

- **Ansible** installed on your control machine.
- **Docker** and **Docker Compose** installed on the monitoring server.
- **SSH access** with appropriate privileges to all managed servers.
  
## Inventory Setup

1. **Define servers** in the inventory file located at `inventory/hosts.ini`. 
2. Group servers based on the services that should be installed and monitored.

Example:
```ini
[web_servers]
server1_ip
server2_ip

[database_servers]
server3_ip
server4_ip

[monitoring_server]
monitoring_server_ip
```

## Configuration Files

The configuration files for each service are structured to allow for easy addition and modularity. Key files and folders include:

- `roles/<service_name>/`: Contains installation, configuration, and metric setup for each service.
- `templates/`: Holds Jinja2 templates for configuration files (such as Prometheus configuration).
- `group_vars/` and `host_vars/`: Define service-specific variables or per-server settings.

## Setting Up the Environment

### Step 1: Install and Configure Services

Run the playbook `setup_services.yml` to install and configure the necessary services as specified in the inventory file. This playbook checks if each service is installed and started; if a service is missing or inactive, it installs and starts it.

```bash
ansible-playbook playbooks/setup_services.yml -i inventory/hosts.ini
```

### Step 2: Deploy Monitoring Stack

Run the playbook `deploy_monitoring.yml` to configure the monitoring server. This will:
- Install Docker (if not already installed).
- Clone and configure Dockprom.
- Generate `prometheus.yml` with dynamic targets based on the inventory file.
- Start Dockprom using Docker Compose.

```bash
ansible-playbook playbooks/deploy_monitoring.yml -i inventory/hosts.ini
```

### Customizing the Monitoring Stack

The `prometheus.yml` file dynamically includes all active services based on the inventory. Ensure all servers are grouped correctly in `hosts.ini` to have their metrics collected.

## Adding a New Service

When adding a new service, follow these steps to ensure it's monitored and configured correctly:

1. **Create a Role for the Service:**
   - Run `ansible-galaxy init roles/<new_service_name>` to create the necessary role structure.
   - Define tasks for installation, configuration, and starting of the service in `roles/<new_service_name>/tasks/main.yml`.
   - Set up metrics for the new service by creating a metrics exporter configuration, if available.

2. **Add Service Variables:**
   - Define service-specific variables in `group_vars/<service_group>.yml` or `host_vars/<hostname>.yml`.
   - Example: If adding PostgreSQL, add necessary variables in `group_vars/postgresql.yml`.

3. **Add the Service to Inventory:**
   - Add the new server's IP to the appropriate group in `inventory/hosts.ini` (e.g., `[database_servers]`).
   - Ensure the service name matches the defined group for automatic configuration.

4. **Update `prometheus.yml`:**
   - The `prometheus.yml` file is dynamically generated, so running the playbooks will automatically include the new service if configured correctly in the inventory and `roles/`.

5. **Run the Playbooks Again:**
   - Execute `setup_services.yml` and `deploy_monitoring.yml` again to apply the new configuration.

## Adding a New Server

1. **Update Inventory:**
   - Add the new server's IP to `inventory/hosts.ini` under the relevant group (e.g., `[web_servers]`).

2. **Run Playbooks:**
   - Execute `setup_services.yml` and `deploy_monitoring.yml` to configure the new server with the required services and metrics.

