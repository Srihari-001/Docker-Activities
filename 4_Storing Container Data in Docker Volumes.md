## Introduction

Storing data within a container image is one option for automating a container with data, 
but it requires a copy of the data to be in each container you run.
For static files, this can be a waste of resources. Instead, 
you can store one copy of the static files in a Docker volume for easy sharing between containers.
In this lab, you will learn how Docker volumes interact with containers, 
create new volumes, attach them to containers, and clean up space left by anonymous volumes.

## Solution

**Log in to the server using the provided credentials:**

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>

or 

vagrant up and vagrant ssh (centos vagrant file)

Switch to this new user:

(name : "srihari") (password : "ComplexPwd@321")

su - srihari
```

**Discover Anonymous Docker Volumes:**

Check the Docker images:

```bash
docker images
```

Run a container using the postgres:12.1 repository:

```bash
docker run -d --name db1 postgres:12.1
```

Run a second container using the same image:

```bash
docker run -d --name db2 postgres:12.1
```

Check the status of the containers:

```bash
docker ps
```

List the anonymous volumes:

```bash
docker volume ls
```

Inspect the db1 container:

```bash
docker inspect db1 -f '{{ json .Mounts }}' | python -m json.tool
```
NOTE:
The `docker inspect` command is used to obtain detailed information about 
Docker objects, such as containers, images, networks, etc. 
In this case, you are inspecting a Docker container named `db1` with a specific format 
and using `python -m json.tool` to pretty-print the JSON output.

Let's break down the command:
- `docker inspect db1`: This part of the command specifies that you want to inspect the Docker container named `db1`.
- `-f '{{ json .Mounts }}'`: This part of the command uses the `-f` flag to format the output. 
It specifies a Go template that extracts the `.Mounts` field from the container's information. 
The `.Mounts` field contains information about the mounted volumes in the container.
- `| python -m json.tool`: This part of the command pipes the JSON output to `python -m json.tool`, 
which is a Python module for formatting JSON data. 
This is done to make the JSON output more readable.
The purpose of this command is to display detailed information about the mounted volumes in the `db1` container, 
including the source and destination paths of the volumes.

Alternative ways to achieve a similar result include using tools like `jq` 
or directly parsing the output with shell commands. Here's an example using `jq`:
{docker inspect db1 | jq '.[0].Mounts'}
This command uses `jq` to filter and display only the `.Mounts` field from the JSON output of `docker inspect db1`. 
The result should be similar to the original command's output.

Inspect the db2 container:

```bash
docker inspect db2 -f '{{ json .Mounts}}' | python -m json.tool
```

Create a third container using the --rm flag:

```bash
docker run -d --rm --name dbTmp postgres:12.1
```

Check the status of the container:

```bash
docker ps -a
```

List the anonymous volumes:

```bash
docker volume ls
```

Stop the db2 and dbTmp containers:

```bash
docker stop db2 dbTmp
```

List the anonymous volumes:

```bash
docker volume ls
```

Check the status of the containers:

```bash
docker ps -a
```

**Create a Docker Volume:**

Create a Docker volume:

```bash
docker volume create website
```
NOTE:
When you execute the command `docker volume create website`, 
Docker creates a named volume named "website" that is managed by Docker itself. 
Named volumes are created in the Docker volumes directory, usually located 
at `/var/lib/docker/volumes` on the host system. 
This command specifically creates a new, empty volume named "website."

Here's a breakdown of what happens in the backend:
1. **Creation of Volume Metadata:**
   Docker creates metadata for the volume, 
   storing information about the volume, 
   such as its name, driver (default is `local`), 
   and a unique identifier.

2. **Volume Directory:**
   A directory corresponding to the volume is 
   created in the Docker volumes directory (`/var/lib/docker/volumes`). 
   In this case, a directory named "website" would be created.

3. **Mountpoint:**
   The volume is associated with a mountpoint inside the containers. 
   The actual data stored in the volume is located 
   at this mountpoint within the containers.

Named volumes, created explicitly with `docker volume create`, 
are independent of any specific container. 
They exist to persistently store and share data between containers.
This is different from volumes created implicitly 
when you run a container with the `-v` flag or `--volume` option. 
In those cases, Docker creates anonymous volumes that are typically 
associated with the lifecycle of the container. 
These anonymous volumes are also managed in the Docker volumes directory, 
but they have auto-generated names.

To summarize, named volumes created with `docker volume create` are explicitly named, 
persist independently of containers, and can be used by multiple containers. 
On the other hand, volumes created implicitly during container creation are often tied 
to the container's lifecycle and might get automatically removed when the container is removed.

Verify that the volume was created successfully:

```bash
docker volume ls
```

Copy the widget-factory-inc data to the website container:

```bash
sudo cp -r /home/cloud_user/widget-factory-inc/web/* /var/lib/docker/volumes/website/_data/
```
NOTE:
Why do we need to copy? Can't we use /home/cloud_user/ folder as volume for containers?

The reason for copying is related to how Docker volumes work. 
When you mount a volume into a Docker container, 
it replaces the existing data in the container at the mount point with the contents of the volume. 
In this case, the /home/cloud_user/widget-factory-inc/web/ directory contains the data 
you want to use in your container. By copying it to the Docker volume, 
you ensure that the initial state of the volume in the container is set to the contents of this directory.
While it is possible to mount the /home/cloud_user/ folder as a volume, 
doing so would expose the entire home directory to the container, 
which might not be desirable for security and isolation reasons. 
It's generally a good practice to only expose the specific data or directories that your application needs.

Why only sudo for this cmd?

The sudo command is used to run the cp command with elevated privileges. 
In many systems, copying files to directories outside of your home directory (like /var/lib/docker/) 
requires elevated permissions. The sudo command allows the command to run with superuser (root) privileges, 
providing the necessary permissions to write to the Docker volume.
If the user running the command already has the required permissions 
to write to /var/lib/docker/volumes/website/_data/, then sudo may not be necessary. 
However, in many cases, Docker and its associated directories are owned by the root user, 
requiring sudo for write access.


List the copied data:

```bash
sudo ls -l /var/lib/docker/volumes/website/_data/
```

**Use the Website Volume with Containers:**

Run a Docker container with the website volume:

```bash
docker run -d --name web1 -p 80:80 -v website:/usr/local/apache2/htdocs:ro httpd:2.4
```
NOTE:

docker run: This command is used to run a container from a specified image.

-d: It stands for "detached" mode. 
This means the container runs in the background, 
and the terminal is not attached to the container's process.

--name web1: Assigns the name "web1" 
to the running container for easier identification and management.

-p 80:80: Maps port 80 on the host 
to port 80 on the container. 
This is necessary for accessing the web server running inside the container. 
You could use -p 8080:80 if you want to map it to a different port on your host.

-v website:/usr/local/apache2/htdocs:ro: Mounts the Docker volume named "website" 
to the /usr/local/apache2/htdocs directory inside the container in read-only mode. 

Let's break down the components:

website: The name of the Docker volume that will be mounted.

/usr/local/apache2/htdocs: The path inside the container where the volume will be mounted.

ro: Specifies that the mounted volume should be read-only. 
This is a security measure to ensure that the container cannot modify the files on the host.

httpd:2.4: Specifies the Docker image and its version to be used for the container. 
In this case, it's the Apache HTTP Server version 2.4.



Why not 8080:80 used?

You can use 8080:80 or any other port mapping based on your preference. 
The example uses 80:80, the default HTTP port, to make it more standard.

What is mounting?

Mounting is the process of associating a directory (in this case, a Docker volume) 
from the host machine with a directory inside the container. 
This allows data to be shared between the host and the container.
Why -v for mount?

The -v flag is used to specify the volume to mount. 
It tells Docker that you are performing a volume mount operation.

Why only to /usr/local/apache2/htdocs?

/usr/local/apache2/htdocs is the default document root directory for Apache HTTP Server. 
This is where the web server looks for files to serve. 
You can choose a different path based on your application's structure.

Why ro?

ro stands for "read-only." This is a security measure, 
ensuring that the container cannot modify the files on the host. 
If you need write access, you can use rw instead.

Will the path change for Nginx?

Yes, the path would typically change for Nginx. 
Nginx uses a different default document root, often /usr/share/nginx/html.

How will I know the path to copy the contents?

You can check the official documentation or Docker Hub page for the specific image you are using. 
It usually mentions the default document root or 
where you should place your web content inside the container.


Check the status of the container:

```bash
docker ps
```

In a web browser, verify connectivity to the container:

[PUBLIC_IP_ADDRESS](http://<PUBLIC_IP_ADDRESS>:80)

Run a second container with the --rm flag:

```bash
docker run -d --name webTmp --rm -v website:/usr/local/apache2/htdocs:ro httpd:2.4
```

Check the status of the containers:

```bash
docker ps
```

Stop the webTmp container:

```bash
docker stop webTmp
```

Check the status of the containers:

```bash
docker ps -a
```

Verify that the website can still be accessed through a web browser:

[PUBLIC_IP_ADDRESS](http://<PUBLIC_IP_ADDRESS>)

**Clean Up Unused Volumes:**

Clean up the unused volumes:

```bash
docker volume prune
```

When a container is stopped (but not removed), 
the data associated with its volumes is not immediately deleted. 
Docker retains the volume data, allowing you to potentially start the container again, 
and the data will still be available. 
This is why, when you run docker volume prune with stopped containers, 
it might not free up as much space as when the containers are removed.

On the other hand, when you remove a container (docker rm), 
Docker removes the association between the container and its volumes. 
If the volume is not shared with other containers, 
and there are no other references to it, 
Docker considers the volume as "unused" and eligible for pruning. 
This can result in a larger amount of data being freed up 
when you run docker volume prune.

Check the currently running containers:

```bash
docker ps -a
```

Remove the db2 container:

```bash
docker rm db2
```

Clean up the unused volumes again:

```bash
docker volume prune
```

List the current volumes:

```bash
docker volume ls
```

**Back Up and Restore the Docker Volume:**

Switch to the root user:

```bash
sudo -i
```

Find where the website volume data is stored:

```bash
docker volume inspect website
```

Copy the Mountpoint.

Back up the volume:

```bash
tar czf /tmp/website_$(date +%Y-%m-%d-%H%M).tgz -C /var/lib/docker/volumes/website/_data .
```
tar: It stands for "tape archive." 
While historically used for tape backups, 
it's commonly used for packaging files and directories together.

czf: These are options for the tar command:
c: Create a new archive.
z: Compress the archive using gzip.
f: Use archive file or device, specified next in the command.
/tmp/website_$(date +%Y-%m-%d-%H%M).tgz: This is the destination path and filename 
for the compressed archive. 

Let's break it down:
/tmp/: It specifies the target directory, in this case, the /tmp directory.
website_$(date +%Y-%m-%d-%H%M).tgz: This is the filename, and it has several components:
website_: A prefix for the filename.
$(date +%Y-%m-%d-%H%M): This is a command substitution. 
It gets replaced by the current date and time formatted as "Year-Month-Day-HourMinute." 
It helps create a unique filename, preventing overwrites.
.tgz: This is the file extension, indicating that it's a compressed archive using gzip.
-C /var/lib/docker/volumes/website/_data: This option changes the directory before performing any operations. 
In this case, it changes to /var/lib/docker/volumes/website/_data, 
and then . (dot) indicates that all files and directories in that location should be included in the archive.

Why tar?

Explanation: tar stands for "tape archive," 
and it's a command-line utility for compressing files and creating archives. 
In this context, tar is used to bundle the contents of the directory into a single file for easy transfer and backup.

Any other alternative to tar?

Explanation: While tar is commonly used for archiving and compressing files, 
there are alternatives like zip and gzip. 
However, the choice depends on the requirements 
and compatibility with the tools used in a particular environment.

Why czf?

Explanation:
c: Create a new archive.
z: Compress the archive using gzip.
f: Specify the filename of the archive.
This combination of options (czf) is used to create a compressed archive file.

Why .tgz?

Explanation: The file extension .tgz indicates that 
it's a tar archive compressed with gzip. 
It's a common convention to use this extension to denote a compressed tar file.

Why $ in $(date +%Y-%m-%d-%H%M)?

Explanation: The $() syntax is used for command substitution. 
It allows the output of the date command to be used as part of the filename. 
In this case, it includes the current date and time in the format "YYYY-MM-DD-HHMM."

Why (date +%Y-%m-%d-%H%M), why not any other alternative?

Explanation: The date command with the specified format 
ensures a standardized representation of the current date and time. 
Alternative formats may not provide the same level of clarity or sorting ability 
when dealing with multiple backup files.

Why -C, not -c or cp?

Explanation:
-C is used to change to the specified directory before performing any operations. 
In this case, it ensures that the command operates in the /var/lib/docker/volumes/website/_data directory.
-c is not applicable in this context. It is often used in other commands for different purposes.
cp is a command for copying files and directories. While it could be used for copying, 
it doesn't provide the archiving and compression features that tar offers.

Verify that the data was backed up properly:

```bash
ls -l /tmp/website_*.tgz
```
Note:
the * in this command is essentially saying "match any characters after website_ in the filename." 
It allows the command to list all files in the /tmp/ directory 
that start with website_ and have any characters after that. 
This is useful when there are multiple backup files with different timestamps, 
and you want to list or operate on all of them.

For example, if you have files like website_2023-11-03-1200.tgz 
and website_2023-11-03-1215.tgz, the * will match both, 
and the ls command will list details for both files.


List the contents of the tgz file:

```bash
tar tf <BACKUP_FILE_NAME>.tgz
```
NOTE:
tar: This is the command-line utility for manipulating tarball archives.
t: This option specifies that you want to list the contents of the archive.
f: This option indicates that the next argument is the filename of the archive.
<BACKUP_FILE_NAME>.tgz: This is the name of the compressed tarball file.
So, when you run tar tf <BACKUP_FILE_NAME>.tgz, 
you are instructing the tar command to list the contents 
of the compressed tarball file specified by <BACKUP_FILE_NAME>.tgz 
without extracting them. The tf options together mean "list files."

Exit out of root:

```bash
exit
```

Run a new container using the website volume, and create a backup:

```bash
docker run -it --rm -v website:/website -v /tmp:/backup bash tar czf /backup/website_$(date +%Y-%m-%d-%H-%M).tgz -C /website .
```
docker run: This command is used to run a new container.

-it: These are two options:
-i: Keep STDIN open even if not attached. This allows you to interact with the container.
-t: Allocate a pseudo-TTY. This provides an interactive terminal inside the container.
--rm: Automatically remove the container when it exits. This is useful for temporary containers.
-v website:/website: This option mounts the Docker volume named 
website into the container at the path /website. 
This allows the container to access the data stored in the website volume.
-v /tmp:/backup: This option mounts the local directory /tmp 
into the container at the path /backup. 
This allows the container to write backup files to the local /tmp directory.
bash: This is the command to run inside the container. 
It starts an interactive Bash shell.
tar czf /backup/website_$(date +%Y-%m-%d-%H-%M).tgz -C /website .: This command, 
executed inside the container, does the following:
tar czf /backup/website_$(date +%Y-%m-%d-%H-%M).tgz: Create a compressed tarball (.tgz file) 
with the current date and time in the filename. The -C /website option specifies the source directory, 
and . indicates that all files and directories in that directory should be included in the tarball.
The trailing . at the end of the tar command indicates that 
the contents of the /website directory should be included in the tarball.

Verify that the data was backed up properly:

```bash
ls -l /tmp/website_*.tgz
```

Switch to the root user:

```bash
sudo -i
```

Change to the /docker/volumes/ directory:

```bash
cd /var/lib/docker/volumes/
```

List the volumes:

```bash
ls -l
```

Change to the website/_data directory:

```bash
cd website/_data/
```

Remove the contents of the directory:

```bash
rm * -rf
```

Verify that the backups are still available:

```bash
ls -l /tmp/website_*.tgz
```

Restore the data to the current directory:

```bash
tar xf /tmp/<BACKUP_FILE_NAME>.tgz .
```

Verify that the data was restored successfully:

```bash
ls -l
```
