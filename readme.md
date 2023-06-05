dck-proxy-stack
===

Basic reverse proxy and container management with traefik and portainer

#

## Initial setup (Docker)

- Upgrade system and install dependencies
 ```console
 sudo apt-get update && sudo apt-get upgrade -yml
 sudo apt-get install apt-transport-https ca-certificates curl software-properties-common apache2-utils
 ```

- Add the GPG Key for the official Docker Registry
 ```console
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 ```

- Add the Docker repository to APT sources
 ```console
 echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 ```

- Update list of packages
 ```console
 sudo apt-get update
 ```

- Make sure you are about to install from the Docker repo instead of the default Ubuntu repo
 ```console
 apt-cache policy docker-ce
 ```

- Finally, install Docker
 ```console
 sudo apt-get install -y docker-ce
 ```

- Check if Docker is running
 ```console
 sudo systemctl status docker
 ```

- Configure Docker to run without sudo
 ```console
 sudo usermod -aG docker ${USER}
 ```

- Check last version of docker compose plugin at <https://github.com/docker/compose/releases>
- Download docker compose plugin
 ```console
 mkdir -p ~/.docker/cli-plugins/
 curl -SL https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
 ```

- Set the correct permissions so that the docker compose command is executable
 ```console
 chmod +x ~/.docker/cli-plugins/docker-compose
 ```

- Check docker compose version
 ```console
 docker compose version
 ```

- Download and install nvm (just in case)
 ```console
 sudo curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
 sudo source ~/.bashrc
 command -v nvm
 ```

- create docker network "proxy"
 ```console
 sudo docker network create proxy
 ```

- Reload bash before starting the stack
 ```console
 source ~/.bashrc
 ```

- Visit your portainer site and create the inital Admin User

#

## Usage

- Create an A entry in your domain dns pointing to your server's public IP for traefik
- Create an A entry in your domain dns pointing to your server's public IP for portainer
- Create an A entry in your domain dns pointing to your server's public IP for each or your services and/or apps
- Copy all files to a folder in your server.
- Put your services' and apps' docker files in the services folder.
  - You have to add these labels to the service's docker-compose file
 Don't forget to change <service_name> and <service_dns_name> with yours service's data!:

    ```
    labels:
      - "traefik.enable=true"
      - "traefil.docker.network=proxy"
      - "traefik.http.routers.<service_name>.rule=Host(`<service_dns_name>`)"
      - "traefik.http.routers.<service_name>.entrypoints=websecure"
    ```

- Generate credential for traefik users. You can use  >> echo $(htpasswd -nb <username> <password>
- Edit /core/traefik-data/configurations/dynamic.yml file and change <user_credentials> with the generated credentials.
- Edit /core/traefik-data/traefik.yml file and change <contact@domain.com> with your contact email.
- Edit /core/docker-compose file and change <TRAEFIK_HOST> and <PORTAINER_HOST>.
- Check /core/traefik-data/acme.json has the correct permissions
 ```console
 sudo chmod 600 <path.to>/acme.json
 ```

- Run traefick and portainer
 ```console
 docker compose -f ./core/docker-compose.yml up -d
 ```

#

## Directory Hierarchy

```
|—— LICENSE
|—— core
|    |—— docker-compose.yml
|    |—— portainer-data
|    |—— traefik-data
|        |—— acme.json
|        |—— configurations
|            |—— dynamic.yml
|        |—— traefik.yml
|—— services
```

#
