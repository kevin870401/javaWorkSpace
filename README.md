# Creating A Java Workspace
## 1. Install Pre-requisites Tools
You only need to do these steps once for the first time on your **Windows** machine.

*NOTE:* Whenever you see a vagrant command (e.g. `vagrant up`) below, these are always run on your host **Windows** machine from **PowerShell** while in the directory of your project's repo where the `Vagrantfile` is.

*NOTE:* Testers will need **local admin** on their machines to use VirtualBox + Vagrant after the installation, it will not work without it.

1. Install the following tools
  - Git `1.9.5` [download](http://repositdev1.ms.otpp.net/binary/github/git/1.9.5/Git-1.9.5-preview20141217.exe)
  - VirtualBox `4.3.20` [download](http://repositdev1.ms.otpp.net/binary/oracle/virtualbox/4.3.20/VirtualBox-4.3.20-96997-Win.exe)
  - Vagrant `1.7.2` [download](http://repositdev1.ms.otpp.net/binary/hashicorp/vagrant/1.7.2/vagrant_1.7.2.msi)
1. Add `git` to your path if it isnt' already there
    - Add the Git binaries to your path (`C:\Program Files (x86)\Git\bin`)
1. (:exclamation:) Change the default VM dir to a drive with at least `20GB+` of space: `VirtualBox > File > Preferences > General > Default Machine Folder` (do **NOT** use your `H:` drive which is the default value!)
  - Your primary drive (i.e. `C:\`) is likely an SSD, so use that for your VMs for better performance if you have the space
  - **Restart** your Windows host computer for this change to take effect

## 2. WARNING: Patch Nights...
(:exclamation:) Be sure to shutdown your VMs with `vagrant halt` on patch nights so that if your Windows box is restarted abruptly, it doesn't corrupt your VM. If your VMs corrupted, you will lose any code changes you have not pushed up to Github

## 3. Create A New Workspace VM
1. Customize what you want to install by copying an existing runlist from `<YOUR_PROJECT>/workspace` to `<YOUR_PROJECT>/workspace/run_list_<USERNAME>.json` and customize the list (do this on github and commit it)
1. Open **Powershell** with **Run As Administrator** [[1]](#1) on your **Windows** host and run this
  - `cd C:\git; git clone http://src/<ORG>/<PROJECT>.git; cd <PROJECT>; vagrant up`
    - `<ORG>` is your project's Github org (e.g. `tim`)
    - `<PROJECT>` is your project's Github project name (e.g. `com.otpp.tim.timwebapp`)
1. Wait about :clock1: **5-20 minutes** for the provisioning to complete (don't use ur VM during this time)
1. Apply WebSense fixes for the browsers (if any)
    1. Launch the installed browsers **inside** your VM, then close them (i.e. Firefox, Chrome)
    1. In your PowerShell on your **windows** host, run `vagrant provision; vagrant reload` to install the WebSense fixes for the browsers and restart the VM

# Updating An Existing Workspace VM
1. Save all your work and make sure you push any commits (in case something screws up and your VM gets messed up)
1. Close all the applications in your VM including IntelliJ
1. From **PowerShell** on your **Windows** host in your project repo's dir, run this while your VM is running
    - `git pull; vagrant provision; vagrant reload`
1. Wait :clock1: **a few minutes** for the provisioning to complete

# Vagrant
## Usage
- `vagrant up` start your VM (provisions on first run)
- `vagrant halt` shuts down your VM (always use this to turn off your VM) (NOTE: Always shutdown your VM during patch nights to avoid VM corruption)
- `vagrant status` shows the status of the VM
- `vagrant reload` restarts a running VM
- `vagrant destroy` deletes your VM (NOTE: Remember to **push** any local commits and backup any files you need to somewhere outside your VM!)
- `vagrant provision` while your VM is up will re-run the idempotent Chef recipes to update/fix any installs you have according to what the recipes do
- `vagrant global-status --prune` to clean up old vagrant vm instances that have been destroyed
- `vagrant suspend` suspends your to disk (:exclamation: sometimes suspend looses the link to ur VM, not reliable)
- `vagrant resume` resumes a suspended VM

## VM Maintenance
- VM User
    - Username: `vagrant`
    - Password: `vagrant`
- Whenever the system update dialog box appears in the VM, close all your applications inside the VM but leave the VM running, then run `vagrant provision` to have Chef do the updates for you

## How to Create A Vagrant Base Box
Steps based on this [article](http://www.skoblenick.com/vagrant/creating-a-custom-box-from-scratch/#tab-t10)

Try to refresh our Vagrant box image with the latest updates on every major or minor release of Xubuntu which seems to happen every few months, that way users don't have to do more than a few months of updates each time the provision with a stale Vagrant box image.

1. Create a new VM in VirtualBox
  - Name `vagrant-xubuntu64-14.04.2`
  - Type `Linux`
  - Version `Ubuntu (64 bit)`
  - Memory `512 RAM`
  - `Create a virtual hard drive now`
  - Disk file type `VMDK`
  - `Fixed disk size`
  - Disk space `20 GB`
1. VM settings
  - General > Advanced >
    - Shared Clipboard `Bidirectional`
    - Drag'n'Drop `Bidirectional`
  - System > Motherboard
    - Chipset `ICH9`
    - Extended Features `Enable I/O APIC`
    - Hardwared Clock in UTC Time
  - System > Processor
    - `Enable PAE/NX`
  - System > Acceleration
    - `Enable VT-x/AMD-v`
    - `Enable Nested Paging`
  - Display > Video >
    - Video Memory `128 MB`
    - Disable 2D and 3D acceleration
  - Storage > Controller
    - Type `AHCI`
  - Network > Adapter 1
    - `NAT`
    - Adapter Type `Paravirtualized Networkd (virtio-net)`
  - Network > Advanced > Port Forwarding > Insert new rule:
    - Name `SSH`
    - Protocol `TCP`
    - Host Port `2222`
    - Guest Port `22`
1. Start the VM and install Xubuntu using the downloaded ISO as the startup disk
  - Timezone `Toronto`
  - User details
    - Name `vagrant`
    - Computer name `vagrant-xubuntu64-14.04.2`
    - Username `vagrant`
    - Password `vagrant`
    - Select `Log in automatically`
1. Restart VM
1. Install VirtualBox Guest Additions
  - Run Xubuntu Software Updater
  - `sudo apt-get install -y gcc build-essential linux-headers-$(uname -r) dkms`
  - VirtualBox > Devices > Insert Guest Additions CD image...
  - Open terminal on mounted image > `sudo ./VBoxLinuxAdditions.run`
  - Unmount Guest Additions ISO
    - Right-click CD mount shortcut on desktop > Eject volume
  - Restart
1. Setup SSH for Vagrant
  - `sudo visudo` and add these lines
    - `Defaults:vagrant !requiretty`
    - `Defaults env_keep = "SSH_AUTH_SOCK"`
    - `vagrant ALL=NOPASSWD: ALL`
  - `sudo mkdir -p /home/vagrant/.ssh`
  - `sudo chmod 0700 /home/vagrant/.ssh`
  - `sudo wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys`
  - `sudo chmod 0600 /home/vagrant/.ssh/authorized_keys`
  - `sudo chown -R vagrant /home/vagrant/.ssh`
  - `sudo apt-get install -y openssh-server`
  - `sudo vi /etc/ssh/sshd_config`
    - Uncomment `AuthorizedKeysFile %h/.ssh/authorized_keys`
    - Uncomment and modify `PasswordAuthentication no`
  - `sudo service ssh restart`
1. Update the system
  - Curl bash our WebSense cookbook so you can do a system update
  - Run `sudo ap-get update --fix-missing` to fix outdated apt repo links in your vagrant image
  - Do a system update/upgrade
1. Package VM as a Vagrant box
  - Shutdown VM
  - Open terminal in a dir (e.g. `~/vagrant-repo/`) for your vagrant boxes and run
    - `vagrant package --base vagrant-xubuntu64-14.04.2-tuned --output vagrant-xubuntu64-14.04.2-tuned.box`
1. [Upload](http://repositdev1.ms.otpp.net:9000/upload) the new Vagrant base box to our internal binary store to share it
  1. Your box will now be available here `http://repositdev1.ms.otpp.net/binary/canonical/xubuntu/14.04/vagrant-xubuntu64-14.04.2-tuned.box`
1. You now have a box you can add to Vagrant's box list
  - `vagrant box add --name vagrant-xubuntu64-14.04.2 http://repositdev1.ms.otpp.net/binary/canonical/xubuntu/14.04/vagrant-xubuntu64-14.04.2-tuned.box`
  - **OR**
  - Specify in your Vagrantfile the location of the box and box name
    - `config.vm.box = "vagrant-xubuntu64-14.04.2"`
    - `config.vm.box_url = "http://repositdev1.ms.otpp.net/binary/canonical/xubuntu/14.04/vagrant-xubuntu64-14.04.2-tuned.box"`

# Workspace Vagrantfile
- Add a Vagrant file to your project's repo based on this the sample `Vagrantfile` in this repo
- Update the config section at the top of the `Vagrantfile` for your project's workspace cookbook and project repo
- Add run lists for everyone on project by putting them in your project's repo under `workspace` following the naming convention `workspace/run_list_<USERNAME>.json`
    - e.g. `YOUR_PROJECT/workspace/run_list_thanesn.json`
- Add `.vagrant/**` to your `.gitignore`
- `vagrant up`

# Appendix
1. <a name="2"/> There seems to be a bug either in VirtualBox or Vagrant caused by your home dir starting off as the `H:` drive, which keeps moving your default VM folder back to the `H:` drive. We can't store full VMs on our `H:` drives because it is shared and has limited space, and you will get a visit from IT support asking you to move it off there. So if you run PowerShell as admin, it seems to start the shell in your `C:` drive and not revert back to the `H:` drive.

# TODO
- [ ] make virtualbox guest additions script idempotent by detecting if the shared folder /vagrant is there or not to see if guest additions need to be reinstalled
- [ ] create a script that is run automatically and regularly by bamboo that downloads the currently uploaded base vagrant box, spins up a vm instance using it, ssh's into the box and does a system update, security patches, full ubuntu upgrade, etc. then re-packages the vagrant box into an updated base box, then re-uploads it to binary store
  - make backup of existing box, then re-upload new one with same name so people don't have to update their Vagrantfiles?
  - insures our base boxes don't get stale overtime and not lead to very long update times for new workspaces
- [ ] have vagrant file pass in username and user's otpp email from windows envirnoment variables into workspace cookbook and have a gitconfig set uesr's git configs so that the ydont' have to do this step manually
  - When you Git push for the first time from intellij
    - Name: `FULL NAME`
    - Email: `FULL_NAME@otpp.com`
    - `Set and Commit`
    - For `Setup Master Password`, leave it **empty** and just hit `OK`
