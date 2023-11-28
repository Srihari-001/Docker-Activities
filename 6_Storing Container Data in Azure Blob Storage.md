# Storing Container Data in Azure Blob Storage

## Introduction

Using Docker volumes is the preferred method of locally storing container data. 
Volume support is built directly into Docker, making it an easy tool to use for storage, 
as well as more portable. However, storing container data in Docker volumes 
still requires us to back up the data in those volumes on our own. 
There is another option: storing our container data in the Cloud. 
It's not a solution for every problem, but this lab demonstrates another tool at our disposal.

This lab shows how to mount a Blob Storage container onto our local system as a directory. 
We will then mount that directory into our Docker container. 
We will use an httpd container to serve the contents of that bucket as a webpage, 
but we can use it to share any common data between containers. 
This will demonstrate how flexible Docker can be. 
We can make changes to our bucket, and all our containers using the Blob Storage 
container will near-instantly have access to the content.

## Solution

### Log in to the server using the credentials provided:

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

### Configuration and Installation

1. **Obtain Azure Login Credentials:**

   ```bash
   az login
   ```

   - Copy the code provided by the command.
   - Open a browser and navigate to https://microsoft.com/devicelogin.
   - Enter the code copied in a previous step and click Next.
   - Use the login credentials from the lab page to finish logging in.

   Switch back to the terminal and wait for the confirmation.

2. **Prepare the Storage:**

   - Find the name of the Storage account:

     ```bash
     az storage account list | grep name | head -1
     ```
	az storage account list: This Azure Command-Line Interface (CLI) command is used 
	to list all the storage accounts associated with the current Azure subscription.

	|: This is a pipe operator, and it is used to pass the output of the command 
	on its left as input to the command on its right.

	grep name: This is a Unix command-line utility for searching plain-text data sets 
	for lines that match a regular expression. In this case, 
	it is filtering lines containing the word "name."

	head -1: This command is used to display the first few lines of a text file or pipeline. 
	In this context, it is extracting the first line from the filtered output.

     Copy the name of the Storage account to the clipboard.

   - Export the Storage account name:

     ```bash
     export AZURE_STORAGE_ACCOUNT=<COPIED_STORAGE_ACCOUNT_NAME>
     ```
	 Note:
	 here we are assigning the account name to the variable, to use that variable in next steps instead of entire account name
	 and for better readability

   - Retrieve the Storage access key:

     ```bash
     az storage account keys list --account-name=$AZURE_STORAGE_ACCOUNT
     ```

     Copy the key1 "value" for later use.

   - Export the key value:

     ```bash
     export AZURE_STORAGE_ACCESS_KEY=<KEY1_VALUE>
     ```
	 Note:
	 here we are assigning the key value to the variable, to use that variable in next steps instead of entire account name
	 and for better readability

   - Install blobfuse:

     ```bash
     sudo rpm -Uvh https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm
     sudo yum install blobfuse fuse -y
     ```
	sudo: This command is used to execute the subsequent command with superuser privileges. 
	It stands for "superuser do."

	rpm: This is the Red Hat Package Manager, a command-line package management tool 
	used on Red Hat-based Linux distributions. It's used for installing, updating, 
	and managing software packages.

	-Uvh: These are options for the rpm command:

	-U: Upgrade the package if it is already installed or install it if not.
	-v: Provide verbose output, displaying detailed information about the installation progress.
	-h: Display hash marks (#) during package installation to indicate progress.
	https://packages.microsoft.com/config/rhel/7/packages-microsoft-prod.rpm: 
	This is the URL to the Microsoft repository configuration package for RHEL 7. 
	It contains the necessary configuration files to enable the system to access Microsoft's repository 
	for installing software packages.

	Explanation:

	The command is necessary because it sets up the system to access Microsoft's repository, 
	allowing the installation of Microsoft software packages. 
	In this specific case, it's preparing the system for the installation of blobfuse—a tool 
	for mounting Azure Blob Storage as a file system.
	
	sudo: This command is used to execute the subsequent command with superuser privileges. 
	It stands for "superuser do."

	yum: This is the package manager for Red Hat-based Linux distributions. 
	It is used for installing, updating, and removing software packages on the system.

	install: This is the subcommand used with yum to install one or more packages.

	blobfuse: This is the package name for the Azure Storage FUSE driver (blobfuse). 
	It allows you to mount an Azure Blob Storage container as a file system.

	fuse: This is the Filesystem in Userspace (FUSE) package. 
	FUSE allows non-privileged users to create their own file systems without 
	modifying the kernel code. It's essential for the operation of blobfuse.

	-y: This flag is used to automatically answer "yes" 
	to prompts during the installation process, 
	avoiding the need for manual confirmation.

	Explanation:

	blobfuse Package:

	blobfuse is a user-mode file system driver for Azure Blob Storage. 
	It enables you to mount a Blob Storage container as a directory on your Linux file system.
	This is particularly useful when you want to access and 
	interact with Azure Blob Storage using standard file I/O commands.
	fuse Package:

	FUSE (Filesystem in Userspace) is a software interface that 
	allows non-privileged users to create their own file systems 
	without modifying the kernel code.
	FUSE enables blobfuse to operate in user space, providing a bridge between user space 
	and the kernel for file system operations.
	Installation Process:

	The yum install command installs the specified packages and their dependencies from the configured repositories.
	The -y flag is used to automatically answer "yes" to any prompts that may appear during the installation process.

   - Modify the fuse.conf configuration file:

     ```bash
     sudo sed -ri 's/# user_allow_other/user_allow_other/' /etc/fuse.conf
     ```
	Note: we are giving non-root users to have privilege to mount File system.
	sudo: Executes the command with superuser privileges.

	sed: Stream editor for filtering and transforming text.

	-ri: Options for sed:

	-r: Interpret regular expressions without escaping special characters.
	-i: Edit files in-place (i.e., save back to the original file).
	's/# user_allow_other/user_allow_other/': The sed command to search for the specified pattern 
	(# user_allow_other) and replace it with the desired string (user_allow_other).

	/etc/fuse.conf: The file path where the changes should be applied, in this case, the FUSE configuration file.

	The reason for this command is that the FUSE (Filesystem in Userspace) 
	configuration file (/etc/fuse.conf) often contains commented lines, 
	and these comments can prevent certain configurations from taking effect. 
	In this case, uncommenting the user_allow_other line allows non-root users to mount FUSE filesystems.

3. **Use the Azure Blob Storage Container:**

   - Create necessary directories:

     ```bash
     sudo mkdir -p /mnt/widget-factory /mnt/blobfusetmp
     ```
	sudo: This prefix is used to execute the following command with elevated privileges, typically as a superuser.

	mkdir: This command is used to create directories.

	-p: This option ensures that the command creates parent directories as needed. 
	If the parent directories do not exist, they are created.

	/mnt/widget-factory: This is the first directory being created. 
	It is intended to be the mount point for the Azure Blob Storage container.

	/mnt/blobfusetmp: This is the second directory being created. 
	It is intended to be used as a temporary path for BlobFuse, 
	a tool that facilitates mounting Azure Blob Storage as a file system. 
	The temporary directory is used for caching purposes.

	In summary, these two directories serve different purposes:

	/mnt/widget-factory: This directory is the main mount point for Azure Blob Storage. 
	Files from the Blob Storage container will be accessible here.

	/mnt/blobfusetmp: This directory is used by BlobFuse as a temporary cache. 
	It helps optimize performance by storing temporary data to 
	reduce the need for repeated interactions with Azure Blob Storage.

   - Change ownership of the directories:

     ```bash
     sudo chown cloud_user /mnt/widget-factory/ /mnt/blobfusetmp/
     ```

   - Mount the Blob Storage from Azure:

     ```bash
     blobfuse /mnt/widget-factory --container-name=website --tmp-path=/mnt/blobfusetmp -o allow_other
     ```

   - Copy website files into the Blob Storage container:

     ```bash
     cp -r ~/widget-factory-inc/web/* /mnt/widget-factory/
     ```

   - Verify the copy worked:

     ```bash
     ll /mnt/widget-factory/
     ```

   - Verify the files made it to Azure Blob Storage:

     ```bash
     az storage blob list -c website --output table
     ```

4. **Run a Docker Container:**

   ```bash
   docker run -d --name web1 -p 80:80 --mount type=bind,source=/mnt/widget-factory,target=/usr/local/apache2/htdocs,readonly httpd:2.4
   ```

   - Once the command is complete, open a web browser and navigate to the public IP address of the server.
   - Verify the website is up and running.

## Conclusion

Congratulations, you've completed this hands-on lab!