## Deploy Airflow (Kubernetes Executor) using Rancher Fleet for Bytewax Pipelines
![Diagram](https://airflow.apache.org/docs/apache-airflow/stable/_images/arch-diag-kubernetes.png)

`helm repo add airflow-stable https://airflow-helm.github.io/charts`

`helm repo update`

## Create namespace
`kubectl create ns pipelines`

## Generate public and private keys for git sync provider 
`ssh-keygen -t rsa -b 4096 -C "your-mail@gmail.com"`
## Add the public key to repository settings deploy keys

## Build custom docker image with bytewax dependencies in pipelines repository
`docker build -f .\Airflow -t airflow:bytewax-2.7.3 .`

## Configure gmail SMTP for email on failure alert

## In gmail settings select Forwarding and POP/IMAP tab and update status to enable in IMAP access 
## Enable 2-Factor authentication for your gmail account
## visit `https://myaccount.google.com/apppasswords` to create a secure password

## Configure sendgrid SMTP for email on failure alert

## Create a account at `https://sendgrid.com
## In dashboard go to settings select sender authentication verify a single sender if status is not verified 
## In Email Api select Integration guide and choose SMTP relay and generate API key
## airflow-smtp-smtp-user secret value is 'apikey' for sendgrid SMTP

## Configure external postgres database
```
CREATE DATABASE airflow_postgres_db;
CREATE USER airflow_postgres_user WITH PASSWORD 'airflow_postgres_password';
GRANT ALL PRIVILEGES ON DATABASE airflow_postgres_db TO airflow_postgres_user;
```
## Connect to airflow_db to run query;
```
GRANT ALL ON SCHEMA public TO airflow_postgres_user;
```
## Create postgres password as a secret
## Update host, port, database, user in values.yaml at airflow.externalDatabase

## Configure external redis
## Update host, port, databaseNumber in values.yaml at airflow.externalRedis

## Create Secrets
```
kubectl create secret generic airflow --from-file=airflow-ssh-git-secret=~/.ssh/airflow_ssh_key \
									  --from-literal="airflow-core-fernet-key=uuid" \
									  --from-literal="airflow-webserver-secret-key=uuid" \
									  --from-literal="airflow-postgres-password=airflow_postgres_password" \
									  --from-literal="airflow-redis-password=" \
									  --from-literal="airflow-smtp-smtp-mail-from=example@gmail.com" \
									  --from-literal="airflow-smtp-smtp-user=apikey" \
									  --from-literal="airflow-smtp-smtp-password=password" \
									  --from-literal="redis-password=" \
									  --from-literal="postgres-password=password" \
									  --from-literal="clickhouse-password=password" \
									  --from-literal="wod-auth=username:password" --namespace pipelines
```
## Deploy using fleet
```
kubectl apply -f .\deploy.yaml
```
## View dashboard
`kubectl port-forward svc/airflow-web 8080:8080 --namespace pipelines

