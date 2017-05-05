### docker-compose for nginx proxy with LE

Running nginx as a proxy with SSL support (via the free services of the Let's Encrypt project) on the Docker platform is possible using a combination of images and a docker-compose file.  This project makes the process very straightforward as follows:

```bash
curl -O https://raw.githubusercontent.com/ekkis/nginx-proxy-LE-docker-compose/master/docker-compose.yml
docker network create --subnet=172.28.0.0/16 --opt com.docker.network.bridge.name=nginx-proxy nginx-proxy
docker-compose up -d
```

Please note that the setup uses an external network that needs to be created first (the second command above), a one-time step required per docker host.  This allows you to define services in a separate docker-compose file and join those that need proxying to the external network.  Here's how it's done:

```
version: '3'

services:
   myservice:
      <other-details not shown>
      networks:
         - nginx-proxy

networks:
   nginx-proxy:
      external: true
```

Additionally, a subnet is specified in the network definition, which enables processes within your containers that need to be told where to accept request from to indicate the particular subnet in question.  For example, to tell a bitcoin server to accept requests from any process within the proxy network you could:

```
  bitcoind:
    image: seegno/bitcoind:latest
    <other-details-omitted>
    command:
      -server
      -rpcuser=$BTCUSER
      -rpcpassword=$BTCPASS
      -rpcallowip=172.28.0.0/16
      -printtoconsole
    networks:
      - nginx-proxy
```

Finally, the `--opt` passed during network creation makes sure to name the real interface as well since Docker otherwise assigns a random `br-<random hex string>` name to it.  Naming the real interface facilitates fine-tuning the IP tables using the name

#### Debugging

To use the staging servers from Let's Encrypt (instead of production which rate limits), create a `.env` file in the same directory where the docker-compose is located with the contents below, like this:

```bash 
echo "ACME_CA_URI=https://acme-staging.api.letsencrypt.org/directory" >> .env
```
#### Components:

For further information regarding the various components in this solution, please refer to each project:

* [The official NGINX image](https://github.com/nginxinc/docker-nginx)
* [JWilder's docker-gen](https://github.com/jwilder/docker-gen)
* [JrCs' docker-letsencrypt nginx-proxy companion ](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion)
