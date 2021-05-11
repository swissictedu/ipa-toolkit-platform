# IPA Toolkit <br> <small>platform</small>

The IPA Toolkit implements helpful tools to master the mighty IPA process. This is the platform repository, which contains aids to configure the whole system.

## Getting started
The following steps describe how to set up the IPA toolkit.

### Environment
These steps describe how to set up the system environment on Ubuntu 20.04 LTS:

1. Install updates <br> `apt update && apt upgrade`
1. Install Docker dependencies <br> `apt install apt-transport-https ca-certificates curl gnupg lsb-release`
1. Import Dockers GPG key <br> `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`
1. Add Dockers apt repository <br> `echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
1. Install Docker <br> `apt update && apt install docker-ce docker-ce-cli containerd.io`
1. Download Docker Compose <br> `curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
1. Give executable permission to Docker Compose <br> `chmod +x /usr/local/bin/docker-compose`
1. Validate the installation of Docker (>= 20.10.6) <br> `docker -v`
1. Validate the installation of Docker Compose (>= 1.29.1) <br> `docker-compose -v`

### Application system
1. Install git <br> `apt install git`
1. Clone this repository <br> `git clone https://github.com/ictorg/ipa-toolkit-platform`
1. Copy the `docker-compose.yml`.
1. Make sure that the DNS is routing all subdomains to the host where the individual services run on.
1. Replace all `example.com` with the domain where the application runs on. <br> `sed -i "s/example.com/example.ch/g" docker-compose.yml`
1. Change contact email for SSL certificates in `traefik.yml`
1. Configure the following environment variables:
   - **ipa-toolkit-backend**
     - `SECRET_KEY_BASE`: Generate with `rails secret`
     - `SYSTEM_EMAIL_SERVER`: IMAP server
     - `SYSTEM_EMAIL_DOMAIN`: Domain
     - `SYSTEM_EMAIL_ADDRESS`: E-mail account address (username)
     - `SYSTEM_EMAIL_PASSWORD`: E-mail account password
   - **ipa-toolkit-frontend**
     - `API`: Escape uri so it works with seed (e.g. `"https:\\/\\/api.example.com\\/graphql"`)
1. Run application system <br> `docker-compose up -d`
1. Initialize database
   1. Create database <br> `docker exec -it $(docker ps -f name=backend_ -q) rails db:create`
   1. Load database schema <br> `docker exec -it $(docker ps -f name=backend_ -q) rails db:schema:load`

### Upgrade application system
1. Pull new images <br> `docker-compose pull`
1. Recreate containers <br> `docker-compose up -d`
1. Migrate database <br> `docker exec -it $(docker ps -f name=backend_ -q) rails db:migrate`