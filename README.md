# BloodHoundAD community edition
### KCSIRTs Startup guide

Pre-requisites
---
1. Latest docker installed, this guide utilises Linux as OS


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
