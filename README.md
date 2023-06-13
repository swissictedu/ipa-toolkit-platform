# IPA Toolkit <br> <small>platform</small>

The IPA Toolkit implements helpful tools to master the mighty IPA process. This is the platform repository, which contains aids to configure the whole system.

## Getting started

The following steps describe how to set up the IPA toolkit.

### Environment

These steps describe how to set up the system environment on Ubuntu 22.04 LTS:

1. Install updates

   ```bash
   apt update && apt upgrade
   ```

1. Install Docker dependencies

   ```bash
   apt install apt-transport-https ca-certificates curl gnupg lsb-release
   ```

1. Import Dockers GPG key

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

1. Add Dockers apt repository
   - On x86_64

     ```bash
     echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
     ```

   - On ARM

     ```bash
     echo "deb [arch=arm64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
     ```

1. Install Docker

   ```bash
   apt update && apt install docker-ce docker-ce-cli containerd.io
   ```

1. Create directory for Docker cli plugins

   ```bash
   mkdir -p /usr/local/lib/docker/cli-plugins
   ```

1. Download `docker-compose` executable
   - On x86_64

     ```bash
     curl -SL https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-x86_64 -o /usr/local/lib/docker/cli-plugins/docker-compose
     ```

   - On ARM

     ```bash
     curl -SL https://github.com/docker/compose/releases/download/v2.4.1/docker-compose-linux-aarch64 -o /usr/local/lib/docker/cli-plugins/docker-compose
     ```

1. Give executable permission to Docker Compose

   ```bash
   chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
   ```

1. Validate the installation of Docker (>= `20.10.17`)

   ```bash
   docker version
   ```

1. Validate the installation of Docker Compose (>= `2.4.1`)

   ```bash
   docker compose version
   ```

The following sources were used:

- [Documentation: Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Documentation: Docker Compose V2](https://docs.docker.com/compose/cli-command/#installing-compose-v2)

### Application system

1. Install git

   ```bash
   apt install git
   ```

1. Clone this repository

   ```bash
   cd /srv && git clone https://github.com/ictorg/ipa-toolkit-platform
   ```

1. Copy the `docker-compose.yml`.

   ```bash
   cp docker-compose.yml docker-compose.prod.yml
   ```

1. Make sure that the DNS is routing all subdomains to the host where the individual services run on.
1. Replace all `example.com` with the domain where the application runs on.

   ```bash
   sed -i "s/example.com/example.ch/g" docker-compose.prod.yml
   ```

1. Change contact email for SSL certificates in `traefik.yml`
1. Configure the following environment variables in `docker-compose.prod.yml`:
   - **ipa-toolkit-backend**
     - `DEFAULT_HOST`: The backends hostname
     - `SECRET_KEY_BASE`: Generate with `rails secret`
     - `SYSTEM_EMAIL_SERVER`: SMTP server
     - `SYSTEM_EMAIL_DOMAIN`: Domain
     - `SYSTEM_EMAIL_ADDRESS`: E-mail account address (username)
     - `SYSTEM_EMAIL_PASSWORD`: E-mail account password
   - **ipa-toolkit-frontend**
     - `API`: Escape uri so it works with sed (e.g. `"https:\\/\\/api.example.com\\/graphql"`)
1. Run application system

   ```bash
   docker compose -f docker-compose.prod.yml up -d
   ```

1. Initialize database
   1. Create database

      ```bash
      docker exec -it $(docker ps -f name=backend..$ -q) rails db:create
      ```

   1. Load database schema

      ```bash
      docker exec -it $(docker ps -f name=backend..$ -q) rails db:schema:load
      ```

   1. Create administrator account

      ```bash
      docker exec -it $(docker ps -f name=backend..$ -q) rails db:seed
      ```

### Release a new version

1. Merge the release pull request and wait for completion of the pipeline.

### Upgrade application system

1. Pull new images

   ```bash
   docker compose -f docker-compose.prod.yml pull
   ```

1. Recreate containers

   ```bash
   docker compose -f docker-compose.prod.yml up -d
   ```

1. Migrate database

   ```bash
   docker exec -it $(docker ps -f name=backend..$ -q) rails db:migrate
   ```

### Maintain application system

- Create a database backup

  ```bash
  docker exec -it $(docker ps -f name=backend-postgres..$ -q) pg_dumpall -c -U postgres > toolkit_dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql
  ```

- Archive storage

  ```bash
  tar -czvf toolkit_storage_`date +%d-%m-%Y"_"%H_%M_%S`.tar.gz /var/lib/docker/volumes/ipa-toolkit-platform_storage/_data/
  ```
