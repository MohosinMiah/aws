helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus


helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana


apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana-prometheus-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
    alb.ingress.kubernetes.io/certificate-arn: <your-ssl-certificate-arn>
spec:
  rules:
    - http:
        paths:
          - path: /grafana
            backend:
              serviceName: grafana
              servicePort: 3000
          - path: /prometheus
            backend:
              serviceName: prometheus-server
              servicePort: 9090
