<!---
{
  "id": "d1bee1c7-d88a-4f00-a44e-3e402f6ee826",
  "teaches": "Docker Volumes",
  "depends_on": ["21351032-2a40-4101-8c29-513fdaf37365"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-05-14",
  "keywords": ["docker", "volumes", "persistent storage", "bind mounts", "named volumes"]
}
--->

# Docker Volumes

> In this exercise you will learn how to use Docker volumes for persistent and shared storage. Furthermore we will explore how bind mounts and named volumes differ and how to use them in containers.

## Introduction

When working with containers, the file system is typically ephemeral — any changes made during a container's lifetime are lost once it stops or is removed. Docker volumes address this by providing persistent storage that exists independently of containers. Volumes can be used to share data between containers, persist database files, or simply retain logs or outputs across container sessions.

There are two main types of volumes in Docker: named volumes and bind mounts. Named volumes are managed by Docker and stored in its default volume location. Bind mounts, in contrast, map specific directories or files from the host filesystem into the container. Both serve different purposes and have distinct advantages depending on your workflow.

This exercise will guide you through creating and using both types of volumes. You will also learn how to inspect and clean up volumes after use.

### Further Readings and Other Sources

* [Docker Docs: Volumes Overview](https://docs.docker.com/storage/volumes/)
* [Docker Bind Mounts vs Volumes](https://docs.docker.com/storage/bind-mounts/)
* [YouTube: Docker Volumes Explained](https://www.youtube.com/watch?v=pTFZFxd4hOI)
* [Docker Deep Dive - Nigel Poulton](https://nigelpoulton.com/books/)

## Tasks

### Task 1: Use a Bind Mount

1. Create a file on your host that will be shared with the container:

   ```bash
   echo "Hello from the host" > hostfile.txt
   ```

   This creates a simple text file named `hostfile.txt` in your current directory. The file contains a message that will be read from inside the container.

2. Run a Docker container and bind mount the file into the container:

   ```bash
   docker run --rm -v $(pwd)/hostfile.txt:/data/hostfile.txt -it ubuntu bash
   ```

   The `-v` flag specifies a volume. Here, we are mapping the current working directory's `hostfile.txt` to the path `/data/hostfile.txt` inside the container. The container is started in interactive mode with a Bash shell.

3. Once inside the container, confirm the file is accessible:

   ```bash
   cat /data/hostfile.txt
   ```

   This should display the content "Hello from the host". Try editing the file with `echo "Updated from container" >> /data/hostfile.txt` and then inspect the changes on your host machine.

4. Exit the container:

   ```bash
   exit
   ```

   Since `--rm` was used, the container is automatically removed after exit.

### Task 2: Use a Named Volume

1. Create a named volume managed by Docker:

   ```bash
   docker volume create myvolume
   ```

   Docker creates a volume and assigns it a storage location. You can reuse this volume across multiple containers.

2. Start a container that mounts this volume:

   ```bash
   docker run -dit --name voltest -v myvolume:/app ubuntu
   ```

   This mounts the `myvolume` volume into the `/app` directory in the container. Any files written to `/app` are stored in the volume.

3. Write data into the volume from inside the container:

   ```bash
   docker exec voltest bash -c "echo 'Hello from inside' > /app/incontainer.txt"
   ```

   This writes a new file into the volume. Since the volume is persistent, the file remains even after the container stops.

4. Verify the persistence by accessing the volume from another container:

   ```bash
   docker run --rm -v myvolume:/data ubuntu cat /data/incontainer.txt
   ```

   The file created in the previous container is visible here, confirming that volumes provide persistent shared storage.

5. Cleanly stop the container:

   ```bash
   docker stop voltest
   ```

### Task 3: Inspect and Remove Volumes

1. List all existing Docker volumes:

   ```bash
   docker volume ls
   ```

   This displays all volumes managed by Docker, including `myvolume`.

2. Inspect a specific volume to see its details:

   ```bash
   docker volume inspect myvolume
   ```

   This reveals metadata such as mount points, creation date, and whether it is in use.

3. Remove the volume after ensuring it is unused:

   ```bash
   docker rm -f voltest
   docker volume rm myvolume
   ```

   This sequence stops and removes the container, then deletes the volume. Always confirm no containers are using a volume before attempting removal.

## Questions

1. What is the difference between a named volume and a bind mount?
2. How can you ensure data written by a container is saved even after it stops?
3. What are some practical scenarios for using volumes?
4. How does Docker manage the storage location of named volumes?
5. Why might you choose to use `--rm` in some of these examples?

## Advice

Docker volumes are a crucial feature for anyone using containers beyond simple tests. They help decouple data from application logic and make containers suitable for real production systems. Named volumes offer safe and isolated storage, while bind mounts provide flexibility and speed during development. Make it a habit to clean up unused volumes and be cautious when mounting sensitive host directories. Review the previous exercise on [background containers and script mounts](#) to deepen your understanding of how containers and file systems interact.
