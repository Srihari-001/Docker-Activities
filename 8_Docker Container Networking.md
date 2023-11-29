# Docker Container Networking

## Introduction

Each container should serve a single purpose, such as running one application like a web server. 
Containers can be powerful by themselves, but when connected together, they are far more useful. 
For example, a web server container can be connected to a database container to provide application storage. 
Docker provides multiple options for networking containers.

In this lab, you'll explore a few of the common types of networks that Docker supports 
and learn how containers within those networks interact.

## Solution

### Log in to the server using the credentials provided:

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

### Explore the Default Network

1. List the default networks:

    ```bash
    docker network ls
    ```
Note:
Bridge Network:

Definition: A bridge network is the default network type when you create a new container. 
It allows containers to communicate with each other using container names as hostnames.
Use Case: Commonly used when you want to isolate containers from the host machine and other networks.

Host Network:

Definition: In the host network mode, a container shares the network namespace with the Docker host. 
It bypasses the Docker virtual network and connects directly to the host's network interfaces.
Use Case: Useful when you want a container to have the same network namespace as the host, 
essentially sharing the network stack.

None Network:

Definition: In the none network mode, a container has no network access. It's a completely isolated network namespace.
Use Case: Useful when you want to run a container without any network connectivity, providing maximum isolation.

Busybox:

Definition: BusyBox is a lightweight and versatile Unix toolset that 
provides several stripped-down Unix tools in a single executable. 
In the context of the lab, it is used as a minimalistic container image 
to demonstrate networking concepts.
Use Case: Often used in Docker tutorials and labs to keep the container image size small. 
It provides essential command-line utilities in a single binary.

2. Run an httpd container named web1, without specifying a network, and see which network it uses:

    ```bash
    docker run -d --name web1 httpd:2.4
    docker inspect web1
    ```

    Take note of the IPAddress.

3. Run a container using the busybox image and see if you can connect to the web1 server:

    ```bash
    docker run --rm -it busybox
    ```

4. Check the container's networking and verify it is in the same IP range as web1:

    ```bash
    ip addr
    ```

5. Ping the web1 container using the IP address:

    ```bash
    ping <WEB1_IP_ADDRESS>
    ```

6. Attempt to ping the web1 container by name:

    ```bash
    ping web1
    ```

7. Attempt to access web1 using wget:

    ```bash
    wget <WEB1_IP_ADDRESS>
    ```

8. Exit the container:

    ```bash
    exit
    ```

### Explore Bridge Networks

9. Create a new bridge network named test_application:

    ```bash
    docker network create test_application
    ```

10. Run an httpd container named web2, in the test_application network:

    ```bash
    docker run -d --name web2 --network test_application httpd:2.4
    ```

11. Check the status of the container:

    ```bash
    docker ps -a
    ```

12. Verify that web2 was added to the test_application network:

    ```bash
    docker inspect web2
    ```

13. Run a container using the busybox image and see if you can connect to the web2 server, within the test_application network:

    ```bash
    docker run --rm -it --network test_application busybox
    ```

14. Check the container's networking and verify it is in the same IP range as web2:

    ```bash
    ip addr
    ```

15. Ping the web2 container using the IP address:

    ```bash
    ping <WEB2_IP_ADDRESS>
    ```

16. Attempt to ping the web2 container by name:

    ```bash
    ping web2
    ```

17. Using wget, attempt to access web2 with the hostname:

    ```bash
    wget web2
    ```

18. Attempt to ping web1:

    ```bash
    ping <WEB1_IP_ADDRESS>
    ```

19. Attempt to access web1 using wget:

    ```bash
    wget <WEB1_IP_ADDRESS>
    ```

20. Exit the container:

    ```bash
    exit
    ```
Note:
The connectivity works in the test_application network because it is a user-defined bridge network, 
which provides internal DNS resolution by default. When you run containers in this network, 
they can communicate with each other using container names, 
as the internal DNS service resolves these names to the respective container IP addresses.

In contrast, when you use the default bridge network or the host network, 
there is no built-in internal DNS resolution mechanism. In the default bridge network, 
containers are assigned dynamic IP addresses, and they are not directly reachable by name. 
In the host network mode, containers share the network namespace with the host, 
so they use the host's networking directly.

Let's break down the behavior in each network:

Default Bridge Network:

Containers get dynamic IP addresses.
No internal DNS resolution by default.
You need to use container IP addresses for communication.

Host Network:

Containers share the network namespace with the host.
They use the host's networking directly.
No internal DNS resolution by default.

User-Defined Bridge Network (test_application in this case):

Containers get static IP addresses within the user-defined bridge network.
Internal DNS resolution is provided by Docker, allowing you to use container names for communication.
It's isolated from the host network by default.

### Explore the Host Network

21. Run an httpd container named web3 on the host network:

    ```bash
    docker run -d --name web3 --network host httpd:2.4
    ```

22. Check the status of the container:

    ```bash
    docker ps -a
    ```

23. Attempt to connect to web3 directly from the server:

    ```bash
    wget localhost
    ```

24. Stop web3:

    ```bash
    docker stop web3
    ```

25. Attempt to connect to web3 directly from the server again:

    ```bash
    wget localhost
    ```

26. Start web3:

    ```bash
    docker start web3
    ```

27. Run a container using the busybox image and see if you can connect to the web3 server:

    ```bash
    docker run --rm -it --network host busybox
    ```

28. ping web3

29. Using wget, attempt to access localhost within the busybox image:

    ```bash
    wget localhost
    ```

30. Attempt to ping web2:

    ```bash
    ping <WEB2_IP_ADDRESS>
    ```

31. Attempt to ping web1:

    ```bash
    ping <WEB1_IP_ADDRESS>
    ```

## Conclusion

Congratulations — you've completed this hands-on lab!