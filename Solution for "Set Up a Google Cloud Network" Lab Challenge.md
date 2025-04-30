### Solution for Set Up a Google Cloud Network: Challenge Lab

#### Task 1. Create networks

##### Requirements

1. Create a VPC network named `vpc-network-0s3h` with two subnets: `subnet-a-xrbk` and `subnet-b-puzt`. Use a **Regional** dynamic routing mode.
2. For `subnet-a-xrbk` set the region to `us-east1`.
   - Set the **IP stack type** to **IPv4 (single-stack)**
   - Set IPv4 range to `10.10.10.0/24`
3. For `subnet-b-puzt` set the region to `us-east4`.
   - Set the **IP stack type** to **IPv4 (single-stack)**
   - Set IPv4 range to `10.10.20.0/24`

###### Solutions for Task 1

1. Create a VPC network

    ```bash
    gcloud compute networks create vpc-network-0s3h --subnet-mode custom 
    ```

2. Create subnets

    ```bash
    gcloud compute networks subnets create subnet-a-xrbk \
    --network vpc-network-0s3h \
    --region us-east1 \
    --range 10.10.10.0/24
    ```

3. Create subnets

    ```bash
    gcloud compute networks subnets create subnet-b-puzt \
    --network vpc-network-0s3h \
    --region us-east4 \
    --range 10.10.20.0/24
    ```

#### Task 2. Add firewall rules

##### Requirements for Task 2

1. Create a firewall rule named rljc-firewall-ssh.
   - For the network, use vpc-network-0s3h.
   - Set the priority to 1000, the traffic to Ingress and action to Allow
   - The targets should be set to all instances in the network and the IPv4 ranges to 0.0.0.0/0
   - Set the Protocol to TCP and port to 22

2. Create a firewall rule named uvhx-firewall-rdp.
   - For the network, use vpc-network-0s3h.
   - Set the priority to 65535, the traffic to Ingress and action to Allow
   - The targets should be set to all instances in the network and the IPv4 ranges to 0.0.0.0/24
   - Set the Protocol to TCP and port to 3389

3. Create a firewall rule named vtyg-firewall-icmp.
   - For the network, use vpc-network-0s3h.
   - Set the priority to 1000, the traffic to Ingress and action to Allow
   - The targets should be set to all instances in the network and the IPv4 ranges to 10.10.10.0/24 and 10.10.20.0/24
   - Set the Protocol to icmp

###### Solutions for Task 2

1. Create a firewall ssh

    ```bash
    gcloud compute firewall-rules create "rljc-firewall-ssh" \
    --allow tcp:22 \
    --network "vpc-network-0s3h" \
    --source-ranges "0.0.0.0/0" \
    --direction=INGRESS \
    --priority 1000
    ```

2. Create a firewall rdp

    ```bash
    gcloud compute firewall-rules create "uvhx-firewall-rdp" \
    --allow tcp:3389 \
    --network "vpc-network-0s3h" \
    --source-ranges "0.0.0.0/24" \
    --priority 65535
    ```

3. Create a firewall icmp

    ```bash
    gcloud compute firewall-rules create "vtyg-firewall-icmp" \
    --allow icmp \
    --network "vpc-network-0s3h" \
    --source-ranges "10.10.10.0/24","10.10.20.0/24" \
    --priority 1000
    ```

#### Task 3. Add VMs to your network

##### Requirements for Task 3

1. Create an instance name `us-test-01` in `subnet-a-xrbk` and set the zone to `us-east1-d`.

2. Create an instance name `us-test-02` in `subnet-b-puzt` and set the zone to `us-east4-c`.

##### Solutions for Task 3

1. Create an instance

    ```bash
    gcloud compute instances create us-test-01 \
    --subnet subnet-a-xrbk \
    --zone us-east1-d
    ```

2. Create an instance

    ```bash
    gcloud compute instances create us-test-02 \
    --subnet subnet-b-puzt \
    --zone us-east4-c
    ```
