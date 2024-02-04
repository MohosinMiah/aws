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
https://github.com/stacksimplify/aws-eks-kubernetes-masterclass/blob/master/15-EKS-HPA-Horizontal-Pod-Autoscaler/README.md



