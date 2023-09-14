# BloodHoundAD community edition
### KCSIRTs Startup guide

Pre-requisites
---
1. Latest docker installed on your desired platform  
  This guide is based on Rocky Linux with docker-ce, but Docker Desktop and other versions are alternatives

Installation procedure for docker-ce on Rocky Linux 9 with detailed information, is available on [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-rocky-linux-9#step-2-executing-docker-command-without-sudo-optional)

Docker-install (Step 1 and 2 summarised)
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
1. Edit and set the `BH_HOSTNAME` variable within the `.env` file to your desired FQDN  
  Eg. to bloodhound.localhost
1. Create a DNS record for the FQDN set, eg. in local hostfile or DNS server
1. Start the docker containers using docker compose  
  `docker compose up -d`
1. Get the randomly generated password, within the Bloodhound application container.  Usually named `bloodhound-bloodhound-1`  
  `docker ps`  
  `docker logs bloodhound-bloodhound-1`
1. Access bloodhounds web UI and use "admin" as username and the random password, then set a new password.


Cleanup BloodHoundAD
---
- Shut down and cleanup all stored data, including bloodhound user database  
  `docker compose down --volumes`


Creating EDR Exceptions
===
NB! SharpHound and AzureHound will be quarantined/removed by EDR solutions, so remeber to create a special policy for allowing this tool before downloading.  
This guide will show exception template for Microsoft 365 Defender.

Create a new group to target this rule
1. Create a new group in Azure AD, and add the device, which will run Sharphound.
   * Open (Microsoft Intune admin center)[https://endpoint.microsoft.com/]
   * Open the `Groups` view
   * Create a `New group` with the following parameters
       * Group type: `Security`
       * Azure AD roles...: `No`
       * Membership type: `Assigned`
   * Click "Select members" and add the desired **devices**
  
Create the exception rule for SharpHound
1. Create Exception policy, below is a guide on Microsoft 365 Defender
  1. Open [Microsoft 365 Dender Endpoint Security Policies](https://security.microsoft.com/policy-inventory?osPlatform=Windows)  
     If not opened by the link, it resides under `Endpoints/Configuration Management` -> `Endpoint security policies`
  1. `Create new Policy`
     * Select platform: `Windows`
     * Select template: `Microsoft Defender Antivirus exclusions`
     * Click `Create policy`
  1. Basics
     * Enter a name and description
  1. Settings
     * Under settings->Defender->Ecluded Paths, add the path where sharphound is to be stored.  
        * Eg. `C:\utils\sharphound\` or `C:\users\[username]\Downloads\`
  1. Assignments
     * Add the previously generated group, with Target type `Include`
  1. Review and save the new policy 
1. Wait for the policy to be applied

SharpHound
===
1. Download the latest release from the Github repo to the excluded location
   Select the `SharpHound-vX.X.X.zip`  
   [SharpHound release](https://github.com/BloodHoundAD/SharpHound/releases)
1. Unpack
2. Create a folder to store the generated output


NB! If this computer is not domain joined, utilise step 1 and 2 steps to get SharpHound to work as desired.  
**Otherwise, skip to step 3**
1. Configure the devices DNS to be the domain controller
1. Spawn a CMD shell as a domain user  
   `runas /netonly /user:[DOMAIN]\[USERNAME] cmd.exe`
1. Execute SharpHound and let it run for the desired amount of time.  Examples follows below  
   [SharpHound documentation](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481151861019)  
   NB! if running secure LDAP in your domain, use the `--secureldap` parameter
1. Gather the data to a secure storage
1. Cleanup the tool and data from the device
1. Remove / disable the exception rule from Microsoft 365 Defender.

Execution examples for SharpHound
---
`.\SharpHound.exe -c All --Loop --loopduration 23:00:00 --loopinterval 00:05:00 --outputdirectory [PATH TO OUTPUTFOLDER] -d [DOMAIN]`  
This runs a 23H collection session, which collects data on everything every 5 minutes.

`.\SharpHound.exe -c All --Loop --loopduration 08:00:00 --loopinterval 00:03:00 --outputdirectory [PATH TO OUTPUTFOLDER] -d [DOMAIN]`  
This runs a 8H collection session, which collects data every 3 minutes.
