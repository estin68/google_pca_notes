### Solution for Implement Load Balancing on Compute Engine: Challenge Lab

##### Prerequisite
Run the following gcloud commands in Cloud Shell to set the default region and zone for your lab:
*Sample*:
```bash 
gcloud config set compute/zone "us-west3-b"
export ZONE=$(gcloud config get compute/zone)

gcloud config set compute/region "us-west3"
export REGION=$(gcloud config get compute/region)
```

#### Task 1. Create a project jumphost instance

###### Requirements

- Name the instance Instance name.
- Create the instance in the ZONE zone.
- Use an e2-micro machine type.
- Use the default image type (Debian Linux).

    ```bash
    gcloud config set compute/region us-west3
    ```

    ```bash
    gcloud config set compute/zone us-west3-b
    ```

    ```bash
    gcloud compute instances create nucleus-jumphost-712 \
    --machine-type=e2-micro \
    --zone=us-west3-b \
    --image-family=debian-11 \
    --image-project=debian-cloud
    ```

#### Task 2. Set up an HTTP load balancer

1. Create a startup.sh file

    ```bash
    cat << EOF > startup.sh
    #! /bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
    EOF
    ```

2. **Create an instance template**

    ```bash
    gcloud compute instance-templates create web-server-template \
        --metadata-from-file startup-script=startup.sh \
        --tags network-lb-tag \
        --machine-type e2-medium
        --global
    ```

3. **Create a managed instance group**

    ```bash
    gcloud compute instance-groups managed create web-server-group \
        --base-instance-name web-server \
        --size 2 \
        --template web-server-template \
        --zone us-west3-b
    ```

4. **Create a firewall rule to allow traffic (80/tcp)**

    ```bash
    gcloud compute firewall-rules create allow-tcp-rule-684 \
        --allow tcp:80 \
        --target-tags network-lb-tag
    ```

5. **Create a health check**

    ```bash
    gcloud compute http-health-checks create http-basic-check
    ```

6. **Create a backend service and attach the managed instance group**

    ```bash
    gcloud compute instance-groups managed set-named-ports web-server-group \
        --named-ports http:80

    gcloud compute backend-services create web-server-backend \
        --protocol HTTP \
        --port-name=http \
        --http-health-checks http-basic-check \
        --global

    gcloud compute backend-services add-backend web-server-backend \
        --instance-group web-server-group \
        --instance-group-zone=us-west3-b \
        --global
    ```

7. **Create a URL map and target HTTP proxy to route requests**

    ```bash
    gcloud compute url-maps create web-server-map \
        --default-service web-server-backend

    gcloud compute target-http-proxies create http-lb-proxy \
        --url-map web-server-map
    ```

8. **Create a forwarding rule**

    ```bash
    gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
    ```

9. **List forwarding rules**

    ```bash
    gcloud compute forwarding-rules list

    Sample ouput:
    NAME: http-content-rule
    REGION: 
    IP_ADDRESS: xx.x.xx.xxx
    IP_PROTOCOL: TCP
    TARGET: http-lb-proxy
    ```

10. **Wait for the website to be created behind the HTTP load balancer**

    Wait for approximately 5-7 minutes for the website to be created behind the HTTP load balancer.
    You can alternatively open browser tab to the IP xx.x.xx.xxx that you get from `forwarding-rules`

