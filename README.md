# Install and Setup Minikube (MacOS) 
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
minikube start # docker should be running

# Create app1 and app2 deployments
kubectl create -f app1-deployment.yaml
kubectl create -f app2-deployments.yaml

# Create app1 and app2 Services
kubectl create -f app1-services.yaml
kubectl create -f app2-services.yaml

# Install NGINX ingress controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Apply ingress configuration 
kubectl apply -f ingress.yaml

# Enable Minikube Ingress Add-on
minikube addons enable ingress

# Cmd to check ingress-nginx-controller pod name
kubectl get pods -n ingress-nginx

# Enable port forwarding 
kubectl port-forward pod/{ingress-nginx-controller-POD_NAME} 8080:80 -n ingress-nginx

# Add rate-limiting configuration
kubectl edit configmap -n ingress-nginx ingress-nginx-controller 

data:
  log-format: '$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$request_time" "$upstream_response_time" "$http_x_client_id"';
  limit-req-status-code: "429" 

# Access the endpoints using curl or a browser:
http://localhost:8080/v1 → Should route to app1.
http://localhost:8080/v2 → Should route to app2.
http://localhost:8080/random → Should return a 404.  

# Verify Rate-limiting
for i in {1..10}; do curl http://localhost:8080/v1; done   -> Should be return "429 Too Many Requests" in some requests. 
for i in {1..10}; do curl http://localhost:8080/v2; done   -> Should be retrun "429 Too Many Requests" in some requests.      

