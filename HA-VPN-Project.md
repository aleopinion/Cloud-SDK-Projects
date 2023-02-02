**To list your projet and to Confirm that your project has been setup properly**
```
gcloud config list project
```
**GOOGLE CLOUD VPC NETWORK SETUP/CREATION**
```
gcloud compute networks create vpc-demo --subnet-mode custom
```

**TO CREATE SUBNET#1 WITHIN YOUR CLOUD VPC NETWORK IN A PARTICULAR REGION** 
```
gcloud beta compute networks subnets create vpc-demo-subnet1 \
--network vpc-demo --range 10.1.1.0/24 --region us-central1
```

**TO CREATE SUBNET#2 WITHIN YOUR CLOUD VPC NETWORK IN A PARTICULAR REGION** 
```
gcloud beta compute networks subnets create vpc-demo-subnet2 \
--network vpc-demo --range 10.2.1.0/24 --region us-east1
```

**TO CREATE FIREWALL RULE WITHIN YOUR CLOUD VPC NETWORK vpc-demo** 
```
gcloud compute firewall-rules create vpc-demo-allow-internal \
  --network vpc-demo \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 10.0.0.0/8
  ```

**TO CREATE FIREWALL RULE WITHIN YOUR CLOUD VPC NETWORK vpc-demo** 
``` 
gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
    --network vpc-demo \
    --allow tcp:22,icmp
```

**ON-PREMISE VPC NETWORK SETUP/CREATION**
```
gcloud compute networks create on-prem --subnet-mode custom
```

**TO CREATE SUBNET WITHIN YOUR ON-PREM VPC NETWORK IN A PARTICULAR REGION**
```
gcloud beta compute networks subnets create on-prem-subnet1 \
--network on-prem --range 192.168.1.0/24 --region us-central1

**TO CREATE FIREWALL RULE WITHIN YOUR CLOUD ON-PREM VPC NETWORK on-prem** 
gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
    --network on-prem \
    --allow tcp:22,icmp\
    --source-ranges 104.48.110.51
```

**HA-VPN GATEWAY CREATION IN THE CLOUD NETWORK**
```
gcloud beta compute vpn-gateways create vpc-demo-vpn-gw1 --network vpc-demo --region us-central1
```

**TO DESCRIBE THE ATTRIBUTES OF THE GATEWAY JUST CREATED**
```
gcloud beta compute vpn-gateways describe vpc-demo-vpn-gw1 --region us-central1
```

**CLOUD ROUTER CREATION**
```
gcloud compute routers create vpc-demo-router1 \
    --region us-central1 \
    --network vpc-demo \
    --asn 65001
```

**CREATING THE ON-PREMISE ROUTER**
```
gcloud compute routers create on-prem-router1 \
    --region us-central1 \
    --network on-prem \
    --asn 65002
```

**CREATING ON-PREMISE GATEWAY**
```
gcloud beta compute vpn-gateways create on-prem-vpn-gw1 --network on-prem --region us-central1
```

**ESTABLISHING THE FIRST VPN TUNNEL FOR THE CLOUD NETWORK**
```
gcloud beta compute vpn-tunnels create vpc-demo-tunnel0 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 0
```

**ESTABLISHING THE SECOND VPN TUNNEL FOR THE CLOUD NETWORK**
```
gcloud beta compute vpn-tunnels create vpc-demo-tunnel1 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 1
```

**ESTABLISHING THE FIRST VPN TUNNEL FOR THE ON-PREM NETWORK**
```
gcloud beta compute vpn-tunnels create on-prem-tunnel0 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 0
```

**ESTABLISHING THE SECOND  VPN TUNNEL FOR THE ON-PREM NETWORK**
```
gcloud beta compute vpn-tunnels create on-prem-tunnel1 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region us-central1 \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 1
```

**TO ESTABLISH THE INTERFACE FOR BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 0** CLOUD => Enables you to establish the interface for cloud router
```
gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel0-to-on-prem \
    --ip-address 169.254.0.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel0 \
    --region us-central1
```

**TO ESTABLISH BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 0** CLOUD
```
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel0 \
    --interface if-tunnel0-to-on-prem \
    --peer-ip-address 169.254.0.2 \
    --peer-asn 65002 \
    --region us-central1
```

**TO ESTABLISH THE INTERFACE BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 1** CLOUD => Enables you to establish the interface for cloud router
```
gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel1-to-on-prem \
    --ip-address 169.254.1.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel1 \
    --region us-central1
```

**TO ESTABLISH BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 1**
```
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel1 \
    --interface if-tunnel1-to-on-prem \
    --peer-ip-address 169.254.1.2 \
    --peer-asn 65002 \
    --region us-central1
```

**TO ESTABLISH THE INTERFACE FOR BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 0** ON-PREM => Enables you to establish the interface for on-prem router
```
gcloud compute routers add-interface on-prem-router1 \
    --interface-name if-tunnel0-to-vpc-demo \
    --ip-address 169.254.0.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel0 \
    --region us-central1
```

**TO ESTABLISH BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 0** ON-PREM
```
gcloud compute routers add-bgp-peer on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel0 \
    --interface if-tunnel0-to-vpc-demo \
    --peer-ip-address 169.254.0.1 \
    --peer-asn 65001 \
    --region us-central1
```


**TO ESTABLISH THE INTERFACE FOR BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 1** ON-PREM => Enables you to establish the interface for on-prem router
```
gcloud compute routers add-interface  on-prem-router1 \
    --interface-name if-tunnel1-to-vpc-demo \
    --ip-address 169.254.1.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel1 \
    --region us-central1
```

**TO ESTABLISH BORDER GATEWAY PROTOCOL (BGP) FOR TUNNEL 1** ON-PREM
```
gcloud compute routers add-bgp-peer  on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel1 \
    --interface if-tunnel1-to-vpc-demo \
    --peer-ip-address 169.254.1.1 \
    --peer-asn 65001 \
    --region us-central1
```

**TO DESCRIBE THE CLOUD ROUTER**
```
gcloud compute routers describe vpc-demo-router1 \
    --region us-central1
```

**FIREWALL RULE PROVIDE ACCESS FROM ON-PREMISE NETWROK TO CLOUD NETWORK**
```
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
    --network vpc-demo \
    --allow tcp,udp,icmp \
    --source-ranges 192.168.1.0/24
```

**FIREWALL RULE TO PROVIDE ACCESS FROM CLOUD NETWORK TO ON-PREMISE NETWORK**
```
gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
    --network on-prem \
    --allow tcp,udp,icmp \
    --source-ranges 10.1.1.0/24,10.2.1.0/24
```

**TO LIST THE TUNNELS** => ALL YOUR PEERINGS, ESTABLISHMENTS,
```
gcloud beta compute vpn-tunnels list
```

**TO VERIFY THE DETAILS OF THE 1ST CLOUD TUNNEL** => detailedStatus: Tunnel is up and running = OK
```
gcloud beta compute vpn-tunnels describe vpc-demo-tunnel0 \
      --region us-central1
```

**TO VERIFY THE DETAILS OF THE 2ND CLOUD TUNNEL** => detailedStatus: Tunnel is up and running = OK
```
gcloud beta compute vpn-tunnels describe vpc-demo-tunnel1 \
      --region us-central1
```

**TO VERIFY THE DETAILS OF THE 2ND ON-PREM TUNNEL** => detailedStatus: Tunnel is up and running = OK
```
gcloud beta compute vpn-tunnels describe on-prem-tunnel0 \
      --region us-central1
```

**TO VERIFY THE DETAILS OF THE 2ND ON-PREM TUNNEL** => detailedStatus: Tunnel is up and running = OK
```
gcloud beta compute vpn-tunnels describe on-prem-tunnel1 \
      --region us-central1
```

**CHANGE BGP CONFIG FROM REGION TO GLOBAL**
```
gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
```

**TO VERIFY UPDATE OF CHANGING BGP CONFIG FROM REGION TO GLOBAL** => routing Mode: GLOBAL
```
gcloud compute networks describe vpc-demo
```

