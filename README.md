# Observability & Automation Stack

This project implements a complete observability stack for Oracle Database using Prometheus, Grafana, and n8n for automation/AI integration.

## Architecture

1.  **Databases**:
    - **Oracle Database (`oracle-db`)**
    - **MySQL Database (`mysql-db`)**
    - **PostgreSQL Database (`postgres-db`)**
2.  **Exporters**:
    - **OracleDB Exporter (`oracledb-exporter`)**: Port `9161`.
    - **MySQL Exporter (`mysqld-exporter`)**: Port `9104`.
    - **Postgres Exporter (`postgres-exporter`)**: Port `9187`.
3.  **Prometheus (`prometheus`)**: Scrapes metrics from all exporters every 15 seconds.
4.  **Grafana (`grafana`)**: Visualizes metrics. Pre-configured with Prometheus as the data source.
5.  **n8n (`n8n`)**: Workflow automation tool to process alerts and integrate with LLMs.

## Prerequisites

- Docker
- Docker Compose

## Getting Started

1.  **Start the Stack**:
    ```bash
    docker-compose up -d
    ```

2.  **Access Services**:
    - **Grafana**: [http://localhost:3000](http://localhost:3000) (User: `admin`, Pass: `admin`)
    - **Prometheus**: [http://localhost:9090](http://localhost:9090)
    - **n8n**: [http://localhost:5678](http://localhost:5678) (User: `admin`, Pass: `admin`)
    - **Oracle DB**: `localhost:1521` (System/Password: `system`/`oracle`)
    - **MySQL DB**: `localhost:3306` (User/Pass: `appuser`/`apppassword`, Root Pass: `root`)
    - **Postgres DB**: `localhost:5432` (User/Pass: `postgres`/`postgres`)

## Configuration Details

### 1. Grafana Dashboards
- Log in to Grafana.
- Go to **Dashboards** > **Import**.
- You can import a standard Oracle Dashboard (e.g., ID `3333` or `12463` from Grafana Labs) or create your own using the Prometheus datasource.

### 2. Setting up LLM Integration with n8n
To achieve the goal of using AI to predict errors:

1.  **Create a Workflow in n8n**:
    - Open n8n at [http://localhost:5678](http://localhost:5678).
    - Create a new workflow.
2.  **Add a Trigger**:
    - Use a **Webhook** node (for Grafana Alerts) or a **Schedule** node (to run periodically, e.g., every hour).
3.  **Fetch Metrics**:
    - Use an **HTTP Request** node to query Prometheus API (`http://prometheus:9090/api/v1/query`) for key metrics.
    - Examples:
        - Oracle: `oracle_session_count`
        - MySQL: `mysql_global_status_threads_connected`
        - Postgres: `pg_stat_activity_count`
4.  **Analyze with LLM**:
    - Add an **AI/LLM Chain** node (if available in your n8n version) or a generic **HTTP Request** node to call OpenAI/Anthropic API.
    - **Prompt Example**:
      > "Here are the current Oracle DB metrics: {json_metrics}. Analyze them for potential performance bottlenecks or space issues. Provide a summary and recommended actions."
5.  **Action**:
    - Use a **Slack**, **Email**, or **Discord** node to send the LLM's analysis to the operations team.

### 3. Grafana Alerting
- Configure Grafana Alerting to send a webhook to your n8n Webhook URL when specific thresholds are breached (e.g., Tablespace > 90%).

## Troubleshooting
- **Oracle Connection**: If the exporter fails to connect, ensure the `oracle-db` container is healthy (`docker ps`).
- **Prometheus Targets**: Check [http://localhost:9090/targets](http://localhost:9090/targets) to see if `oracle-db` is UP.
