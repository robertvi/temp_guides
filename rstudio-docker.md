# RStudio Docker
This guide explains how to build an RStudio docker container including the tidyverse set of packages as well as any custom R packages you require. It shows how to test your packages import correctly, how to run test scripts on test data before uploading the container to the TRE and finally running RStudio on your TRE data. We assume you have already installed [Docker Desktop](https://www.docker.com/products/docker-desktop/) on your local computer and are familiar with running basic commands in Powershell or a terminal, and with creating and editing small script files using a programmer's text editor such as [VSCode](https://code.visualstudio.com/).

## Build the RStudio Docker container
On your local machine create a file called `Dockerfile` using a text editor such as [VSCode](https://code.visualstudio.com/) containing the following and save it into a new empty folder:

```Dockerfile
#use a rocker.org base image containing RStudio and tidyverse
FROM rocker/tidyverse
```

This premade image contains RStudio and the tidyverse packages. See the [rocker project](https://rocker-project.org/images/versioned/rstudio.html) for details and other image types they provide. If you need to add custom R packages modify the file as follows (here we use caret, plotly and sf as the example additional R packages, where sf also requires the Ubuntu system packages libudunits2-dev libproj-dev and libgdal-dev to be installed first):

```Dockerfile
#use a rocker.org base image containing RStudio and tidyverse
FROM rocker/tidyverse

#install any system dependencies for your custom R packages like this:
RUN apt-get update && apt-get install -y libudunits2-dev libproj-dev libgdal-dev

#install your custom R packages like this:
RUN Rscript -e "install.packages(c('caret', 'plotly', 'sf'))"
```

Now open Powershell or a terminal into the same folder as the Dockerfile and build the docker image using (don't omit the final `.`):
```bash
docker build -t my-rstudio .
```

## Test import any custom R packages
It's a good idea to test your custom packages import correctly into R without throwing any dependency errors before you save and upload your new container to the TRE, as these can only be fixed with an internet connection which the TRE lacks. To test this run your new R container on your local computer in Powershell or a terminal using the following:

```bash
docker run --rm -it my-rstudio R
```

Once the R command line prompt appears do a test import of each of your custom packages:

```bash
library(caret)
library(plotly)
library(sf)
```

If any dependency errors appear paste them into an online search engine or AI chatbot to workout which Ubuntu system packages need to be installed. Stop the container using `quit()`. Add the required packages into the Dockerfile by appending the package names to the existing `apt-get install` command and remake the container using the `docker build` command again. Repeat until you can import all your R packages.

## Test run on local data

To run test data through your pipeline locally launch the container in its default mode, which starts RStudio. Here we assume any test scripts and data are somewhere under the current working where you enter the `docker run` command from, therefore we mount the current folder `.` to appear inside the container in the container's home folder `/home/rstudio` where you'll be able to access it from RStudio:

```bash
docker run --rm -it -p 8787:8787 -v .:/home/rstudio/shared my-rstudio
```

You should see a message saying what the password was set to. Open RStudio in your browser by visiting the URL `locahost:8787`, enter `rstudio` as the username and the password you were shown. You should now be able to see the files in the launch directory and run your test scripts on your test data. Add any additional R or system packages you find you require as a result of your tests to the Dockerfile and repeat the testing steps until you are happy with the container you've built.

## Save to tar file, ingress and load
Save the final tested image to a tar file:
```bash
docker save -o my-rstudio.tar my-rstudio
```

Ingress the `my-rstudio.tar` file into the TRE using the Airlock.

Then from your TRE desktop load the image into Docker:
```bash
docker load -i /shared/inbound/$(whoami)/my-rstudio.tar
```

## Run the container in the TRE
Run the container using the following command which will mount your home and shared folders so they appear under the home folder within the container. Be sure to save anything you need to keep under one of these folders as everything else in the container will be deleted once it stops running. Note also that the container is working directly with your data in the shared folders, not a temporary copy of them, so use all the normal precautions when working with important files.

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

You may want to change the password from `rstudio`, although you should be the only user able to access RStudio running on your TRE desktop due to the lack of any internet connection. If you remove `-e PASSWORD=rstudio` altogether it will set a different random password each time you launch the container.

Open Firefox ('Activities', then Search 'Firefox') and navigate to `http://localhost:8787`, enter `rstudio` as the username and the password you chose. You should now be able to access all your data and scripts on `/home` and `/shared` from within RStudio and have all the packages you installed.

Once done using it, the container can be stopped by going to the terminal window you started it from and hitting `CTRL-C`.