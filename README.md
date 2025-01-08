# Qubic Node - VM - SetUp ENGLISH

Creado por: Marti Ras
Hora de creación: January 7, 2025 3:41 PM

## ENGLISH

- Some general comments before starting. Currently, the virtual machine running the node where the 'Qubic network' operates is on a [Hetzner](https://robot.hetzner.com/server) server for the following reasons: firstly, because it was indicated in their guides and kavatak (Qubic testing staff who guided me) also has it set up this way. And secondly, because to configure the VM where the node will run, you need a computer with Ubuntu 22.04 operating system, amd64, with 64GB of RAM and a static IP.
- For this first installation, the PC I used to connect to the server runs Ubuntu, so I also used TigerVNC Viewer for Linux. However, I will expand this section for Windows, because the smart contract compilation and deployment must be done with a Windows PC, and there I will probably use RemoteRipple VNC Viewer.
- I will use 'Qubic network' as a general term to refer to the Qubic network, whether testnet or mainnet. The difference will be that when running the node, if the qubic.efi file (boot loader) has been compiled with the port in the range between 31841 and 31844, we are on testnet. While if it is compiled to run on port 21841, we are on mainnet. Currently, qubic.efi is compiled for testnet (31841).
- Section 5.4. ultimately isn't very useful at this moment, as we will need to develop specific commands once the contract is deployed on mainnet.
- For various configurations throughout the tutorial, it is necessary to have some resources that must be downloaded to the computer/server. **All of them are available in this [Google Drive](https://drive.google.com/file/d/18cl0n0_r36OcYPZ6_oWXYyDSiOGBXaMK/view?usp=drive_link)** folder.

## 1. Hetzner Server Configuration

To run a [qubic node](https://github.com/qubic/core) you need a computer with the following characteristics:

- Bare Metal Server/Computer with at least 8 Cores (high CPU frequency with AVX2 support). AVX-512 support is recommended; check supported CPUs [here](https://www.epey.co.uk/cpu/e/YTozOntpOjUwOTc7YToxOntpOjA7czo2OiI0Mjg1NzUiO31pOjUwOTk7YToyOntpOjA7czoxOiI4IjtpOjE7czoyOiIzMiI7fWk6NTA4ODthOjY6e2k6MDtzOjY6IjQ1NjE1MCI7aToxO3M6NzoiMjM4Nzg2MSI7aToyO3M6NzoiMTkzOTE5OSI7aTozO3M6NzoiMTUwMjg4MyI7aTo0O3M6NzoiMjA2Nzk5MyI7aTo1O3M6NzoiMjE5OTc1OSI7fX1fYjowOw==/)
- At least 500GB of RAM
- 1Gb/s synchronous internet connection
- A USB Stick or SSD/HD attached to the Computer (via NVMe M.2 or USB)
- UEFI Bios

Additionally, to install the virtual machine where the node will run, you must have a computer with Ubuntu 22.04 operating system, amd64, with 64GB of RAM and a static IP. Therefore, a 'Dedicated Server EX44', which meets all requirements.

![Hetzner Robot Dashboard. Ordering panel for product selection.](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image.png)

Hetzner Robot Dashboard. Ordering panel for product selection.

In my case, the server IP is 95.216.15.174 and my user is root.

![Hetzner Robot Dashboard. Server Panel, detail view in the 'IP' tab of the server IP and MAC.](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image1.png)

Hetzner Robot Dashboard. Server Panel, detail view in the 'IP' tab of the server IP and MAC.

The first time, if you don't have an SSH key, you can generate a new one using the following command in the Ubuntu terminal:

`ssh-keygen -t rsa -b 4096 -C "pc014-3080@anarojo"`

- `t rsa`: Specifies that you will use an RSA type key.
- `b 4096`: Indicates that the key size will be 4096 bits for greater security.
- `C "your_email@example.com"`: Adds a comment (my computer name@user) to identify the key.

I didn't configure any password for this key and saved the file with the name **'hetzner_server_ssh'** in the default suggested path. Then, to view the public key later, execute the following command in the terminal:

`cat /root/.ssh/id_rsa.pub`

The text that appears should be copied into the 'Public Key' text container, accept the data loss conditions, and apply the server installation/configuration.

![Hetzner Robot Dashboard. Server Panel, detail view in the Linux tab, where for the first time you can select the OS configuration, language and public key for the SSH connection. ](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image2.png)

Hetzner Robot Dashboard. Server Panel, detail view in the Linux tab, where for the first time you can select the OS configuration, language and public key for the SSH connection. 

Therefore, to connect to the server from my PC via ssh, you only need to execute the following command:

`ssh -i ~/./hetzner_server_ssh [root@95.216.15.174](mailto:root@95.216.15.174)`

- `./hetzner_server_ssh`: Specifies the path where the 'hetzner_server_ssh' file is located, this part of the command must be replaced with the corresponding relative path.
- `root`: Is the UserName on the server, by default. If creating a new server, use the corresponding user name. This information is received by email once the product is acquired.
- `95.216.15.174`: The server IP, which should also be modified accordingly if creating a new server.

**Note:** When I've had to reset the server because it crashed and I could no longer connect to it, I had to apply '**Execute an automatic hardware reset**' and then reinstall Linux, directly loading the 'Public key' that I had saved.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image3.png)

Normally, after this reset, when trying to connect to the server using: `ssh -i ~/./hetzner_server_ssh [root@95.216.15.174](mailto:root@95.216.15.174)` the following warning appears:

>>>WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!   
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:feJb1fNiGiIRc/0zvJb2r24IIAONMtr1b/560kEP0HU.
Please contact your system administrator.<<<

This can be resolved by clearing the old key from the 'known_hosts' file with the following command:

`ssh-keygen -f "/home/pc014-3080/.ssh/known_hosts" -R "95.216.15.174"`

Finally, before starting with the virtual machine installation, we update the system:

`sudo apt update && sudo apt upgrade -y`

## 2. Virtual Machine Configuration

### 2.1. Installing the Desktop Environment and VNC Server

On the server, once we have updated the package list (sudo apt update), we install the Xfce desktop environment:

`sudo apt install xfce4 xfce4-goodies`

Once this installation is complete, we install TightVNC server:

`sudo apt install tightvncserver`

To verify that the installation was successful and perform its initial configuration, we use the `vncserver` command to set a secure password and create the initial config files. When executing this command, the following dialog will appear in the terminal requesting you to enter and verify a password to access the machine remotely:

```
   Output
   You will require a password to access your desktops.

   Password:
   Verify:

```

The password must be between 6 and 8 characters. Passwords longer than 8 characters will be automatically truncated. In this case, the **password** used is: **123456QV** (Q for Qubic, V for Vottun 🙂)

Once the password is verified, you will be given the option to create a view-only password. This way, users who log in with the view-only password won't be able to control the VNC instance with the mouse or keyboard. In this case, this option was not selected.

Next, the process creates the necessary default configuration files and connection information for the server:

```
   Output
   Would you like to enter a view-only password (y/n)? n
   xauth:  file /root/.Xauthority does not exist

   New 'X' desktop is your_hostname:1

   Creating default startup script /root/.vnc/xstartup
   Starting applications specified in /root/.vnc/xstartup
   Log file is /root/.vnc/srv650887:1.log

```

### 2.2. VNC Server Configuration

Next, we stop the vncserver activity with the command-,

`vncserver -kill :1`

Before modifying the xstartup file, let's make a backup of the original:

`mv ~/.vnc/xstartup ~/.vnc/xstartup.bak`

Now we can create a new xstartup file and open it in the terminal 'editor' using:

`nano ~/.vnc/xstartup`

Actually, the text we need to add to this file consists of the following lines. This configuration will start XFCE as the graphical environment:

#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &

Save and close (with Ctrl+O to save, Enter to save the filename, Ctrl+X to exit). Finally, we launch the VNC server again granting it permissions:

`sudo chmod +x ~/.vnc/xstartup`

`vncserver`

As a result, this dialog will be shown in the terminal:

```
  New 'X' desktop is your_hostname:1

  Starting applications specified in /home/sammy/.vnc/xstartup
  Log file is /home/sammy/.vnc/your_hostname:1.log

```

### 2.3. Installing VNC Viewer on local PC

If you're using Windows, Kavatak recommends using RemoteRipple on the local machine from [RemoteRipple VNC Viewer](https://remoteripple.com/). In my case, since my **local machine** is Ubuntu, I used TigerVNC Viewer as an alternative. To do this, we simply follow these steps:

`sudo apt install tigervnc-standalone-server tigervnc-common -y`

Afterwards, we create a password using the command: `vncpasswd`

To run TigerViewer, simply execute the command: `vncviewer` in the local machine's terminal, and enter the server's IP. After clicking 'Connect' it will request a password, if it was configured during installation. In my case, the password is: **123456QV**.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image4.png)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image5.png)

This will display the following interface. Actually, you can change the screen resolution by specifying the parameters when starting the server, using the command `vncserver -geometry 1920x1080`, but it's not essential.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image6.png)

Right now, all the files needed to configure and launch the node in the virtual machine are located in /root/, as shown in the image. 

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image7.png)

### 2.4. VirtualBox Installation on the Server

Back to the server terminal, we execute the following commands to obtain VirtualBox from Oracle's repository:

`sudo apt update`
`sudo apt install -y wget gnupg software-properties-common`
`wget -q [https://www.virtualbox.org/download/oracle_vbox_2016.asc](https://www.virtualbox.org/download/oracle_vbox_2016.asc) -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] [https://download.virtualbox.org/virtualbox/debian](https://download.virtualbox.org/virtualbox/debian) $(lsb_release -cs) contrib"`
`sudo apt update'`

Next, we install VirtualBox: `sudo apt install -y virtualbox-7.0`

Finally, we download the extensions:

`wget [https://download.virtualbox.org/virtualbox/7.1.4/Oracle_VM_VirtualBox_Extension_Pack-7.1.4.vbox-extpack](https://download.virtualbox.org/virtualbox/7.1.4/Oracle_VM_VirtualBox_Extension_Pack-7.1.4.vbox-extpack)`

And lastly, we install them:
`sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-7.1.4.vbox-extpack`

Note: We can verify that VirtualBox is correctly installed with the following commands: `VBoxManage --version` and `VBoxManage list extpacks`.

Actually, the virtual machine for the Qubic node could be configured with VBoxManage commands, but Kavatak (and the Qubic team in general) prefer that we use the VirtualBox GUI to be able to see the node logs in the console. Therefore, in section 2.6. I review the creation and configuration of the VM, although the steps I followed match those in the Qubic [documentation](https://github.com/XARKUR/Qubic/blob/main/Qubic-Node.md#3virtual-box-configuration).

### 2.5. Package Downloads

For section 2.6. of VM configuration, it's necessary to download the compressed file 'qubic.zip' on the server. And in the subsections of section 3, for qubic node configuration, three other files need to be downloaded: '136.zip', 'Qubic.efi', and 'boradcastComputorTestnet'. Since these are files shared through Google Drive, the `wget` command doesn't work properly and either corrupts the files or doesn't download them correctly.

Therefore, we'll use the `gdown` command from pipx. To do this, we first install it with the command: `sudo apt install pipx` and then: `pipx install gdown`

The terminal will show us the following text:

```jsx
installed package gdown 5.2.0, installed using Python 3.12.3
These apps are now globally available
- gdown
⚠️  Note: '/root/.local/bin' is not on your PATH environment variable. These apps will not be globally
accessible until your PATH is updated. Run pipx ensurepath to automatically add it, or manually modify
your PATH in your shell's config file (i.e. ~/.bashrc).
done! ✨ 🌟 ✨
```

Warning! We must configure the path or the command may not be recognized because it can't be found. For this, simply indicate the source: `nano ~/.bashrc`
`source ~/.bashrc`

With this, you can now download any Google Drive link using the command:

`gdown [https://drive.google.com/](https://drive.google.com/uc?id=1BGlxZWXgySolRv0g_SulUT3-dNK8G_UF)uc?id={id_file}`

### 2.6. VM Configuration for the Qubic Node

To begin this step, it is necessary to download and decompress the ['qubic.zip'](https://drive.google.com/file/d/17yEcYNUvUhiET5oX_DEupQIQo6WwvgBT/view?usp=drive_link) file using gdown. After extracting it in the same root, the file path is: **/root/qubic/qubic.vhd**.

A VM can be created and its configuration assigned directly from the server terminal with the following commands, and when executing the last line, the node (which is loaded with the .vhd file) will start and the VM GUI will open. If you don't want to launch the GUI, you can replace --type gui with --type headless.

```jsx
VBoxManage createvm --name "QUBIC" --ostype "Other_64" --register
VBoxManage modifyvm "QUBIC" --nic1 nat --nictype1 virtio --cableconnected1 on
...
VBoxManage storagectl "QUBIC" --name "SATA Controller" --add sata --controller IntelAhci
VBoxManage storageattach "QUBIC" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium /root/qubic/qubic.vhd
VBoxManage startvm "QUBIC" --type gui
```

Even so, the above methodology is discouraged, and it is better to do it through the interface. Therefore, one way to open the GUI is to directly open the terminal from VirtualBox VMs>Open in Terminal.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image8.png)

And this will open a terminal where using the startup command `virtualbox` launches the GUI.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image9.png)

As shown in the image, to create a new VM click on the 'New' button.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image10.png)

Assign a name and select Linux as operating system, with Oracle Linux (64-bit) version.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image11.png)

We must also select at least 60gb of RAM (as it is at the limit of acceptable/valid) for the maximum of 64gb. And at least 8 CPU nodes.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image12.png)

Select 'Do not add a virtual hard disk'.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image13.png)

And confirm the configuration. Once done, we can select the VM from the left pane, click on the 'Settings' button and review the configuration or modify it.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image14.png)

We must ensure that the 'Enable EFI (special OSes only)' feature is active. To do this, go to Settings>System>Motherboard.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image15.png)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image16.png)

Now, we configure the port of the node, so that it executes the testnet. Therefore, we navigate to Settings>Network>Adapter 1. We deploy 'Advanced' and, firstly, we verify that the 'Adapter_Type' selected is Paravirtualized Network (virtio-net). Secondly, clicking on the 'Port Forwarding' button opens the 'Port Forwarding Rules' window where you can add a new rule, example: Rule 1, with TCP protocol and both host and guest port will be 31844. 

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image17.png)

As for the load file, we navigate to Settings>Storage and in the storage devices, selecting 'Controller: IDE' we can click on the 'Add hard disk' button.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image18.png)

This opens a window of the computer's file directory, in which we can browse for the location of 'qubic.vhd' and add it. 

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image19.png)

Finally, save all the changes with the 'Ok' button and run the machine with the 'Start' button. If everything is correct, you will initially see the following information in the console, and after 5 seconds it will go black. This is correct. We have to make a partition, load a new qubic.efi, the node data and execute a command to activate the 'Qubic network' when the node is launched, to get the network running. We see this in detail in the next section.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image20.png)

## 3. Proper Configuration of the Qubic Node and Qubic Network

### **3.1. VHD File Download and Mounting**

For this configuration step, we need to download 2 required files using `gdown`

1. **Zip file with blockchain data**: `136.zip`.
2. **EFI file required for VM boot**: `Qubic.efi`.

```bash
gdown https://drive.google.com/uc?id=1LtAgPftLzTosQ99O3kFEsEvySFNBzaKy
gdown https://drive.google.com/uc?id=17U1XncFPpo942s9IdPomWE_Uu18jcSRU
```

### 3.2. **VHD File Mounting**

First, we create a folder/directory for mounting, with the command:

`sudo mkdir -p /mnt/qubic`

Next, we connect the VHD file:

`sudo losetup -f --show --partscan /root/qubic/qubic.vhd`
`sudo mount /dev/loop0p1 /mnt/qubic`

At this point, we can verify the qubic contents with the command: `ls /mnt/qubic`
You should see something like this:

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image21.png)

We will replace the Qubic.efi file and all old files with the contents of folder 136. Therefore, first we delete all old files, making sure not to delete the /efi/boot/ folder:

`sudo rm -rf /mnt/qubic/contract* /mnt/qubic/score* /mnt/qubic/spectrum* /mnt/qubic/system*`

We extract 136.zip directly into the /mnt/qubic folder to add the new files:

`sudo unzip /root/136.zip -d /mnt/qubic`

And we copy the Qubic.efi file to its path:

`sudo cp Qubic.efi /mnt/qubic/efi/boot/`

Finally, if we check the folder contents again, it should be visible with the command: `ls -lh /mnt/qubic`

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image22.png)

### 3.3. **VHD Unmounting**

All that remains is to unmount and release the VHD file. For this, we use the commands opposite to mounting:

`sudo umount /mnt/qubic`

`sudo losetup -d /dev/loop0`

### **3.4**. **Node and Qubic Network Startup**

Now, when starting the VM with VirtualBox GUI for visual monitoring, and executing the node. You should see something similar to the image shown below. In this case, 17420000 is the tick information, 136 is the epoch, and if we look carefully, the Indices value shows the '?' symbol. This means that although the node is running, the network is not properly loaded.

Another error is also observed: Latest tick created tick = 4'294'967'293, this is an incorrect format. According to Kavatak, I need to set my server clock to UTC to avoid this issue.

![image.png](Qubic%20Node%20-%20VM%20-%20SetUp%20ENGLISH%201745532b965980378d01f2abc64738e2/69ab3b94-51f9-4928-b4d8-23c772248bcd.png)

This error was resolved by using the following commands from the server terminal. To set the clock to UTC+00 timezone: `sudo timedatectl set-timezone UTC` and to display the set date and time, the command `sudo timedatectl` was used.

I'm not sure if this actually resolved anything, but it was done.

![image.png](Qubic%20Node%20-%20VM%20-%20SetUp%20ENGLISH%201745532b965980378d01f2abc64738e2/image%2023.png)

To resolve the node-network issue, we need to follow these steps.

1. When starting the node, it starts in aux mode, therefore, before starting the network, we need to change the mode to Main. To do this, we press f12 three times to switch to Main/Main. The sequence is: aux/aux-->Main/aux-->aux/Main-->Main/Main-->aux/aux.
2. Once the node is set to Main/Main, we simply need to load the 'broadcastComputorTestnet' file from the terminal. To do this, with the file downloaded, we execute the following commands:

`chmod +x broadcastComputorTestnet`

`./broadcastComputorTestnet 95.216.15.174 136 31844`

With all this, the 'Qubic network' should launch correctly.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image24.png)

To finish, it is enough to close the vm to stop the execution of the node as well.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image25.png)

## 4. Connecting to Qubic Testnet through VM (without having to configure it, just using it)

**Step 1.** We connect to the server using: `ssh -i ~/./hetzner_server_ssh [root@95.216.15.174](mailto:root@95.216.15.174)`

**Step 2.** We start the VNC using the command: `vncserver`

It should indicate port +1 or +2 (New 'X' desktop is Ubuntu-2404-noble-amd64-base:1', in this case it's 1. If it shows 2, or any higher value, it might be because we haven't closed the previous connection (vncserver -kill:1).

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image26.png)

**Step 3.** In a new terminal on the local PC, start TigerViewer with the command: `vncviewer`.

This will open a window to confirm the connection with the server by verifying IP and port (in our case, 1). If everything is correct, click **'Connect'** and in the following 'VNCauthentication' window enter the password: **123456QV** and click **'OK'**.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image27.png)

**Step 4.** The VHD needs to be reconfigured, as it cannot contain the scores, system files (or debug, in case it was generated) when launching the node. Therefore, it is recommended to **perform the steps defined in subsection 5.3**, except for step 3 if a new Qubic.efi is not needed. If the VHD is not cleaned, the node will fail.

**Step 5.** To open the VM from the server interface, we open a new terminal from the **folder > VirtualBox VMs > Open in Te**

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image8.png)

**Step 6**. When a terminal opens, we use the startup command `virtualbox` to launch the VM with GUI.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image9.png)

**Step 7**. As shown in the image, to execute the node and run the Qubic network, we must select **'QUBICbridge'** in the left panel and click the '**Start**' button.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image28.png)

**Step 8**. Once the network is launched, we must press 'F12' 3 times until switching to 'MAIN&MAIN' mode, as it starts in aux&aux by default. And each time we press it, we change modes

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image29.png)

**Step 9.** Next we must load the indices with the command `./broadcastComputorTestnet 95.216.15.174 136 31844`

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image30.png)

And if everything goes well, we should see how the indices parameter shows the text 'Owning 676 indices' and the ticks advance. In the following screenshot, you can see how it went from tick 17420011 to 17420012. (The initialization of this tick was defined in the Qubic.efi compilation at 17420000)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image31.png)

## 5. Qubic.efi Compilation and SC Deployment on Qubic Testnet

### 5.1. Project Parameter Configuration

The project is structured in the following way to simplify the organization and configuration of the necessary components:

The `contracts` folder in the project root contains the `QubicOrderContract.h` file, which replicates the content of the smart contract included in the `core` submodule. This allows for easy location and reading of the contract if you only need to review its content.

The `core` submodule points to the `vottundev/core` fork. Within the `core` folder, the `contracts` directory includes the `EthBridge.h` smart contract. This is the smart contract included in the Qubic.sln core solution.

**Configuration Parameters:**

Before compiling the project, the following parameters must be modified in the following files available in the **core/src** submodule

- **`public_settings.h`:**
    - **Line 94:**
        
        ```cpp
        #define TESTNET_EPOCH_DURATION 1000
        
        ```
        
        Defines the duration of an epoch. On the testnet, the epoch will change after 1,000 ticks. On powerful memos, a maximum of 5,000 ticks can be set.
        
    - Epoch and tick settings:
        
        ```cpp
        #define EPOCH 136
        #define TICK 17420000
        
        ```
        
        The initial epoch and initial tick are defined here. These may need to be adjusted depending on the use case.
        

**`qubic.cpp`:**

- **Configuración del puerto:**
Se debe configurar el puerto en el cual se iniciará el nodo. En este caso, se ejecuta en el puerto 31844.
    
    ```cpp
    #define PORT 31844
    
    ```
    
- **`private_settings.h`:**
    - **Private Keys:**
        
        ```cpp
        static unsigned char computorSeeds[][55 + 1] = { };
        
        ```
        
        Here the private keys (indexes) must be configured. There are a total of 676 keys in the Qubic network. We must copy and paste the 676 lines of the message_publicseed.txt file.
        
    - **Nodes o peers:**
        
        ```cpp
        static const unsigned char knownPublicPeers[][4] = {
        		{95, 216, 15, 174},
            {127, 0, 0, 1},
            {45, 143, 199, 16}
        };
        
        ```
        
        ![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image32.png)
        
        We need to replace the example IP address with the node's IP address (95, 216, 15, 174) and the echo server (45, 143, 199, 16), and add some more. Ideally 4, but I used 3.
        

Finally, we must include the contract in `contract_core/contract_def.h` with the following definitions after `#include "contracts/QVAULT.h"`:

```cpp
#undef CONTRACT_INDEX
#undef CONTRACT_STATE_TYPE
#undef CONTRACT_STATE2_TYPE

#define ETHBRIDGE_CONTRACT_INDEX 11
#define CONTRACT_INDEX ETHBRIDGE_CONTRACT_INDEX
#define CONTRACT_STATE_TYPE ETHBRIDGE
#define CONTRACT_STATE2_TYPE ETHBRIDGE2
#include "contracts/EthBridge.h"
```

This means that our contract corresponds to **index 11**. When deployed, our contract address is the following: **LAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKPTJ**

### 5.2. Project Compilation to Generate Qubic.efi

Now we simply need to make sure we configure the build in **Release** and **x64** and click the **'LocalWindows Debugger'** button (or ctrl+B). If there are no errors, a pop-up will appear like the one shown in the second image, which is just a warning that the application cannot be launched, but the Qubic.efi file has been compiled successfully.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image33.png)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image34.png)

So, let's look for this file in our project folder: core/x64/Release/Qubic.efi

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image35.png)

In my case, I replace this Qubic.efi in the Google Drive folder so I know I can download it on any machine through its link.

### 5.3. Replace Qubic.efi on the Server

After downloading the Qubic.efi file from Google Drive to the server (I use gdown as mentioned in point 2.5), we replace Qubic.efi in the vhd.

To do this, I make sure the node is turned off to remount the vhd. These are basically the steps from points 3.2 and 3.3. Although this process can be seen in the screenshots, I list the commands below:

1. Download the Qubic.efi file: `gdown [https://drive.google.com/](https://drive.google.com/uc?id=1BGlxZWXgySolRv0g_SulUT3-dNK8G_UF)uc?id={id_file}`
2. Connect the vhd: `sudo losetup -f --show --partscan /root/qubic/qubic.vhd`
`sudo mount /dev/loop0p1 /mnt/qubic`
3. Copy/replace the Qubic.efi file in /efi/boot of the vhd: `sudo cp Qubic.efi /mnt/qubic/efi/boot/`
4. I usually check the contents to see which files it contains using the command: `ls -lh /mnt/qubic`
5. Note: before launching the node again, we must delete the score*, system* files, and if a debug.log was generated, we delete that too: `sudo rm -rf /mnt/qubic/score*  /mnt/qubic/system*`
6. We finish and release the vhd: `sudo umount /mnt/qubicsudo losetup -d /dev/loop0`

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image36.png)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image37.png)

5.4. Using Qubic-cli as an intermediary to read network events

Quick explanation to clarify. qubic-cli is basically a command-line tool that simplifies direct access to the Qubic node. What it does is translate commands into specific requests that the node understands. Therefore, it acts as a bridge to interact with the node

- It is a command-line tool.
- Simplifies direct access to the Qubic node.
- Translates commands into specific requests that the node understands.
- Acts as a **bridge** to interact with the node without needing to directly implement the node's communication protocol.

**-include qubic-cli compilation here-**

[https://github.com/qubic/qubic-cli/](https://github.com/qubic/qubic-cli/)

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image38.png)

## 6. Interaction with the SC

Finally, to interact with the SC, we will use the [**qubic-http](https://github.com/qubic/qubic-http)** service, which acts as a bridge to the Qubic network. To implement it, they include a docker-compose.yml file, in which you can configure the environment variables of an active node, either a "trusted" node from the Qubic network or a locally operated node (our case).

Through this service, essential functions should be possible such as balance verification, transaction transmission, tick information, and block information.

**Step 1.** Create an empty file called `docker-compose.yml` in my current directory: `touch docker.compose.yml`

**Step 2.** Open it: `nano docker-compose.yml` and copy the contents of the [docker-compose.yml](https://docs.google.com/document/d/1CV9Q8bGoaadBePHuhxdXONa333Jr1A6l_EcQBR-Wxrs/edit?usp=drive_link) file from Google Drive. In the drive file, the environment values are already set, which are our node's IP and port:

**QUBIC_NODES_QUBIC_PEER_LIST: "95.216.15.174"**

**QUBIC_NODES_QUBIC_PEER_PORT: "31844"**

Note: To exit the terminal editor, press shift+W, enter, shift+X.

**Step 3.** Now we should launch the node and make sure it's running correctly.

**Step 4.** Then we start it: `docker compose up -d`

From this point, we can interact with either the node itself or directly with the smart contract. Example of interaction with the node, reading the current tick: `curl [http://127.0.0.1:80/v1/tick-info](http://127.0.0.1/v1/tick-info)`

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image39.png)

**Important**: Any write operation that requires calling an sc function involves creating a transaction. Any read operation requires calling the query endpoint in qubic-http.

Below I provide an example of reading a public sc function. We will read '**getOrder**', which takes a uint64 as an input parameter.

```jsx
curl -X 'POST' \
'[http://127.0.0.1:80/v1/querySmartContract](http://127.0.0.1/v1/querySmartContract)' \
-H 'accept: application/json' \
-H 'Content-Type: application/json' \
-d '{
"contractIndex": 11,
"inputType": 2,
"inputSize": 8,
"requestData": "AAAAAAAAAAA="
}'
```

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image40.png)

Example to read 'getAdminID' which has as input parameter a uint8 (any value is valid, I pass 0.

```jsx
curl -X 'POST' \
'[http://127.0.0.1:80/v1/querySmartContract](http://127.0.0.1/v1/querySmartContract)' \
-H 'accept: application/json' \
-H 'Content-Type: application/json' \
-d '{
"contractIndex": 11,
"inputType": 12,
"inputSize": 1,
"requestData": "AA=="
}'
```

By default, it is initialized to address 0. A default address must be set at initialization.

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image41.png)

To execute transactions, you can use the accounts that are in [message_publicID.txt](https://drive.google.com/file/d/1i55D5u7rWNHBpxBcKAuIkNVpXV7FnrzW/view?usp=drive_link) with their private key for signing, which can be found in the file [message_privatekey](https://drive.google.com/file/d/1AkLR1_kwRe3O2UJ84wgJBeJXmJyi1qLD/view?usp=sharing).

In this example, I passed the first ID from 'message_publicID' as the ID from which the transaction will be signed (and in line 62 of the example, the corresponding private key must be passed, which is the first one from the corresponding 'message_privatekey'). Destination ID is always the contract address (ours is index 11) and adminID would be the new address that we want to set as admin (I took the second publicID from the txt).

![image.png](https://github.com/vottundev/SetUp-Qubic-Node/blob/main/setupimg/image42.png)

Using qubic-cli you can check if the transaction has been included in the block (tick), in this case, the transaction has been found.
