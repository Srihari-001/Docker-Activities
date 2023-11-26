# Introduction
If you run your website from a pre-built base image, 
it will require a manual process to set up the container each time it runs. 
For repeatability and scalability, the container, and 
your website code should be made into an image.

In this lab, you will start with a base webserver image,
modify settings in the container for your website, 
and then create images from the container. 
You'll demonstrate the importance of small changes to your container, 
and how they affect your image. 
Lastly, you will use your new images to create containers to see your hard work in action.

# Solution
Log in to the server using the credentials provided:

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>

or 

vagrant up and vagrant ssh (centos vagrant file)

Switch to this new user:

(name : "srihari") (password : "ComplexPwd@321")

su - srihari
```

## Get and Run the Base Image
Retrieve the httpd image:

```bash
docker pull httpd:2.4
```

Run the image:

```bash
docker run --name webtemplate -d httpd:2.4
```
--name webtemplate: Sets a custom name for the container, in this case, "webtemplate".
-d: Detaches the container and runs it in the background.
httpd:2.4: Specifies the Docker image and its version to be used for the container. 
In this case, it's the Apache HTTP Server version 2.4.
If you want to expose ports from the container to the host, you should use the -p option

Check the status of the webtemplate container:

```bash
docker ps
```

## Install Tools and Code in the Container
Log in to the container:

```bash
docker exec -it webtemplate bash
```
NOTE:
The `docker exec` command is used to execute a command inside a running container.
 Here's a breakdown of the command:

- **`docker exec`**: This is the Docker command to execute a command in a running container.
- **`-it`**: These are options specifying the mode of interaction with the container:
  - `-i` stands for interactive mode, allowing you to interact with the container.
  - `-t` allocates a pseudo-TTY, essentially providing a terminal interface.

- **`webtemplate`**: This is the name of the running container where the command will be executed.

- **`bash`**: This is the command to be executed inside the container. 
In this case, it's starting an interactive Bash shell. 
If you're working with an image that uses a different shell 
(e.g., Alpine Linux images may use `sh` instead of `bash`), 
you would specify that shell.

So, when you run:

- `-it` ensures an interactive terminal session.
- `webtemplate` is the name of the container.
- `bash` is the command to be executed inside the container.

### Alternative Ways to Enter a Container:

- **Without Bash:**
  If you omit `bash` at the end, Docker will use the default command specified in the Dockerfile. 
  For example, if the Dockerfile ends with `CMD ["nginx", "-g", "daemon off;"]`, 
  you'll enter the container with that command, not Bash.

- **Using Another Shell:**
  If the container has a different shell, you can specify it. 
  For example, if the container uses the Alpine Linux image, 
  you might use `sh` instead of `bash`.

  example: "docker exec -it webtemplate sh"

- **Non-Interactive Mode:**
  If you don't need an interactive session, you can omit the `-it` flags:

Run `apt update` and install `git`:

```bash
apt update && apt install git -y
```

The choice between apt and yum depends on the Linux distribution used in the Docker image. 
In the provided lab steps, the base image is likely to be based on a Debian/Ubuntu distribution, 
where apt is the package manager. If you are using an image based on CentOS or RHEL, 
you would typically use yum instead.

Regarding the necessity of running apt update before installing any packages, 
it is a good practice to ensure that the package information on the system is up-to-date. 
This step fetches the latest package information from the repositories, 
allowing the package manager to make informed decisions about 
which versions of packages are available and compatible.

When you run apt install git -y, it installs Git and 
its dependencies from the repositories specified in the package manager's configuration. 
The package manager knows where to find the necessary packages because 
it consults the repository information obtained during the apt update step. 
The download link and package details are retrieved from the configured package repositories.

Here's a breakdown of the commands:

apt update: Fetches the latest package information from the configured repositories.
apt install git -y: Installs the Git package along with its dependencies, 
using the information obtained from the updated package repositories.

Clone the website code from GitHub:

```bash
git clone https://github.com/linuxacademy/content-widget-factory-inc.git /tmp/widget-factory-inc
```

Verify that the code was cloned successfully:

```bash
ls -l /tmp/widget-factory-inc/
```

List the files in the `htdocs/` directory:

```bash
ls -l htdocs/
```

Remove the `index.html` file:

```bash
rm htdocs/index.html
```

Copy the webcode from `/tmp/` to the `htdocs/` folder:

```bash
cp -r /tmp/widget-factory-inc/web/* htdocs/
```

Verify that they were copied over successfully:

```bash
ls -l htdocs/
```
NOTE:
In the provided lab instructions, the /tmp/widget-factory-inc/ directory is used as a temporary location
 to clone the website code from GitHub before copying it into the htdocs/ folder within the Apache HTTP Server container. 
 Let's break down the reasoning behind this choice:

Temporary Directory (/tmp/):
/tmp/ is often used as a temporary directory in Linux systems.
It is typically writable by all users, making it a convenient location for temporary operations.

Cloning the Repository:
The website code is cloned from the GitHub repository (https://github.com/linuxacademy/content-widget-factory-inc.git) 
into /tmp/widget-factory-inc/.
Cloning into a temporary directory allows for easy cleanup after the code is copied into the container.
htdocs/ vs. /var/www/html/:

The choice of htdocs/ is more of a convention, and it could be adjusted based on your specific container configuration.
In the context of Apache HTTP Server, the default web root is often set to /var/www/html/. 
However, the provided lab instructions use the htdocs/ directory, which is a common name for the web root in various setups.

Copying the Code:
After cloning the website code into /tmp/widget-factory-inc/, 
the cp command is used to copy the contents of this directory into the htdocs/ folder within the container.
The htdocs/ directory is a common choice for the web root directory where the web server looks for files to serve.

Alternative Locations:
It's possible to use different directories for the web root, 
such as /var/www/html/ or any other directory configured in the Apache server.
The key is to ensure that the web server is configured to look at the correct directory for serving content.
In summary, the use of /tmp/ as an intermediate location for cloning 
allows for flexibility in managing temporary files. The choice of htdocs/ 
as the destination for the website code within the container aligns with common practices for web server configurations. 
Alternative locations could be used based on specific requirements and configurations in a real-world scenario.

Exit the container:

```bash
exit
```

## Create an Image from the Container
Copy the Container ID:

```bash
docker ps
```

Create an image from the container:

```bash
docker commit <CONTAINER_ID> example/widgetfactory:v1
```
NOTE:
The `docker commit` command is used to create a new image from a container's changes. 
It essentially captures the current state of a container, including any modifications 
or additions made during its runtime, and saves it as a new image. 

Here's an explanation of each aspect of the command:
- `docker commit <CONTAINER_ID>`: This specifies the container from which the image should be created. 
You need to provide the unique identifier of the container using its Container ID.
- `example/widgetfactory:v1`: This is the name and tag of the new image. 
It follows the convention `<repository>/<image name>:<tag>`. 
The tag is optional, but it helps in versioning. 
In this case, `v1` is used to denote the first version of the image.

### Why use `docker commit` instead of `docker cp`?
- `docker cp` is used for copying files or 
directories from a container's file system to the host machine. 
It's a one-time copy, and the changes are not saved as a new image. 
`docker commit`, on the other hand, captures the entire container state 
and saves it as a reusable image.

### Is it a must to mention `v1`?
- No, it's not mandatory to mention a tag like `v1`. 
Tags are used for versioning or labeling different versions of the same image. 
If you don't specify a tag, Docker will default to `latest`. 
For example, `example/widgetfactory:latest`.

### Why use Container ID and not Container Name?
- Container IDs are unique identifiers for containers, ensuring specificity. 
Container names are not necessarily unique, and multiple containers could have the same name. 
Using the Container ID avoids ambiguity.

### What happens in the backend?
When you run the `docker commit` command, 
Docker takes a snapshot of the container's file system and packages it into a new image. 
It essentially creates a new layer on top of the existing image, incorporating any changes made in the container. 
This new image can then be used to run containers with the same state as the original container.

In summary, `docker commit` is used to persist changes made to a container as a new image, 
making it a convenient way to create customized images based on modifications in a running container.

Verify that the image was created successfully:

```bash
docker images
```

Take note of the image size.

## Clean up the Template for a Second Version
Log in to the container:

```bash
docker exec -it webtemplate bash
```

Remove the cloned code from the `/tmp/` directory:

```bash
rm -rf /tmp/widget-factory-inc/
```

Use `apt` to uninstall `git` and clean the cache:

```bash
apt remove git -y && apt autoremove -y && apt clean
```

Exit the container:

```bash
exit
```

Check the status of the container:

```bash
docker ps
```

Create an image from the updated container:

```bash
docker commit <CONTAINER_ID> example/widgetfactory:v2
```

Verify that both images are now running:

```bash
docker images
```

Delete the v1 image:

```bash
docker rmi example/widgetfactory:v1
```

## Run Multiple Containers from the Image
Run multiple containers using the new image:

```bash
docker run -d --name web1 -p 8081:80 example/widgetfactory:v2
docker run -d --name web2 -p 8082:80 example/widgetfactory:v2
docker run -d --name web3 -p 8083:80 example/widgetfactory:v2
```

Check the status of the containers:

```bash
docker ps
```

Stop the base webtemplate image:

```bash
docker stop webtemplate
```

Verify that only the created containers are running:

```bash
docker ps
```

Using a web browser, verify that the containers are running successfully:

```html
<SERVER_PUBLIC_IP_ADDRESS>:8081
<SERVER_PUBLIC_IP_ADDRESS>:8082
<SERVER_PUBLIC_IP_ADDRESS>:8083
```

# Conclusion
Congratulations — you've completed this hands-on lab!
```

You can use this Markdown document as a guide for documenting your steps on platforms like GitHub.