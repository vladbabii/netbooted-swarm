# Install portainer to manage the swarm

Portainer is nice because it can auto deploy agents on all nodes automatically so we don't have to do anything else for each worker node - netboot it and use it.

On the server do the following  (replace <serverhostname> with the actual server hostname)

```
mkdir -p /storage/portainer/data
cd /storage/portainer
echo "docker stack deploy -c portainer-agent-stack.yml portainer" > start.sh
echo "docker stack rm portainer" > stop.sh
chmod +x *.sh
```
then edit file portainer-agent-stack.yml
```
version: '3.2'

services:
  agent:
    image: portainer/agent:2.19.1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    restart: always
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:2.19.1
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
    networks:
      - agent_network
    restart: always
    deploy:
      mode: replicated
      replicas: 1
      placement:
#        constraints: [node.role == manager]
        constraints:
          - node.hostname == <serverhostname>

networks:
  agent_network:
    driver: overlay
    attachable: true

volumes:
  portainer_data:
    driver: local
    driver_opts:
      type: 'none'
      o: 'bind'
      device: '/storage/portainer/data'
```

then start it with ./start.sh

Portainer should be available at https://<server ip>:9443 after a bit of time

