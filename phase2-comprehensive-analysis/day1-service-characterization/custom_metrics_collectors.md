# Custom Metrics Collectors
*Directory structure and implementation for custom metrics collection*

## Directory Structure
```
custom_metrics_collectors/
├── python/
│   ├── complexity_metrics_collector.py
│   ├── build_performance_collector.py
│   ├── resource_efficiency_collector.py
│   └── requirements.txt
├── kubernetes/
│   ├── metrics_collector_deployment.yaml
│   ├── service_monitor.yaml
│   ├── rbac.yaml
│   └── configmap.yaml
├── scripts/
│   ├── collect_service_metrics.sh
│   ├── push_to_prometheus.sh
│   ├── heroku_metrics_collector.py
│   └── github_actions_metrics.py
└── docker/
    ├── Dockerfile
    ├── docker-compose.yml
    └── entrypoint.sh
```

## python/complexity_metrics_collector.py
```python
#!/usr/bin/env python3
"""
Service Complexity Metrics Collector
Collects and calculates complexity scores for GitOps vs Traditional CI/CD research
"""

import time
import requests
import subprocess
import json
import os
from datetime import datetime
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway
from typing import Dict, List, Optional

class ComplexityMetricsCollector:
    def __init__(self):
        self.registry = CollectorRegistry()
        self.prometheus_gateway = os.getenv('PROMETHEUS_PUSHGATEWAY', 'localhost:9091')
        
        # Initialize Prometheus metrics
        self.complexity_score = Gauge(
            'service_complexity_score', 
            'Service complexity score based on weighted components',
            ['service_name', 'methodology', 'technology_stack'],
            registry=self.registry
        )
        
        self.build_time = Gauge(
            'service_build_time_seconds',
            'Time taken to build service in seconds', 
            ['service_name', 'technology_stack'],
            registry=self.registry
        )
        
        self.resource_efficiency = Gauge(
            'service_resource_efficiency',
            'Resource efficiency percentage',
            ['service_name', 'resource_type'],
            registry=self.registry
        )
        
        self.deployment_frequency = Gauge(
            'service_deployment_frequency_per_day',
            'Number of deployments per day',
            ['service_name', 'methodology'],
            registry=self.registry
        )

    def calculate_complexity_score(self, service_data: Dict) -> float:
        """Calculate weighted complexity score"""
        weights = {
            'codebase': 0.20,
            'build': 0.25, 
            'resource': 0.20,
            'tech_stack': 0.15,
            'external_deps': 0.10,
            'deployment': 0.10
        }
        
        score = 0
        for component, weight in weights.items():
            if component in service_data:
                score += service_data[component] * weight
                
        return round(score, 2)

    def collect_service_metrics(self, services_config: Dict):
        """Collect metrics for all configured services"""
        
        for service_name, config in services_config.items():
            print(f"Collecting metrics for {service_name}...")
            
            # Calculate complexity score
            complexity = self.calculate_complexity_score(config.get('complexity_components', {}))
            
            # Set Prometheus metrics
            self.complexity_score.labels(
                service_name=service_name,
                methodology=config.get('methodology', 'unknown'),
                technology_stack=config.get('technology_stack', 'unknown')
            ).set(complexity)
            
            if 'build_time_seconds' in config:
                self.build_time.labels(
                    service_name=service_name,
                    technology_stack=config.get('technology_stack', 'unknown')
                ).set(config['build_time_seconds'])
            
            if 'resource_efficiency' in config:
                for resource_type, efficiency in config['resource_efficiency'].items():
                    self.resource_efficiency.labels(
                        service_name=service_name,
                        resource_type=resource_type
                    ).set(efficiency)
            
            if 'deployment_frequency_per_day' in config:
                self.deployment_frequency.labels(
                    service_name=service_name,
                    methodology=config.get('methodology', 'unknown')
                ).set(config['deployment_frequency_per_day'])

    def push_metrics(self):
        """Push metrics to Prometheus Gateway"""
        try:
            push_to_gateway(
                self.prometheus_gateway, 
                job='complexity-metrics-collector',
                registry=self.registry
            )
            print(f"✅ Metrics pushed to {self.prometheus_gateway}")
        except Exception as e:
            print(f"❌ Failed to push metrics: {e}")

# Service configuration based on Day 1 analysis
SERVICES_CONFIG = {
    "user_service": {
        "methodology": "GitOps",
        "technology_stack": "Python_FastAPI_PostgreSQL", 
        "complexity_components": {
            "codebase": 7.5,
            "build": 8.5,
            "resource": 7.0,
            "tech_stack": 8.0,
            "external_deps": 6.0,
            "deployment": 10.0
        },
        "build_time_seconds": 142,
        "deployment_frequency_per_day": 2.0,
        "resource_efficiency": {
            "memory": 23,
            "cpu": 5
        }
    },
    "order_service": {
        "methodology": "GitOps",
        "technology_stack": "Python_FastAPI_PostgreSQL_Redis",
        "complexity_components": {
            "codebase": 7.5,
            "build": 8.5, 
            "resource": 7.5,
            "tech_stack": 9.0,
            "external_deps": 8.0,
            "deployment": 9.5
        },
        "build_time_seconds": 123,
        "deployment_frequency_per_day": 3.5,
        "resource_efficiency": {
            "memory": 13,
            "cpu": 3
        }
    },
    "product_service": {
        "methodology": "Traditional",
        "technology_stack": "NodeJS_Express_MongoDB",
        "complexity_components": {
            "codebase": 7.0,
            "build": 5.5,
            "resource": 3.0,
            "tech_stack": 7.5,
            "external_deps": 4.5,
            "deployment": 4.0
        },
        "build_time_seconds": 67,
        "deployment_frequency_per_day": 0.2,
        "resource_efficiency": {
            "memory": 16,
            "cpu": 0  # Not available
        }
    },
    "cart_service": {
        "methodology": "Traditional", 
        "technology_stack": "Java_SpringBoot_WebFlux_Redis",
        "complexity_components": {
            "codebase": 6.5,
            "build": 7.5,
            "resource": 8.0,
            "tech_stack": 9.0,
            "external_deps": 7.0,
            "deployment": 6.5
        },
        "build_time_seconds": 47,
        "deployment_frequency_per_day": 0.1,
        "resource_efficiency": {
            "memory": 92,
            "cpu": 0  # Not available
        }
    }
}

if __name__ == "__main__":
    collector = ComplexityMetricsCollector()
    collector.collect_service_metrics(SERVICES_CONFIG)
    collector.push_metrics()
```

## python/build_performance_collector.py
```python
#!/usr/bin/env python3
"""
Build Performance Metrics Collector
Collects build timing and performance data from GitHub Actions and local builds
"""

import requests
import os
import json
from datetime import datetime, timedelta
from prometheus_client import CollectorRegistry, Histogram, Gauge, push_to_gateway

class BuildPerformanceCollector:
    def __init__(self):
        self.registry = CollectorRegistry()
        self.github_token = os.getenv('GITHUB_TOKEN')
        self.repo = 'kousaila502/ecommerce-microservices-platform'
        
        # Prometheus metrics
        self.build_duration = Histogram(
            'build_duration_seconds',
            'Build duration in seconds',
            ['service_name', 'build_type', 'status'],
            registry=self.registry
        )
        
        self.build_efficiency = Gauge(
            'build_efficiency_seconds_per_complexity',
            'Build time per complexity point', 
            ['service_name', 'methodology'],
            registry=self.registry
        )

    def get_github_workflow_runs(self, days_back=7):
        """Get recent GitHub Actions workflow runs"""
        if not self.github_token:
            print("❌ GITHUB_TOKEN not set")
            return []
            
        since = (datetime.now() - timedelta(days=days_back)).isoformat()
        
        url = f"https://api.github.com/repos/{self.repo}/actions/runs"
        headers = {
            'Authorization': f'token {self.github_token}',
            'Accept': 'application/vnd.github.v3+json'
        }
        params = {'created': f'>{since}', 'per_page': 100}
        
        try:
            response = requests.get(url, headers=headers, params=params)
            response.raise_for_status()
            return response.json().get('workflow_runs', [])
        except Exception as e:
            print(f"❌ Failed to get workflow runs: {e}")
            return []

    def collect_build_metrics(self):
        """Collect build performance metrics"""
        
        # Local build times from Day 1 analysis
        local_builds = {
            'user_service': {'duration': 142, 'complexity': 7.8, 'methodology': 'GitOps'},
            'order_service': {'duration': 123, 'complexity': 8.2, 'methodology': 'GitOps'},
            'product_service': {'duration': 67, 'complexity': 5.4, 'methodology': 'Traditional'},
            'cart_service': {'duration': 47, 'complexity': 7.5, 'methodology': 'Traditional'}
        }
        
        for service, data in local_builds.items():
            # Record build duration
            self.build_duration.labels(
                service_name=service,
                build_type='local_docker',
                status='success'
            ).observe(data['duration'])
            
            # Calculate and record efficiency
            efficiency = data['duration'] / data['complexity']
            self.build_efficiency.labels(
                service_name=service,
                methodology=data['methodology']
            ).set(efficiency)
        
        # GitHub Actions workflow data
        workflow_runs = self.get_github_workflow_runs()
        
        for run in workflow_runs:
            if run['conclusion'] == 'success':
                # Calculate duration in seconds
                created = datetime.fromisoformat(run['created_at'].replace('Z', '+00:00'))
                updated = datetime.fromisoformat(run['updated_at'].replace('Z', '+00:00'))
                duration = (updated - created).total_seconds()
                
                # Determine service from workflow name
                service_name = self.extract_service_name(run['name'])
                if service_name:
                    self.build_duration.labels(
                        service_name=service_name,
                        build_type='github_actions',
                        status=run['conclusion']
                    ).observe(duration)

    def extract_service_name(self, workflow_name):
        """Extract service name from workflow name"""
        name_mapping = {
            'User Service': 'user_service',
            'Order Service': 'order_service', 
            'Product Service': 'product_service',
            'Cart Service': 'cart_service'
        }
        
        for workflow, service in name_mapping.items():
            if workflow in workflow_name:
                return service
        return None

    def push_metrics(self):
        """Push metrics to Prometheus Gateway"""
        try:
            push_to_gateway(
                os.getenv('PROMETHEUS_PUSHGATEWAY', 'localhost:9091'),
                job='build-performance-collector',
                registry=self.registry
            )
            print("✅ Build performance metrics pushed")
        except Exception as e:
            print(f"❌ Failed to push build metrics: {e}")

if __name__ == "__main__":
    collector = BuildPerformanceCollector()
    collector.collect_build_metrics()
    collector.push_metrics()
```

## scripts/collect_service_metrics.sh
```bash
#!/bin/bash
# Service Metrics Collection Script
# Collects metrics from Kubernetes and Heroku services

set -euo pipefail

# Configuration
PROMETHEUS_GATEWAY="${PROMETHEUS_GATEWAY:-localhost:9091}"
METRICS_DIR="/tmp/service_metrics"
TIMESTAMP=$(date +%s)

# Create metrics directory
mkdir -p "$METRICS_DIR"

echo "🔍 Collecting service metrics..."

# Function to collect Kubernetes metrics
collect_k8s_metrics() {
    echo "📊 Collecting Kubernetes metrics..."
    
    # Get pod resource usage
    kubectl top pods -n research-apps --no-headers | while read line; do
        pod_name=$(echo $line | awk '{print $1}')
        cpu_usage=$(echo $line | awk '{print $2}' | sed 's/m//')
        memory_usage=$(echo $line | awk '{print $3}' | sed 's/Mi//')
        
        service_name=$(echo $pod_name | cut -d'-' -f1-2)
        
        # Push to Prometheus
        cat << EOF | curl -X POST "$PROMETHEUS_GATEWAY/metrics/job/k8s-metrics-collector/instance/$service_name" --data-binary @-
# HELP kubernetes_pod_cpu_usage_millicores Pod CPU usage in millicores
# TYPE kubernetes_pod_cpu_usage_millicores gauge
kubernetes_pod_cpu_usage_millicores{pod_name="$pod_name",service_name="$service_name",namespace="research-apps"} $cpu_usage

# HELP kubernetes_pod_memory_usage_mebibytes Pod memory usage in MiB
# TYPE kubernetes_pod_memory_usage_mebibytes gauge
kubernetes_pod_memory_usage_mebibytes{pod_name="$pod_name",service_name="$service_name",namespace="research-apps"} $memory_usage
EOF
    done
    
    # Get ArgoCD application status
    kubectl get applications -n argocd -o json | jq -r '.items[] | 
        select(.metadata.name | contains("service")) | 
        "\(.metadata.name) \(.status.sync.status) \(.status.health.status)"' | \
    while read app_name sync_status health_status; do
        sync_value=$([ "$sync_status" = "Synced" ] && echo 1 || echo 0)
        health_value=$([ "$health_status" = "Healthy" ] && echo 1 || echo 0)
        
        cat << EOF | curl -X POST "$PROMETHEUS_GATEWAY/metrics/job/argocd-metrics-collector/instance/$app_name" --data-binary @-
# HELP argocd_application_sync_status ArgoCD application sync status (1=synced, 0=not synced)
# TYPE argocd_application_sync_status gauge
argocd_application_sync_status{application_name="$app_name"} $sync_value

# HELP argocd_application_health_status ArgoCD application health status (1=healthy, 0=not healthy)
# TYPE argocd_application_health_status gauge  
argocd_application_health_status{application_name="$app_name"} $health_value
EOF
    done
}

# Function to collect Heroku metrics
collect_heroku_metrics() {
    echo "📊 Collecting Heroku metrics..."
    
    if [ -z "${HEROKU_API_KEY:-}" ]; then
        echo "⚠️ HEROKU_API_KEY not set, skipping Heroku metrics"
        return
    fi
    
    # Heroku apps to monitor
    apps=("ecommerce-product-service-56575270905a" "ecommerce-cart-service-f2a908c60d8a")
    
    for app in "${apps[@]}"; do
        echo "Collecting metrics for $app..."
        
        # Get dyno information
        dyno_info=$(heroku ps --app="$app" --json 2>/dev/null || echo "[]")
        
        if [ "$dyno_info" != "[]" ]; then
            echo "$dyno_info" | jq -r '.[] | "\(.type) \(.state) \(.size)"' | \
            while read dyno_type state size; do
                state_value=$([ "$state" = "up" ] && echo 1 || echo 0)
                
                cat << EOF | curl -X POST "$PROMETHEUS_GATEWAY/metrics/job/heroku-metrics-collector/instance/$app" --data-binary @-
# HELP heroku_dyno_status Heroku dyno status (1=up, 0=down)
# TYPE heroku_dyno_status gauge
heroku_dyno_status{app_name="$app",dyno_type="$dyno_type",size="$size"} $state_value
EOF
            done
        fi
        
        # Collect service health
        service_url=""
        service_name=""
        
        case $app in
            "ecommerce-product-service-56575270905a")
                service_url="https://$app.herokuapp.com/health"
                service_name="product_service"
                ;;
            "ecommerce-cart-service-f2a908c60d8a")
                service_url="https://$app.herokuapp.com/health"
                service_name="cart_service"
                ;;
        esac
        
        if [ -n "$service_url" ]; then
            if curl -sf "$service_url" > /dev/null; then
                health_status=1
            else
                health_status=0
            fi
            
            cat << EOF | curl -X POST "$PROMETHEUS_GATEWAY/metrics/job/service-health-collector/instance/$service_name" --data-binary @-
# HELP service_health_status Service health status (1=healthy, 0=unhealthy)
# TYPE service_health_status gauge
service_health_status{service_name="$service_name",platform="heroku"} $health_status
EOF
        fi
    done
}

# Function to collect build performance metrics
collect_build_metrics() {
    echo "📊 Collecting build performance metrics..."
    
    # Day 1 build times (from actual measurements)
    declare -A build_times=(
        ["user_service"]="142"
        ["order_service"]="123" 
        ["product_service"]="67"
        ["cart_service"]="47"
    )
    
    declare -A complexity_scores=(
        ["user_service"]="7.8"
        ["order_service"]="8.2"
        ["product_service"]="5.4"
        ["cart_service"]="7.5"
    )
    
    for service in "${!build_times[@]}"; do
        build_time=${build_times[$service]}
        complexity=${complexity_scores[$service]}
        efficiency=$(echo "scale=2; $build_time / $complexity" | bc)
        
        cat << EOF | curl -X POST "$PROMETHEUS_GATEWAY/metrics/job/build-metrics-collector/instance/$service" --data-binary @-
# HELP service_build_time_seconds Service build time in seconds
# TYPE service_build_time_seconds gauge
service_build_time_seconds{service_name="$service"} $build_time

# HELP service_build_efficiency_ratio Build time per complexity point
# TYPE service_build_efficiency_ratio gauge
service_build_efficiency_ratio{service_name="$service"} $efficiency
EOF
    done
}

# Main execution
main() {
    echo "🚀 Starting service metrics collection..."
    echo "Timestamp: $(date)"
    echo "Prometheus Gateway: $PROMETHEUS_GATEWAY"
    
    collect_k8s_metrics
    collect_heroku_metrics  
    collect_build_metrics
    
    echo "✅ Service metrics collection completed"
}

# Run main function
main "$@"
```

## kubernetes/metrics_collector_deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-metrics-collector
  namespace: research-apps
  labels:
    app: custom-metrics-collector
    component: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: custom-metrics-collector
  template:
    metadata:
      labels:
        app: custom-metrics-collector
    spec:
      serviceAccountName: metrics-collector
      containers:
      - name: metrics-collector
        image: metrics-collector:latest
        imagePullPolicy: Always
        env:
        - name: PROMETHEUS_PUSHGATEWAY
          value: "prometheus-pushgateway.monitoring.svc.cluster.local:9091"
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-credentials
              key: token
        - name: HEROKU_API_KEY
          valueFrom:
            secretKeyRef:
              name: heroku-credentials  
              key: api-key
        - name: COLLECTION_INTERVAL
          value: "60"
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi" 
            cpu: "100m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
      restartPolicy: Always

---
apiVersion: v1
kind: Service
metadata:
  name: custom-metrics-collector
  namespace: research-apps
  labels:
    app: custom-metrics-collector
spec:
  selector:
    app: custom-metrics-collector
  ports:
  - name: http
    port: 8080
    targetPort: 8080
  type: ClusterIP

---
apiVersion: v1
kind: Secret
metadata:
  name: github-credentials
  namespace: research-apps
type: Opaque
stringData:
  token: "your-github-token-here"

---
apiVersion: v1
kind: Secret
metadata:
  name: heroku-credentials
  namespace: research-apps
type: Opaque
stringData:
  api-key: "your-heroku-api-key-here"
```

## scripts/heroku_metrics_collector.py
```python
#!/usr/bin/env python3
"""
Heroku Metrics Collector
Collects resource usage and performance metrics from Heroku platform
"""

import os
import requests
import json
import time
from datetime import datetime
from prometheus_client import CollectorRegistry, Gauge, push_to_gateway

class HerokuMetricsCollector:
    def __init__(self):
        self.registry = CollectorRegistry()
        self.heroku_api_key = os.getenv('HEROKU_API_KEY')
        self.prometheus_gateway = os.getenv('PROMETHEUS_PUSHGATEWAY', 'localhost:9091')
        
        if not self.heroku_api_key:
            raise ValueError("HEROKU_API_KEY environment variable required")
        
        # Prometheus metrics
        self.dyno_memory_usage = Gauge(
            'heroku_dyno_memory_usage_bytes',
            'Heroku dyno memory usage in bytes',
            ['app_name', 'dyno_type', 'dyno_name'],
            registry=self.registry
        )
        
        self.dyno_memory_quota = Gauge(
            'heroku_dyno_memory_quota_bytes', 
            'Heroku dyno memory quota in bytes',
            ['app_name', 'dyno_type'],
            registry=self.registry
        )
        
        self.dyno_status = Gauge(
            'heroku_dyno_status',
            'Heroku dyno status (1=up, 0=down)',
            ['app_name', 'dyno_type', 'dyno_name'],
            registry=self.registry
        )
        
        self.app_response_time = Gauge(
            'heroku_app_response_time_ms',
            'Application response time in milliseconds',
            ['app_name', 'endpoint'],
            registry=self.registry
        )

    def get_heroku_headers(self):
        """Get Heroku API headers"""
        return {
            'Authorization': f'Bearer {self.heroku_api_key}',
            'Accept': 'application/vnd.heroku+json; version=3',
            'Content-Type': 'application/json'
        }

    def get_app_dynos(self, app_name):
        """Get dyno information for an app"""
        url = f"https://api.heroku.com/apps/{app_name}/dynos"
        
        try:
            response = requests.get(url, headers=self.get_heroku_headers())
            response.raise_for_status()
            return response.json()
        except Exception as e:
            print(f"❌ Failed to get dynos for {app_name}: {e}")
            return []

    def get_app_metrics(self, app_name):
        """Get metrics for an app (if available)"""
        # Note: Heroku metrics API requires paid dynos
        # For Basic dynos, we simulate based on dashboard data
        
        # Simulated metrics based on Day 1 dashboard data
        app_metrics = {
            'ecommerce-product-service-56575270905a': {
                'memory_usage_mb': 79.6,
                'memory_quota_mb': 512,
                'response_time_ms': 31
            },
            'ecommerce-cart-service-f2a908c60d8a': {
                'memory_usage_mb': 470.8,
                'memory_quota_mb': 512,
                'response_time_ms': None  # Not available
            }
        }
        
        return app_metrics.get(app_name, {})

    def test_app_health(self, app_name):
        """Test application health and response time"""
        app_urls = {
            'ecommerce-product-service-56575270905a': 
                'https://ecommerce-product-service-56575270905a.herokuapp.com/health',
            'ecommerce-cart-service-f2a908c60d8a':
                'https://ecommerce-cart-service-f2a908c60d8a.herokuapp.com/health'
        }
        
        url = app_urls.get(app_name)
        if not url:
            return None
        
        try:
            start_time = time.time()
            response = requests.get(url, timeout=10)
            response_time_ms = (time.time() - start_time) * 1000
            
            if response.status_code == 200:
                return {
                    'status': 'healthy',
                    'response_time_ms': response_time_ms
                }
            else:
                return {
                    'status': 'unhealthy', 
                    'response_time_ms': response_time_ms
                }
        except Exception as e:
            print(f"❌ Health check failed for {app_name}: {e}")
            return {'status': 'unhealthy', 'response_time_ms': None}

    def collect_app_metrics(self, app_name):
        """Collect all metrics for a specific app"""
        print(f"📊 Collecting metrics for {app_name}...")
        
        # Get dyno information
        dynos = self.get_app_dynos(app_name)
        
        for dyno in dynos:
            dyno_name = dyno.get('name', 'unknown')
            dyno_type = dyno.get('type', 'unknown')
            state = dyno.get('state', 'unknown')
            
            # Dyno status (1=up, 0=down)
            status_value = 1 if state == 'up' else 0
            self.dyno_status.labels(
                app_name=app_name,
                dyno_type=dyno_type,
                dyno_name=dyno_name
            ).set(status_value)
        
        # Get app metrics (memory, performance)
        metrics = self.get_app_metrics(app_name)
        
        if 'memory_usage_mb' in metrics:
            self.dyno_memory_usage.labels(
                app_name=app_name,
                dyno_type='web',
                dyno_name='web.1'
            ).set(metrics['memory_usage_mb'] * 1024 * 1024)  # Convert to bytes
        
        if 'memory_quota_mb' in metrics:
            self.dyno_memory_quota.labels(
                app_name=app_name,
                dyno_type='web'
            ).set(metrics['memory_quota_mb'] * 1024 * 1024)  # Convert to bytes
        
        # Test application health and response time
        health = self.test_app_health(app_name)
        if health and health['response_time_ms']:
            self.app_response_time.labels(
                app_name=app_name,
                endpoint='/health'
            ).set(health['response_time_ms'])

    def collect_all_metrics(self):
        """Collect metrics for all monitored Heroku apps"""
        apps = [
            'ecommerce-product-service-56575270905a',
            'ecommerce-cart-service-f2a908c60d8a'
        ]
        
        for app in apps:
            self.collect_app_metrics(app)

    def push_metrics(self):
        """Push metrics to Prometheus Gateway"""
        try:
            push_to_gateway(
                self.prometheus_gateway,
                job='heroku-metrics-collector',
                registry=self.registry
            )
            print(f"✅ Heroku metrics pushed to {self.prometheus_gateway}")
        except Exception as e:
            print(f"❌ Failed to push Heroku metrics: {e}")

if __name__ == "__main__":
    collector = HerokuMetricsCollector()
    collector.collect_all_metrics()
    collector.push_metrics()
```

## python/requirements.txt
```
prometheus-client==0.17.1
requests==2.31.0
kubernetes==27.2.0
PyYAML==6.0.1
python-dotenv==1.0.0
click==8.1.7
schedule==1.2.0
```

## docker/Dockerfile
```dockerfile
FROM python:3.11-slim

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    kubectl \
    bc \
    jq \
    && rm -rf /var/lib/apt/lists/*

# Install Heroku CLI
RUN curl https://cli-assets.heroku.com/install.sh | sh

# Set working directory
WORKDIR /app

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY python/ ./
COPY scripts/ ./scripts/

# Make scripts executable
RUN chmod +x scripts/*.sh

# Create non-root user
RUN useradd -m -u 1000 metrics-collector
USER metrics-collector

# Expose port for health checks
EXPOSE 8080

# Set environment variables
ENV PYTHONPATH="/app"
ENV PROMETHEUS_PUSHGATEWAY="localhost:9091"

# Health check endpoint
COPY docker/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

## docker/entrypoint.sh
```bash
#!/bin/bash
set -euo pipefail

# Health check endpoint
python3 -c "
import http.server
import socketserver
import threading
import time

class HealthHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/health':
            self.send_response(200)
            self.send_header('Content-type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'OK')
        elif self.path == '/ready':
            self.send_response(200)
            self.send_header('Content-type', 'text/plain')
            self.end_headers()
            self.wfile.write(b'READY')
        else:
            self.send_response(404)
            self.end_headers()

    def log_message(self, format, *args):
        pass  # Suppress log messages

def start_health_server():
    with socketserver.TCPServer(('', 8080), HealthHandler) as httpd:
        httpd.serve_forever()

# Start health server in background
health_thread = threading.Thread(target=start_health_server, daemon=True)
health_thread.start()

print('Health server started on port 8080')

# Main metrics collection loop
while True:
    try:
        print(f'Running metrics collection at {time.strftime(\"%Y-%m-%d %H:%M:%S\")}')
        
        # Run collectors
        exec(open('complexity_metrics_collector.py').read())
        time.sleep(10)
        exec(open('build_performance_collector.py').read())
        time.sleep(10)
        exec(open('resource_efficiency_collector.py').read())
        time.sleep(10)
        exec(open('../scripts/heroku_metrics_collector.py').read())
        
        print('✅ Metrics collection cycle completed')
        
        # Wait for next collection cycle
        collection_interval = int(os.environ.get('COLLECTION_INTERVAL', '300'))
        time.sleep(collection_interval)
        
    except KeyboardInterrupt:
        print('Shutting down metrics collector...')
        break
    except Exception as e:
        print(f'❌ Error in metrics collection: {e}')
        time.sleep(60)  # Wait before retry
" &

# Keep container running
wait
```