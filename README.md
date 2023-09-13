# BloodHoundAD community edition
### KCSIRTs Startup guide

Pre-requisites
---
1. Latest docker installed, this guide utilises Rocky Linux as OS with docker-ce  
  Install procedure for docker-ce on Rocky Linux 9, with detailed informartion, is available on [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-rocky-linux-9#step-2-executing-docker-command-without-sudo-optional)

Docker-install (summarised)
---
The following is a set of commands to update the repo list, add the repo for docker-ce, installing docker-ce, starting the service and enabling it as a permanent service.
Then finally it adds the current user to the docker group, so it doesn't require sudo privileges to run containers.
```
sudo dnf check-update
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
sudo usermod -aG docker $(whoami)
```


Configurations
---
1. Set the `BH_HOSTNAME` variable within the `.env` file  
  Eg. to bloodhound.localhost
1. Update DNS record for the FQDN set, eg. in local hostfile or DNS server
1. Start the docker containers using docker compose  
  `docker compose up -d`
1. Get the randomly generated password, within the Bloodhound application container.  Usually named `bloodhound-bloodhound-1`  
  `docker ps`  
  `docker logs bloodhound-bloodhound-1`
1. Access bloodhounds web UI and use "admin" as username and the random password, then set a new password.


Commands
---
- Start Bloodhound on a clean docker server  
  `docker compose up`
- Shutdown and cleanup all stored data, including bloodhound user database  
  `docker compose down --volumes`
