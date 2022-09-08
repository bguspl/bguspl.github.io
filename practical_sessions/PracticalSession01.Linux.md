Practical session 1 - To do at home
Take home message
You MUST get familiar with your Linux account and handle your environment properly . Failing to do so, will cause you big headache when things go wrong due to wrong use of storage space. Your assignments might disappear and you might find yourself wasting a lot of time with system problems.
In the other hand, learning the system now, will benefit in the future (until you graduate). You may discover that the automatic backup system is easy, saving time and many more….

Very important: Create your Linux account
All 2nd year cs students can open an account in the department's computer system. You will use this account in many courses until you will graduate. With your new account you will be able to use the computer science's labs such as in building 34.
Your first task is to open this account.

Go to https://oldweb.cs.bgu.ac.il/help/passwd.html In the following window enter your credentials - BGU username and password. acc2.png
You'll need to read the agreement, sign at the bottom, and choose your CS password acc4.png
If everything went ok, you should be getting this screen:
acc6.png

Your password is valid for both Linux and Windows. (One password for both OS)

Each computer in the department holds 2 Run Time Environments (RTE) - Linux and Windows. You can choose which RTE to activate when the computer is rebooting.

This is also a good time to get familiar with the system team in room 007 building 37. They are the guys that will help you when you have a crisis in your account.

Need help?
Having problems with your account? the cs-computers is not working? Send a mail to help@cs.bgu.ac.il
If the problem is in a particular department computer please state the computer name and its mac address (see http://frodo.cs.bgu.ac.il/wiki/Support_Request?highlight=mac+address

We suggest that when ever you encounter a problem in the labs/accounts/computers etc. let the lab know in order to make life easier and the departments facilities in idle shape.

Prologue - Introduction to Linux
We assume this is your first encounter with Linux and your CS-account.
Enter your CS-account terminal
Step-by-step (START) Choose which of the following method applies to you:
Case sensitive
Linux exhibits case sensitivity; that is, words can differ in meaning based on differing use of uppercase and lowercase letters. Words with capital letters do not always have the same meaning when written with lowercase letters. For example, the files blabla and BlaBla are different. This could cause you problems when you are moving windows code to Linux.
From Linux in the lab
In the current version:
Click the K-menu button (parallel to Window's Start)
Application
System
Konsole or Terminal
You are now logged in to the terminal of the local computer.
From home
In order to connect remotely to the lab computers, you'll need to setup VPN. Follow the instructions here: VPN Setup
Once you've setup VPN, check this page for the list of available lab computers: Computers
From your Home Linux
Enter the command: ssh -X <username>@<computer-name>.cs.bgu.ac.il
For example, ssh -X spl211@cs314row1-1.cs.bgu.ac.il.

You might see the following message:labs3.png
type yes and enter.

From your Home Windows
Download putty for connecting to any computer in the lab. For putty documentation see instructions.
When using putty go to "Connection" -> "SSH" -> "X11" and enable X11 forwarding. putty2.png
Then go back to "Session", enter the hostname you wish to connect, and click "Open" putty1.png

Verify your Terminal
You should now have an open terminal in front of you.
Check your present-working-directory, type:
pwd
You should see: /users/studs/bsc/2021/your-name
Obviously, the line is different per student.
Step-by-step (END)
Basic Concepts: directories
What is the meaning of /users/studs/bsc/2021/your-name?
This is a directory. The first / is the root directory of the system. Under it there is the "users" directory, under that "studs" and so on.
This directory is called yourhome-directory. It is backed-up regularly. This directory can, therefore, be used for important files that you want to be saved.
To see how much space you have allocated to you and how much you are currently using, use the commands: quota or du

It is recommended to save all your work (e.g. your text, source code, assignments, homework, poems and stories you might have) in your home directory because it is automatically and frequently backup bu the system.

More info on your Unix account can be found here.
Student's question: let me see if I understand you correctly. It is recommended to save all file assignment files at my home directory.
Answer: Well, not exactly. The best thing you can do is to work with a source control. This is the right way to develop software, especially where a team of people may change the same files. See Revision control. In our department we have http://frodo.cs.bgu.ac.il/wiki/SVN server, it's free and it is very easy to use. We encourage you to use source control.

It is better to adapt good habits as earlier as you can!

Basic Commands
Step-by-step (START)
ls
Type: ls
You will see the list of files in your present working directory. Now, typels -lha . Can you see the different?
pwd
Type: pwd
You will see your present working directory.
cd
Type: cd stud
Type: cd your-name
pwd (to verify this).
Type: cd ~
This brought you back to your home-directory.
Type: du -h ~
Type: du -s ~/ to check how much storage you are currently using in your home directory.
man
Type: man ls
You will see the manual (help page) for the ls command. Use this command to learn about the rest of the commands. To quit press "q".
Step-by-step (END)

cp
rm
cat
mkdir
du
Tab will auto-complete your command (if only one option exist) or show all options available.
Use the up-arrow to repeat to the previous command

FTP
You can access your home directory files using FTP. To do so, download WinSCP, a free open source ftp utility that you can use to FTP into your account. This way you can easily copy files from and to your account from your local PC.
To connect to your account, enter the hostname you wish to connect, and your linux username and password, just like in the putty case. winscp.png


Personal WebPage Setup - Optional
You can create a personal home page inside our department as : http://www.cs.bgu.ac.il/~YOUR_USER_NAME
To create your home page you need to:

Create directory ".html" : In Linux type mkdir .html
Inside .html create index.html or index.php file which will be your leading home page :
enter the directory: cd .html
creating index file: touch index.html
Make sure your home dir & .html has at least 711 permission (so that the web server will be able to read files under ~/.html):
to change permission to home dir: chmod 711 ~YOURUSERNAME
change permission to .html: chmod 711 ~/.html/
The files under .html should be world readable (permission 755) : chmod -R 755 ~/.html/*
Protection
For protection from other users that can access to your html directories, you can creat two files:
.htpasswd in some directory (in home directory for example)
.htaccess in directory that you want to protect.

File .htpasswd may be created by standard Unix utility htpasswd.
Just type htpasswd -c .htpasswd username and after enter a password for.

File .htaccess may contains next lines:

    AuthName "My Secure Directory"
    AuthType Basic
    AuthUserFile /absolute/directory/path/to/.htpasswd
    AuthGroupFile /dev/null
    <Files
    ~ "^\.ht(access|passwd)">
    Order allow,deny
    Deny from all
</Files>
<Limit GET POST>
    require valid-user
</Limit>
Pay attention that .htaccess has right absolute path to your

.htpasswd
.

Your personal home page must be noncommercial and must not contain any illegal or offensive data!

Virtual Machine
The frontal checks will be done with a virtual machine, containing an ubuntu 18.04 image, you are required to verify that your assignment works in this virtual machine.
In order to run the VM, download the image from here, install the relevant virtual box distribution (e.g. Linux, Mac, Windows. Use 6.1.4) from here.
After installing, run VirtualBox, and import the virtual machine image. The image contains all the relevant tools for coding and compiling, if you need to install additional software you are allowed to install it.
