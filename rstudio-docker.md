# RStudio Docker

On your local machine create a file called `Dockerfile` using a text editor such as [VSCode](https://code.visualstudio.com/) containing the following and save it into a new empty folder:

```Dockerfile
FROM rocker/tidyverse
```

This premade image contains RStudio and tidyverse packages. See the [rocker project](https://rocker-project.org/images/versioned/rstudio.html) for details and other image types they provide.

Now open Powershell or a terminal into the same folder as the Dockerfile and build the docker image using:
```bash
docker build -t my-rstudio .
```

Save the image to a tar file:
```bash
docker save -o my-rstudio.tar my-rstudio
```

Ingress the `my-rstudio.tar` file into the TRE using the Airlock.

Then from your TRE desktop load the image:
```bash
docker load -i /shared/inbound/$(whoami)/my-rstudio.tar
```

Run the container using the following command which will mount your home and shared folders so they appear under the home folder within the container. Be sure to save anything you need to keep under one of these folders as everything else in the container will be deleted once it stops running.

```bash
docker run -it --rm \
  -e PASSWORD=rstudio \
  -v /shared:/home/rstudio/shared \
  -v /home/$(whoami):/home/rstudio/home \
  -p 8787:8787 \
  --userns=keep-id:uid=1000,gid=100 \
  --security-opt=label=disable \
  my-rstudio
```

You may want to change the password from `rstudio`, although you should be the only user able to access RStudio running on your TRE desktop, which in anycase only has access to the same files you can see already without logging in to it.

Open Firefox ('Activities', then Search 'Firefox') and navigate to `http://localhost:8787`, enter `rstudio` as the username and the password you chose.

Once done using it, the container can be stopped by going to the terminal window you started it from and hitting `CTRL-C`.