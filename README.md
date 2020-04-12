![Go](https://github.com/masoudd/minimal_ssh_honeypot/workflows/Go/badge.svg?branch=master)

Minimal SSH Honeypot
====================

This is a SSH honeypot designed to only log the SSH connection attempts on a given port.  It
does not go any further than that and that's by design.

It is inspired by [sshoney](https://github.com/ashmckenzie/sshoney) and by proxy by [server_complex.go](https://github.com/Scalingo/go-ssh-examples/blob/master/server_complex.go) (thanks @max107 :))

How?
----

It works by listening on a non-privileged port (2222 by default) and pretends to be an SSH
server.  When an SSH client connects, it logs the connection details (IP, username, password and SSH clienr version) to stdout and then rejects the login attempt.

run with --help to see available flags and their default values.

Basic setup
-----------

Install the source and binary:

```shell
go get -u github.com/masoudd/minimal_ssh_honeypot
```

Ensure `${GOPATH}/bin` is in your `${PATH}` (so you can run `minimal_ssh_honeypot` from any directory):

```shell
$ export PATH:${PATH}:${GOPATH}/bin
```
You first need to generate a key pair for the server.

```shell
ssh-keygen -f ./host.key -N ''
Generating public/private rsa key pair.
Your identification has been saved in ./host.key.
Your public key has been saved in ./host.key.pub.
The key fingerprint is:
SHA256:5QH4ForyXNVRuUPPuKtyg2//swPLtw4c3DyS0idTpUk ash@ashmckenzie
The key's randomart image is:
+---[RSA 2048]----+
|       ....o..E .|
|      . o.. o. + |
|     . + .o. =+  |
|  . . o oo ++=o  |
|   + . .S o Oo=  |
|    o      oo* . |
|         . .o+   |
|        o + +.+  |
|         =o+.+== |
+----[SHA256]-----+
```

Now run it:

```shell
$ minimal_ssh_honeypot -hostkey host.key
time="2015-08-28T08:59:58+10:00" level=info msg="listening on 2222"
```

It is now logging to stdout and listening on port 2222 which is not the standard SSH port (22 is).  This is deliberately setup this way to ensure:

1. You are not locked out of a remote server by default
2. The service is not running as root

Proceed to the [Running live](#running-live) section for the best way to run this on a real server.

Running live
------------

The honey pot listens on port 2222 by default. This can be changed with the -port argument

Once you have it running (ideally as the least privileged user, e.g. `nobody`) it's time to setup an IPTables rule to redirect the traffic.
Change _eth0_ to the interface you want to use.

WARNING: Please, please be very careful when adding this rule you don't lock yourself out!
==========================================================================================

```
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 22 -j REDIRECT --to-port 2222
```
