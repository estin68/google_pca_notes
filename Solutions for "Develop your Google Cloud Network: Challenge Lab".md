### Solutions for Develop your Google Cloud Network: Challenge Lab

#### Task 1. Create development VPC manually

##### Solutions for Task 1

```bash
gcloud config set compute/zone "us-central1-c"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "us-central1"
export REGION=$(gcloud config get compute/region)

export MY_DB_PW=$(gcloud config list --format 'value(core.project)')
# Or you can just use $GOOGLE_CLOUD_PROJECT as DB password
```

```bash
gcloud compute networks create griffin-dev-vpc --subnet-mode custom
```

```bash
gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region $REGION --range=192.168.16.0/20

gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region $REGION --range=192.168.32.0/20
```

#### Task  2. Create production VPC manually

##### Solutions for Task 2

```bash
gsutil cp -r gs://cloud-training/gsp321/dm .

cd dm

sed -i s/SET_REGION/$REGION/g prod-network.yaml

gcloud deployment-manager deployments create prod-network \
--config=prod-network.yaml

cd ..
```

#### Task 3. Create bastion host

##### Solutions for Task 3

```bash
gcloud compute instances create bastion \
    --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt \
    --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt \
    --tags=ssh \
    --zone=$ZONE

gcloud compute firewall-rules create fw-ssh-dev \
    --source-ranges=0.0.0.0/0 \
    --target-tags=ssh \
    --allow=tcp:22 \
    --network=griffin-dev-vpc

gcloud compute firewall-rules create fw-ssh-prod \
    --source-ranges=0.0.0.0/0 \
    --target-tags=ssh \
    --allow=tcp:22 \
    --network=griffin-prod-vpc
```

#### Task 4. Create and configure Cloud SQL Instance

##### Solutions for Task 4

```bash
gcloud sql instances create griffin-dev-db \
    --region=$REGION \
    --root-password=$MY_DB_PW

gcloud sql connect griffin-dev-db

# Run the following SQL commands WordPress
CREATE DATABASE wordpress;
CREATE USER "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%";
FLUSH PRIVILEGES;

exit;
```

#### Task 5. Create Kubernetes cluster

##### Solutions for Task 5

```bash
gcloud container clusters create griffin-dev \
 --network griffin-dev-vpc \
 --subnetwork griffin-dev-wp \
 --machine-type e2-standard-4 \
 --num-nodes 2 \
 --zone $ZONE
```

#### Task 6. Prepare the Kubernetes cluster

##### Solutions for Task 6

```bash
gcloud container clusters get-credentials griffin-dev --zone $ZONE
```

```bash
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .

cd wp-k8s
```

```bash
sed -i -e 's/username_goes_here/wp_user/g' -e 's/password_goes_here/stormwind_rules/g' wp-env.yaml
# Verify the username and password is replaced
cat wp-env.yaml 
```

```bash
kubectl create -f wp-env.yaml
```

```bash
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

#### Task 7. Create a WordPress deployment

##### Solutions for Task 7

```bash
INSTANCE_ID=$(gcloud sql instances describe griffin-dev-db --format="value(connectionName)")

sed -i s/YOUR_SQL_INSTANCE/$INSTANCE_ID/g wp-deployment.yaml

kubectl create -f wp-deployment.yaml

kubectl create -f wp-service.yaml
```

#### Task 8. Enable monitoring 

##### Solutions for Task 8

```bash
WORDPRESS_EXTERNAL_IP=$(kubectl get service wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Verify the IP address
echo $WORDPRESS_EXTERNAL_IP

gcloud monitoring uptime create quicklab \
    --resource-type=uptime-url \
    --resource-labels=host=$WORDPRESS_EXTERNAL_IP
```

#### Task 9. Provide access for an additional engineer

##### Solutions for Task 9

```bash
# Replace the username 2
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
  --member="user:<username 2>" \
  --role="roles/editor"
```
