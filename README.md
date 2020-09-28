# jordvisser - Portainer agent
This docker image is a repack of the HomeCentr [portainer agent](https://github.com/homecentr/docker-portainer-agent) to use alpine latest as the original [portainer agent](https://portainer.io) does.


# Orginal HomeCentr README - Portainer agent

## Usage

```yml
version: "3.7"
services:
  portainer-agent:
    build: .
    image: jordvisser/portainer-agent
    restart: unless-stopped
    environment:
      PORTAINER_AGENT_ARGS: ""   
    ports:
      - "9001:9001/tcp"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

## Environment variables

| Name | Default value | Description |
|------|---------------|-------------|
| PUID | 7077 | UID of the user portainer agent should be running as. The UID must have sufficient rights to read from the Docker socket. |
| PGID | 7077 | GID of the user portainer agent should be running as. You must set the PUID if you want to set the PGID variable. |
| PORTAINER_AGENT_ARGS |  | Command line [arguments](https://portainer-agent.readthedocs.io/en/stable/configuration.html#available-flags) to the portainer-agent executable. |
| FIRST_HEALTHCHECK_TIMEOUT | 20 (seconds) | Sets how long the start script will wait for the first execution of healthcheck (see the explanation below). If you see the container is restarting and the last message in output is "Signal from healthcheck not received in time, failing the execution...", try increasing this timeout. |

## Exposed ports

| Port | Protocol | Description |
|------|------|-------------|
| 9001 | TCP | Agent API |

## Volumes

| Container path | Description |
|-------------|-----------------|
| /data | portainer agent data |

> Make sure you mount the Docker socket.

## Healthcheck - why is it so complicated...

When running in a Swarm cluster, Portainer agent does a DNS lookup to discover other agents. If the container has healthcheck, the DNS record for the container is only added to the Swarm's internal DNS when the healthcheck succeeds. This creates a bit of chicken and egg problem. Healthcheck waits for the agent to start and the agent requires the healthcheck to succeed before it's actually started.

Workaround:
- The healtcheck starts as soon as the container starts and the first execution immedialy succeeds
- Swarm adds record for the container into the internal DNS
- Bash script which is used to start the Portainer agent waits until it gets a signal from healthcheck and only then it actually starts the agent

## Security
The container is regularly scanned for vulnerabilities and updated. Further info can be found in the [Security tab](https://github.com/homecentr/docker-portainer-agent/security).

### Container user
The container supports privilege drop. Even though the container starts as root, it will use the permissions only to perform the initial set up. The portainer process runs as UID/GID provided in the PUID and PGID environment variables.

:warning: Do not change the container user directly using the `user` Docker compose property or using the `--user` argument. This would break the privilege drop logic.

:bulb: To grant a user the permission to read Docker socket, you can set the PGID to the ID of the docker group on host.

Note: PUID or PGID must give access to docker.sock

