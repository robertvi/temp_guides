# Docker Jupyter

## Using a custom image
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

Save the image to a tar file:
```bash
docker save -o my-jupyter-lab.tar my-jupyter-lab
```

Ingress the `my-jupyter-lab.tar` file into the TRE using the Airlock.

Then from your TRE desktop load the image:
```bash
docker load -i ~/shared/inbound/$(whoami)/my-jupyter-lab.tar
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

When the container starts running it will show some links. Open Firefox ('Activities', then Search 'Firefox'), navigate to the link that looks something like `http://localhost:8888/lab?token=...` or `http://127.0.0.1:8888/lab?token=...`.
