Connecting To A Server
======================

This chapter will show you how to connect to a server using the SSH ("secure shell") protocol, how to interact with that server through a remote shell, and how to move data and code between your local computer, the Internet, and the server. Along the way, you will learn about how SSH proves your identity to a server, how to configure your local computer to make using SSH more comfortable, and about basic server etiquette.

:::{admonition} Learning Objectives
+ Learn how to connect to a remote server using the SSH protocol.
+ Learn how to co-exist with other users on the same server.
+ Learn about SSH passwords and keys.
+ Learn how to move data between your local PC, the Internet, and a server using `wget`/`curl`, `sftp`, and `scp`.
+ Learn about the POSIX directory structure and access permissions.
:::

What Is A Server?
-----------------

Generally, a server is just some computer that can be accessed over a network, typically the Internet. In this context, a computing server is, more specifically, a computer that gives users access to a remote command line from where they can run programs on that computer. The reason to run programs on a server is that servers typically offer more memory and disk space and more computing performance than any user's local computer, and can therefore work on bigger problems.

Unlike users' local computers, servers typically run some UNIX-like operating system, which in almost all cases is Linux, they are typically controlled exclusively via the command line, they are typically shared by multiple users, often multiple users at the same time, and they are operated and maintained by dedicated people, called system administrators or system operators (SysOps for short).

How To Connect To A Server
--------------------------

Users typically connect to a server using the SSH ("secure shell") protocol, by using the `ssh` command installed on their local computers (in the following, we will use "SSH" to refer to the SSH protocol, i.e., the language spoken between a local computer and a server to establish and maintain secure connections, and `ssh` to refer to the command line program that implements the client side of the SSH protocol). The "secure" in SSH denotes the fact that the entirety of the connection between a user's local computer and the remote server is enctrypted using strong cryptography, meaning that it is practically impossible for a third party (anybody besides the user, the user's local computer, the server, and the server's administrators) to read any of the data sent over the connection.

In order to be able to connect to a server over SSH, users need to have an account on that server. Unlike most web-based services, where users who visit a site for the first time can usually sign up by filling out an online form and then use the service immediately, computing servers in this context typically have some special procedure for requesting an account. A new user typically has to contact a system administrator or fill out some online form to request an account, and if the administrators accept the request, they will typically assign the new user an account name and some way to authenticate themselves to the server.

"Authentication" here means some way for the user to prove that they really are who they claim they are, and that they have permission to access the server. In the SSH protocol, there are two main authentication methods: SSH passwords and SSH keys. We will discuss these in detail in the following sections.

With that in mind, once a new user has an account on a server and a way to authenticate themselves, they can log in to the server using the aforementioned `ssh` command:

````
ssh -l <account name> <server name>
````

where `<account name>` is the name of the account that was assigned to the user, and `<server name>` is the Internet host name or IP address of the server. An alternative formulation of the same command is

````
ssh <account name>@<server hame>
````

which uses the "@" notation familiar from email addresses.

Running either one of these commands will establish a connection between the user's local computer and the remote server, and start a new shell session on the remote computer. As part of setting up that connection, the `ssh` command might ask for the user's password, in which case the user will have to enter the password that was assigned to them by the server's administrator. After the SSH connection has been established and while it remains active, anything typed into the terminal on which the `ssh` command was run will be forwarded to the shell on the remote server, and any output from that remote shell or programs run within it will be forwarded to, and printed by, the terminal on the user's local computer. In other words, users can interact with the remote shell running on the remote server in the same way as with the local shell on their local computer, but any commands executed in that shell will be run on the remote server. Finally, when the user shuts down the remote shell, for example via the `exit` command, the SSH connection will be closed, and the terminal on the user's local computer will control the local shell again.

An example SSH session might proceed like this:

````
me@mypc$ ssh testuser@testserver
Last login: Mon Dec  4 12:00:59 2023 from 192.168.2.109
testuser@testserver$ date
Tue Dec  5 09:03:13 PST 2023
testuser@testserver$ ls ~
src data Documents Downloads Temp ToDoList.txt
testuser@testserver$ exit
logout
Connection to testserver closed.
me@mypc$ 
````

In this example, the user first ran the `date` command, which printed the current time, then ran `ls ~` to see the list of files in their home directory, and finally returned control to their local PC by `exit`ing from the remote shell. Importantly, the `date` command printed the current time *on the remote server,* not on the user's local computer, and the `ls ~` command listed the files in the user's home directory *on the remote server,* not on the user's local computer, because both of those commands were *executed* on the remote server. It is only the *output* from running those commands on the remote server that are forwarded to and printed on the user's local computer.

There is a second way to use `ssh` to quickly execute a single command on a remote server without entering an interactive shell. This is done by appending the command line that should be executed on the remote server to the `ssh` command itself, like so:

````
me@mypc$ ssh testuser@testserver date
Tue Dec  5 09:10:14 PST 2023
me@mypc$ 
````

This example connected to the same account on the same server as before, executed the `date` command on the server to print *the server's* current time, and then immediately returned control to the user's local computer.

Basic Server Etiquette
----------------------

Don't forkbomb a server the first time you log in! (or the second, third, ... time, either)
(J/K, obviously not written yet.)

SSH Passwords
-------------

One of the main ways how SSH establishes a user's identity on a remote server is via passwords. SSH passwords work identically to the passwords used to log in to local computers or to access web services like Google. Importantly, SSH passwords are *not* managed by the SSH protocol itself. They are, in fact, standard passwords in the server's operating system (typically Linux). This means they follow the same rules as standard passwords, and have the same drawbacks and caveats.

If a server uses SSH passwords for authentication, that server's administrators will typically assign some random password to a new user and communicate that password to them through some insecure channel, usually email. As a result, a new user is usually expected to change their initial password the first time they log into their new account. As SSH passwords are simply operating system-level passwords on the server, they can be changed in the same way as local passwords, using the `passwd` command:

````
me@mypc$ ssh testuser@testserver
Last login: Tue Dec  5 09:02:20 2023 from 192.168.2.109
testuser@testserver$ passwd
Changing password for user testuser.
Current password: 
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
testuser@testserver$ 
````

The `passwd` command will first ask for the user's current password, i.e., the one that was assigned during account creation. It will then ask for the new password, check that password against its internal password rules, whatever they may be, and then ask for the new password for a second time to catch typos. If the new password passes the internal checks and is entered identically for the second time, the `passwd` command will print a confirmation message. If something went wrong, `passwd` will print an error message like this:

````
passwd: Authentication token manipulation error
````

If `passwd` printed a confirmation message, the new password is immediately activated and must be used on the next login. If it printed an error message, the current password is not changed and must continue to be used, and `passwd` must be run again to actually change it.

:::{important} SSH Password Hygiene

Due to SSH passwords being standard passwords stored on a remote server, users should **always:**

+ Assume that their password on any given server **will** be leaked or stolen at some point.
+ Use different passwords for different servers, to minimize the damage when one server they use will inevitably be hacked, and their password for that server will be stolen. By using different passwords for different servers, hackers will not be able to use the password they stole from one server to access the same user's accounts on other servers.
:::

SSH Keys
--------

The second main way how SSH establishes a user's identity on a remote server is via SSH keys. SSH keys are specific to the SSH protocol, and much more secure than SSH passwords, both for the user and the server, to the point that many servers no longer allow SSH passwords for login and require SSH keys.

Unlike SSH passwords, which are just strings of characters, SSH keys are cryptographic keys, typically based on the RSA algorithm, which is a super interesting topic for another day (https://en.wikipedia.org/wiki/RSA_cryptosystem). RSA uses a two-part asymmetric scheme consisting of a *private key* and a *public key.* Without going into the details of how RSA cryptography works, a public key is used to *encrypt* data, and the associated private key is used to *decrypt* data. Unlike in *symmetric* cryptography systems, in asymmetric systems like RSA public keys can be shared *freely, with anyone,* without impacting security. In fact, many people append their public keys to their email signatures, or post them on their personal web pages. Also unlike SSH passwords, SSH keys are not generated on or assigned by the server, but are property of the user, are generated and stored on the user's own computer, and a user's private keys are not shared with a server as part of establishing a connection to that server.

In the context of SSH authentication, RSA keys are used to establish a user's identity. First, the server's administrators associate the user's **public** key with the user's account (typically by asking the user to send them their public key via email or an online form, which is safe to do because public SSH keys can be freely shared with anyone). Then, when the user attempts to connect to the server over SSH, the server takes a small piece of data, encrypts it with the user's **public** key, and sends it to the `ssh` client program running on the user's computer. The `ssh` program will then decrypt the server's message using the user's **private** key, which is stored on the local computer, and subsequently prove to the server that it was able to decode the message (the details of how this proof works are very interesting, but beyond the scope of this reader). Being able to decode the server's message, in turn, proves to the server that the user knows the private key associated with the public key stored with the user's account, and is therefore who they claim to be. As a result of the assumption that anyone who knows a user's **private** SSH key *is* that user, while **public** SSH keys can be shared freely, **private** SSH keys *must* be kept private and safe. If a user's **private** SSH key(s) get accessed by hackers, those hackers will be able to access any of that user's accounts using those keys. It is therefore a good idea to protect one's SSH keys with passwords (more on that later).

SSH keys are created by running the `ssh-keygen` command on a user's local computer:

````
me@mypc$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/me/.ssh/id_rsa): /home/me/.ssh/newkey
Enter passphrase (empty for no passphrase):
Enter same passphrase again: 
Your identification has been saved in /home/me/.ssh/newkey
Your public key has been saved in /home/me/.ssh/newkey.pub
The key fingerprint is:
SHA256:dhJNxMNE02Gfe4IQuk64/v7wj/fcYc+LdxOZufF/rnU me@mypc
The key's randomart image is:
+---[RSA 3072]----+
|         o=*+.   |
|  .      o+oo    |
| . .    ..o.     |
|. .    S..       |
|.. . .  So. +    |
|o.  . .+oo *     |
|o. .. +.+   =   E|
|..  oooo.+ + . .o|
|..oo.+.o..o . o+o|
+----[SHA256]-----+
me@mypc$ 
````

These are the steps of the above procedure in detail:

* `ssh-keygen -t rsa` executes the SSH key generation program and instructs it to generate an SSH key using the RSA algorithm (`-t rsa`). There are several different key algorithms, and servers may require or recommend specific ones, but the most common key algorithm is RSA.
* `ssh-keygen` asks for the name of a file in which to store the **private** key. By default, as indicated in the prompt, private RSA keys will be stored in a file named id_rsa in a hidden `.ssh` directory in the user's home directory (`/home/me`). If the user simply presses the Enter key at this prompt, `ssh-keygen` will use the default file name, which is fine unless the user wants to maintain multiple keys.
* Next, `ssh-keygen` asks for an optional passphrase to protect the new private key. As mentioned above, it is crucial that **private** SSH keys be kept private and safe. As such, it is a good idea to protect them with a passphrase (which is just another term for password) that will prevent hackers from using the private key in case they get access to it somehow. This passphrase is not stored anywhere, and **must never be given to anyone.** When the `ssh` program needs to access a private key to connect to a server, it will ask the user for the passphrase, use it to access the private key, and then immediately forget it again.
* Then, `ssh-keygen` prints the name of file containing the **public** SSH key associated with the new private SSH key, which is just the name of the private SSH key file with ".pub" appended to it.
* Finally, `ssh-keygen` will print the new key's hash "fingerprint" and the key's "randomart," which are both unimportant to anyone who is not a cryptography nerd.

From a practical perspective, key-based authentication has a number of important differences to password-based authentication:

:::{important} SSH Key Hygiene
+ **Public** SSH keys can be freely shared with anyone, without concern. They could be published via email, on web pages, or in phone books.
+ **Private** SSH keys, unlike SSH passwords, are only stored on the user's local computer, and are *not* shared with a server while an SSH connection is established.
+ Because **public** SSH keys are safe to share, and **private** SSH keys are not shared with servers during connection establishment, it is safe to use the same SSH public/private key pair for any number of different servers. Even if one server gets hacked, the information that hackers could potentially glean would *not* allow them to log into the same user's account on other servers.
+ **Private** SSH keys **must be kept private.**
+ SSH keys are only as secure as the user's local computer. If a user's computer gets stolen or hacked, thieves or hackers could potentially gain access to that user's accounts on all servers for which the user has SSH keys, unless the keys themselves are protected by strong passphrases.
+ **NEVER EVER AT ABSOLUTELY NO TIME GIVE ANYONE YOUR PRIVATE SSH KEY EVER!**
+ If an SSH key is protected by a strong passphrase, it is practically impossible to use by a hacker even if said hacker were able to access the key files on a user's computer somehow.
+ **NEVER EVER AT ABSOLUTELY NO TIME GIVE ANYONE YOUR SSH KEY PASSPHRASE EVER!**
:::

The .ssh Configuration Directory
--------------------------------

The SSH protocol suite stores configuration data in a hidden `.ssh` directory in a user's home directory. Because this directory contains some data that is private to the user, such as private SSH keys, many SSH programs will refuse to operate if this directory or its contents have incorrect sets of permissions, i.e., access privileges.

:::{important} Access privileges for the `.ssh` directory
+ The `.ssh` directory **must** be readable, writable, and executable by the user.
+ The `.ssh` directory **should not** be readable, writable, or executable by the user's group and by other users.
+ The `.ssh` directory's permissions, as printed by `ls -l`, **should** be `drwx------.`.
+ In case of doubt, one should set the `.ssh` directory's permissions via `chmod u+rwx,go-rwx ~/.ssh`, or, equivalently, `chmod 0700 ~/.ssh`.
+ **Public** key files (`id_rsa.pub` etc.) **must** be readable and writable by the user, and **may** be readable, but **must not** be writable, by the user's group and by other users. In short, their permissions **should** be `-rw-r--r--.`. To set, `chmod u+rw,go+r,go-w ~/.ssh/<key file>.pub`.
+ **Private** key files (`id_rsa` etc.) **must** be readable and writable by the user, and **must not** be readable or writable by the user's group or other users. In short, their permissions **must** be `-rw-------.`. To set, `chmod u+rw,go-rw ~/.ssh/<key file>.pub`.
+ The `~/.ssh/config` file **must** be readable and writable by the user, and **must not** be readable or writable by the user's group or other users. In short, its permissions **must** be `-rw-------.`. To set, `chmod u+rw,go-rw ~/.ssh/config`.
+ Other files (`known_hosts`, `authorized_keys`, etc.) **must** be readable and writable by the user, and **should not** be readable or writable by the user's group or other users. In short, their permissions **should** be `-rw-------.`. To set, `chmod u+rw,go-rw ~/.ssh/<file>`.
:::

`~/.ssh/known_hosts`
--------------------

`~/.ssh/known_hosts` contains the public SSH keys of servers to which the user has connected in the past. These are stored to prevent so-called "man-in-the-middle" attacks, where a third party attempts to incercept SSH connection traffic by pretending to be a server to which a user wants to connect, accepting incoming SSH connnections from such a user, and then transparently forwarding those connections and their content to the actual server. While SSH connections are secure between the two ends of the connection, this setup would allow a third party to see unencrypted data travelling across the connection.

When a user connects to a server to which they have connected in the past, the `ssh` program compares that server's public SSH key to the one in the `known_hosts` file. If the keys do not match, `ssh` refuses to establish the connection, because it can not prove that the server is the same one to which it connected before, meaning that the server is potentially being impersonated.

In rare circumstances, servers may legitimately change their SSH keys. If `ssh` indicates a server key mismatch, the best approach is to directly contact the administrators of that server to confirm that the key was legitimately changed. If that is the case, the old key must manually be removed from the `known_hosts` file by opening that file with a text editor such as `vi`, finding the outdated server key (`ssh` will print the number of the line containing the mismatching key), delete it, and then save the edited file. Upon running `ssh` again, the program will not find the previous key, and accept the server's new key and add it to the `known_hosts` file.

`~/.ssh/config`
---------------

`~/.ssh/config` contains local configuration data for SSH tools, such as the `ssh` program. The most useful settings in that file are per-server settings that associate account names and potentially SSH key files with server names, to simplify managing different user identities on different servers. For example, if a user's `~/.ssh/config` file contains the following text:

````
Host testserver
User testuser
IdentityFile ~/.ssh/id_rsa

Host anotherserver
User m34754_a2
IdentityFile ~/.ssh/anotherserver_rsa
````

then it is no longer necessary to specify the testuser account name when logging into server testserver, and specifying the (non-standard) `~/.ssh/anotherserver_rsa` SSH key file and the m34754_a2 account name when logging into server anotherserver. In other words, instead of

````
me@mypc$ ssh testuser@testserver
````
and
````
me@mypc$ ssh -i ~/.ssh/anotherserver_rsa m34754_a2@anotherserver
````

the user can simply log into these two servers via

````
me@mypc$ ssh testserver
````
and
````
me@mypc$ ssh anotherserver
````

How To Move Data Between A Local Computer, The Internet, And A Server
---------------------------------------------------------------------

An established SSH connection between a user's local computer and remote server sends the user's input to the remote server, and sends the output of the shell running on the remote server to the user's local computer, but it does not otherwise send data or code between those two ends. Therefore, uploading data from a user's local computer to a server, downloading data from the Internet to a server, and downloading data from a server to a user's local computer require additional tools.

Download Files From The Internet
--------------------------------

The first pair of commands to download data from the Internet to *any* computer, including a user's local computer and a server to which a user is connected via SSH, are `wget` and `curl`. Both of these commands do basically the same thing, but for historical reasons, one or the other (or sometimes both) may be installed on any specific computer. It is therefore important to know the basics of using both of them.

`wget`
------

`wget` (as in "web get") is the simpler of the two. It can be used to copy one or more files, entire directories, or even entire directory hierarchies, from the Internet via a multiple protocols including HTTP and HTTPS. Its most common use is to download files from web servers. Usually, one downloads a file from a web server via a web browser, by right-clicking on a hyperlink in a web page and selecting "Save Link As..." from the menu that pops up in response. In its simplest form, `wget` does the same, but from the command line and without requiring further user input. To download the file referenced by a given URL, one simply passes that URL on `wget`'s command line:

````
me@mypc$ wget https://www.google.com
--2023-12-05 11:17:29--  https://www.google.com/
Resolving www.google.com (www.google.com)... 142.251.46.196, 2607:f8b0:4005:812::2004
Connecting to www.google.com (www.google.com)|142.251.46.196|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html                        [ <=>                                              ]  18.89K  --.-KB/s    in 0.001s  

2023-12-05 11:17:30 (22.5 MB/s) - ‘index.html’ saved [19348]

me@mypc$ ls -l index.html
-rw-r--r--. 1 me me 19348 Dec  5 11:17 index.html
me@mypc$ 
````

The above command will download the entirety of the Internet and store it on the user's local computer.

No, just kidding, it will download `index.html`, the HTML file that renders Google's frontpage, the one that is shown when visiting the https://www.google.com URL with a web browser. It's a surprisingly small file, and fun to look at.

In the same way, `wget` can download files referenced by any other URLs. A common practical way to do this is to visit the web site hosting a desired file using a web browser, to locate a hyperlink for the file one wants to download, then to right-click on that hyperlink and select "Copy Link" in the pop-up menu, and finally to paste the copied link into `wget`'s command line. And because `wget` is command line-based, this can be used to download a file to a server by executing `wget` on that server via an SSH connection:

````
me@mypc$ ssh testuser@testserver
Last login: Tue Dec  5 10:32:57 2023 from 192.168.2.109
testuser@testserver$ wget https://www.nasa.gov/wp-content/uploads/2023/03/135918main_bm1_high.jpg
--2023-12-05 11:27:28--  https://www.nasa.gov/wp-content/uploads/2023/03/135918main_bm1_high.jpg
Resolving www.nasa.gov (www.nasa.gov)... 192.0.66.108, 2a04:fa87:fffd::c000:426c
Connecting to www.nasa.gov (www.nasa.gov)|192.0.66.108|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 195439 (191K) [image/jpeg]
Saving to: ‘135918main_bm1_high.jpg’

135918main_bm1_high.jpg       100%[=================================================>] 190.86K  --.-KB/s    in 0.007s  

2023-12-05 11:27:28 (27.9 MB/s) - ‘135918main_bm1_high.jpg’ saved [195439/195439]

testuser@testserver$ 
````

When downloading a URL, `wget` will store the file referenced by that URL in the current directory from which `wget` was run, using the same file name as the file on the server (in the above example, `135918main_bm1_high.jpg`). This is often not useful, because the names of files on web servers are commonly auto-generated and non-descriptive. Instead, it is possible to specify a desired location and name for the downloaded file via the `-O <file name>` (that is *uppercase* O for "output file name") command line option for `wget`. The `-O <file name>` option must appear *before* the URL of the file to be downloaded. While at it, one can also reduce the amount of extra information `wget` prints while downloading via the `-q` (for "quiet") command line option:

````
testuser@testserver$ wget -q -O BlueMarble1972.jpg https://www.nasa.gov/wp-content/uploads/2023/03/135918main_bm1_high.jpg
testuser@testserver$ ls -l BlueMarble.jpg
-rw-rw-r-- 1 testuser testuser 195439 Apr  6  2023 BlueMarble1972.jpg
testuser@testserver$ 
````

True to form for a UNIX utility, wget will print *absolutely nothing* in quiet mode unless there is an error. The output file name specified via `-O` can contain absolute or relative paths, such as `Images/BlueMarble1972.jpg`, or `/home/testuser/Images/BlueMarble1972.jpg` or `~/Images/BlueMarble1972.jpg` for short.

`wget` can download multiple files in one go by using wildcards in the URL, such as

````
wget -q https://www.nasa.gov/wp-content/uploads/2023/03/*.jpg
````

which will download all JPEG image files in the March 2023 upload section of NASA's main web site (which is probably not a good idea!), or by referencing the name of a directory instead of a file, as in

````
wget -q https://www.nasa.gov/wp-content/uploads/2023/03/
````

(*also* not a good idea!). `wget` can even download an entire directory hierarchy via the `-r` (for "recursive") command line option:

````
wget -q -r https://www.nasa.gov/wp-content/uploads/
````

which will download all media files NASA ever uploaded to their web site (now this is an outright *terrible* idea!), and recreate the directory hierarchy of the web site on the user's local computer, i.e., the Blue Marble image would be stored in `www.nasa.gov/wp-content/uploads/2023/03/135918main_bm1_high.jpg`.

By default, `wget` stores downloaded files or directories in the current directory. This can be changed via the `-P <root directory>` option (for "directory prefix"). If given, directories and files will be created underneath the given directory, which can be identified by an absolute or relative path.

`curl`
------

`curl` (as in "copy URL") is another command line utility that can download files from the Internet. `curl` supports even more remote file access protocols than `wget`, and, according to its own documentation (`man curl`), "[its] number of features will make your head spin."

In its simplest invocation, `curl` will download the file referenced by a given URL and print its contents to the terminal. Instead of printing to the terminal, which is typically not useful, output redirection can then be used to save `curl`'s output to a file, or to pipe it as input into another program:

````
testuser@testserver$ curl https://www.nasa.gov/wp-content/uploads/2023/03/135918main_bm1_high.jpg > BlueMarble1972.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 19206    0 19206    0     0   137k      0 --:--:-- --:--:-- --:--:--  137k
testuser@testserver$ 
````

Instead of redirecting output to a file, `curl` can also be instructed to write to a file directly by using the `-o` or `-O` command line options. `-o` (*lowercase* o) needs a file name, while `-O` (*uppercase* O) uses the file name component of the given URL. In other words, `curl -O <URL>` aligns with the default behavior of `wget <URL>`, and `curl -o <file name> <URL>` aligns with `wget -O <file name> <URL>`. There is one big difference, however: unlike `wget`, `curl`'s `-o` option only accepts file names and does not support paths. To create the named file in a directory other than the current directory, one has to add the `--output-dir` (notice two dashes!) option. This behavior means that it is often easier to ignore the `-o` option, and use output redirection instead.

Like `wget`, `curl` will print continuous status updates while downloading, and write download statistics when done. Also like `wget`, these updates can be disabled, but unlike `wget`, the associated command line option is `-s` (for "silent"), and `curl` in silent mode will not even print a message if there is an error while downloading a file, unless error messages are re-enabled by adding `-S` (*uppercase* S, for "not so silent") after `-s`.

Transfer Files Between A Local Computer And A Server
----------------------------------------------------

Unlike `wget` and `curl`, the next pair of commands can be used to transfer files between a user's local computer and a remote server. The first, `scp`, is non-interactive and behaves similar to the standard UNIX `cp` command, but with the added ability to access files on remote servers. The second command, `sftp`, creates an interactive session on the remote server, similar to `ssh`, but unlike `ssh`, it does not connect to a standard UNIX shell, but to a custom shell that supports local and remote file operations and transfers. While `scp` can only transfer files between computers, `sftp` additionally supports file management operations like creating and removing directories, moving, renaming, and deleting files, etc.

`scp`
-----

`scp` is straightforward to use. Just like `cp`, it takes the name of a source file, and the name of a destination file or the name of a directory into which to copy the source file. Unlike `cp`, the source file and/or destination file or directory can exist on different computers. This is indicated by prefixing a regular path with the name of a server followed by a colon, as in `<server name>:<path>`, which will then refer to the path of the given name on the indicated remote computer. The server name can additionally be prefixed with an account name followed by an "@" sign, as in `<account name>@<server name>:<path>`. For example,

````
me@mypc$ scp BlueMarble1972.jpg testuser@testserver:Images/
````

will copy a `BlueMarble1972.jpg` file from the current directory on the local computer into the `Images` directory in the testuser account on the testserver server. Remote paths start from the remote user's home directory, unless they start with a `/`, which indicates that they start from the root directory of the remote server's file system. When a local or remote destination path identifies a directory, the destination file will be created in that destination directory with the same name as the source file. In that case, the destination directory must already exist, or the copy operation will fail. The command

````
me@mypc$ scp testuser@testserver:Images/BlueMarble1972.jpg .
````

will do the exact opposite, copying `BlueMarble1972.jpg` from the `Images` directory under the testuser account's home directory on server testserver to the current directory on the local computer. The `.` at the end of the command line is UNIX shorthand for the current directory, and thus tells `scp` to give the destination file the same name as the source file, and to store it in the current directory on the local computer. Equivalently, one could have specified the name of the destination file explicitly:

````
me@mypc$ scp testuser@testserver:Images/BlueMarble1972.jpg BlueMarble1972.jpg
````

`scp` also supports wildcards in source paths to copy more than one file at a time. For example,

````
me@mypc$ scp testuser@testserver:Images/*.jpeg Images
````

will copy all JPEG files (or, rather, files whose names end in `.jpeg`) contained in testuser's `Images` directory on the remote server to the `Images` directory on the local computer.

Like `cp`, `scp` supports recursive copying of entire directory hierarchies using the `-r` (*lowercase* r for "recursive") option. Note that the equivalent option for `cp` is `-R` (*uppercase* R). When used with `-r`, both the source and destination paths specified on `scp`'s command line must be directories.

`sftp`
------

Talk about `sftp` here.

The POSIX Directory Structure And Permissions
---------------------------------------------

As servers are typically UNIX machines, the layout of their filesystems typically follow the POSIX Filesystem Hierarchy Standard. This means the file system is generally split between system directories, user home directories, and "other" directories. System directories contain programs and data shared by all users on the server, while user home directories contain programs and data private to each user. "Other" directories can contain a mix of these two.

System Directories
------------------

As system directories contain data and programs that are shared by all users, their contents are typically readable by all users, but can only be modified by the superuser (root). In other words, their permissions are typically `drwxr-xr-x.`, and the permissions of the files contained within them are typically `-rw-r--r--.` for data files and `-rwxr-xr-x.` for program files. The most relevant system directories are:

* `/bin` and `/sbin` contain programs that are essential for operating a UNIX system, such as `ls`, `cat`, `cp`, etc. `sbin` contains user-level programs; `/sbin` contains programs that are only meant to be used by administrators.
* `/lib` and `/lib<arch>` (e.g., `/lib64`) contain library files (libraries are bundles of common code shared by multiple programs) essential to programs in `/bin` and `/sbin`.
* `/usr` is a second-level hierarchy containing the majority of utilities and applications installed on a UNIX system.
* `/usr/bin`, `/usr/sbin`, `/usr/lib`, `/usr/lib<arch>` contain programs, administrator-level programs, and libraries shared by those programs, respectively.
* `/usr/include` contains *include files* that are used by developers to create and build applications on the server, primarily using the libraries in `/usr/lib` and `/usr/lib<arch>`.
* `/usr/local` is a third-level hierarchy containing utilities and applications that have been built specifically for one server, i.e., are not part of the "standard" operating system. It in turn contains `bin`, `sbin`, `lib`, `lib<arch>`, `include`, etc. directories.
* `/etc` contains host-specific system-wide configuration files. According to UNIX lore, "etc" *does not* stand for "etcetera," and is pronounced "etsy" or sometimes "ee-tee-cee".
* `/opt` contains optional software packages accessible to all users on a system.
* `/media` contains mount points for removable media like CD-ROMs. In some UNIX versions, `/media` also contains mount points for temporary storage devices such as USB drives
* `/mnt` contains mount points for temporary storage devices such as USB drives. In some UNIX versions, these appear in `/media` instead.

User Home Directories
---------------------

Users' home directories appear inside the `/home` hierarchy, and are named by each user's account name, such as `/home/me` or `/home/testuser`. As home directories are supposed to be private to each user, they can typically be read and written by the user owning them, can be read but not written by the user's group, and are neither readable nor writable by other users. In short, their permissions are typically `drwxr-x---.`.

Sometimes a user might accidentally misconfigure their home directory's permissions to allow read or even write access by other users. Exploiting such a mistake by snooping in other user's home directories is considered a serious invasion of privacy, and *changing* files in other users' home directories is considered a capital offense and will most probably result in a lifetime ban from the server.

Other Directories
-----------------

Other directories are a mixed bag. The most relevant one is `/tmp`, which holds temporary files that are not meant to exist beyond the lifetime of whichever program created them. Many programs that need to create temporary files to operate create them here, and as such, the `/tmp` directory is typically readable and writable by all users. The files contained within it, on the other hand, are owned by the users who created them, respectively, and are typically readable and writable by those users, and neither readable nor writable by other users. A server's operating system or administrators might start deleting files in `/tmp` without warning if the server's file system starts to run out of space -- or for any other reason, really.
