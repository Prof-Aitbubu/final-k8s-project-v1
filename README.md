                    *** "Introduction" ***
To successfully set up this project, familiarize yourself with its 10 main components:

1)Setting up the CloudSQL Instance
2)Cloning the Repositories
3)Creating Secrets for Database Access
4)Creating Dockerfiles for Backend and Frontend
5)Pushing Images to Google Container Registry (GCR)
6)Writing YAML Files for Deployments
7)Creating ClusterIP Services
8)Installing Nginx Controller and Creating Ingress
9)Configuring Domain Name with Cloud DNS and ExternalDNS
10)Obtaining a Certificate for Your Domain with Cert-Manager

                    *** Part I - "Setting up the CloudSQL Instance" ***
Ensure your Ubuntu system has "mysql-client" installed:

sudo apt update
sudo apt install mysql-client

Follow these steps:

1)Set your default project:
gcloud config set project PROJECT_ID

2)Enable the SQL Admin API:
gcloud services enable sqladmin.googleapis.com

3)Connect to your Cloud SQL instance:
gcloud sql connect INSTANCE_NAME --user=USERNAME

4) Inside your database, execute:
a) \dt to list tables
b)Copy data from "PSQL.txt" in "awesome_cats_backend"

                    *** Part II - "Cloning the Repositories" ***
Clone the backend and frontend repositories:
git clone https://github.com/AntTechLabs/awesome_cats_backend.git
git clone https://github.com/AntTechLabs/awesome_cats_frontend.git


                    *** Part III - "Creating a Secret with Hostname, Username, Password, and Database Name" ***
Use Kubernetes Secrets to securely manage sensitive information:

apiVersion: v1
kind: Secret
metadata:
  name: SECRET_NAME
type: Opaque
data:
  host: BASE64_ENCODED_HOST
  username: BASE64_ENCODED_USERNAME
  password: BASE64_ENCODED_PASSWORD
  dbname: BASE64_ENCODED_DBNAME

                  *** Part IV - "Dockerfile for Frontend and Backend Images" ***
Create Dockerfiles for your Node.js applications:

For frontend:

FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]

For backend (same structure as frontend)

                  *** Part V - "Pushing Images to Private Repositories in GCR" ***
Prepare Docker and GCR:

sudo systemctl restart docker
sudo docker info
gcloud auth configure-docker

Adjust and push your images to GCR.
docker build -t awesome_cats_backend .
docker tag awesome_cats_backend:latest <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/awesome_cats_backend:latest
docker push <your-account-id>.dkr.ecr.<your-region>.amazonaws.com/awesome_cats_backend:latest

                  *** Part VI - "Writing YAML Files for Deployments" ***
Create deployment YAML files for backend and frontend, specifying replicas and environment variables.

a) Backend Deployment
Create backend-deployment.yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: <your-backend-image>
        ports:
        - containerPort: 3000
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: host
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: DB_NAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: dbname
b)Frontend Deployment
Create frontend-deployment.yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: <your-frontend-image>
        ports:
        - containerPort: 3000

                      *** Part VII - "Creating ClusterIP Services" ***
Define ClusterIP services for backend and frontend deployments.
a)Backend Service
Create backend-service.yaml:
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
b) Frontend Service
Create frontend-service.yaml:
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP

                        *** Part VIII - "Installing Nginx Controller and Creating Ingress" ***
a) Install Nginx Ingress Controller and configure ingress for routing.

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

b) Create Ingress:
Create ingress.yaml:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: awesome-cats-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: <your-domain-name>
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

                                  *** Part IX - "Configuring the Domain Name with Cloud DNS and ExternalDNS" ***
Set up Cloud DNS Zone, install ExternalDNS, and create ingress.yaml for routing.
a)Create a hosted zone for your domain.
b)Install ExternalDNS:
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install external-dns bitnami/external-dns --set provider=google --set google.project=<your-project-id> --set domainFilters[0]=<your-domain-name> --set rbac.create=true

                              *** Part X - "Obtaining a Certificate for Your Domain with Cert-Manager" ***
Install Cert-Manager, create Issuer or ClusterIssuer YAML, and generate a certificate for your domain.

a)Install Cert-Manager:
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.0/cert-manager.yaml

b)Create Issuer or ClusterIssuer:
Create a file issuer.yaml:
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: <your-email>
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

                                            *** Conclusion ***
These adjustments aim to streamline the flow and clarity of your setup instructions. Adjustments have been made for clarity and logical flow, ensuring each step is clear and actionable.

Questions?
If you have any questions or need further assistance, please feel free to contact me via abdyibraeva.cheber@gmail.com 

Thanks for Setting Up Awesome Cats!
Thank you for using this guide to set up the Awesome Cats application. Happy coding!😊
