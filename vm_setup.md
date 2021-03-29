# Setting Up a VM Instance on Google Compute Engine

## Part 1: Creating the VM Instance

1. Click create instance.

2. Set the following fields:
   - Region: `us-central1`
   - Zone: `us-central1-a`
   - Machine Configuration:
     - Series: N1
     - Custom Machine Type: 4 vCPU, 32 GB RAM
     - No GPU (we'll add that later)
   - Display Device (under CPU platform and GPU): Check the option Turn on display device
   - Boot Disk:
     - OS: Ubuntu 18.04 LTS
     - Disk Type: Standard persistent disk
     - Size: 350 GB
   - Firewall: check both options
   - Verify that the monthly cost is $170.16.
3. Click Create.

## Part 2: Set Up Firewall for Display Device

1. Under Compute Engine/VM instances/Related Actions, click Setup Firewall Rules

2. Click "Create firewall rule"

3. Set the following fields:
   - Name: `vnc-viewer-display`
   - Logs: Off
   - Targets: All instances in the network
   - Source IP ranges: `0.0.0.0/0`
   - Protocols and ports: `tcp:5901`

## Part 3: Initial Setup of the VM Instance

1. Note the External IP address from the Google Cloud console for later use.

2. Connect to the VM instance via SSH.

3. Run the following to install the VNC server:

   ```bash
   sudo apt-get update
   sudo apt-get install tightvncserver
   ```

4. Run the following to install Ubuntu desktop:

   ```bash
   sudo apt-get install ubuntu-desktop
   ```

5. Run the following to install Gnome components:

   ```bash
   sudo apt-get -y install gnome-shell autocutsel gnome-core gnome-panel gnome-themes-standard
   ```

6. Run and configure the tightvncserver using the following. When prompted, create a password.

   ```bash
   vncserver
   ```

7. This automatically creates a session. Kill the session by running:

   ```bash
   vncserver -kill :1
   ```

8. Edit the file ~/.vnc/xstartup:

   ```bash
   vim ~/.vnc/xstartup
   ```

9. Remove all contents and paste the following, then save:

   ```bash
   #!/bin/sh
   autocutsel -fork
   xrdb $HOME/.Xresources
   xsetroot -solid grey
   export XKL_XMODMAP_DISABLE=1
   export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unity"
   export XDG_MENU_PREFIX="gnome-flashback-"
   unset DBUS_SESSION_BUS_ADDRESS
   gnome-session --session=gnome-flashback-metacity --disable-acceleration-check --debug &amp;
   ```

10. Start a new session:

    ```bash
    vncserver -geometry 1440x900
    ```

11. In VNC Viewer, type in `<external_ip>:5901` (e.g. `35.238.123.226:5901`)

12. Finish setting up Ubuntu.

## Part 4: Setup Git and Clone Repository

1. Run the following to install git:

   ```bash
   sudo apt-get install git-all
   ```

2. Create an SSH key:

   - Follow the instructions here: <https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent>
   - Make sure you select the **Linux** version of instructions! (it'll default to Mac)
   - Save the key in the default location
   - If you get an error using `xclip`, run the following and copy the contents:

     ```bash
     less ~/.ssh/id_ed25519.pub
     ```

3. Clone the repo inside Documents:

   ```bash
   cd Documents
   git clone git@github.com:Brian0615/CSC413FinalProject.git
   ```

## Part 5: Installing Conda

The instructions below are for Miniconda, a minimal installer for conda.

1. Change to `/tmp` directory:

   ```bash
   cd /tmp
   ```

2. Download installer for Miniconda:

   ```bash
   curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   ```

3. Install Miniconda under `/home/<username>/miniconda3`. Enter `yes` for any prompts that show up.

   ```bash
   bash Miniconda3-latest-Linux-x86_64.sh
   ```

4. Close and restart the shell before continuing. (i.e. connect via SSH again)

5. Update conda.

   ```bash
   conda update conda
   ```

6. Create a Python 3.6 environment then activate:

   ```bash
   conda create --name py36 python=3.6
   conda activate py36
   ```

7. Add channels for installing packages later:

   ```bash
   conda config --add channels conda-forge
   ```
