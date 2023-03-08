# Launch the date logging application
## Installing the application
First, you need to pull the project from a remote repository or other sources.

In some cases, it may be necessary to install a compiler for C++ called g++ .

The project must be placed in a specific directory on your host machine. The path and file tree might look like this:
`
/your/path/to/project
`
- Project file-tree:
     - app/*
     - config.json
     - Dockerfile
     - README.md
 
## Launching an application using Docker

First, you need to go to the root directory of the project in CMD.

It is necessary to build the image using the following command in the terminal:
> docker build -t log_image:latest .

This command creates an image and names it _log_image_ with the _latest_ tag. Dot "." means that the image will be created from the current directory.

Next, you should create a volume using the command
> docker volume create my_volume

Here, _my_volume_ means the name of the created volume.

After that, you need to start the Docker container using the following command:
> docker run -it --name new_container -e USER=$USER -v /home/vitaliisavchuk/projects/datetime_logger_cpp/config.json:/dockerbuildlog/config.json -v my_volume:/dockerbuildlog/app/output log_image
