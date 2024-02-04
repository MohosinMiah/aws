Certainly! Below is a simplified example of deploying Prometheus and Grafana on AWS EKS using Helm charts, with persistent volume using EBS and an ALB Ingress Controller.

1. **Install AWS ALB Ingress Controller**:
   - Follow the official AWS documentation to install the ALB Ingress Controller in your EKS cluster.

2. **Create PersistentVolumeClaim YAML files**:
   - Create two files, `prometheus-pvc.yaml` and `grafana-pvc.yaml`, to define PersistentVolumeClaims for Prometheus and Grafana respectively:

**prometheus-pvc.yaml**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: aws-ebs
```

**grafana-pvc.yaml**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: aws-ebs
```

3. **Install Prometheus using Helm**:
   - Add Prometheus Helm repository:
     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     ```
   - Install Prometheus using Helm:
     ```bash
     helm install prometheus prometheus-community/prometheus --set server.persistentVolume.enabled=true --set server.persistentVolume.storageClass=aws-ebs -f prometheus-pvc.yaml
     ```

4. **Install Grafana using Helm**:
   - Add Grafana Helm repository:
     ```bash
     helm repo add grafana https://grafana.github.io/helm-charts
     ```
   - Install Grafana using Helm:
     ```bash
     helm install grafana grafana/grafana --set persistence.enabled=true --set persistence.storageClassName=aws-ebs --set persistence.size=5Gi -f grafana-pvc.yaml
     ```

5. **Expose Grafana and Prometheus using AWS ALB Ingress**:
   - Create Ingress resources for both Grafana and Prometheus to expose them externally via ALB.

Here's a simple example of how the Ingress resources might look:

**grafana-ingress.yaml**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: grafana
                port:
                  number: 3000
```

**prometheus-ingress.yaml**:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prometheus-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: prometheus-server
                port:
                  number: 9090
```
Apply these Ingress resources using `kubectl apply -f <filename>`.

This example provides a basic setup. Depending on your specific requirements and configurations, you may need to adjust the configurations accordingly. Make sure you have configured proper security groups and IAM roles for accessing resources within your AWS environment.


#  Connect to RDS Database using kubectl and create usermgmt schema/db
https://github.com/stacksimplify/aws-eks-kubernetes-masterclass/blob/master/06-EKS-Storage-with-RDS-Database/README.md

```
kubectl run -it --rm --image=mysql:5.7.22 --restart=Never mysql-client -- mysql -h usermgmtdb.c7hldelt9xfp.us-east-1.rds.amazonaws.com -u dbadmin -pdbpassword11

mysql> show schemas;
mysql> create database usermgmt;
mysql> show schemas;
mysql> exit
```

# EKS - Horizontal Pod Autoscaling (HPA)

Sure, I'll provide you with a full example of setting up Horizontal Pod Autoscaling (HPA) in an AWS EKS cluster deployed in a private subnet. 

First, let's set up the HPA:

1. **Deploy a sample application**:
   We'll deploy a sample application that we can scale horizontally. Here's a simple Deployment manifest for a sample Nginx web server:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Apply this Deployment using `kubectl apply -f nginx-deployment.yaml`.

2. **Enable metrics server**:
   Horizontal Pod Autoscaler requires metrics from the Metrics Server. Deploy Metrics Server in your cluster:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

3. **Create HorizontalPodAutoscaler**:
   Next, create a HorizontalPodAutoscaler to scale the Nginx Deployment based on CPU utilization:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
```

This HPA will scale the `nginx-deployment` Deployment based on CPU utilization, targeting an average CPU utilization of 50%. It will scale between 2 and 10 replicas.

Apply the HPA using `kubectl apply -f nginx-hpa.yaml`.

4. **Test the scaling**:
   To test the scaling, you can generate load on the Nginx Deployment. One simple way to generate load is by using a load testing tool like `hey` or `wrk`.

For example, you can use `hey` to send a large number of requests to the Nginx service:

```bash
hey -z 1m -c 10 http://<nginx-service-ip>
```

This command sends requests to the Nginx service endpoint for 1 minute with a concurrency of 10 clients.

5. **Observe scaling**:
   Monitor the scaling behavior by checking the number of replicas in the Deployment and the HPA status:

```bash
kubectl get deployment nginx-deployment
kubectl get hpa nginx-hpa
```

You should see the number of replicas in the Deployment increasing or decreasing based on the load, and the HPA scaling up or down accordingly.

This example demonstrates the basic setup of Horizontal Pod Autoscaling in an AWS EKS cluster deployed in a private subnet. You can adjust the configurations and test with different workloads to observe how the autoscaling behavior adapts to changing demand.

https://github.com/stacksimplify/aws-eks-kubernetes-masterclass/blob/master/15-EKS-HPA-Horizontal-Pod-Autoscaler/README.md



