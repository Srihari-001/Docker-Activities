# Building Container Images Using Dockerfiles

## Introduction

Creating a container image by hand is possible, but it requires manual processes. There has to be a more automatic way to build images. Manual processes do not scale and are not easily version controlled. Docker provides a solution to this problem - the Dockerfile. In this lab, you will create a Dockerfile to build an image and host a static website.

## Solution

### Log in to the server using the credentials provided:

```bash
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

### Build a First Version

1. Change to the widget-factory-inc directory:

   ```bash
   cd widget-factory-inc
   ```

2. Create a Dockerfile that uses httpd:2.4 as the base image:

   ```bash
   vim Dockerfile
   ```

   In the new file, insert the following:

   ```Dockerfile
   FROM httpd:2.4
   RUN apt update -y && apt upgrade -y && apt autoremove -y && apt clean && rm -rf /var/lib/apt/lists*
   ```

   Save the file:

   - Press `ESC`
   - Type `:wq`

3. Verify that the file was saved successfully:

   ```bash
   cat Dockerfile
   ```

4. Build the 0.1 version of the widgetfactory image using the Dockerfile:

   ```bash
   docker build -t widgetfactory:0.1 .
   ```

5. Set variables to examine the image's size and layers:

   ```bash
   export showLayers='{{ range .RootFS.Layers }}{{ println . }}{{end}}'
   export showSize='{{ .Size }}'
   ```

6. Compare the httpd and widgetfactory images:

   ```bash
   docker images
   ```

7. Show the widgetfactory image's size:

   ```bash
   docker inspect -f "$showSize" widgetfactory:0.1
   ```

8. Show the layers:

   ```bash
   docker inspect -f "$showLayers" widgetfactory:0.1
   ```

9. Show the layers of the httpd:2.4 image:

   ```bash
   docker inspect -f "$showLayers" httpd:2.4
   ```

10. Compare the layers. Are they the same?

### Load the Website into the Container

11. Open the Dockerfile:

    ```bash
    vim Dockerfile
    ```

12. Remove the Apache welcome page from the image by adding the following:

    ```Dockerfile
    RUN rm -f /usr/local/apache2/htdocs/index.html
    ```

    Save the file:

    - Press `ESC`
    - Type `:wq`

13. Build version 0.2 of the widgetfactory image:

    ```bash
    docker build -t widgetfactory:0.2 .
    ```

14. Inspect both versions of the widgetfactory image to see the differences in size and layers:

    ```bash
    docker images
    ```

15. Show the widgetfactory:0.1 image's size:

    ```bash
    docker inspect -f "$showSize" widgetfactory:0.1
    ```

16. Compare it to the image size for widgetfactory:0.2:

    ```bash
    docker inspect -f "$showSize" widgetfactory:0.2
    ```

17. Using an interactive terminal, check the htdocs folder for widgetfactory:0.2. Are the website files in the folder?:

    ```bash
    docker run --rm -it widgetfactory:0.2 bash
    ls htdocs
    ```

    Exit the container:

    ```bash
    exit
    ```

18. Show the layers for the widgetfactory:0.1 image:

    ```bash
    docker inspect -f "$showLayers" widgetfactory:0.1
    ```

19. Show the layers for the widgetfactory:0.2 image and compare the two:

    ```bash
    docker inspect -f "$showLayers" widgetfactory:0.2
    ```

20. Open the Dockerfile:

    ```bash
    vim Dockerfile
    ```

21. Add the website data to the container by adding the following to the end of the file:

    ```Dockerfile
    WORKDIR /usr/local/apache2/htdocs
    COPY ./web .
    ```

    Save the file:

    - Press `ESC`
    - Type `:wq`

22. Build version 0.3 of the widgetfactory image:

    ```bash
    docker build -t widgetfactory:0.3 .
    ```

23. Inspect versions 0.2 and 0.3 to see the differences in size and layers:

    ```bash
    docker images
    ```

24. Show the widgetfactory:0.2 image's size:

    ```bash
    docker inspect -f "$showSize" widgetfactory:0.2
    ```

25. Compare it to the image size for widgetfactory:0.3:

    ```bash
    docker inspect -f "$showSize" widgetfactory:0.3
    ```

26. Show the layers for the widgetfactory:0.2 image:

    ```bash
    docker inspect -f "$showLayers" widgetfactory:0.2
    ```

27. Show the layers for the widgetfactory:0.3 image and compare the two:

    ```bash
    docker inspect -f "$showLayers" widgetfactory:0.3
    ```

28. Using an interactive terminal, check the htdocs folder for widgetfactory:0.3:

    ```bash
    docker run --rm -it widgetfactory:0.3 bash
    ls -l
    ```

### Run a Container from the Image

29. Run a container from the widgetfactory:0.3 image. What command does it use to run the server? Remember to publish the web server port:

    ```bash
    docker run --name web1 -p 80:80 widgetfactory:0.3
    ```

30. Exit the container:

    - Press `CTRL+C`

31. Check the status of the container:

    ```bash
    docker ps -a
    ```

32. Start the container:

    ```bash
    docker start web1
    ```

33. Using docker exec connect to the web1 container:

    ```bash
    docker exec -it web1 bash
    ```

34. View the website files in the container:

    ```bash
    ls -l
    ```

35. Exit the container:

    ```bash
    exit
    ```

36. Retrieve the main website page from the container:

    ```bash
    wget localhost
    ```

37. Compare it to the copy on the server:

    ```bash
    diff index.html web/index.html
    ```

##

 Read Me

Note: Many of the things done in this lab are intentionally inefficient. These tasks are done to demonstrate certain aspects of Docker and highlight common pitfalls. They do not represent best practices.

## Conclusion

Congratulations — you've completed this hands-on lab!