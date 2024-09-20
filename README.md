# Udacity project 3 - Cloud Devops

## Getting Started

### Step 1. Create an EKS cluster & update config
```bash
eksctl create cluster --name project3-cluster --region us-east-1 --nodegroup-name project3-nodes --node-type m6g.large --nodes 1 --nodes-min 1 --nodes-max 2

aws eks --region us-east-1 update-kubeconfig --name project3-cluster
kubectl config current-context
```
#### View config eks
```bash
kubectl config view
```

### Step 3. Configure ENV variable and Secrets
```bash
kubectl apply -f deployment/configmap.yaml
kubectl apply -f deployment/secrets.yaml

```

### Step 4. Deploy Database
#### Step 4.1. deploy
```bash
kubectl apply -f deployment/pv.yaml
kubectl apply -f deployment/pvc.yaml
kubectl apply -f deployment/postgresql-deployment.yaml
kubectl apply -f deployment/postgresql-service.yaml
```
#### Step 4.2. forward port & run seed file in db
```bash
kubectl port-forward service/postgresql-service 5433:5432 &
export DB_PASSWORD=mypassword
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < ./db/1_create_tables.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < ./db/2_seed_users.sql
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < ./db/3_seed_tokens.sql
```

### Step 5. Deploy application
```bash
export DB_PASSWORD=`kubectl get secret project3-secrets -o jsonpath='{.data.password}' | base64 --decode`
export DB_USERNAME=`kubectl get configMap project3-config-map -o jsonpath='{.data.DB_USERNAME}'`
export DB_NAME=`kubectl get configMap project3-config-map -o jsonpath='{.data.DB_NAME}'`
kubectl apply -f deployment/coworking.yaml
```
#### Step 5.1 Check pod deployment, service
```bash
kubectl get pods
kubectl get service
```
#### Step 5.2 Attach to cloudwatch
```bash
aws iam attach-role-policy \
--role-name eksctl-project3-cluster-nodegroup--NodeInstanceRole-DctCC7c22uLg \
--policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name project3-cluster

```
