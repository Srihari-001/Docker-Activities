## Introduction

After Docker installation, the best way to familiarize yourself is to run containers from prebuilt images. This lab explores Docker Hub for images that run a website. You will find suitable images, get them into your environment, and experiment with running, stopping, and deleting containers. Additionally, you'll learn how to use existing data in containers.

## Solution

### Step 1: Log in to the server

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>

or 

vagrant up and vagrant ssh (centos vagrant file)

Switch to this new user:

(name : "srihari") (password : "ComplexPwd@321")

su - srihari

```

### Step 2: Explore Docker Hub

- Sign in to Docker Hub.
- Search for "httpd" at the top of the page.
- In the left-hand menu, filter for Application Infrastructure and Official Images.
- Select the httpd project.
- Click the Tags tab under the latest version.
- Select the linux/amd64 tag.
- In the list of available images, also select the nginx image.
- Review the "How to use this image" section.

### Step 3: Get and View httpd

In the Docker instance:

# Verify Docker installation
```bash

docker ps

```

#docker ps
The docker ps command is used to list the currently running containers. 
This is often done before and after running a new container 
to check the status and confirm that the new container is running.

Purpose: It provides information about the containers that are currently active.
Usage: docker ps lists only running containers. 
To include stopped containers, you can use docker ps -a.

# Pull the httpd:2.4 image
```bash

docker pull httpd:2.4

```
docker pull httpd:2.4
Purpose: This command pulls the httpd image with the version tag 2.4 from Docker Hub.
Explanation: The docker pull command fetches the specified image from a registry 
(in this case, Docker Hub) and stores it on your local machine. 
The httpd:2.4 tag specifically refers to version 2.4 of the httpd (Apache HTTP Server) image. 
If you omit the tag, Docker pulls the latest version by default.

# Run the httpd container
```bash

docker run --name httpd -p 8080:80 -d httpd:2.4

```
docker run Command:
Purpose: This command creates and starts a new container from the specified image.
Explanation:
--name httpd: Sets the name of the container to "httpd". 
This is optional but provides a human-readable identifier.
-p 8080:80: Maps port 8080 on the host to port 80 on the container. 
This allows you to access the web server inside the container via port 8080 on the host machine.
-d: Runs the container in detached mode, meaning 
it runs in the background and doesn't block the terminal.
httpd:2.4: Specifies the image and 
its version (tag) to be used for creating the container.

# Check the status of the container
```bash

docker ps

```

In a web browser, test connectivity to the container:

```plaintext
<PUBLIC_IP_ADDRESS>:8080
```

### Step 4: Run a Copy of the Website in httpd

In the content-widget-factory-inc directory:

```bash
# Clone the Widget Factory Inc repository
git clone https://github.com/linuxacademy/content-widget-factory-inc
```
# Change to the content-widget-factory-inc directory

```
cd content-widget-factory-inc
```
# Check the files
```
ls
```
# Move to the web directory
```
cd web
```
# Check the files
```
ls
```
# Stop and remove the httpd container
```
docker stop httpd
docker rm httpd
```
# Verify that the container has been removed
```
docker ps -a
```
# Run the container with website data
```bash
docker run --name httpd -p 8080:80 -v $(pwd):/usr/local/apache2/htdocs:ro -d httpd:2.4
```
Volume Mounting (-v $(pwd):/usr/local/apache2/htdocs:ro):

Explanation:
The -v option is used to mount volumes, providing a way to share data between the host and the container.
$(pwd):/usr/local/apache2/htdocs:ro specifies the source path on the host, 
the destination path in the container, and sets the volume as read-only (ro).
Reason:
$(pwd) resolves to the present working directory on the host. 
This is often used to mount the current project's code into the container.
/usr/local/apache2/htdocs is the default document root for Apache in the container. 
By mounting the host's code into this directory, any changes made on the host are reflected in the container.
Setting the volume as read-only (ro) is a security measure. 
It ensures that the container cannot modify the files on the host, preventing accidental or malicious changes.

# Check the status of the container
```
docker ps
```

In a web browser, check connectivity to the container:

```plaintext
<PUBLIC_IP_ADDRESS>:8080
```

### Step 5: Get and View Nginx

Using Docker:

# Pull the latest version of nginx
```
docker pull nginx
```
# Verify that the image was pulled successfully
```
docker images
```
# Run the container using the nginx image
```
docker run --name nginx -p 8081:80 -d nginx
```
# Check the status of the container
```
docker ps
```

# Verify connectivity to the nginx container
```
<PUBLIC_IP_ADDRESS>:8081
```

### Step 6: Run a Copy of the Website in Nginx

# Stop and remove the nginx container
```
docker stop nginx
docker rm nginx
```
# Verify that the container has been removed
```
docker ps -a
```

# Run the nginx container and mount the website data
```
docker run --name nginx -v $(pwd):/usr/share/nginx/html:ro -p 8081:80 -d nginx
```

# Check the status of the container
```
docker ps
```

In a web browser, verify connectivity to the container:

```plaintext
<PUBLIC_IP_ADDRESS>:8081
```

### Step 7: Stop and Clean Up

```bash
# Stop and remove the nginx container
```
docker stop nginx
docker rm nginx
```

# Verify that the container has been removed
```
docker ps -a
```
**Differences between Apache HTTP Server (httpd) and Nginx:**
Differences between Apache HTTP Server (httpd) and Nginx:

Architecture:

Apache HTTP Server (httpd): Uses a process-based architecture where each connection is handled by a separate process.
Nginx: Uses an event-driven, asynchronous architecture that is more resource-efficient for handling a large number of concurrent connections.
Resource Consumption:

Apache HTTP Server (httpd): Consumes more memory per process, and the process-based model can result in higher memory usage under heavy loads.
Nginx: Typically has lower memory usage due to its efficient event-driven model.
Concurrency Handling:

Apache HTTP Server (httpd): Handles concurrency through a multi-process model where each process handles one connection at a time.
Nginx: Handles concurrency efficiently through an asynchronous, non-blocking model, allowing it to scale better under heavy loads.
Modules and Configuration:

Apache HTTP Server (httpd): Known for its extensive module ecosystem, providing a wide range of features. Configuration is done using an Apache-specific syntax.
Nginx: Has a modular architecture but tends to have a more streamlined core. Configuration is generally simpler and follows a straightforward syntax.
Use Cases:

Apache HTTP Server (httpd): Traditionally used for a wide range of use cases, including dynamic content generation with technologies like PHP.
Nginx: Often preferred for serving static content, acting as a reverse proxy, and handling a large number of concurrent connections efficiently.
Why do we need them:

Web Server:

Both Apache HTTP Server and Nginx serve as web servers, handling incoming HTTP requests and delivering web content to clients.
Load Balancing:

They can act as load balancers, distributing incoming traffic across multiple servers to ensure optimal resource utilization and reliability.
Reverse Proxy:

Serve as reverse proxies, forwarding requests from clients to backend servers, often used to improve security, performance, and simplify server management.
Static Content Delivery:

Efficiently serve static content (HTML, CSS, images) to clients, optimizing performance for websites.
Application Support:

Support for different programming languages and applications. Apache HTTP Server is known for its broad support for various modules and applications, while Nginx is often chosen for its efficient handling of static content and proxy capabilities.
Which is better:

The choice between Apache HTTP Server (httpd) and Nginx depends on the specific use case, performance requirements, and personal or organizational preferences.
For serving static content, handling a large number of concurrent connections, and acting as a reverse proxy, Nginx is often considered more efficient and lightweight.
Apache HTTP Server, with its extensive module ecosystem, is a solid choice for dynamic content generation and supporting a wide range of applications.
In summary, neither is universally "better" – it depends on the specific needs of the project or environment. Many factors, including performance requirements, ease of configuration, and specific feature needs, should be considered when choosing between Apache HTTP Server and Nginx.