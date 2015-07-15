---
layout: post
title: Share Files between Windows and Linux
categories: linux
tags: [linux, windows, share]
---

##Windows as host(Using CIFS)
###Operations on Windows
Slect the folder you would like to share, then right-click, follow options as follow: select `property`, change to `share` tab, click `share`, then select user and authory. Finishing share by click `share` button.
###Operations on Linux
1. Install CIFS

    ```sh
    sudo apt-get install cifs-utils
    ```
2. Mounting shared folders
    1. Create a mount point whereever you like, for example:

        ```sh
        sudo mkdir /media/windows_share
        ```
    2. Add to configure file
        edit file `/etc/fstab`, add a new line: 
        
        ```
        //windows_ip/share_folder /media/windows_shre cifs username=windows_user,password=windows_passwd,iocharset=utf-8,sec=ntl 0 0
        ```
        If you would like to do something in the shared folder(eg: compile a c program), you may encounter a fatal error:
        >Value too large for defined data type  

        You have to append `nounix`, `noserverino` to the fourth.

##Linux as host(Using ftp or samba)
###Sharing via ftp
1. Install vsftpd
    
    ```sh
    sudo apt-get install vsftpd
    ```
2. Configuring vsftpd
    Edit file `/etc/vsftd.conf`, simply uncomment these lines to acquire read/write acess.

    ```
    annoymous_enable=NO
    local_enable=YES
    write_enable=YES
    ```
    More options can be found in the file.
3. Start service

    ```sh
    sudo service vsftpd start
    sudo chkconfig vsftpd on
    ```
    First command start vsftpd service, and the second enable service on startup.
4. On windows, you can access your linux `HOME` folder via a ftp client or a browser.

###Sharing via samba
1. Install samba

    ```sh
    sudo apt-get install samba samba-common
    ```
2. add a samba user

    ```sh
    sudo smbpasswd -a username
    ```
3. Edit file `/etc/samba/smb.conf`, simply uncomment these lines:

    ```
    [homes]
        comment = Home Directores
        browseable = yes
        writeable = yes
    ```
4. Start service

    ```sh
    sudo service smbd start
    sudo chkconfig smbd on
    ```
5. Check samba user with `pdbedit -Lv`.
6. Right-click `computer`, select `Map Network Drive`, type server info `\\server\shared_folder` in the `Folder` box, then type your smb user credential.

##Which is better
Generally, ftp is faster than samba/CIFS[1](http://arstechnica.com/civis/viewtopic.php?t=221436). However, samba and CIFS is more convenient. If you just have to transfer a very big file, ftp is better. If you have a limited storage or you are working on a ARM board, CIFS is the best choice.

Ref: [Mount Windows Shares Permanently](https://wiki.ubuntu.com/MountWindowsSharesPermanently)
