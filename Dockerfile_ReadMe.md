# Docker Instructions for MSSeg2 Challenge

* Build the Docker image using the following command: `docker build -f Dockerfile.cpu -t <your-docker-username>/msseg2_pipeline1:v1 .`
  * The Dockerfile is divided into 4 parts: (i) Pulls all the relevant base images, copies the required folders, sets the environment (`ENV`) variables, 
 and installs the relevant libraries, (ii) Installs the ANIMA package for validation, (iii) Sets up the model and (iv) Runs test functions for ensuring successful
 build.
  * An interesting feature about this file is in its usage of Docker's `multi-stage builds` feature.
    * In short, it allows one to use multiple `FROM` statements so that only relevant features of certain base images can be copied from one build stage to another
instead of using multiple Dockerfiles for each new base image.
    * The `COPY --from="x"` command is the one that makes this possible. It copies the files/artefacts built in the previous stage to the latest build stage.
    * Note that this is also economic in that only the base image defined in the final `FROM` statement is saved and all the intermediate build stages are left  
    behind.
  * Specifically, the first three stages install the preprocessing libraries, ANTs (v.2.2.0) (`Stage 0`), SCT (5.3.0) (`Stage 1`), and FSL (5.0.11) (`Stage 2`), 
respectively.
  * The final stage (`Stage 3`) forms the base image. It sets up the PATH variables, installs ANIMA, and prepares the image for model and inference.

* Publish Docker image using the command: `docker push <your-docker-username>/msseg2_pipeline1:v1`
  * Make sure that you are logged in CLI, otherwise run `docker login`.

* Run inference with: `docker run --entrypoint=/bin/sh --rm -v <MOUNT_DIRECTORY>:<WORKING_DIRECTORY> -w <WORKING_DIRECTORY>/ uzaymacar/msseg2_pipeline1:v1 process.sh` from this folder or any other folder which contains `process.sh` script.
	* `<MOUNT_DIRECTORY>` should contain the two FLAIR images `flair_time01.nii.gz` and `flair_time02.nii.gz`.
	* The `<WORKING_DIRECTORY>/` should be a non-existing path inside your container. This call will create a working directory inside the container, and copy all the files from your local `<MOUNT_DIRECTORY>`.
	* To run inference, you can also (i) `docker run -i --rm -v <MOUNT_DIRECTORY>:<WORKING_DIRECTORY> -w <WORKING_DIRECTORY>/ -t <your-docker-username>/msseg2_pipeline1:v1 /bin/bash` and then once you are in the container, (ii) `sh process.sh`. For this option to work, you also need `process.sh` script inside the specified `<WORKING_DIRECTORY>`.

**NOTE 1**: On a MacBook Pro with 3.5 GHz Dual-Core Intel Core i7 with 8GB memory, the build takes ~1.5 hours. 

**NOTE 2**: For Docker app in MacOS, you also need to change the allocated memory and cores from Settings.

The content and the skeletal structure for this Dockerfile is adapted from [MICCAI MS Lesion Segmentation Challenge- 2021](https://gitlab.inria.fr/amasson/lesion-segmentation-challenge-miccai21/-/tree/master/example_method) repository.
Further details can be found there.
