# Prometheus Setup and Configuration Documentation

 This script provides an overview of the Prometheus monitoring setup and related components.

## Prometheus Service Status
 To check the status of the Prometheus service, use the following command:
```
 sudo systemctl status prometheus
```
## Ports Used
Prometheus listens on port 9090, and the Node Exporter listens on port 9100.
```
9090 -> prometheus port
9100 -> node exporter port 
```
## Configuration Changes
After making any changes to the prometheus.yaml configuration file, it is necessary to restart the Prometheus service to apply the changes.
```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "go-email-v6-countfiles"
    static_configs:
      - targets:
          - "localhost:9100"                           
```

## Node Exporter
The Node Exporter is a utility that collects system metrics.A shell script is used to count and write the number of files in a specified directory.
```
#!/bin/bash

# Directory to monitor
DIR_PATH="/home/purshotamp/codebase/go-emailapi-v6"

# Count the number of files in the directory
FILE_COUNT=$(find "$DIR_PATH" -type f | wc -l)

# Output in Prometheus format
echo "number_of_files_in_directory{directory=\"$DIR_PATH\"} $FILE_COUNT"
```

## File Counting Script
The script count_files.sh is designed to count the number of files in a directory.It is scheduled to run every minute using a cron job, as per the requirements.

## Cron Job Configuration
The following cron job will execute the count_files.sh script every minute and output the results to file_count.prom, which is read by the Node Exporter.
```
* * * * * /home/purshotamp/codebase/prometheus/count_files.sh > /var/lib/node_exporter/textfile_collector/file_count.prom 2>> /home/purshotamp/codebase/check/cron.log
```
## Node Exporter Execution
 To run the Node Exporter with the textfile collector, use the following command:
```
pwd
/home/purshotamp/node_exporter-1.8.2.linux-amd64
purshotamp@NL1519:~/node_exporter-1.8.2.linux-amd64$ ./node_exporter --collector.textfile.directory="/var/lib/node_exporter/textfile_collector"
```
# End of Documentation
