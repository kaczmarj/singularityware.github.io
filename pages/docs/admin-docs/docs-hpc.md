---
title: Singularity on HPC
sidebar: admin_docs
permalink: docs-hpc
folder: docs
toc: false
---

One of the architecturally defined features in Singularity is that it can execute containers like they are native programs or scripts on a host computer. As a result, integration with schedulers is simple and runs exactly as you would expect. All standard input, output, error, pipes, IPC, and other communication pathways that locally running programs employ are synchronized with the applications running locally within the container.

Additionally, because Singularity is not emulating a full hardware level virtualization paradigm, there is no need to separate out any sandboxed networks or file systems because there is no concept of user-escalation within a container. Users can run Singularity containers just as they run any other program on the HPC resource.

### Workflows
We are in the process of developing Singularity Hub, which will allow for generation of workflows using Singularity containers in an online interface, and easy deployment on standard research clusters (e.g., SLURM, SGE). Currently, the Singularity core software is installed on the following research clusters, meaning you can run Singularity containers as part of your jobs:

- The <a href="http://sherlock.stanford.edu" target="_blank" class="no-after">Sherlock cluster</a> at <a href="https://srcc.stanford.edu/" class="no-after" target="_blank">Stanford University</a>
- <a href="https://www.xsede.org/news/-/news/item/7624" target="_blank" class="no-after">SDSC Comet and Gordon</a> (XSEDE)

### Integration with MPI
Another result of the Singularity architecture is the ability to properly integrate with the Message Passing Interface (MPI). Work has already been done for out of the box compatibility with Open MPI (both in Open MPI v2.x as well as part of Singularity). Here is a document that illustrates the Open MPI/Singularity workflow:

From the above image you can follow the invocation pathway:
1. mpirun is called by the resource manager or the user directly from a shell
2. Open MPI then calls the process management daemon (ORTED)
3. The ORTED process launches the Singularity container requested by the mpirun command
4. Singularity builds the container and namespace environment
5. Singularity then launches the MPI application within the container
6. The MPI application launches and loads the Open MPI libraries
7. The Open MPI libraries connect back to the ORTED process via the Process Management Interface (PMI)
8. At this point the processes within the container run as they would normally directly on the host at full bandwidth! This entire process happens behind the scenes, and from the user's perspective running via MPI is as simple as just calling mpirun on the host as they would normally.

#### Code
Below is an example snippet of building and installing OpenMPI into a container and then running an example MPI program through Singularity. 
 
```bash
$ # Include the appropriate development tools into the container (notice we are calling
$ # singularity as root and the container is writeable)
$ sudo singularity exec -w /tmp/Centos-7.img yum groupinstall "Development Tools"
$
$ # Clone the OpenMPI GitHub master branch in current directory (on host)
$ git clone https://github.com/open-mpi/ompi.git
$ cd ompi
$
$ # Build OpenMPI in the working directory, using the tool chain within the container
$ singularity exec /tmp/Centos-7.img ./autogen.pl
$ singularity exec /tmp/Centos-7.img ./configure --prefix=/usr/local
$ singularity exec /tmp/Centos-7.img make
$
$ # Install OpenMPI into the container (notice now running as root and container is writeable)
$ sudo singularity exec -w /tmp/Centos-7.img make install
$
$ # Build the OpenMPI ring example and place the binary in this directory
$ singularity exec /tmp/Centos-7.img mpicc examples/ring_c.c -o ring
$
$ # Install the MPI binary into the container at /usr/bin/ring
$ sudo singularity copy /tmp/Centos-7.img ./ring /usr/bin/
$
$ # Run the MPI program within the container by calling the MPIRUN on the host
$ mpirun -np 20 singularity exec /tmp/Centos-7.img /usr/bin/ring


Process 0 sending 10 to 1, tag 201 (20 processes in ring)
Process 0 sent to 1
Process 0 decremented value: 9
Process 0 decremented value: 8
Process 0 decremented value: 7
Process 0 decremented value: 6
Process 0 decremented value: 5
Process 0 decremented value: 4
Process 0 decremented value: 3
Process 0 decremented value: 2
Process 0 decremented value: 1
Process 0 decremented value: 0
Process 0 exiting
Process 1 exiting
Process 2 exiting
Process 3 exiting
Process 4 exiting
Process 5 exiting
Process 6 exiting
Process 7 exiting
Process 8 exiting
Process 9 exiting
Process 10 exiting
Process 11 exiting
Process 12 exiting
Process 13 exiting
Process 14 exiting
Process 15 exiting
Process 16 exiting
Process 17 exiting
Process 18 exiting
Process 19 exiting
```

### Notes
Supported Open MPI Version(s): To achieve proper container'ized Open MPI support, you must use Open MPI version 2.1 which at the time of this writing has not been released yet. The above example builds from the current master development branch of Open MPI.
