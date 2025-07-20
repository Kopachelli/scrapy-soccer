# Deployment Guide

This guide covers deploying the Soccer Data Scraper in various environments, from local development to production.

## Table of Contents

1. [Local Development](#local-development)
2. [Docker Deployment](#docker-deployment)
3. [Cloud Deployment](#cloud-deployment)
4. [Production Setup](#production-setup)
5. [Monitoring and Logging](#monitoring-and-logging)
6. [Troubleshooting](#troubleshooting)

## Local Development

### Prerequisites

- Python 3.7+
- pip
- MySQL (optional, for database storage)

### Setup

1. **Clone the repository**
```bash
git clone https://github.com/zseta/scrapy-soccer.git
cd scrapy-soccer
```

2. **Create virtual environment**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Configure settings**
Edit `soccer/settings.py`:
```python
# For database storage (optional)
DB_SETTINGS = {
    'db': 'soccer_db',
    'user': 'soccer_user',
    'passwd': 'soccer_pass',
    'host': 'localhost',
}

# Enable database pipeline
ITEM_PIPELINES = {
    'soccer.pipelines.CleaningPipeline': 300,
    'soccer.pipelines.DatabasePipeline': 301  # Uncomment to enable
}
```

5. **Run the spider**
```bash
python soccer/run.py
```

## Docker Deployment

### Dockerfile

```dockerfile
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    default-libmysqlclient-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m -u 1000 scraper && chown -R scraper:scraper /app
USER scraper

# Expose port (if running API)
EXPOSE 8000

# Default command
CMD ["python", "soccer/run.py"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  scraper:
    build: .
    container_name: soccer-scraper
    environment:
      - DB_HOST=mysql
      - DB_USER=soccer_user
      - DB_PASS=soccer_pass
      - DB_NAME=soccer_db
    depends_on:
      - mysql
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
    restart: unless-stopped

  mysql:
    image: mysql:8.0
    container_name: soccer-mysql
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: soccer_db
      MYSQL_USER: soccer_user
      MYSQL_PASSWORD: soccer_pass
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: soccer-redis
    ports:
      - "6379:6379"
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    container_name: soccer-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - scraper
    restart: unless-stopped

volumes:
  mysql_data:
```

### Running with Docker

1. **Build and run**
```bash
docker-compose up -d
```

2. **View logs**
```bash
docker-compose logs -f scraper
```

3. **Stop services**
```bash
docker-compose down
```

## Cloud Deployment

### AWS Deployment

#### EC2 Setup

1. **Launch EC2 instance**
```bash
# Using AWS CLI
aws ec2 run-instances \
    --image-id ami-0c02fb55956c7d316 \
    --instance-type t3.medium \
    --key-name your-key-pair \
    --security-group-ids sg-xxxxxxxxx \
    --subnet-id subnet-xxxxxxxxx
```

2. **Install dependencies**
```bash
# Connect to instance
ssh -i your-key.pem ubuntu@your-instance-ip

# Update system
sudo apt-get update
sudo apt-get install -y python3-pip python3-venv mysql-server

# Clone repository
git clone https://github.com/zseta/scrapy-soccer.git
cd scrapy-soccer

# Setup Python environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. **Configure MySQL**
```bash
sudo mysql_secure_installation
sudo mysql -u root -p
```

```sql
CREATE DATABASE soccer_db;
CREATE USER 'soccer_user'@'localhost' IDENTIFIED BY 'soccer_pass';
GRANT ALL PRIVILEGES ON soccer_db.* TO 'soccer_user'@'localhost';
FLUSH PRIVILEGES;
```

4. **Setup systemd service**
```bash
sudo nano /etc/systemd/system/soccer-scraper.service
```

```ini
[Unit]
Description=Soccer Data Scraper
After=network.target mysql.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/scrapy-soccer
Environment=PATH=/home/ubuntu/scrapy-soccer/venv/bin
ExecStart=/home/ubuntu/scrapy-soccer/venv/bin/python soccer/run.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

5. **Start service**
```bash
sudo systemctl daemon-reload
sudo systemctl enable soccer-scraper
sudo systemctl start soccer-scraper
```

#### RDS Setup

1. **Create RDS instance**
```bash
aws rds create-db-instance \
    --db-instance-identifier soccer-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password your-password \
    --allocated-storage 20
```

2. **Update settings**
```python
DB_SETTINGS = {
    'db': 'soccer_db',
    'user': 'admin',
    'passwd': 'your-password',
    'host': 'your-rds-endpoint.amazonaws.com',
}
```

### Google Cloud Platform

#### Compute Engine

1. **Create instance**
```bash
gcloud compute instances create soccer-scraper \
    --zone=us-central1-a \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud
```

2. **Setup application**
```bash
# Connect to instance
gcloud compute ssh soccer-scraper --zone=us-central1-a

# Install dependencies
sudo apt-get update
sudo apt-get install -y python3-pip python3-venv

# Clone and setup
git clone https://github.com/zseta/scrapy-soccer.git
cd scrapy-soccer
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### Cloud SQL

1. **Create Cloud SQL instance**
```bash
gcloud sql instances create soccer-db \
    --database-version=MYSQL_8_0 \
    --tier=db-f1-micro \
    --region=us-central1
```

2. **Create database and user**
```bash
gcloud sql databases create soccer_db --instance=soccer-db
gcloud sql users create soccer_user --instance=soccer-db --password=soccer_pass
```

## Production Setup

### Environment Configuration

1. **Environment variables**
```bash
# .env file
DB_HOST=your-db-host
DB_USER=soccer_user
DB_PASS=soccer_pass
DB_NAME=soccer_db
REDIS_URL=redis://localhost:6379
LOG_LEVEL=INFO
ENVIRONMENT=production
```

2. **Update settings**
```python
import os
from dotenv import load_dotenv

load_dotenv()

DB_SETTINGS = {
    'db': os.getenv('DB_NAME'),
    'user': os.getenv('DB_USER'),
    'passwd': os.getenv('DB_PASS'),
    'host': os.getenv('DB_HOST'),
}

LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
```

### Security Configuration

1. **Firewall rules**
```bash
# Allow only necessary ports
sudo ufw allow 22    # SSH
sudo ufw allow 3306  # MySQL (if external)
sudo ufw allow 80    # HTTP
sudo ufw allow 443   # HTTPS
sudo ufw enable
```

2. **SSL/TLS setup**
```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d your-domain.com
```

### Backup Strategy

1. **Database backup**
```bash
#!/bin/bash
# backup.sh
DATE=$(date +%Y%m%d_%H%M%S)
mysqldump -u soccer_user -p soccer_db > backup_$DATE.sql
gzip backup_$DATE.sql
aws s3 cp backup_$DATE.sql.gz s3://your-backup-bucket/
```

2. **Automated backups**
```bash
# Add to crontab
0 2 * * * /path/to/backup.sh
```

## Monitoring and Logging

### Logging Configuration

1. **Structured logging**
```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
        }
        return json.dumps(log_entry)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('soccer_scraper.log'),
        logging.StreamHandler()
    ]
)
```

2. **Log rotation**
```bash
# /etc/logrotate.d/soccer-scraper
/home/ubuntu/scrapy-soccer/soccer_scraper.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 ubuntu ubuntu
}
```

### Monitoring Setup

1. **Prometheus configuration**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'soccer-scraper'
    static_configs:
      - targets: ['localhost:8000']
```

2. **Grafana dashboard**
```json
{
  "dashboard": {
    "title": "Soccer Scraper Metrics",
    "panels": [
      {
        "title": "Matches Scraped",
        "type": "stat",
        "targets": [
          {
            "expr": "soccer_matches_scraped_total"
          }
        ]
      }
    ]
  }
}
```

## Troubleshooting

### Common Issues

1. **Database connection errors**
```bash
# Check MySQL status
sudo systemctl status mysql

# Check connection
mysql -u soccer_user -p -h your-host soccer_db

# Check firewall
sudo ufw status
```

2. **Memory issues**
```bash
# Monitor memory usage
htop
free -h

# Check for memory leaks
ps aux | grep python
```

3. **Network issues**
```bash
# Test connectivity
ping stat-football.com
curl -I http://stat-football.com

# Check DNS
nslookup stat-football.com
```

### Performance Optimization

1. **Database optimization**
```sql
-- Add indexes
CREATE INDEX idx_match_date ON matches(match_date);
CREATE INDEX idx_league ON matches(league);
CREATE INDEX idx_teams ON matches(home_team, away_team);

-- Analyze table
ANALYZE TABLE matches;
```

2. **Scrapy optimization**
```python
# settings.py
CONCURRENT_REQUESTS = 16
CONCURRENT_REQUESTS_PER_DOMAIN = 8
DOWNLOAD_DELAY = 1
RANDOMIZE_DOWNLOAD_DELAY = 0.5
```

### Health Checks

1. **Application health check**
```python
# health_check.py
import requests
import mysql.connector
import redis

def check_database():
    try:
        conn = mysql.connector.connect(**DB_SETTINGS)
        conn.close()
        return True
    except:
        return False

def check_redis():
    try:
        r = redis.Redis.from_url(REDIS_URL)
        r.ping()
        return True
    except:
        return False

def check_website():
    try:
        response = requests.get('http://stat-football.com', timeout=10)
        return response.status_code == 200
    except:
        return False
```

2. **Automated health checks**
```bash
#!/bin/bash
# health_check.sh
python health_check.py
if [ $? -ne 0 ]; then
    echo "Health check failed" | mail -s "Soccer Scraper Alert" admin@example.com
fi
```