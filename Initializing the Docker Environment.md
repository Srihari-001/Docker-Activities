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

### Step 3: Adding Docker Repository

Add the CentOS-specific Docker repository using `yum-config-manager`:

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

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

Feel free to use this Markdown content and add it to your documentation or GitHub repository. This format provides a clear structure with explanations for each step.