# Airflow-Monitoring-with-StatsD-Prometheus-and-Grafana

Project Overview

This project provides a complete monitoring system for Apache Airflow using StatsD, Prometheus, and Grafana. It leverages Docker Compose to orchestrate services and enables tracking Airflow DAG runs, tasks, and performance metrics through Grafana dashboards, with data scraped by Prometheus and exposed by StatsD.
Prerequisites

- Docker and Docker Compose installed on your machine.
- Basic knowledge of Airflow, monitoring tools (StatsD, Prometheus), and Grafana for visualization.

Setup Instructions

1. Start the Services

  Start the entire setup with the following command:
    
    docker-compose up -d

  This command will initialize the following services:

  - Airflow Scheduler
  - Airflow Webserver (available at http://localhost:8081)
  - MySQL as the Airflow metadata database
  - StatsD Exporter for collecting Airflow metrics
  - Prometheus for scraping metrics from StatsD (available at http://localhost:9092)
  - Grafana for visualizing metrics (available at http://localhost:3000)

2. Accessing the Services

  - Airflow Webserver: http://localhost:8081 (Default login: admin / airflow)
  - Prometheus Dashboard: http://localhost:9092
  - Grafana Dashboard: http://localhost:3000 (Default login: admin / grafana)

3. Airflow Configuration

  In the provided setup, the Airflow instance is pre-configured to use StatsD for emitting metrics. The StatsD configuration in airflow.cfg ensures that metrics are captured and passed to the StatsD Exporter.

    global:
      scrape_interval:     5s
      evaluation_interval: 15s

    scrape_configs:
      - job_name: airflow
        scheme: http
        metrics_path: metrics
        static_configs:
          - targets: ['host.docker.internal:9102']
            labels:
              airflow_id: 'airflow'

4. Prometheus Configuration

  The prometheus/prometheus.yaml file is pre-configured to scrape metrics from the StatsD Exporter at http://localhost:9102.

  Configuration for Prometheus scraping StatsD metrics is provided in the prometheus.yaml:

    scrape_configs:
    - job_name: 'airflow'
    static_configs:
      - targets: ['statsd-exporter:9102']


5. Grafana Configuration

   Grafana is pre-configured to use Prometheus as a data source and comes with a Grafana dashboard to visualize Airflow metrics like DAG run times, task failures, and system performance.

   To access the Grafana dashboard:

  - Log into Grafana at http://localhost:3000 with the default login (admin / grafana).
  - Add Prometheus as a data source using the URL http://prometheus:9090.
  - Import the provided Grafana dashboard JSON (if included) to visualize Airflow metrics.

6. MySQL Configuration

  MySQL is used as the metadata database for Airflow. The configuration details are:

  - Port: 3307
  - User: root
  - Password: airflow
  - Database: airflow

  MySQL is configured to initialize automatically when starting the containers and has a health check configured to ensure it's up and running before Airflow services start.

7. StatsD Configuration

   The statsd.yaml file defines how StatsD metrics emitted by Airflow are mapped to specific counters, gauges, and timers. These metrics are processed by Prometheus and visualized in Grafana for real-time monitoring of Airflow's performance and health.
  Key Metrics Mapped:

    Counters:
        Job Start/End: Tracks how many times a job starts or ends.
            Example: af_agg_job_start, af_agg_job_end
        Operator Failures/Successes: Tracks success or failure of specific operators.
            Example: af_agg_operator_failures, af_agg_operator_successes
        Task Failures/Successes: Tracks task failures and successes.
            Example: af_agg_ti_failures, af_agg_ti_successes

    Gauges:
        DAG Processing Metrics: Monitors various DAG-related metrics, such as size, import errors, and parse times.
            Example: af_agg_dag_processing_total_parse_time, af_agg_dag_processing_last_runtime
        Executor Slots: Tracks the number of open and running slots in Airflow's executor.
            Example: af_agg_executor_running_tasks, af_agg_executor_open_slots

    Timers:
        DAG Task Duration: Measures the time taken for each DAG task to complete.
            Example: af_agg_dag_task_duration
        DAG Run Duration: Tracks the total time for a DAG run to succeed or fail.
            Example: af_agg_dagrun_duration_success, af_agg_dagrun_duration_failed

    Custom Counter:
        A custom metric my_custom_task_counter tracks custom task increments in DAGs like my_dag (defined in the Airflow DAG).



    
