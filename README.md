# Assignment for the "1-01 Getting Started with OpenFOAM" unit

## Goals

- Get familiarized with the created docker container.
- Practice engaging in individual assignments.

## Basic-level skills

Access your remote machine with the most basic command (The first > denotes that 
the user is a regular user):

```bash
(loc:~) ssh -i ~/.ssh/remotesshkey.pem linux1@xxx.xxx.xxx.xxx
```

Where:
- `~/.ssh/remotesshkey.pem` should point to the path of the SSH key-pair file on
  your local machine.
- `linux1` is the (default) user you intend to act as, on the remote machine.
- `xxx.xxx.xxx.xxx` is the specific IP address of the remote machine.

> Adding `-x -C` flags to the SSH command might help if you experience 
> sluggish terminal behavior!

### Basic system commands

At this point you're on a Red Had system. You may want to try running
the following commands and see if you can answer the upcoming questions:

1. `df -h` to recon the filesystem
2. `lscpu` to list CPU information
3. `cat /etc/os-release` to display the content of the `/etc/os-release` file.
 
> By the way, you can use TAB for basic auto-completion :smile:

Here are the questions:

a. How much "free" disk space you have? (hint: look for the device `/dev/sda*`)
b. What is the maximal CPU frequency we can reach on this machine?
c. What OS the machine is running?

### Becoming the super user

To be able to (globally) install any application/program on Red HAt sysytems, we use
the `yum` command.
To install, say vim (a text editor):

```bash
yum install vim
```

That doesn't work, huh! Regular users (eg. `linux1`) can't install packages system-wide.
That's one action among many that only special user accounts have access to.

The super user on a Red Had system (and all other Linux distributions) is called `root`.

You can become root on the remote machine by executing the following command:

```bash
(rem:~) sudo -s
```
When you want to get back to your regular user (`linux1`), just hit `Ctrl-D`, or type
(the first `$` sign indicates that the following command is run as root)

```bash
(rem:~)$ exit
```

You should now be able to install a couple of recommended packages:

```bash
(rem:~) sudo -s
(rem:~)$ yum install -y vim nano tmux
```
> Vim and Nano are "text editors". Vim is more suited for the advanced users; Linux-beginners
> should definitely stick the more "intuitive" Nano (Just replace vim with nano in the following
> commands).

> Tmux is a useful application when it comes to leaving your simulations running even after
> logging out of the SSH session.


### Attach to the OpenFOAM docker container

If you have followed what we did in the unit, the script we used to install docker
and create our OpenFOAM container should have instantiated the container and left it running
in the background.

The most basic method to attach to an interactive shell on that container is to run (as root):

```bash
$ docker -it openfoam bash
```

where:
- `-it` tells docker to create an "interactive terminal"
- `openfoam` is the name of the target container
- `bash` is the command to run: the shell command itself!

You'll be presented with a prompt similar to this one:
```bash
(of@containerid:~/OpenFOAM/of-7/run)$ __
```
This tells us that:
- We are acting as the `of` user inside the container with the unique id `containerid`.
- the current directory is `/home/of/OpenFOAM/of-7/run`

Try to run the same commands we ran back in the **Basic system commands** section and
answer the same questions.

> At this point, you should press Ctrl-D multiple times until you leave the **SSH session**

## Intermediate-level skills

The docker container actually exposes port 8888 to the remote system - for running jupyter
notebooks - so, if you are interrested, you can also forward the same port from the remote
machine to your local one. This should happen when you initiate the SSH session:

```bash
(loc:~) ssh -x -C -i ~/.ssh/remotesshkey.pem -L 8888:localhost:8888 linux1@xxx.xxx.xxx.xxx
```

> You should alias the previous command to `l1c` or something

Now if you run `jupyter notebook --no-browser` inside the container on the remote machine, 
and then go to
`http://localhost:8888/?token=....` (the link is provided in the output of the previous command)
in your local browser, you should be able to access the jupyter server.

Try it out, it's fun.

One other thing an intermediate user would attempt to do, is to mount a directory of the remote 
machine directly on his/her local machine using, for example, the SSHFS tool.

Simply put, 
```bash
> mkdir -p ~/ResEngCourse/myproject    # Create a local directory to mount things on
> sshfs -o IdentityFile=~/.ssh/remotesshkey.pem\
        linux1@xxx.xxx.xxx.xxx:/path/to/remote_directory ~/ResEngCourse/myproject
```

You can then use `df -h` locally (or an equivalent command) to check if the directory
is mounted correctly.

Mounting the remote directory on the local filesystem in this way is generally a better
option than relying on your text editor's ability to deal with remote files. Now you 
can use your local tools to edit these files!

To unmout, run `fusermount -u ~/ResEngCourse/myproject`. That's it.

In addition, you may have noticed that once you leave the SSH session, all processes
started by that particular session get stopped. To prevent this from happening:
- Install tmux (or any other multiplexer you might prefer) on the remote system: 
  `sudo yum install -y tmux`
- Instead of running the command you want to be persistent in the shell directly, start tmux 
  and run it there.
- Press `Ctrl-b` then `d` to detach from the tmux session
- You can leave the SSH session now, the process your started on the tmux session
  won't be killed
- If you want to get back to that process, just type `tmux attach`.

## Advanced-level skills

The remote has an X server installed by default :smile:, you can leverage it to forward GUI
applications to your local machine.

> A note: I use this trick for nothing else than visually **monitoring simulation results**
> (which are running on remote machines) on my local machine!

For this to work, we need the `xauth` package to be installed on the remote machine:
```bash
(rem:~) sudo yum -y install xauth
```

In addition, X11 Forwarding is enabled by default on the server, take a look at
`/etc/ssh/sshd_config` file, it should say:

```bash
X11Forwarding yes
```

Make sure to enable trusted X11 forwarding as well on your local machine (in your local
`/etc/ssh/ssh_config`, you may have to uncomment these lines):
```bash
ForwardX11 yes
ForwardX11Trusted yes
```

Also, don't forget to restart the SSH service
```bash
(loc/rem:~) sudo systemctl restart sshd
```

To engage in an X-Forwarding-Enabled SSH session, you can run (whithout 8888-port forwarding):
```bash
(loc:~) ssh -XC -c aes128-gcm@openssh.com \
      -i ~/.ssh/testmachine.pem linux1@165.165.xxx.xxx
```
