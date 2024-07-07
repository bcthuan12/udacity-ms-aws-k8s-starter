# Coworking Space Service Extension

The Coworking Space Service is a set of APIs that enables users to request one-time tokens and administrators to authorize access to a coworking space. This service follows a microservice pattern and the APIs are split into distinct services that can be deployed and managed independently of one another.

For this project, you are a DevOps engineer who will be collaborating with a team that is building an API for business analysts. The API provides business analysts basic analytics data on user activity in the service. The application they provide you functions as expected locally and you are expected to help build a pipeline to deploy it in Kubernetes.

## Getting Started

### Dependencies

#### Local Environment

1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster
4. `helm` - apply Helm Charts to a Kubernetes cluster

#### Remote Resources

1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

### Setup

#### 1. Configure a Database

Set up a Postgres database.
Before you start, ensure you are connected to your K8s cluster

```bash
kubectl get namespace

```

1. Apply PersistentVolume

```bash
kubectl apply -f postgres-pv.yaml
```

2. Apply PersistentVolumeClaim

```bash
kubectl apply -f postgres-pvc.yaml
```

3. Apply Postgres Deployment

```bash
kubectl apply -f postgres-deploy.yaml
```

4. Apply Postgres Service

```bash
kubectl apply -f postgres-service.yaml
```

This should set up a Postgre deployment at `<SERVICE_NAME>-postgresql.default.svc.cluster.local` in your Kubernetes cluster. You can verify it by running `kubectl svc`

The password can be retrieved with the following command:

```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default <SERVICE_NAME>-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

echo $POSTGRES_PASSWORD
```

5. Test Database Connection

- Connecting Via Port Forwarding

```bash
kubectl port-forward service/postgresql-service 5433:5432 &
```

6. Run Seed Files
   We will need to run the seed files in `db/` in order to create the tables and populate them with data.

```bash
export DB_PASSWORD=mypassword
PGPASSWORD="$DB_PASSWORD" psql --host 127.0.0.1 -U myuser -d mydatabase -p 5433 < <FILE_NAME.sql>
```

### 2. Running the Analytics Application Locally

In the `analytics/` directory:

1. Install dependencies

```bash
pip install -r requirements.txt
```

2. Run the application (see below regarding environment variables)

```bash
<ENV_VARS> python app.py
```

There are multiple ways to set environment variables in a command. They can be set per session by running `export KEY=VAL` in the command line or they can be prepended into your command.

- `DB_USERNAME`
- `DB_PASSWORD`
- `DB_HOST` (defaults to `127.0.0.1`)
- `DB_PORT` (defaults to `5432`)
- `DB_NAME` (defaults to `postgres`)

If we set the environment variables by prepending them, it would look like the following:

```bash
DB_USERNAME=username_here DB_PASSWORD=password_here python app.py
```

The benefit here is that it's explicitly set. However, note that the `DB_PASSWORD` value is now recorded in the session's history in plaintext. There are several ways to work around this including setting environment variables in a file and sourcing them in a terminal session.

3. Verifying The Application

- Generate report for check-ins grouped by dates
  `curl <BASE_URL>/api/reports/daily_usage`

- Generate report for check-ins grouped by users
  `curl <BASE_URL>/api/reports/user_visits`
