# Setup Monitoring System
The is about setting up Prometheus, Grafana, Node Exporter, and cAdvisor on Ubuntu, then configuring alerting rules and email notifications via Grafana.


Step 1: Install Prometheus, Grafana, Node Exporter & cAdvisor 

      # Update system:
     sudo apt update && sudo apt upgrade -y
              
      #Install Prometheus:
      # Create user
      sudo useradd --no-create-home --shell /bin/false prometheus  
      # Create directories
      sudo mkdir /etc/prometheus
      sudo mkdir /var/lib/prometheus
  
      # Download Prometheus
      # If your system have amd architecture 
      wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
      OR
      # If your system have arm architecture (e.g apple M1,M2...)
      wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-arm64.tar.gz

      tar xvf prometheus-*.tar.gz # change * with version number/folder name (prometheus-2.54.1.linux-arm64) you have installed
      
      # Move binaries
      cd prometheus-2.45.0.linux-amd64
      sudo cp prometheus promtool /usr/local/bin/
      
      # Move configuration
      sudo cp -r consoles/ console_libraries/ /etc/prometheus/
      sudo cp prometheus.yml /etc/prometheus/
      sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
      
      #Install Node Exporter
      wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
      tar xvf node_exporter-*.tar.gz
      sudo cp node_exporter-*/node_exporter /usr/local/bin/
      
      #Install cAdvisor (Docker required)
      sudo apt install docker.io -y
      sudo docker run -d \
        --name=cadvisor \
        --volume=/:/rootfs:ro \
        --volume=/var/run:/var/run:ro \
        --volume=/sys:/sys:ro \
        --volume=/var/lib/docker/:/var/lib/docker:ro \
        -p 8080:8080 \
        gcr.io/cadvisor/cadvisor:latest
  
      #Install Grafana
      sudo apt-get install -y apt-transport-https software-properties-common
      sudo mkdir -p /etc/apt/keyrings/
      wget -q -O - https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
      echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
      sudo apt-get update
      sudo apt-get install grafana -y
      sudo systemctl enable --now grafana-server

Step 2: Access Applications

Prometheus → http://localhost:9090

Node Exporter → http://localhost:9100/metrics

cAdvisor → http://localhost:8080

Grafana → http://localhost:3000
 (default login: admin/admin)

 

Step 3: Add Prometheus as Datasource in Grafana

Go to Grafana → Connections → Data sources.

Choose Prometheus.

Enter URL: http://localhost:9090 → Save & Test.

Step 4: Create Alert Rules in Grafana

Go to Alerting → Alert rules and create rules:

High CPU usage (>20% for 3 mins)
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[3m])) * 100) > 20

<img width="1302" height="654" alt="image" src="https://github.com/user-attachments/assets/4245c919-b29e-43dd-8f03-ed3a955ff25f" />
<img width="1274" height="702" alt="image" src="https://github.com/user-attachments/assets/eaadee76-6919-471a-814a-28be2839bec7" />
<img width="1292" height="697" alt="image" src="https://github.com/user-attachments/assets/67dcb696-fb22-452d-a1be-822478999b46" />



Low memory (<20%)
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 < 20

Container not running (1 min)
sum(container_last_seen{container_label_com_docker_swarm_service_name!=""}) < 1

<img width="2608" height="1368" alt="image" src="https://github.com/user-attachments/assets/311a0fab-d824-431f-9b91-57858eb56e15" />


Step 5: Integrate Gmail as Contact Point

Go to Alerting → Contact points.

Create a new contact point → choose Email.

Enter Gmail address (e.g., your_gmail@gmail.com).
<img width="1296" height="677" alt="image" src="https://github.com/user-attachments/assets/a421c3c0-57e3-457d-9150-605d817fad35" />


Configure Grafana SMTP

Edit /etc/grafana/grafana.ini:

    [smtp]
    enabled = true
    host = smtp.gmail.com:587
    user = your_gmail@gmail.com
    password = your_app_password
    from_address = your_gmail@gmail.com
    from_name = Grafana

⚠️ For Gmail, you must generate an App Password (from Google Account → Security → App Passwords).

Restart Grafana:

    sudo systemctl restart grafana-server


Test email → Alerts should now be sent.

🧪 6. Testing Alerts

    #Test High CPU
    sudo apt install stress-ng -y
    stress-ng --cpu 4 --timeout 300
    
    #Test Low Memory
    stress-ng --vm 1 --vm-bytes 80% --timeout 300
    
    #Test Container Not Running
    sudo docker stop cadvisor
    # Restart later:
    sudo docker start cadvisor


✅ At this point you’ll have:
-  Prometheus, Grafana, Node Exporter & cAdvisor installed
- Prometheus integrated with Grafana
-  Alert rules for CPU, memory, container state
- Email alerts sent to Gmail


<img width="2592" height="1378" alt="image" src="https://github.com/user-attachments/assets/76d2a448-f3fa-4b98-9d6a-8d5b497a704e" />




