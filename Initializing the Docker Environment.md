### Step 1: Log in to the Server

Log in to the server using the provided credentials:

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>

or 

vagrant up and vagrant ssh (centos vagrant file)

Switch to this new user:

(name : "srihari") (password : "ComplexPwd@321")

su - srihari

```

### Step 2: Installing Docker

To use Docker, we first need to install the necessary prerequisites. Execute the following commands:

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
yum-utils:
Purpose: yum-utils is a collection of utilities and plugins for the yum package manager. 
It provides additional functionality beyond the basic package management features.
Reason for Docker: Some Docker installation scripts and tools use yum-config-manager, 
a utility provided by yum-utils, to manage repositories and enable specific configurations.

device-mapper-persistent-data:
Purpose: This package contains the device-mapper storage driver's persistent metadata. 
Device Mapper is one of the storage drivers used by Docker to manage the storage of container images and filesystem layers.
Reason for Docker: Docker relies on storage drivers to manage the layers of container images efficiently. 
The device-mapper storage driver is one of the options available for Docker to manage container storage.

lvm2:
Purpose: lvm2 stands for Logical Volume Manager, 
a system for managing logical volumes or filesystems that use the device-mapper driver.
Reason for Docker: Docker uses storage drivers, and 
lvm2 provides the necessary tools for managing logical volumes,
which can be utilized by the device-mapper storage driver.

### Step 3: Adding Docker Repository

Add the CentOS-specific Docker repository using `yum-config-manager`:

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

In Step 3, adding the CentOS-specific Docker repository using yum-config-manager is essential 
to ensure that you are installing Docker from an official and up-to-date repository. 
Let's break down why this step is necessary:

Official Repository:
The Docker packages provided by the official Docker project are hosted in a repository managed by Docker.
Adding this repository ensures that you get the latest and official releases of Docker.
Using the official repository is recommended as it guarantees the authenticity and integrity of the Docker packages.

Latest Versions:
Docker regularly releases updates and new versions with bug fixes, security patches, and additional features. 
Adding the Docker repository allows you to install the latest version available for your system.
This step helps you stay current with the latest improvements and 
ensures that you have access to the most recent Docker features.

Compatibility with CentOS:
Adding the CentOS-specific Docker repository ensures that the Docker packages are configured and 
optimized for compatibility with CentOS, which is a popular Linux distribution.
This step helps avoid potential issues related to package versions, dependencies, and 
configurations that might arise when using generic repositories.

Ease of Installation:
Adding the Docker repository simplifies the installation process. 
Once the repository is added, you can install Docker using the package manager (yum in this case) without having to manually download 
and configure the package.

Convenient Updates:
When Docker releases updates, you can easily update your Docker installation to the latest version by using the package manager. 
This ensures that your Docker environment remains secure and up-to-date.

### Step 4: Installing Docker

Install Docker Community Edition:

```bash
sudo yum -y install docker-ce
```

### Step 5: Enabling the Docker Daemon

To ensure that Docker starts automatically with the system, enable and start the Docker daemon:

```bash
sudo systemctl enable --now docker
```

### Step 6: Configuring User Permissions

Add the lab user to the Docker group to run Docker commands without using `sudo`. Note that you need to log out and log back in for the changes to take effect:

```bash
sudo usermod -aG docker srihari
```

### Step 7: Running a Test Image

Verify that your Docker environment is set up correctly by running a test container:

```bash
docker run hello-world
```

## Conclusion

Congratulations! You've successfully completed this hands-on lab. Your Docker environment is now configured, and you're ready to start working with containers.
```