# kuber
info about kuber

# Port Forwarding
kubectl port-forward svc/argocd-server -n argocd 8080:443


# Get Value from chart
helm upgrade -i my-release oci://ghcr.io/nginxinc/charts/nginx-ingress -f vaule.yml



# add ip to load balancer
kubectl patch svc traefik -p '{"spec":{"externalIPs":["192.168.0.194"]}}'





---------------
# get https with traefik
source : https://www.fosstechnix.com/kubernetes-traefik-ingress-letsencrypt/
## 1: Install Helm 3 on Kubernetes Cluster
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

## 2: Install Traefik Ingress Controller on Kubernetes using Helm 3

```
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik
```

## 3. Creating Deployment and service for nginx app
```
sudo nano nginx-deploy.yml
```

```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-app
  namespace: default
  labels:
    app: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: "nginx"
```
after this
```
sudo nano nginx-svc.yml
```
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
  namespace: default
spec:
  selector:
    app: nginx-app
  ports:
  - name: http
    targetPort: 80
    port: 80
```
after this
```
kubectl create -f nginx-deploy.yml
kubectl create -f nginx-svc.yml
```
---------------------------------------------------------
## 4. Creating Traefik Ingress Resources and Exposing the apps
```
sudo nano traefik-ingress.yml
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traefik-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: nginxapp.fosstechnix.info
    http:
      paths:
      - backend:
          service:
            name: nginx-app
            port:
              number: 80
        path: /
        pathType: Prefix
```

```
kubectl create -f traefik-ingress.yml   
```
## 5. Pointing Traefik Ingress Loadbalancer in Domain Name provider
---------------------------------------------------------

## 6: Configure cert manager for Traefik Ingress
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.crds.yaml
```
```
helm install   cert-manager jetstack/cert-manager   --namespace cert-manager   --create-namespace   --version v1.12.0
```

## 7: Kubernetes Traefik Ingress LetsEncrypt
```
sudo nano  letsencrypt-issuer.yml
```

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: default
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: fosstechnixinfo@gmail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: traefik
```
```
kubectl apply -f letsencrypt-issuer.yml
```
## 8: Creating Traefik Ingress Let’s Encrypt TLS Certificate
```
sudo nano letsencrypt-cert.yml
```

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: nginxapp.fosstechnix.info
  namespace: default
spec:
  secretName: nginxapp.fosstechnix.info-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: nginxapp.fosstechnix.info
  dnsNames:
  - nginxapp.fosstechnix.info
```

```
kubectl apply -f letsencrypt-cert.yml
```


## 9: Point Traefik Ingress Let’s Encrypt Certificate in Traefik Ingress Resource

```
kubectl edit ingress traefik-ingress
```

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    kubernetes.io/ingress.class: traefik
  creationTimestamp: "2021-04-22T03:20:24Z"
  generation: 2
  name: traefik-ingress
  namespace: default
  resourceVersion: "5902"
  uid: 62300582-7b91-4f56-a229-75f9664f9334
spec:
  rules:
  - host: nginxapp.fosstechnix.info
    http:
      paths:
      - backend:
          service:
            name: nginx-app
            port:
              number: 80
        path: /
        pathType: Prefix
  tls:
  - hosts:
    - nginxapp.fosstechnix.info
    secretName: nginxapp.fosstechnix.info-tls
status:
  loadBalancer:
    ingress:
    - hostname: afbcd905b614842c29702ddd7481784d-1960968212.ap-south-1.elb.amazonaws.com
```



## 10: Install Elk With Helm and Update Kibana Dashboard 
```
helm repo add opensearch https://opensearch-project.github.io/helm-charts/
helm install opensearch opensearch/opensearch -f values.yaml 
helm install dashboard opensearch/opensearch-dashboards -f dash.yml
```

for update and remove all kibana index for dashboard
```
curl -u admin:admin -k -XDELETE "https://localhost:9200/.kibana_1"

```
