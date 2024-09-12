# Hospital Networks 

c7200 

```
enable
configure terminal
interface g2/0
ip address dhcp
no shutdown
exit

interface g4/0
ip address 10.0.0.1 255.255.255.0
no shutdown
exit

ip dhcp pool lan
network 10.0.0.0 255.255.255.0
default-router 10.0.0.1
dns-server 8.8.8.8 1.1.1.1
lease 7
exit

interface g2/0
ip nat outside

interface g4/0
ip nat inside

access-list 1 permit 10.0.0.0 0.0.0.255
ip nat inside source list 1 interface g2/0 overload

exit

write memory
```

[dcm4che](https://github.com/dcm4che/dcm4chee-arc-light.git "dcm4che") 
4 vCPU, 4 GB DRAM, 8 GiB HDD

```
sudo tee /etc/netplan/50-cloud-init.yaml > /dev/null <<EOL
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 10.0.0.200/24
      gateway4: 10.0.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOL

sudo netplan apply

sudo snap install docker
sudo docker network create dcm4chee_network
sudo docker rm -f $(sudo docker ps -a -q)
sudo docker run --network=dcm4chee_network --name ldap \
           -p 389:389 \
           -v /home/ubuntu/dcm4chee-arc/ldap:/var/lib/openldap/openldap-data \
           -v /home/ubuntu/dcm4chee-arc/slapd.d:/etc/openldap/slapd.d \
           -d dcm4che/slapd-dcm4chee:2.6.6-32.0
sudo docker run --network=dcm4chee_network --name db \
           -p 5432:5432 \
           -e POSTGRES_DB=pacsdb \
           -e POSTGRES_USER=pacs \
           -e POSTGRES_PASSWORD=pacs \
           -v /etc/localtime:/etc/localtime:ro \
           -v /etc/timezone:/etc/timezone:ro \
           -v /home/ubuntu/dcm4chee-arc/db:/var/lib/postgresql/data \
           -d dcm4che/postgres-dcm4chee:16.2-32
sudo docker run --network=dcm4chee_network --name arc \
           -p 8080:8080 \
           -p 8443:8443 \
           -p 9990:9990 \
           -p 9993:9993 \
           -p 11112:11112 \
           -p 2762:2762 \
           -p 2575:2575 \
           -p 12575:12575 \
           -e POSTGRES_DB=pacsdb \
           -e POSTGRES_USER=pacs \
           -e POSTGRES_PASSWORD=pacs \
           -e WILDFLY_WAIT_FOR="ldap:389 db:5432" \
           -v /etc/localtime:/etc/localtime:ro \
           -v /etc/timezone:/etc/timezone:ro \
           -v /home/ubuntu/dcm4chee-arc/wildfly:/opt/wildfly/standalone \
           -d dcm4che/dcm4chee-arc-psql:5.32.0
```

Test with storescu 

```
storescu 10.0.0.200 11112 0002.DCM +sd -ll info -aec DCM4CHEE -xy 
```

Mirth Connect 
4 GiB HDD 

```
sudo tee /etc/netplan/50-cloud-init.yaml > /dev/null <<EOL
network:
  version: 2
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 10.0.0.210/24
      gateway4: 10.0.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOL

sudo netplan apply

sudo snap install docker
sudo docker rm -f $(sudo docker ps -a -q)
sudo docker run -d --name mirth-connect \
           -p 8080:8080 \
           -p 3306:3306 \
           nextgenhealthcare/connect
```

