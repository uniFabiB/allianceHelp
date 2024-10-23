# Help for Using the Alliance Clusters
Help for debugging/ssh/development on the Digital Research Alliance of Canada Clusters
## Resources and Source for this
- https://helpwiki.sharcnet.ca/wiki/Online_Seminars

## Useful Information
- Debugging with few Resources on Login Nodes is fine &rarr; Debug/Develop on Login Nodes
  - Narval doesn't have cpu or time limit for login nodes
- [DDT](https://www.youtube.com/watch?v=t5eNtewYgJw) can be used to parallal debug
  - `ssh -Y username@graham.alliancecan.ca` (note that `-Y` (x11 Forwarding) clashes with the ControlMaster [see here](https://github.com/gravitational/teleport/issues/38238))
    - `module load ddt-cpu`
    - `ddt program`
      - can also run just `ddt` and then use the gui

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
	      RemoteCommand module load fftw-mpi/3.3.10 netcdf-fortran/4.6.1; bash  # load the modules at login so that they are available in vs code
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
    - once connected you can do multiple things at once &rarr; **a bit more user friendly**
  - Method [sshfs](https://www.digitalocean.com/community/tutorials/how-to-use-sshfs-to-mount-remote-file-systems-over-ssh)
    - mounts a local folder to the remote folder
    - then you can use it with your prefered way of handling files (nautilus/dolphin/terminal/...)
      - setup:
        - make a local folder which you would like to use for this (say `~/grahamFolder/`)
        - `sshfs username@graham.alliancecan.ca:/home/username/path/to/folder/which/it/should/be ~/grahamFolder/`
    - I think this is the **most user friendly version** togehter with ide
  - Method [IDE](<Visual Studio Code>)
    - handle the files in the ide
    - therefore you need to setup the ide, see [below](<Visual Studio Code>)
    - I think this is also really **user friendly version** togehter with sshfs

## Visual Studio Code
You do not have to use an IDE and can just modify the files with any editor, but I think it is really helpful.
And you can of course use your prefered IDE but in the following I will explain [VS Code](https://code.visualstudio.com/)
- [Here](https://www.youtube.com/watch?v=aR2L-UVmNXA) is a introduction to VS Code and how to use it on the cluster. Follow the basic setup of the video.
- We need fftw
- Compile and **to do: finish**
