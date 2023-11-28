# Storing Container Data in AWS S3

## Introduction

Using Docker volumes is the preferred method of storing container data locally. 
Volume support is built directly into Docker, making it an easy tool to use for storage, 
as well as more portable. However, storing container data in Docker volumes still requires 
you to back up the data in those volumes on your own.

There is another option - storing your container data in the cloud. 
It's not a solution for every problem, but after this lab, 
you'll have another tool at your disposal.

This lab will show you how to mount an S3 bucket onto your local system as a directory. 
You will then mount that directory into your Docker container. 
We will use an httpd container to serve the contents of that bucket as a webpage, 
but you can use it to share any common data between containers.

This will demonstrate how flexible Docker can be. 
You can make changes to your bucket and all of your containers using 
the S3 bucket will near-instantly have access to the content.

## Solution

1. **Log in to the server using the credentials provided:**

   ```bash
   ssh cloud_user@<PUBLIC_IP_ADDRESS>
   ```

2. **Configuration and Installation:**

   Install the awscli, while checking if there are any versions currently installed, and not stopping any user processes:

   ```bash
   pip install --upgrade --user awscli
   ```
	
	Note:
	pip: It's the package installer for Python, 
	commonly used for installing Python packages and managing dependencies.
	install: This is the command to install a Python package.
	--upgrade: This flag ensures that if an older version of the AWS CLI is already installed, 
	it will be upgraded to the latest version.
	--user: This flag installs the AWS CLI in the user's home directory, 
	specifically in the user's Python site-packages directory. 
	This allows users to install and manage Python packages without requiring administrative (root) access.
   
   Configure the CLI:

   ```bash
   aws configure
   ```

   Enter the following:
   - AWS Access Key ID: `<ACCESS_KEY_ID>`
   - AWS Secret Access Key: `<SECRET_ACCESS_KEY>`
   - Default region name: `us-east-1`
   - Default output format: `json`

   Copy the CLI configuration to the root user:

   ```bash
   sudo cp -r ~/.aws /root
   ```
   Note:
   sudo: It is a command that allows a permitted user to execute a command as the superuser or another user, 
   as specified in the sudoers file. In this context, 
   it provides the necessary privileges to copy files to the root user's directory.

	cp: It is the copy command in Linux/Unix.

	-r: It stands for recursive. This option is used when copying directories. 
	It allows the cp command to copy the entire directory and its contents.

	~/.aws: This is the path to the AWS CLI configuration directory 
	in the home directory of the current user. 
	The tilde (~) represents the home directory.

	/root: This is the destination directory. 
	It represents the home directory of the root user.

	Explanation:

	The AWS CLI configuration typically resides in the home directory of the user 
	who configured AWS CLI, under the .aws directory. 
	The root user also needs access to the AWS CLI configuration, 
	especially if administrative tasks need to be performed with elevated privileges. 
	By copying the configuration to the root user's home directory, 
	you ensure that AWS CLI commands can be executed seamlessly by both the current user and the root user.

   Install the s3fs package:

   ```bash
   sudo yum install s3fs-fuse -y
   ```
   Note:
   sudo: This command is used to execute the subsequent command with superuser (root) privileges. 
   It allows users to perform administrative tasks that would otherwise be restricted.

	yum: This is the default package manager for CentOS and other Red Hat-based Linux distributions. 
	It is used for installing, updating, and managing software packages on the system.

	install: This is a yum subcommand that is used to install packages on the system.

	s3fs-fuse: This is the name of the package that is being installed. 
	s3fs-fuse is a FUSE (Filesystem in Userspace) driver that allows mounting Amazon S3 buckets as local directories.

	-y: This is an option for yum that automatically answers "yes" to any prompts, 
	allowing for non-interactive installations. 
	It's useful when you want to script or automate installations without manual intervention.

	Now, let's understand why this command is used:

	s3fs-fuse Package: The s3fs-fuse package provides the necessary tools and 
	libraries to mount an Amazon S3 bucket as a file system using FUSE. 
	This allows you to interact with S3 storage using standard file system operations.

	Installation: The command is used to install the s3fs-fuse package on the system. 
	Without this package, you wouldn't have the required tools to mount and 
	interact with an S3 bucket as a file system.

	Dependency Resolution: YUM automatically resolves and installs 
	any dependencies required by the s3fs-fuse package, ensuring that the 
	necessary components are present for the software to function correctly.

3. **Prepare the Bucket:**

   Create a mount point for the s3 bucket:

   ```bash
   sudo mkdir /mnt/widget-factory
   ```

   Export the bucket name:

   ```bash
   export BUCKET=<S3_BUCKET_NAME>
   ```
	export: This command is used in Unix-like operating systems (including Linux) 
	to set the value of an environment variable. 
	Environment variables are dynamic-named values that affect the way running processes behave on a computer.

	BUCKET: This is the name of the environment variable being created or updated. 
	Environment variables are often used to store configuration settings or 
	other values that various programs or scripts can reference.

	<S3_BUCKET_NAME>: This is a placeholder for the actual name of the S3 bucket. 
	The user is expected to replace <S3_BUCKET_NAME> with the real name of their S3 bucket.

	The reason for using this command in the context of the AWS S3 lab is 
	to create an environment variable (BUCKET) that stores the name of the S3 bucket. 
	This variable can then be referenced later in the script or commands, 
	providing a convenient and flexible way to manage and 
	reuse the S3 bucket name throughout the lab. 
	It enhances script readability, maintainability, and 
	allows for easy updates if the bucket name needs to be changed.

   Mount the S3 bucket:

   ```bash
   sudo s3fs $BUCKET /mnt/widget-factory -o allow_other -o use_cache=/tmp/s3fs
   ```
	sudo: Executes the command with superuser privileges. 
	Many operations related to file systems and mounts require elevated permissions.

	s3fs: This is the command used to mount an S3 bucket as a local file system. 
	It is a user-space file system for interacting with Amazon S3.

	$BUCKET: This is a variable representing the name of the S3 bucket. 
	The actual bucket name is provided when you export it earlier in the script.

	/mnt/widget-factory: This is the local mount point on the server 
	where the contents of the S3 bucket will be accessible. 
	You are creating a directory named "widget-factory" under "/mnt/" to serve as the mount point.

	-o allow_other: This option allows access to the mounted file system 
	by users other than the one who mounted it. 
	In this case, it enables other users (not just the user who executed the mount command) 
	to access the mounted S3 bucket.

	-o use_cache=/tmp/s3fs: This option specifies the cache directory for s3fs. 
	It helps improve performance by caching data locally. 
	The "/tmp/s3fs" directory is used as the cache directory.

	This command essentially mounts the specified S3 bucket onto 
	the local file system at "/mnt/widget-factory" using the s3fs utility. 
	The additional options (allow_other and use_cache=/tmp/s3fs) are used 
	to configure permissions and caching for the mounted S3 bucket.

   Verify that the bucket was mounted successfully:

   ```bash
   ll /mnt/widget-factory
   ```

   Copy the website files to the s3 bucket:

   ```bash
   cp -r ~/widget-factory-inc/web/* /mnt/widget-factory
   ```
   Note:
   Here, we have already mounted s3 bucket file system on mount point "/mnt/widget-factory",
   there are website contents in vm local dir "~/mnt/widget-factory", is copied to
   another dir in vm named "/mnt/widget-factory"(mount point), which is associated with S3 bucket,
   so when we copy these contents from local web folder mount point, we can view these contents in both
   mnt/ folder and s3 ls list

   Verify the files are in the folder:

   ```bash
   ll /mnt/widget-factory
   ```

   Verify the files are in the s3 bucket:

   ```bash
   aws s3 ls s3://$BUCKET
   ```

4. **Use the S3 Bucket Files in a Docker Container:**

   Run an httpd container using the S3 bucket:

   ```bash
   docker run -d --name web1 -p 80:80 --mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly httpd:2.4
   ```
	docker run: This command is used to run a container from a specified image.

	-d: This option runs the container in the background, making it a detached mode. 
	The container runs independently, and the terminal is freed up for other commands.

	--name web1: This option assigns a name to the container, in this case, "web1". 
	Naming containers can make it easier to manage them.

	-p 80:80: This option maps port 80 from the host to port 80 on the container. 
	This allows external access to the container's web server.

	--mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly: 
	This option mounts a bind mount. It links a directory or file on 
	the host machine (/mnt/widget-factory) to a directory in the container (/usr/local/apache2/htdocs). 
	The readonly flag indicates that the mount is read-only, preventing any changes to 
	the mounted directory from within the container.

	httpd:2.4: This is the name of the Docker image and 
	its version (tag) that you want to run. 
	In this case, it's the Apache HTTP Server version 2.4.

	Explanation:
	This command starts an Apache HTTP Server container (httpd:2.4) in detached mode, 
	named "web1", mapping port 80 from the host to the container, 
	and using a bind mount to link the host directory /mnt/widget-factory to 
	the container directory /usr/local/apache2/htdocs in read-only mode.

	The purpose of using this command is to serve the contents of the S3 bucket, 
	which was mounted onto the host machine at /mnt/widget-factory, 
	through an Apache web server running in the Docker container. 
	The readonly flag ensures that the container can't modify the content of 
	the mounted directory, maintaining the read-only nature of the S3 bucket data within the container.

   In a web browser, verify connectivity to the container:

   `<SERVER_PUBLIC_IP_ADDRESS>`

   Check the bucket cache:

   Change to the `/mnt/widget-factory/` directory:

   ```bash
   cd /mnt/widget-factory
   ```

   Create a new page within the bucket:

   ```bash
   cp index.html newPage.html
   ```
	cp: This is the command itself for copying files.

	index.html: This is the source file that you want to copy. 
	It is assumed that there is an existing file 
	named index.html in the current working directory.

	newPage.html: This is the destination file 
	where the content of index.html will be copied. 
	If newPage.html already exists, it will be overwritten with the content of index.html.

	Explanation:

	This command is used to create a copy of the index.html file 
	and name the copy as newPage.html. It's a simple way to duplicate 
	the content of an existing file under a different name. 
	In the context of the mentioned lab, it could be part of demonstrating 
	how changes to files in the S3 bucket are reflected in the Docker container, 
	showcasing the flexibility and synchronization capabilities of the system.

   In a web browser, verify that the new page is accessible:

   `<SERVER_PUBLIC_IP_ADDRESS>/newPage.html`

   Verify that the page was added to the bucket:

   ```bash
   aws s3 ls $BUCKET
   ```

## Conclusion

Congratulations — you've completed this hands-on lab!
```
