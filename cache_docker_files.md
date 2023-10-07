# Local caching of docker files
This removes the requirement of fresh nodes to download all the images from the internet

On the server
```
mkdir -p /storage/docker_caching_registry/data
chmod -R 777 /storage/docker_caching_registry/data
```

In portainer create a new stack with (replace <serverhostname> with the actual hostname)
```
version: '3.2'

services:
  portainer:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
      REGISTRY_HTTP_SECRET: "590hiu5hj54h095uh5hp0p9ejhkberhero9herjkhrejhre"
      REGISTRY_PROXY_REMOTEURL: "https://registry-1.docker.io"
      REGISTRY_PROXY_REMOTEURL_USERNAME: "yfiro"
      REGISTRY_PROXY_REMOTEURL_PASSWORD: "Ng3P2jxX8TSRrE2mmry4sw6YsAU9"
      REGISTRY_STORAGE_CACHE_BLOBDESCRIPTOR: "inmemory"
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - registry_data:/data
    restart: always
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.hostname == <serverhostname>

volumes:
  registry_data:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/storage/docker_caching_registry/data'
```
