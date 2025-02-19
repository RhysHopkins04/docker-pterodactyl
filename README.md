## Wings-Node Creation Guide
Repository to create a new node to attach to the existing panel, when setting up a panel you still need to follow the guide, but it will be broken until a new one is published or that is integrated.

# Pre-Setup
<details>
<summary>Ensure the Pre-setup is done before trying to install a wings node</summary>

## 1. Setup the new system with ubuntu 22.04 LTS
<details>
<summary>Guided Steps:</summary>
<br>

[Guide link](https://ostechnix.com/install-ubuntu-server/)

</details>


## 2. Setup the ufw to allow that ssh port then enable it
<details>
<summary>Guided Steps:</summary>
<br>

1. Check if ufw is active (should be inactive) ```sudo ufw status numbered```

2. Default ssh port is 22, for staggering more servers on the same network we have gone to: 2222, 3222, etc. (may not be correct correct but works)
    
    ```sudo ufw allow <portnum>/<protocol (usually just tcp)>``` optionally add a comment by making it: ```sudo ufw allow <portnum>/<protocol (usually just tcp)> comment '<comment>'```

3. Other ports required by default for pterodactyl are:
	1. 2022 - SFTP port for pterodactyl running servers, we have staggered to ports 2023, 2024
	2. 443 - HTTPS port for connection, we have staggered to ports 8443, 8444
	3. 80 - HTTP port for fallback, we have staggered to ports 8081, 8082
	4. <port allocation range> - For allowing game server connections, we use ports 27000:27099 split between the servers.

        To add a range you do:  ```sudo ufw allow <portnum>:<portnum>/<protocol>```

	5. Enable ufw:  ```sudo ufw enable```

</details>

## 3. Sort out the new ssh port if using non standard
<details>
<summary>Guided Steps:</summary>
<br>

If this is the only server on a network then by default it will be 22 and can be left, otherwise do the below:

1. ```nano /etc/ssh/sshd_config```

2. look for ```#Port 22```, and remove the comment at the beginning and set it to the new ssh port (make sure it is one allowed by ufw)

3. Save the file ```(ctrl x)``` then follow the instructions for saving.

4. Restart ssh services: ```sudo /sbin/service sshd restart```

5. Sometimes restart the server itself ```sudo reboot -h now```

</details>

## 4. Create a new user account for remote access with sudo group
<details>
<summary>Guided Steps:</summary>
<br>

If you are going to have multiple users possibly managing the server itself, ensure you add another user account etc:

1. Create the user account: ```sudo adduser <username>```

2. Grant the user the sudo group: ```sudo usermod -aG sudo <username>```

3. Have the user connect and reset their password ```passwd```

</details>

## 5. install docker through this guide: 
<details>
<summary>Guided Steps:</summary>
<br>

[Guide link or follow below](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

1. ```sudo apt update```

2. ```sudo apt install apt-transport-https ca-certificates curl software-properties-common```

3. ```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg```

4. ```echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null```

5. ```sudo apt update```

6. ```apt-cache policy docker-ce```

7. ```sudo apt install docker-ce```

8. ```sudo systemctl status docker```

</details>

## 6. Allow docker to be ran without the sudo command on user account:
<details>
<summary>Guided Steps:</summary>
<br>

1. ```sudo usermod -aG docker ${USER}```

2. ```su - ${USER}```

3. ```groups```

4. If not on the account that needs the permission use this: ```sudo usermod -aG docker <username>```

</details>

## 7. Install docker compose through this guide: 
<details>
<summary>Guided Steps:</summary>
<br>

[Guide link or follow below](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-22-04)

1. ```mkdir -p ~/.docker/cli-plugins/```

2. ```curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose```

3. ```chmod +x ~/.docker/cli-plugins/docker-compose```

4. ```docker compose version```

</details>
</details>
<br>

# Installing and Setting up a Node:
<details>
<summary>Guided Steps:</summary>
<br>

[Old Guide link or follow below (Below is better)](https://github.com/EdyTheCow/docker-pterodactyl)

### Requirements:

1. Access to the domain DNS records with Edit permissions.
2. A further server to be used as a node running Ubuntu 22.04 Server (LTS)

### Setup the DNS record:

2. Create an A record pointing to the wings server IP, this shouldn't be proxied on Cloudflare (grey cloud).

### Installing and setting up Traefik:

#### Pre-Requisites:

Ensure if the directory is a private repo you create a PAT for it. Follow the steps below:

1. go to ```github.com -> settings -> developer settings -> Personal access tokens -> Fine-grained tokens -> Generate new token```,

2. Select a unique name then set the Resouce owner as the organization, set a short expiration since it isnt needed, set it to only select repositories: ```pterodactyl-docker```, then repository permissions for contents to ```"Read-only"``` and ```generate token```, **ensure you copy it** since it will get auto hidden after leaving the page.

3. Use your github username the PAT was created on, and the PAT in the below git clone statement to allow it to pass auth.

#### Setup:

1. Clone the repo: ```sudo git clone https://<github username>:<paste PAT>@github.com/Wargames-Development/pterodactyl-docker.git /home/```

2. Allow access to the directory: ```sudo chown -R <username>:<username> /home/pterodactyl-docker/```

3. Navigate to the folder for traefik in filezilla ```cd /home/pterodactyl-docker/_base/compose/```

4. Edit the file .env adding the cloudflare zone api token: Ask Glac for the api key and add it under ```CF_DNS_API_TOKEN=<placeholder>``` This will allow it to work for generating ssl certs without port 80. Save and upload the new edited .env

5. Edit the docker-compose.yml and change the ports under ```Services->ports->``` to have the two selected ports for http and https for that node. Save and upload the edited file.

6. Navigate to the folder for traefik's data: ```cd /home/pterodactyl-docker/_base/data/traefik/```

7. Edit the traefik.toml and change the ```[entryPoints]->[entryPoints.web]->address = ":port"``` to the http port, repeat for the .websecure https port (set it to the https port), update the email under ```[certificatesResolvers.letsencrypt.acme]``` to be your email for notifaction of renewals (will tell you if it fails etc).

8. set the acme.json to have the correct permissions: ```sudo chmod 600 /home/pterodactyl-docker/_base/data/traefik/acme.json```

9. You will now need to create a new network called "pterodactyl" that is shared: ```docker network create pterodactyl```

10. Bring up the docker container: Navigate to ```cd /home/pterodactyl-docker/_base/compose/``` and run ```docker compose up -d```

11. Check the logs for the container to see if there were errors: be in the same compose directory and run: ```docker logs base-traefik-1```

12. You can also check the ports in use by doing: ```docker ps```

### Setting up the wing node:

1. Navigate to the directory: ```cd /home/pterodactyl-docker/wings/compose/```

2. Edit the .env: Open the .env and add the FQDN(Fully Qualified Domain Name) that you added as a DNS A Record eg. ```node<num>.example.com``` Save and upload the edited file.

3. Edit the docker-compose.yml and change the port for SFTP, and a port for a label: ```services->wings->ports-> "<SFTP-Port>:<SFTP-Port>"```

    Set the SFTP-Port to the one you set earlier when doing the UFW.

    ```services->wings->labels-> "traefik.http.services.pterodactyl_wings-https.loadbalancer.server.port=<HTTPS-Port>"```

    Set the HTTPS-Port to the one you set earlier when doing the UFW. Save and upload the docker-compose.yml back onto the server.

4. Access the ```panel->admin settings-> nodes``` page and create a new node filling out the information as such:

    ```bash
    Name = <Node Name> # We follow node naming eg. ```Node<number>```
    Description = <Node Descriptor> # We usually follow adding the informatin that the node is running on (server specs),
    Location =  <location of node> # If the node is on a new location remember to make a new location, then try create the node
    Node Visibility = <Public> # If you want anyone to be able to use it etc. (not sure about private),
    FQDN = <FQDN in A record> # If this is wrong it will give: "\"pterodactyl_wings-https\" error: unable to find the IP address for the container"
    Communicate over SSL = <Use SSL Connection> # If this is wrong it will give: "\"pterodactyl_wings-https\" error: unable to find the IP address for the container"
    Behind Proxy = <Behind Proxy> # If this is wrong it will give: "\"pterodactyl_wings-https\" error: unable to find the IP address for the container"
    Daemon Server File Directory = <Leave default> # If changes were made to the default storage locations then this needs to be changed.
    Total Memory = <RAM of server in MIB> # Convert from GB to MiB using below tool, ensure to leave some headway, ususally leave 1-2GB.
    Memory Over-Allocation = <set to 0> # Set to 0 unless you understand fully what it does, allows the node to go beyond the set value.
    Total Disk Space = <Disk-space in MiB> # Convert from GB to MiB using below tool, find using df -h and leave a little space for the system ~50GB
    Disk Over-Allocation = <set to 0> # Set to 0 unless you understand fully what it does, allows the node to go beyond the set value.
    Daemon Port = <HTTPS port set earlier> # Ensure to set to the HTTP S Port not HTTP since we are using SSL.
    Daemon SFTP Port = <SFTP port set earlier> # Ensure this is set to the correct port, used for SFTP usage for individual servers.
    ```

    [Converter for GB to MiB](https://www.xconvert.com/unit-converter/gigabytes-to-mebibytes)
<br>

5. Go to the Configuration tab in the new node, and copy the configuration file.

6. Navigate to ```cd /home/pterodactyl-docker/wings/data/wings/etc/```

7. Paste the content you copied into the config.yml and save and upload it again.

8. Navigate to: ```cd /home/pterodactyl-docker/wings/compose/```

9. Run the command: ```docker compose up -d```

10. Check the logs for the container to see if there were errors: be in the same compose directory and run: ```docker logs pterodactyl-wings-1```

11. You can also check the ports in use by doing: ```docker ps```


### If there are issues for any node with conflicting networks etc, Check the subnets of that server, and or other docker containers on the other nodes

The error may look like:
```bash
    FATAL: [Month Date Time] failed to configure docker environment error= Error response from daemon: invalid pool request: Pool overlaps with other one on this address space

    Stacktrace:
    Error response from daemon: invalid pool request: Pool overlaps with other one on this address space
```

<details>
<summary>Guided Steps:</summary>

### Checking the currently in use subnets of the other servers

If the new node is running on the same network as other nodes, then it will need its subnets to be different to the current ones in use by the other nodes.

Please refer to the documentation that we have for this, or go onto the other servers and check the current subnets of the networks in use using this command.

It will require jq to be installed, so if you get that error install it using ```sudo apt update``` followed by, ```sudo apt install jq```

```docker network inspect $(docker network ls | awk '$3 == "bridge" { print $1}') | jq -r '.[] | .Name + " " + .IPAM.Config[0].Subnet' -```

This will return likely 3 responses if the server failed because of subnet allocations:
bridge 172.17.0.0/16
pterodactyl 172.18.0.0/16
wings0 172.22.0.0/16

This needs to be repeated on all nodes that exist and then adjust in the docker composes the subnets that they are conflicting with.

Ps. remember to delete and re-create the pterodactyl network with a specific non-conflicting subnet:

1. Turn off any containers that are running, both the wings and the traefik using ```docker compose down``` in their compose directories.

2. Remove the old network that doesn't work: ```docker network rm pterodactyl```

3. Create the new network with a set subnet that is free:

    ```docker network create --driver bridge --subnet 172.<set number that doesnt conflict>.0.0/16 pterodactyl```

    **It appears that the pterodactyl network by default likes to create on subnet 172.18.0.0/16, and it also looks like pterodactyl_nw generated by wings somewhere also wants .18, therefore the usual fix is offsetting the pterodactyl network to 172.19.0.0/16 therefore use:** ```docker network create --driver bridge --subnet 172.19.0.0/16 pterodactyl```

4. Put back up both of the containers, traefik first, then wings.

### Further Errors:

If you have attempted to change a value on the node in the panel and it gives you a 500 error loop, recreate the node again and ensure settings are correct such as:

```bash
FQDN = Correct node.example.com address as an a record in cloudflare.
Communicate Over SSL = Use SSL Connection,
Behind Proxy = Behind Proxy
Daemon Port = Correct HTTPS Port
Daemon SFTP Port = Correct SFTP Port
```

</details>
</details>