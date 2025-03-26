# Help for Using the Alliance Clusters
Help for debugging/ssh/development on the Digital Research Alliance of Canada Clusters

## Content
- [Useful Information](<#useful-information>)
- [SSH](#SSH)
- [File Managment](<#file-management>)
- [Visual Studio Code (IDE)](<#visual-studio-code>)
- [Compiling Locally](<#compiling-locally>)


## Useful Information
- Resources for this
  - [Sharcnet Wiki](https://helpwiki.sharcnet.ca/wiki/FAQ)
    - especially their [online seminars](https://helpwiki.sharcnet.ca/wiki/Online_Seminars)
  - [Alliance Wiki](https://docs.alliancecan.ca/wiki/Technical_documentation) 
- 2FA can be simplified (see [SSH Help](#SSH))
- Debugging with few Resources on Login Nodes is fine
  - Narval doesn't have cpu or time limit for login nodes
- [DDT](https://www.youtube.com/watch?v=t5eNtewYgJw) can be used to parallal debug
  - `ssh -Y username@graham.alliancecan.ca` (note that `-Y` (x11 Forwarding) clashes with the ControlMaster [see here](https://github.com/gravitational/teleport/issues/38238))
    - `module load ddt-cpu`
    - `ddt program`
      - can also run just `ddt` and then use the gui

## slurm
- Interactive Job
  - `salloc --time=0-1:00 --mem-per-cpu=4G --ntasks=32 --account=rrg-bprotas`
    - 1 hour interactive job
  - `mpiexec -n 32 prog` or something similar
  - If the connection is lost
    - `squeue -u $(whoami)`
    - `sattach 12345678.interactive`
- Submit Job
  - `ssh` and `cd` to location
  - `sbatch run_graham.sh`
    - where `run_graham.sh` is
      ```
      #!/bin/bash
      #SBATCH --account=rrg-bprotas
      #SBATCH --nodes=1
      #SBATCH --ntasks-per-node=32
      #SBATCH --mem=125000M
      #SBATCH --time=1:00:00
      srun prog
      ```
      - `srun` instead of `mpiexec`
      - `srun` automatically figures out the nodes/cpus
- commands
  - `squeue -u $(whoami)`
    - shows my jobs in the job queue
  - `sacct`
    - shows my previous jobs
  - `scontrol show job 12345678`
    - shows job details
  - previously/other mentioned
    - `sbatch run_graham.sh`
    - `sattach 12345678.interactive`
    - `scancel 12345678`
    - `srun --jobid 12345678 --pty /bin/bash`
    

## SSH
- general syntax `ssh -Y username@graham.alliancecan.ca`
  - `-Y` allows to run graphical user interfaces
- 2FA is manditory now
  - **instead of plugging in the code of your phone you can plug in `1` which will give you a prompt on your phone to confirm**
    - this way you don't need to type the 6 digits all the time
- you can generate a key pair so you don't have to type in your password every time you connect via any kind of ssh
  - [this tutorial](https://www.youtube.com/watch?v=u9k6HikDyqk&t=2186s) shows you how to generate and setup the key pair
- you can modify the local `~/.ssh/config` file
  - mine looks like this
    - ```
      Host *
	      User username                                                         # change to your username

      Host vscode_graham                                                      # the connection to use for graham when accessing via vs code
	      Hostname graham.alliancecan.ca
	      RemoteCommand echo "loading modules ..."; module load fftw-mpi netcdf-fortran; bash # load the modules at login so that they are available in vs code
	      RequestTTY yes                                                        # required for RemoteCommand to work
	      ControlPath ~/.ssh/cm_vscode-%r@%h:%p
	      ControlMaster auto
	      ControlPersist 10m
      
      Host x11_graham
	      Hostname graham.alliancecan.ca
	      ForwardX11 yes

      Host graham
        Hostname graham.alliancecan.ca
	      ControlPath ~/.ssh/cm-%r@%h:%p
	      ControlMaster auto
	      ControlPersist 10m
      ```
      - change the username in `User username`. This allows you to omit the username login in `ssh username@graham.alliancecan.ca` becomes `ssh graham.alliancecan.ca`
      - `ControlPath ...`, `ControlMaster auto`, `ControlPersist 10m` allows you to login without the 2fa for 10 minutes which I think is a bit more convenient [see also the Alliance-Wiki](https://docs.alliancecan.ca/wiki/Multifactor_authentication#Configuring_your_SSH_client_with_ControlMaster)
      - there is a [bug](https://github.com/gravitational/teleport/issues/38238) when combining control master and x11 forwarding. So when using a GUI we need to disable the control master.

## File Management
- Manual Copying to and from the server
  - Method [sshfs](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh)
    - mounts a local folder to the remote folder
    - then you can use it with your prefered way of handling files (nautilus/dolphin/terminal/...)
      - setup:
        - make a local folder which you would like to use for this (say `~/grahamFolder/`)
        - `sshfs username@graham.alliancecan.ca:/home/username/path/to/folder/which/it/should/be ~/grahamFolder/`
      - terminal copy files to local example
        - `find ./output/ -type f -name 'diagScalar*.nc' -exec rsync -h --progress '{}' ~/Desktop/temp/`
    - I think this is the **most user friendly version** togehter with ide
    - If there is an error or the connection broke down `pkill -kill -f "sshfs"`
  - Method [IDE](<#visual-studio-code>)
    - handle the files in the ide
    - therefore you need to setup the ide, see [below](<#visual-studio-code>)
    - I think this is also really **user friendly version** togehter with sshfs
  - Method [scp](https://www.geeksforgeeks.org/scp-command-in-linux-with-examples/)
    - simply copies the file
    - example
      - `scp path/to/local/file.txt username@graham.alliancecan.ca:/home/username/path/to/remote/destionation/filename.txt`
      - 
        copys file.txt to the server
    - can also be used the other way, ...
    - needs (2 factor) authentification every time &rarr; **not user friendly**
  - Method [sftp](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server)
    - opens a connection that allows you to move stuff locally and remotely in the terminal
    - example
      - `sftp username@graham.alliancecan.ca`
        - `put local/path/to/file.txt remote/path/to/filename.txt`
        - `...`
    - once connected you can do multiple things at once &rarr; **a bit more user friendly than scp**

## Visual Studio Code
You do not have to use an IDE and can just modify the files with any editor, but I think it is really helpful.
And you can of course use your prefered IDE but in the following I will explain [VS Code](https://code.visualstudio.com/).
- **It seems like you can not use the Debugger in MPI in VSCode**
- [Here](https://www.youtube.com/watch?v=u9k6HikDyqk) is a introduction to VS Code and how to use it on the cluster. Follow the basic setup of the video so that you can connect in vscode to the cluster
  - We also want the [Fortran Extension](https://marketplace.visualstudio.com/items?itemName=fortran-lang.linter-gfortran)
- To Compile and Launch the code we need to specify how VS Code should do that
  - In the remote project directory create a directory `.vscode`
    - In this directory create the following files
      - `tasks.json` which should look like this
        ```
          {
            "version": "2.0.0",
            "tasks": [
              {
                "type": "shell",
                "label": "make",
                "command": "make"
              },
              {
                "type": "shell",
                "label": "clean",
                "command": "make clean"
              },
              {
                "label": "fastBuild",
                "dependsOn": ["make"]
              },
              {
                "label": "cleanBuild",
                "dependsOrder": "sequence",
                "dependsOn": ["clean", "make"]
              }
            ]
          }
        ```
      - `launch.json` which should look like this
        ```
          {
            "version": "0.0.0",
            "configurations": [
              {
                "name": "Debug (fast)",
                "type": "cppdbg",
                "request": "launch",
                "preLaunchTask": "fastBuild",
                "program": "${workspaceFolder}/prog",
                "cwd":     "${workspaceFolder}",
                "args": [],
                "environment": [],
              },
              {
                "name": "Debug (clean)",
                "type": "cppdbg",
                "request": "launch",
                "preLaunchTask": "cleanBuild",
                "program": "${workspaceFolder}/prog",
                "cwd":     "${workspaceFolder}",
                "args": [],
                "environment": [],
              }
            ]
          }

        ```
  - This should give you the option to clean, compile and run the program named `prog` or to just compile the program and run it
- loading modules
  - For compilation we need to load the modules `fftw` and `netcdf`
    - You can either add the following to the local `~/.ssh/config` file as in the example config file above
      ```
      RemoteCommand module load fftw-mpi/3.3.10 netcdf-fortran/4.6.1; bash  # load the modules at login so that they are available in vs code
      RequestTTY yes                                                        # required for RemoteCommand to work
      ```
      - This will send `module load fftw-mpi/3.3.10 netcdf-fortran/4.6.1; bash` every time you ssh to the server via this profile
    - Or include the following to the in the remote `~/.bashrc` file
      `module load fftw-mpi/3.3.10 netcdf-fortran/4.6.1`
      - The `~/.bashrc` file is executed every time you log on the server
    - Loading the modules in this fashion we don't need to hard code module paths. So the new make file can look like this
      ```
      compiler = mpif90
      progName = prog
      OBJ = global_variables.o initialize.o data_ops.o fftwfunction.o function_ops.o optimization.o maxdLpdt.o
      BASIC_LIB  = -lm -lmpi
      DEB_OPTS = -g -O0
      # -g enable debugging
      # -O0 optimization off (for faster debugging)
      EDI_OPTS = -ffree-line-length-512
      # -ffree-line-length-512 otherwise line split error
      # -fallow-argument-mismatch turns argument missmatch in mpi to warnings (not needed anymore since use mpi instead of include mpif.h)
      OPTIONS = $(DEB_OPTS) $(EDI_OPTS)
      FFTW_DIR      = -I$(EBROOTFFTWMPI)/include
      FFTW_LIB      = -lfftw3_mpi -lfftw3 -lm		# Load FFTW3 Library
      NETCDF_DIR    = -I$(EBROOTNETCDFMINFORTRAN)/include
      NETCDF_LIB    = -lnetcdf -lnetcdff

      $(progName): $(OBJ)
        $(compiler) $(OPTIONS) $(OBJ) $(BASIC_LIB) $(NETCDF_LIB) $(FFTW_LIB) -o $@
      %.o: %.f90
        $(compiler) -c $(OPTIONS) $(NETCDF_DIR) $(FFTW_DIR) $<
      clean:
        rm -f *.o *.mod *.nc *.dat $(progName)
      ```
## Compiling locally
We need the following packages
- for the [fftw3](http://www.fftw.org/fftw2_doc/fftw_6.html) module
  - fftw3-fortran
    - `./configure --enable-mpi`
    - `make`
    - `sudo make install`
  - fftw3-c (I don't think this was needed)
    - `./configure --enable-mpi`
    - `make`
    - `sudo make install`
- for the netcdf module
  - [netcdf-fortran]((https://github.com/Unidata/netcdf-fortran/releases)), for which we need
    - [netcdf-c](https://github.com/Unidata/netcdf-c/releases), for which we need
      - [hdf5](https://www.hdfgroup.org/download-hdf5/source-code/)
        - `configure`
        - `make`
        - `sudo make install`
        - copy the files from `lib`, (`bin`, )`include` to `/usr/local/lib`, (`/usr/local/bin`, )`/usr/local/include` (I don't think we need bin)
      - `ZDIR=/usr/local`
      - `./configure --prefix=${ZDIR}`
      - `make`
      - `sudo make install`
    - `./configure --prefix=${ZDIR}`
    - `make`
    - `sudo make install`

Now all needed modules should be in `/usr/local/lib`, (`/usr/local/bin`, )`/usr/local/include` and the make file
```
compiler = mpif90
progName = prog
OBJ = global_variables.o initialize.o data_ops.o fftwfunction.o function_ops.o optimization.o maxdEdtHeli_main.o
BASIC_LIB  = -lm -lmpi 
LIB = -L /usr/local/lib -lfftw3_mpi -lfftw3 -lnetcdff -lnetcdf
DIR = -I /usr/local/include
$(progName): $(OBJ)
  $(FC) -g $(OBJ) $(LIB) -o $@
%.o: %.f90
  $(FC) -c -g $(DIR) $(BASIC_LIB) -ffree-line-length-512 -fallow-argument-mismatch $<
clean: 
  rm -f *.o *~ *.mod
```
should work
