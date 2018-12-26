# Using Grafana with Prometheus for Alerting and Monitoring

## Instructions & Tasks

In this learning activity, you are tasked with setting up Prometheus and Grafana.

The environment has 4 containers running: prometheus, cadvisor, nginx, and redis.

### Configure Docker

1. Create a daemon.json file for Docker.
2. Add "metrics-addr" and set it to "0.0.0.0:9323".
3. Set "experimental" to true.
4. Restart the Docker service.

### Update Prometheus

Update the prometheus.yml file that is located in /root.
Rename the cadvisor job to prometheus.
1. Under targets, add the following:
2. prometheus:9090
3. node-exporter:9100
4. pushgateway:9091
5. Create a new job called docker.
6. Set the scrape interval to 5 seconds.
7. Create a static config and targets.
8. Add the private IP with port 9323 as a target.

### Update Docker Compose:

1. Open docker-compose.yml.
2. Add a new service called pushgateway.
3. Use the prom/pushgateway image.
4. Name the container pushgateway.
5. Map port 9091 to port 9091 on the container.
6. Add a new service called node-exporter.
7. Use prom/node-exporter:latest for the image.
8. Name the container node-exporter.
9. Map port 9100 to port 9100 on the container.
10. Create a new service called grafana.
11. Use the grafana/grafana image.
12. Name the container grafana.
13. Map port 3000 to port 3000 on the container.
14. Create an environment variable called GF_SECURITY_ADMIN_PASSWORD and set it to password.
15. Make sure that it depends on prometheus and the cadvisor containers.
16. Execute docker-compose up -d to rebuild the environment.
17. Verify that all of your Prometheus targets are healthy by going to http://<PUBLIC_IP>:9090/targets.

### Create a New Grafana Datasource:

1. Log in to Grafana at http://<PUBLIC_IP>:3000/login with the username admin and the password password.
2. Add a new datasource called Prometheus and set it to the prometheus type.
3. Set the URL to http://<PRIVATE_IP>:9090.

### Add the Docker Dashboard to Grafana:

1. Import the new dashboard using the JSON file located here.
2. Set Prometheus to the datasource you created.
3. Set the quick range to Last 5 minutes.

### Add an Email Notification Channel:

1. Create a new Email Notification Channel.
2. Name the channel Email.
3. Add your email address to the notification.

### Create an Alert for CPU Usage:

1. Edit the CPU Usage graph.
2. On the Metrics tab, create a new query: sum(rate(process_cpu_seconds_total[1m])) * 100
3. Hide it from view.
4. Set the alert condition to use the new query. It should be E in the dropdown menu.
5. Set the Is Above condition to 75.
6. In the Notification section, set the alert to Email Alerts.
7. The message should say “CPU usage is over 75%”.
8. Save your changes.

### Configure Docker.

Open /etc/docker/daemon.json and add the following.

```json
{
  "metrics-addr" : "0.0.0.0:9323",
  "experimental" : true
}
```

### Update `prometheus.yml`.

Edit the prometheus.yml file in the /root directory.

vi ~/prometheus.yml
Change the contents to reflect the following:

```yml
scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    static_configs:
    - targets:
      - prometheus:9090
      - node-exporter:9100
      - pushgateway:9091
      - cadvisor:8080

  - job_name: docker
    scrape_interval: 5s
    static_configs:
    - targets:
      - PRIVATE_IP_ADDRESS:9323
```

### Update `docker-compose.yml`.
Edit docker-compose.yml in the /root directory.

vi ~/docker-compose.yml
Change the contents to reflect the following:

```yml
version: '3'
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - 9090:9090
    command:
      - --config.file=/etc/prometheus/prometheus.yml
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
    depends_on:
      - cadvisor
  cadvisor:
    image: google/cadvisor:latest
    container_name: cadvisor
    ports:
      - 8080:8080
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
  pushgateway:
    image: prom/pushgateway
    container_name: pushgateway
    ports:
      - 9091:9091
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    expose:
      - 9100
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - 3000:3000
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=password
    depends_on:
      - prometheus
      - cadvisor
```

### Install the Docker and System Monitoring dashboard.
Click the plus sign (+) on the left side of the Grafana interface. Click Import. Copy the contents of the JSON file included in the lab instructions.

Paste the contents of the JSON file into the Import screen of the Grafana interface, and click Load. In the upper right-hand corner, click on Refresh every 5m and select Last 5 minutes.