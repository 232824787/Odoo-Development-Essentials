# Chapter 1 Getting Started with Odoo Development
*******************

Before we dive into Odoo development, we need to set up our development environment, and you need to learn the basic administration tasks for it.  

In this chapter, you will learn how to set up the work environment, where we will later build our Odoo applications.  

You will also learn how to set up a Debian or Ubuntu system to host our development server instances, and how to install Odoo from the GitHub source code. Then you will learn how to set up  le sharing with Samba, allowing you to work on Odoo  les from a workstation running Windows or any other operating system.  

Odoo is built using the Python programming language and uses the PostgreSQL database for its data storage, so these are the main requirements we should have in our Odoo host.  

To run Odoo from source, we will need to install  rst the Python libraries it depends on. The Odoo source code can then be downloaded from GitHub and executed from source. While we can download a zip or tarball, it's best to get the sources using GitHub, so we'll also have it installed on our Odoo host.  

## Setting up a host for the Odoo server
We will prefer using Debian/Ubuntu for our Odoo server, even though you will still be able to work from your favorite desktop system, be it Windows, Macintosh, or Linux.  

Odoo can run on a variety of operating systems, so why pick Debian at the expense of other operating systems? Because Odoo is developed primarily with the Debian/ Ubuntu platform in mind, it supports Odoo better. It will be easier to  nd help and additional resources if working with Debian/Ubuntu.  

It's also the platform the majority of developers work on, and where most deployments are rolled out. So, inevitably, Odoo developers will be expected to be comfortable with that platform. Even if you're from a Windows background it will be important to have some knowledge about it.  

In this chapter, you will learn how to set up and work with Odoo hosted in a Debian system, using only the command line. For those more at home with a Windows system, we will cover how to set up a virtual machine to host the Odoo server. As a bonus, the techniques you will learn will also allow you to manage Odoo in cloud servers where your only access will be through **Secure Shell (SSH)**.  

>### Note
>Keep in mind that these instructions are intended to set up a new system for development. If you want to try some of them in an existing system, always take a backup ahead of time to be able to restore it in case something goes wrong.  

### Provisions for a Debian host
As explained earlier, we will need a Debian host for our Odoo version 8.0 server. If these are your  rst steps with Linux, you may like to know that Ubuntu is a Debian-based Linux distribution, so they are very similar.  

>### Note
>Odoo is guaranteed to work with the current stable version of Debian or Ubuntu. At the time of writing this book, these are Debian 7 "Wheezy" and Ubuntu 14.04 "Trusty Tahr". Both ship with Python 2.7, necessary to run Odoo.  

If you are already running Ubuntu or another Debian-based distribution, you're set; this machine can also be used as a host for Odoo.  

For the Windows and Macintosh operating systems, it is possible to have Python, PostgreSQL, and all the dependencies installed, and then run Odoo from source natively.  


However, that could prove to be a challenge, so our advice is to use a virtual machine running Debian or Ubuntu Server. You're welcome to choose your preferred virtualization software to get a working Debian system in a VM. If you need some guidance, here is some advice: regarding the virtualization software,
you have several options, such as Microsoft Hyper-V (available in some versions
of Windows), Oracle VirtualBox, or VMWare Player (or VMWare Fusion for Macintosh). VMWare Player is probably easier to use, and free-to-use downloads can be found at https://my.vmware.com/web/vmware/downloads.  

Regarding the Linux image to use, Ubuntu Server is more user friendly to install than Debian. If you're beginning with Linux, I would recommend trying a ready- to-use image. TurnKey Linux provides easy-to-use, preinstalled images in several formats, including ISO. The ISO format will work with any virtualization software you choose, or even on a bare-metal machine you might have. A good option might be the LAPP image, found at http://www.turnkeylinux.org/lapp.  

Once installed and booted, you should be able to log in to a command-line shell.  

If you are logging in using root, your  rst task should be to create a user to use for your work, since it's considered bad practice to work as root. In particular, the Odoo server will refuse to run if you are using `root`.  

If you are using Ubuntu, you probably won't need this since the installation process has already guided you in the creation of a user.  

### Creating a user account for Odoo  为Odoo创建一个账户
First, make sure sudo is installed. Our work user will need it. If logged in as `root`:  

首先，已经安装了sudo。我们要创建的这个用户需要用到它。如果您已经以`root`身份登录shell，请执行以下命令：  

```shell
# apt-get update && apt-get upgrade # Install system updates
# apt-get install sudo  # Make sure 'sudo' is installed
```

The following commands will create an odoo user:  

下面的命令会创建一个odoo用户：  

```shell
# useradd -m -g sudo -s /bin/bash odoo # Create an 'Odoo' user with sudo powers 创建一个拥有sudo权限的用户‘Odoo’
# passwd odoo  # Ask and set a password for the new user 为新用户设置一个密码
```

You can change odoo to whatever username you want. The `-m` option has its home directory created. The `-g sudo` adds it to the sudoers list, so it can run commands as root, and the `-s /bin/bash` sets the default shell to bash, which is nicer to use than the default `sh`.  

你可以将odoo改为任意一个自己喜欢的名字。 `-m` 选项表示用户创建了自己家目录。`-g sudo`将用户添加到sudoer用户列表，这样用户才可以用root身份运行命令，`-s /bin/bash` 设置默认的shell为bash，这比默认使用的`sh`更为友好。  

Now we can log in as the new user and set up Odoo.  

现在我们可以以新用户身份进行登录，并设着Odoo了。  

## Installing Odoo from source
Ready-to-install Odoo packages can be found at nightly.odoo.com, available as Windows (.exe), Debian (.deb), CentOS (.rpm), and source code tarballs (.tar.gz).  

As developers, we will prefer installing directly from the GitHub repository. This will end up giving us more control over versions and updates.  

To keep things tidy, let's work in an /odoo-dev directory inside your home directory. Throughout the book, we will assume this is where your Odoo server is installed.  

First, make sure you are logged in as the user created above, or during the installation process, and not as root. Assuming your user is odoo, you can con rm this with the following command:  

```shell
$ whoami
odoo
$ echo $HOME
/home/odoo
```

Now we can use this script. It shows us how to install Odoo from source in a Debian system:  

```shell
$ sudo apt-get update && sudo apt-get upgrade  # Install system updates
$ sudo apt-get install git  # Install Git
$ mkdir ~/odoo-dev  # Create a directory to work in
$ cd ~/odoo-dev  # Go into our work directory
$ git clone https://github.com/odoo/odoo.git -b 8.0  # Get Odoo source code
$ ./odoo/odoo.py setup_deps  # Installs Odoo system dependencies
$ ./odoo/odoo.py setup_pg  # Installs PostgreSQL & db superuser for unix
user
```

At the end, Odoo should be ready to be used. The ~ symbol is a shortcut for
your home directory (for example, /home/odoo). The git -b 8.0 option asks to explicitly download the 8.0 branch of Odoo. At the time of writing this book, this is redundant, since 8.0 is the default branch, but this may change, so it will make the script time resilient.  

To start an Odoo server instance, just run odoo.py:   

```shell
$ ~/odoo-dev/odoo/odoo.py
```

By default, Odoo instances listen from port 8069, so if we point a browser to http://<server-address>:8069 we will reach that instance. When we are accessing it for the  rst time, it will show us an assistant to create a new database, as shown in the following screenshot:  

img:omit   

But we will learn how to initialize new databases from the command line, now so press Ctrl + C to stop the server and get back to the command prompt.  

## Initializing a new Odoo database 初始化Odoo数据库
To be able to create a new database, your user must be a PostgreSQL superuser. The ./odoo.py setup_pg does that for you; otherwise use the following command to create a PostgreSQL superuser for the current Unix user with:  

```shell
$ sudo createuser --superuser $(whoami)
```

To create a new database we use the command createdb. Let's create a v8dev database:  

```shell
$ createdb v8dev
```

To initialize this database with the Odoo data schema we should run Odoo on the empty database by using the -d option:  

```shell
$ ~/odoo-dev/odoo/odoo.py -d v8dev
```

This will take a couple of minutes to initialize a v8dev database, and will end with an INFO log message Modules loaded. Then the server will be ready to listen to client requests.  

By default, this method will initialize the database with demonstration data, which often is useful on development databases. To initialize a database without demonstration data, add to the command the option: --without-demo-data=all.  

Open `http://<server-name>:8069` in your browser to be presented with the login screen. If you don't know your server name, type the hostname command at the terminal to  nd it, or the ifconfig command to  nd the IP address.  

If you are hosting Odoo in a virtual machine you might need to do some network con guration to be able to use it as a server. The simplest solution is to change the VM network type from NAT to Bridged. With this, instead of sharing the host IP address, the guest VM will have its own IP address. It's also possible to use NAT, but that requires you to con gure port forwarding, so your system knows that some ports, such as 8069, should be handled by the VM. In case you're having trouble, hopefully these details can help you  nd help in the documentation for your chosen virtualization software.  

The default administrator account is admin with password admin. Upon login you are presented with the Settings menu, displaying the installed modules. Remove the Installed  lter and you will be able to see and install any of the of cial modules.  

Whenever you want to stop the Odoo server instance and return to the command line, press `Ctrl + C`. At the bash prompt, pressing the Up arrow key will bring you the previous shell command, so it's a quick way to start Odoo again with the same options. You will see the `Ctrl + C `followed by Up arrow and Enter is a frequently used combination to restart the Odoo server during development.  

### Managing your databases 管理数据库
We've seen how to create and initialize new Odoo databases from the command line. There are more commands worth knowing for managing databases.  

You already know how to use the createdb command to create empty databases, but it can also create a new database by copying an existing one, by using a --template option.  

Make sure your Odoo instance is stopped and you have no other connection open on the v8dev database created above, and run:  

```shell
$ createdb --template=v8dev v8test
```

In fact, every time we create a database, a template is used. If none is speci ed, a prede ned one called template1 is used.  

To list the existing databases in your system use the PostgreSQL utility psql utility with the -l option:  

```shell
$ psql -l
```

Running it we should see listed the two databases we created so far: v8dev and v8test. The list will also display the encoding used in each database. The default is UTF8, which is the encoding needed for Odoo databases.  

To remove a database you no longer need (or want to recreate), use the dropdb command:  

```shell
$ dropdb v8test
```

Now you know the basics to work with several databases. To learn more on PostgresSQL, the of cial documentation can be found at http://www.postgresql. org/docs/  

>### WARNING: 
>The drop database will irrevocably destroy your data. Be careful when using it and always keep backups of your important databases before using it.  

## A word about Odoo product versions 论Odoo的生产版本
At the date of writing, Odoo's latest stable is version 8, marked on GitHub as branch 8.0. This is the version we will work with throughout the book.  

当编写本书的时候，Odoo的最新稳定本是8，在Github上标记为分支8.0。整本书使用的都是这一个版本。  

It's important to note that Odoo databases are incompatible between Odoo major versions. This means that if you run Odoo 8 server against an Odoo/OpenERP 7 database, it won't work. Non-trivial migration work is needed before a database can be used with a later version of the product.  

The same is true for modules: as a general rule a module developed for an Odoo major version will not work with other versions. When downloading a community module from the Web, make sure it targets the Odoo version you are using.
On the other hand, major releases (7.0, 8.0) are expected to receive frequent updates, but these should be mostly  xes. They are assured to be "API stable", meaning
that model data structures and view element identi ers will remain stable. This is important because it means there will be no risk of custom modules breaking due to incompatible changes on the upstream core modules.  

And be warned that the version in the master branch will result in the next major stable version, but until then it's not "API stable" and you should not use it to build custom modules. Doing so is like moving on quicksand: you can't be sure when some changes will be introduced that will make you custom module break.  

## More server configuration options 另外的服务器配置选项
The Odoo server supports quite a few other options. We can check all available options with the --help option:  

```shell
$ ./odoo.py --help
```

It's worth while to have an overview on the most important ones.  

### Odoo server configuration files Odoo的服务端配置文件
Most of the options can be saved in a con guration  le. By default, Odoo will use the .openerp-serverrc  le in your home directory. Conveniently, there is also the --save option to store the current instance con guration into that file:  

```shell
$ ~/odoo-dev/odoo/odoo.py --save --stop-after-init  # save configuration to file
```

Here we also used the `--stop-after-init` option, to have the server stop after it  nishes its actions. This option is often used when running tests or asking to run a module upgrade to check if it installs correctly.  

Now we can inspect what was saved in this default configuration file:  

```shell
$ more ~/.openerp_serverrc  # show the configuration file
```

This will show all con guration options available with the default values for them. Editing them will be effective the next time you start an Odoo instance. Type q to quit and go back to the prompt.  

We can also choose to use a speci c con guration  le, using the `--conf=<filepath>` option. Con guration  les don't need to have all those the options you've just seen. Only the ones that actually change a default value need to be there.  

### Changing the listening port 变更侦听端口
The `--xmlrpc-server=<port>` command allows us to change the default 8069 port where the server instance listens. This can be used to run more than one instances at the same time, on the same server.  

`--xmlrpc-server=<port>`命令允许我们改变服务器实例侦听的默认8069端口。这就可以在相同的服务器上同一时间运行多个实例。  

Let's try that. Open two terminal windows. On the first one run:  

让我们来试一试。打开终端窗口。在第一个终端中运行下面的命令：  

```shell
$ ~/odoo-dev/odoo.py --xmlrpc-port=8070
```

and on the other run:  

接着在另外一个终端运行：  

```shell
$ ~/odoo-dev/odoo.py --xmlrpc-port=8071
```

And there you go: two Odoo instances on the same server listening on different ports. The two instances can use the same or different databases. And the two could be running the same or different versions of Odoo.  

这里你可以看到：在同一个服务器上两个Odoo实例分别侦听了不同的端口。

### Logging 日志
The `--log-level` option allows us to set the log verbosity. This can be very useful to understand what is going on in the server. For example, to enable the debug log level use: `--log-level=debug`  

`--log-level`选项允许我们设置详细日志。这对于理解服务器上所发生的事情非常有帮助。例如，启用调试日志级别为：`--log-level=debug`   

The following log levels can be particularly interesting:  

下面的日志级别都有各自特别的用处：  

- debug_sql to inspect SQL generated by the server
- debug_rpc to detail the requests received by the server
- debug_rpc to detail the responses sent by the server

- debug_sql 检查服务器生成的SQL
- debug_rpc 详细打印服务器接收到的请求
- debug_rpc 详细打印服务端发生的响应

By default the log output is directed to standard output (your console screen), but it can be directed to a log  le with the option `--logfile=<filepath>`.  

默认，日志直接地输出标准输出（在你的控制台屏幕），不过日志也可使用选项`--logfile=<filepath>`直接地记录到日志文件。  

Finally, the `--debug` option will bring up the Python debugger (`pdb`) when an exception is raised. It's useful to do a post-mortem analysis of a server error. Note that it doesn't have any effect on the logger verbosity. More details on the Python debugger commands can be found here: https://docs.python.org/2/library/pdb.html#debugger-commands.  

最后，当一个异常被抛出时，`--debug`选项将引用Python调试器（pdb）。注意该选项对于记录详细日志没有任何的效果。这里，你可以找到关于Python调试器的更多信息：https://docs.python.org/2/library/pdb.html#debugger-commands。  

## Developing from your workstation 在自己工作站上进行开发
You may be running Odoo with a Debian/Ubuntu system, either in a local virtual machine or in a server over the network. But you may prefer to do the development work in your personal workstation, using your favorite text editor or IDE.  

This may frequently be the case for developers working from Windows workstations. But it also may be the case for Linux users that need to work on an Odoo server over the local network.  

A solution for this is to enable  le sharing in the Odoo host, so that  les are easy to edit from our workstation. For Odoo server operations, such as a server restart, we can use an SSH shell (such as PuTTY on Windows) alongside our favorite editor.  

### Using a Linux text editor 使用Linux下的文本编辑器
Sooner or later, we will need to edit  les from the shell command line. In many Debian systems the default text editor is vi. If you're not comfortable with it, then you probably could use a friendlier alternative. In Ubuntu systems the default text editor is nano. You might prefer it since it's easier to use. In case it's not available in your server, it can be installed with:  

```shell
$ sudo apt-get install nano
```

In the following sections we will assume nano as the preferred editor. If you prefer any other editor, feel free to adapt the commands accordingly.  

### Installing and configuring Samba Samba的安装和配置
The Samba project provides Linux  le sharing services compatible with Microsoft Windows systems. We can install it on our Debian/Ubuntu server with:  

```shell
$ sudo apt-get install samba samba-common-bin
```

The samba package installs the file sharing services and the samba-common-bin package is needed for the smbpasswd tool. By default users allowed to access shared  les need to be registered with it. We need to register our user odoo and set a password for its  le share access:  

```shell
$ sudo smbpasswd -a odoo
```

After this the odoo user will be able to access a  leshare for its home directory, but it will be read only. We want to have write access, so we need to edit Sambas, con guration  le to change that:  

```shell
$ sudo nano /etc/samba/smb.conf
```

In the con guration file, look for the [homes] section. Edit its con guration lines so that they match the settings below:  

```shell
   [homes]
      comment = Home Directories
      browseable = yes
      read only = no
      create mask = 0640
      directory mask = 0750
```

For the con guration changes to take effect, restart the service:  

```shell
$ sudo /etc/init.d/smbd restart
```

>### Tip 
>Downloading the example code  
>You can download the example code files for all Packt books you have purchased from your account at http://www.packtpub.com. If you purchased this book elsewhere, you can visit http://www. packtpub.com/support and register to have the files e-mailed directly to you.  

>### 提示
>下载示例代码
>你可以从Packt图书下载所有的示例代码。如果你从其他地方购买本书，你可以访问http://www. packtpub.com/support ，注册后会有文件直接电邮给你。  

To access the  les from Windows, we can map a network drive for the path `\\<my-server-name>\odoo` using the speci c user and password de ned with smbpasswd. When trying to log in with the odoo user, you might  nd trouble with Windows adding the computer's domain to the user name (for example `MYPC\odoo`). To avoid this, use an empty domain by prepending a \ to the login (for example `\odoo`).  

img:omit  

If we now open the mapped drive with Windows Explorer, we will be able to access and edit the contents of the odoo user home directory.  

img:oimt  

## Enabling the on-board technical tools 启动自带的技术工具
Odoo includes some tools that are very helpful for developers, and we will make use of them throughout the book. They are the Technical Features and the Developer Mode.  

These are disabled by default, so this is a good moment to learn how to enable them.  

### Activating the Technical Features 激活技术功能
Technical Features provide advanced server con guration tools.  

They are disabled by default, and to enable them, we need to log in as admin. In the Settings menu, select Users and edit the Administrator user. In the Access Rights tab, you will  nd a Technical Features checkbox. Let's check it and save.  

Now we need to reload the page in our web browser. Then we should see in the Settings menu a new Technical menu section giving access to many Odoo server internals.  

The Technical menu option allows us to inspect and edit all Odoo con gurations stored in the database, from user interface to security and other system parameters. You will be learning more about many of these throughout the book.  

### Activating the Developer mode  激活开发者模式
The Developer mode enables a combobox near the top of Odoo windows, making a few advanced con guration options available throughout the application. It also disables the mini cation of JavaScript and CSS used by the web client, making it easier to debug client-side behavior.  

To enable it, open the drop-down menu from the top-right corner of the browser window, next to the username, and select the About Odoo option. In the About dialog, click on the Activate the developer mode button at the top-right corner.  

After this, we will see a Debug View combo box at the top left of the current form area.  

## Installing third-party modules 安装第三方模块
Making new modules available in an Odoo instance so they can be installed is something that newcomers to Odoo frequently  nd confusing. But it doesn't have to be so, so let's demystify it.  

### Finding community modules 雪照社区模块
There are many Odoo modules available from the Internet. The apps.odoo.com website is a catalogue of modules that can be downloaded and installed in your system. The Odoo Community Association (OCA) coordinates community contributions and maintains quite a few module repositories on GitHub, at https://github.com/OCA/  

因特网上有很多的可用的Odoo模块。

To add a module to an Odoo installation we could just copy it into the addons directory, alongside the official modules. In our case, the addons directory is at ~/odoo-dev/odoo/addons/. This might not be the best option for us, since our Odoo installation is based on a version controlled code repository, and we will want to keep it synchronized with the GitHub repository.  

要对Odoo添加一个模块，你可以直接将模块复制到插件目录，和官方模块放倒一起。对我们而言，插件目录位于~/odoo-dev/odoo/addons/。对我们来说这可能不是最好的选择，因为Odoo的安装基于了版本控制的代码仓库，我们

Fortunately, we can use additional locations for modules, so we can keep our custom modules in a different directory, without having them mixed with the of cial addons.  

As an example, we will download the OCA project department and make its modules available in our Odoo installation. This project is a set of very simple modules adding a Department  eld on several forms, such as Projects or CRM Opportunities.  

To get the source code from GitHub:  

```shell
$ cd ~/odoo-dev
$ git clone https://github.com/OCA/department.git -b 8.0
```

We used the optional -b option to make sure we are downloading the modules for the 8.0 version. Since at the moment of writing 8.0 is the projects default branch we could have omitted it.  

After this, we will have a new `/department` directory alongside the `/odoo` directory, containing the modules. Now we need to let Odoo know about this new module directory.  

### Configuring the addons path 配置插件目录
The Odoo server has a con guration option called addons-path setting where to look for modules. By default this points at the /addons directory where the Odoo server is running.  

Fortunately, we can provide Odoo not only one, but a list of directories where modules can be found. This allows us to keep our custom modules in a different directory, without having them mixed with the of cial addons.  

Let's start the server with an addons path including our new module directory:  

```shell
$ cd ~/odoo-dev/odoo
$ ./odoo.py -d v8dev --addons-path="../department,./addons"
```

If you look closer at the server log you will notice a line reporting the addons path in use: **INFO ? openerp: addons paths**: (...). Con rm that it contains our department directory.  

### Updating the module list 更新模块列表
We still need to ask Odoo to update its module list before these new modules are available to install.  

For this we need the Technical menu enabled, since the Update Modules List menu option is provided by it. It can be found in the Modules section of the Settings menu.  

After running the modules list update we can con rm the new modules are available to install. In the Local Modules list, remove the Apps  lter and search for department. You should see the new modules available.  

## Summary 总结
In this chapter, you learned how to set up a Debian system to host Odoo and to install it from GitHub sources. We also learned how to create Odoo databases and run Odoo instances. To allow developers to use their favorite tools in their personal workstation, we also explained how to con gure  le sharing in the Odoo host.  

We should now have a functioning Odoo environment to work with and be comfortable managing databases and instances.  

With this in place, we're ready to go straight into the action. In the next chapter we will create from scratch our  rst Odoo module and understand the main elements it involves.  

So let's get started!  


