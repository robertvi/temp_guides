# Docker Jupyter
This guide explains how to build a Jupyter Lab / Notebook docker container. We assume you have already installed [Docker Desktop](https://www.docker.com/products/docker-desktop/) on your local computer and are familiar with running basic commands in Powershell or a terminal, and with creating and editing small script files using a programmer's text editor such as [VSCode](https://code.visualstudio.com/).

## Creating a JupyterLab image with custom conda packages
Here we use numpy, pandas and matplotlib as examples packages to add to your docker image, but you can change these to any conda package(s) you require.

On your local machine create a file called `Dockerfile` using a text editor such as [VSCode](https://code.visualstudio.com/) containing the following and save it into a new empty folder:

```Dockerfile
FROM quay.io/jupyter/minimal-notebook:2025-03-14
RUN conda install numpy pandas matplotlib
RUN rmdir /home/jovyan/work
CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]
```

Edit the list of conda packages to include everything you need. Note the `rmdir /home/jovyan/work` command above removes a folder from the container so that you don't accidentally save anything important there, as it will get deleted when the container stops running as we are using the `--rm` option with docker run below.

Now open Powershell or a terminal into the same folder as the Dockerfile and build the docker image using:
```bash
docker build -t my-jupyter-lab .
```

## Test all your modules import correctly
Optionally, but high recommended, here we run the container locally and do a test import of all the python modules before uploading to the TRE, as we can fix any missing packages only when we have an internet connection. Launch the container into an interactive python prompt:

```bash
docker run --rm -it my-jupyter python3
```

When the python prompt appears we import all the custom packages you added to the container (in this example numpy, pandas and matplotlib):

```python
import numpy
import pandas
import matplotlib
exit()
```

Any missing python modules will cause an import error. Use the error message to work out which additional conda packages are required, add them to your Docker file and repeat the build and test process until you can import everything successfully.

## Test using jupyter lab itself
Alternatively or additionally you may want to run tests inside jupyterlab itself on your local machine. Here we mount your current folder into the container so that any test scripts or data there can be accessed from within jupyterlab using the following command in Powershell or the terminal:

```bash
docker run --rm -p 8888:8888 -v .:/home/jovyan/shared my-jupyter
```

Once it launches it should print a message giving you the URL you need to open in your browser in order to access jupyter lab, it should look something like `http://127.0.0.1:8888/lab?token=...`. Once you are finished testing jupyterlab type `CTRL+C` into the window you used to launch the docker command from.

## Move the container into the TRE

Save the image to a tar file:
```bash
docker save -o my-jupyter-lab.tar my-jupyter-lab
```

Ingress the `my-jupyter-lab.tar` file into the TRE using the Airlock.

Then from your TRE desktop load the image:
```bash
docker load -i /shared/inbound/$(whoami)/my-jupyter-lab.tar
```

You now launch the container mounting whichever folder(s) you will need to access. To be able to see them from within jupyterlab you must tell docker to mount them inside the container under the default home folder of the container which is called `/home/jovyan` regardless of what your TRE username is. So to have access to the TRE folder `/shared` we can mount it as `/home/jovyan/shared` within the container. If you also need access to your home folder you can mount TRE folder `/home/$(whoami)` as `/home/jovyan/home` within the container. It makes no difference which folder you run the docker run command from provided you specify the mounts as shown below using absolute paths.

Everything you work on from within jupyter that you need to keep must be saved to somewhere in the folder(s) you have mounted. Anything else will be lost when the container stops running as we will be launching the container in throw away mode (`--rm` below), i.e. it will not save anything you modify within the container itself. This avoids creating a new container image on disk every time you launch it:

```bash
docker run -it --rm \
  -v /shared:/home/jovyan/shared \
  -v /home/$(whoami):/home/jovyan/home \
  -p 8888:8888 \
  --userns=keep-id:uid=1000,gid=100 \
  --security-opt=label=disable \
  my-jupyter-lab
```

When the container starts running it will show some links. Open Firefox ('Activities', then Search 'Firefox') and navigate to the link that looks something like `http://localhost:8888/lab?token=...` or `http://127.0.0.1:8888/lab?token=...`.

Once done using it, the container can be stopped by going to the terminal window you started it from and hitting `CTRL-C`.