# 0. Introduction/Setup.
Creating new user accounts is a basic, common task on any operating system. On Linux, most distributions provide a way to create user accounts via a GUI. However, these don't always give one all of the options that one might need. Also, if one is running a Linux server without a GUI, this luxury will not be available. Fortunately, creating users is easy and straightforward via the command line, if you know the commands to use. This tutorial was tested on Debian 12, but all of these commands should work on basically any Linux distribution.

Note that you will need to either log in as root or run these commands with sudo, as shown below. Also, these commands are all to be run in your Linux terminal.

# 1. Creating a user.
The most basic command for creating a user goes as follows. This doesn't create a home directory (folder) for the user, though. So typically this isn't desirable. For the purposes of the tutorial, I will be creating and dealing with a new user account named `username`.

> sudo useradd username

This command is the one that you are most likely to want to run. The `-m` flag specifies that a folder should be created for the user. In this case, it will create `/home/username` on most systems.

> sudo useradd -m username

The following command allows one to specify the location at which the home folder will be created. Most of the time the default location is what you want. If not, this is the command for you.

> sudo useradd -m -d /somewhere/somename username

The following command creates a user with an expiration date. The `-e` flag does this, and uses the format as seen in the command below. This is for creating temporary user accounts, which can be useful sometimes.

>sudo useradd -e YYYY-MM-DD username

# 2. Setting a user's password.
One will want to set a password so that the user account can actually be logged into. This is accomplished with the command below. You will be prompted to enter a new password for the account, and then prompted again to confirm the password. Note that this command can be used to change the password of any account, provided you have the proper permissions to do so.

> sudo passwd username

# 3. Giving user sudo access. (optional)
One may want to give the new user the ability to run commands with sudo. This is similar to an administrator account on Windows. If you are creating an account for yourself or someone who is responsible, trustworthy, and has reason to need sudo privileges, this is the command for you. Otherwise, you shouldn't give a user these privileges. Not giving a user these privileges if they don't absolutely need them is an excellent way of providing additional security for your system.

Something to note is that the `-a` flag means append in this context. That means that the user will be added to the group or groups specified after the `-G` flag, but stay in whatever groups they are already in. If you do not include the `-a` flag, the user will be removed from all current groups and added only to the ones you specify in this command. Also note that a user can be added to any group using this command, not just the sudo group.

> sudo usermod -a -G sudo username

If this command fails with a message saying there is no sudo group, you may need to install the `sudo` package. Instructions for doing so and configuring the sudoers file on your system are beyond the scope of this tutorial. There are many online resources that can assist with that task, however.
