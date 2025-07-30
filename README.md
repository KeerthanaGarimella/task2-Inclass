INFO8985 - Task 2: OpenTelemetry Collector Setup with PostgreSQL Metrics
This task extends the observability setup to include PostgreSQL metrics and custom SQL query metrics, visualized using SigNoz.

✅ Overview
We use the following OpenTelemetry receivers:

postgresqlreceiver: Collects metrics from the PostgreSQL instance.

sqlqueryreceiver: Executes custom SQL queries and exports the results as metrics.

Both metrics are sent to ClickHouse through the OpenTelemetry Collector and visualized in SigNoz dashboards.

🧰 Prerequisites
Ensure you have the following installed and configured:

 Docker and Docker Compose

 Ansible

 GitHub Codespaces (or local Docker setup)

🗃️ Database Details
A PostgreSQL container is already provisioned via ansible-playbook up.yml.

Database: lcbo

User: lcbo

Password: Secret5555

🚀 Getting Started
Step 1: Start the observability stack (SigNoz + LCBO DB)
bash
ansible-playbook up.yml
This command brings up:

SigNoz core services (ClickHouse, frontend, otel-collector, etc.)

PostgreSQL + pgAdmin4 container with LCBO sample data preloaded

🛠️ If PostgreSQL isn't running (manual run):
bash
docker run --name in-class-work-2-db-1 \
  -e POSTGRES_USER=lcbo \
  -e POSTGRES_PASSWORD=Secret5555 \
  -e POSTGRES_DB=lcbo \
  -p 5432:5432 \
  -d postgres
🔧 Configuration
Update your otel-collector-config.yaml to include the following receivers:

📈 postgresqlreceiver
yaml
receivers:
  postgresql:
    endpoint: "host.docker.internal:5432"
    username: "lcbo"
    password: "Secret5555"
    database: "lcbo"
    tls:
      insecure: true
📊 sqlqueryreceiver
yaml
  sqlquery:
    driver: "postgres"
    datasource: "host=host.docker.internal port=5432 user=lcbo password=Secret5555 dbname=lcbo sslmode=disable"
    queries:
      - sql: "SELECT COUNT(*) AS product_count FROM products"
        metrics:
          - metric_name: "custom.product_count"
            value_column: "product_count"
            value_type: "gauge"
            data_type: "int"
Ensure these receivers are part of the metrics pipeline and use the clickhousemetricswrite exporter.

📦 Stop the Stack
bash
ansible-playbook down.yml
