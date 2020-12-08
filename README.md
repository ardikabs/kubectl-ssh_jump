# kubectl ssh-jump
An extended version from [kubectl-ssh_jump](https://github.com/yokawasa/kubectl-plugin-ssh-jump) with the ability to leverage an existing ssh key-pairs given by ssh-agent to do an ssh jump to the destination node.

# Prerequisites
* Clone this repository
* Move `kubectl-ssh_jump` to `/usr/local/bin`

    ```bash
    $ git clone https://github.com/ardikabs/kubectl-ssh_jump.git
    $ chmod +x kubectl-ssh_jump/kubectl-ssh_jump
    $ mv kubectl-ssh_jump/kubectl-ssh_jump /usr/local/bin
    $ kubectl ssh-jump --help
    ```

# Disclaimer
This script is a simple bash script and expect to run with `ss` command for Linux/GNU OS. Other than Linux/GNU OS, this script will randomized port between `49152 - 59152` to be used without any check, so it is expected in case picked port is already on used then script would be exit with non-zero status code and you need to re-run the script.

# How-to
## Usage
```bash
$ ssh-jump --help
A kubectl plugin for ssh'ing to an instance under kubernetes host private networks

Usage:
  ssh-jump [--help | -h] TARGET_NODE
  ssh-jump [-CP] [-u=<ssh_username>|--username=<ssh_username>]
           [-p=<ssh_port> | --port=<ssh_port>]
           [-o=<ssh_opts> | --ssh-opts=<ssh_opts>]
           [-i=<identity_file> | --identity-file=<identity_file>]
           TARGET_NODE

Examples:

  # Use ssh key from selected file
  ssh-jump -i ~/.ssh/id_rsa ip-10-0-10-217.ap-southeast-1.compute.internal

  # Use ssh private key from ssh-agent
  ssh-jump ip-10-0-10-217.ap-southeast-1.compute.internal

Flags:
  -h, --help           : show this message
  -u, --username       : Target SSH username. Default "centos".
  -p, --port           : Target SSH Port. Default "22".
  -i, --identity-file  : Target SSH identity file.
  -o, --ssh-opts       : SSH additional flags.
  -P, --persistent     : Enable and/or use persistent ssh-jump pod. This flag is mutually exclusive with "--cleanup".
  -C, --cleanup            : Clean up ssh-jump if any. This flag is mutually exclusive with "--persistent". If both are specified, then this flag will take precedence.
```

## SSH'ing to the target node
```
$ ssh-jump -i /path/to/ssh-private-key.pem ip-10-0-30-61.ap-southeast-1.compute.internal
Creating SSH jump host (Pod)...
pod/sshjump-z80ursw created
Forwarding from 127.0.0.1:50183 -> 22
Forwarding from [::1]:50183 -> 22
Handling connection for 50183
Warning: Permanently added 'ip-10-0-30-61.ap-southeast-1.compute.internal' (ECDSA) to the list of known hosts.
Last login: Mon Jul 27 03:06:36 2020 from ip-10-0-10-211.ap-southeast-1.compute.internal
[centos@ip-10-0-30-61 ~]$
.
.
.
[centos@ip-10-0-30-61 ~]$ logout
Connection to ip-10-0-30-61.ap-southeast-1.compute.internal closed.
pod "sshjump-z80ursw" force deleted
```

## Enable ssh-jump persistent mode
```
$ ssh-jump --persistent -i /path/to/ssh-private-key.pem ip-10-0-30-61.ap-southeast-1.compute.internal
Creating SSH jump host (Pod)...
pod/sshjump-z80ursw created
Forwarding from 127.0.0.1:50183 -> 22
Forwarding from [::1]:50183 -> 22
Handling connection for 50183
Warning: Permanently added 'ip-10-0-30-61.ap-southeast-1.compute.internal' (ECDSA) to the list of known hosts.
Last login: Mon Jul 27 03:06:36 2020 from ip-10-0-10-211.ap-southeast-1.compute.internal
[centos@ip-10-0-30-61 ~]$
.
.
.
[centos@ip-10-0-30-61 ~]$ logout
Connection to ip-10-0-30-61.ap-southeast-1.compute.internal closed.

# Try it again
# You will using existing ssh-jump as a jumper
$ ssh-jump --persistent -i /path/to/ssh-private-key.pem ip-10-0-30-61.ap-southeast-1.compute.internal
Use existing SSH jump host (sshjump-z80ursw)
Forwarding from 127.0.0.1:50183 -> 22
Forwarding from [::1]:50183 -> 22
Handling connection for 50183
Warning: Permanently added 'ip-10-0-30-61.ap-southeast-1.compute.internal' (ECDSA) to the list of known hosts.
Last login: Mon Jul 27 03:06:36 2020 from ip-10-0-10-211.ap-southeast-1.compute.internal
[centos@ip-10-0-30-61 ~]$
.
.
.
[centos@ip-10-0-30-61 ~]$ logout
Connection to ip-10-0-30-61.ap-southeast-1.compute.internal closed.

# Cleanup ssh-jump
$ ssh-jump --cleanup -i /path/to/ssh-private-key.pem ip-10-0-30-61.ap-southeast-1.compute.internal
Use existing SSH jump host (sshjump-z80ursw)
Forwarding from 127.0.0.1:50183 -> 22
Forwarding from [::1]:50183 -> 22
Handling connection for 50183
Warning: Permanently added 'ip-10-0-30-61.ap-southeast-1.compute.internal' (ECDSA) to the list of known hosts.
Last login: Mon Jul 27 03:06:36 2020 from ip-10-0-10-211.ap-southeast-1.compute.internal
[centos@ip-10-0-30-61 ~]$
.
.
.
[centos@ip-10-0-30-61 ~]$ logout
Connection to ip-10-0-30-61.ap-southeast-1.compute.internal closed.

Terminating ssh-jump Pods if any
pod "sshjump-z80ursw" force deleted
```