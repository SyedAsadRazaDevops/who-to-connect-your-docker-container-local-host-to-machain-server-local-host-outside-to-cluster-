# who-to-connect-your-docker-container-local-host-to-machain-server-local-host-outside-to-cluster

# How to Connect to Localhost Within a Docker Container

When working with Docker, you usually containerize the services that form your stack and use inter-container networking to communicate between them. Sometimes you might need a container to talk to a service on your host that hasn’t been containerized. Here’s how to access localhost or 127.0.0.1 from within a Docker container.

# Way:1 The Easy Option
Docker Desktop 18.03+ for Windows and Mac supports host.docker.internal as a functioning alias for localhost. Use this string inside your containers to access your host machine.
```
localhost and 127.0.0.1 – These resolve to the container.
host.docker.internal – This resolves to the outside host.
```
If you’re running a MySQL / mongo server on your host, Docker containers could access it by connecting to host.docker.internal:3306 and mongo for port 27017. This is the simplest technique when you’re working on a Windows or Mac machine.

>Docker Engine users on Linux can enable host.docker.internal too via the --add-host flag for docker run. Start your containers with this flag to expose the host string:
```
docker run -d --add-host host.docker.internal:host-gateway my-container:latest
```
The --add-host flag adds an entry to the container’s /etc/hosts file. The value shown above maps host.docker.internal to the container’s host gateway, which matches the real localhost value. You could replace host.docker.internal with your own string if you prefer.

# Way:2 Connecting to the Host Network
Docker provides a host network which lets containers share your host’s networking stack. This approach means localhost inside a container resolves to the physical host, instead of the container itself.

>Containers are launched with the host network by adding the --network=host flag:
```
docker run -d --network=host my-container:latest
```
>Now your container can reference localhost or 127.0.0.1 directly.

If you’re using Docker Compose, modify your container’s service definition to include the network_mode field:
>add them in docher-compose.yaml
```
services:
  my-service:
    network_mode: host
```
There are some caveats to this approach. It’s important to consider all the implications before you use it. Containers ordinarily get their own private network that’s separate to the host’s stack. When you specify --network=host, the container defaults to inheriting shared networking settings from your host.

Any ports exposed by the container will be exposed on the host, even if they’re not explicitly declared with the -p flag. The container’s default hostname will match the host’s, although this can be changed with the --hostname flag.

The host network can be a security concern which breaks the isolation model of Docker containers. It can still be useful in scenarios where you’re confident that running containers won’t conflict with each other or cause problems in your host environment. The host networking mode is also quicker than the default bridge mode as there’s no virtualization layer for traffic to pass through.

# Way:3 Accessing the Host With the Default Bridge Mode
Your host can still be accessed from containers in the default bridge networking mode. You just need to reference it by its Docker network IP, instead of localhost or 127.0.0.1.

Most Docker Engine installations will represent the host as 172.17.0.1 on the default docker0 bridge network. You can check your own IP by running this command on your host:
```
ip addr show docker0
```
Your host’s Docker IP will be shown on the inet line. Connect to this IP address from within your containers to successfully access the services running on your host.

One pitfall of this approach is you might not be able to connect to services which bind directly to localhost. You’ll need to make sure your services are listening for connections on your Docker bridge IP, as well as localhost and 127.0.0.1. Otherwise you’ll see connection refused or similar errors within your container.

# Summary
You’ve got several options when you need to reach outside a Docker container to your machine’s localhost. If you’re on Windows or Mac, it’s best to use the built-in host.docker.internal alias. Linux users can setup something similar with the --add-host flag when starting a container.

Host networking mode is a universal alternative which lets containers share your host’s networking stack. You can reference localhost directly but need to stay aware of the risks and limitations. It’s not a suitable option when strong networking isolation is required.

Sticking with bridge mode can be the best option for workloads which support it. Bind your host’s services to its Docker IP, then use that address to connect from within your container. This lets you use Docker’s per-container virtualized networking while providing a route to your host when it’s required.
