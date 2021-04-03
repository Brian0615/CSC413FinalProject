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
   export XDG_CURRENT_DESKTOP="GNOME-Flashback:Unvncity"
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

## Part 6: Installing Required Packages

> Note: Ensure the packages are installed in the `py36` environment!

### Packages for Hierarchical Multi-Scale Attention

1. Install Pytorch 1.4.0 with CUDA 10.1:

   ```bash
   conda install pytorch==1.4.0 torchvision==0.5.0 cudatoolkit=10.1 -c pytorch
   ```

2. Install the packages for HMSA:

   ```bash
   conda install numpy scikit-learn h5py jupyter scikit-image pillow piexif cffi tqdm dominate opencv nose ninja

   pip install runx==0.0.6

   sudo apt-get install libgtk2.0-dev -y

   sudo rm -rf /var/lib/apt/lists/*
   ```

3. Install Apex (for HMSA). This part requires GPU!

   ```bash
   cd /home/

   sudo git clone https://github.com/NVIDIA/apex.git apex

   cd apex

   python setup.py install --cuda_ext --cpp_ext
   ```

### Packages for ResNeSt

1. We need to install the PyTorch-Encoding pacakge.

   ```bash
   cd ~/Documents/CSC413FinalProject/PyTorch-Encoding/

   python setup.py install
   ```

## Part 7: Downloading Datasets

The HMSA paper uses the Mapillary Vistas and Cityscapes datasets.

### Mapillary Vistas Dataset

1. Create a folder for downloading the datasets.

   ```bash
   cd ~/Documents/

   mkdir data

   cd data

   mkdir mapillary

   cd mapillary
   ```

2. Request access for the research edition dataset [here](https://www.mapillary.com/dataset/vistas/).

3. There are two parts to the dataset file, which will be sent via email. Download, assemble, and unzip them by running the following. This should take ~15-20 mins.

   ```bash
   wget mapillary_vistas_v2_part.z01 "<insert part 1 link here>"

   wget mapillary_vistas_v2_part.zip "<insert part 2 link here>"

   zip -s 0 mapillary_vistas_v2_part.zip --out mapillary-vistas-dataset_public_v2.0.zip

   unzip mapillary-vistas-dataset_public_v2.0.zip
   ```

4. Remove unneeded files.

   ```bash
   rm mapillary_vistas_v2_part.z01

   rm mapillary_vistas_v2_part.zip
   ```

5. The folder structure should look like this:

   ```bash
   mapillary
   ├── README
   ├── config_v1.2.json
   ├── config_v2.0.json
   ├── demo.py
   ├── testing
   │   └── images
   ├── training
   │   ├── images
   │   ├── v1.2
   │   └── v2.0
   └── validation
      ├── images
      ├── v1.2
      └── v2.0
   ```

6. Update `config.py` inside the HMSA code.

   ```bash
   __C.DATASET.MAPILLARY_DIR = '~/Documents/data/mapillary'
   ```

### Cityscapes Dataset

1. Create a folder to store the data.

   ```bash
   cd ~/Documents/data

   mkdir cityscapes

   cd cityscapes
   ```

2. Create an account [here](https://www.cityscapes-dataset.com/).

3. Log in to the Cityscapes using your username and password.

   ```bash
   wget --keep-session-cookies --save-cookies=cookies.txt --post-data 'username=USERNAME&password=PASSWORD&submit=Login' https://www.cityscapes-dataset.com/login/
   ```

4. Download the datasets we need. We need the following files:

   ```bash
   id    filename                      size     est_time    md5sum
   1     gtFine_trainvaltest.zip       241MB    < 1 min     4237c19de34c8a376e9ba46b495d6f66
   2     gtCoarse.zip                  1.3GB    1 min       1c7b95c84b1d36cc59a9194d8e5b989f
   3     leftImg8bit_trainvaltest.zip  11GB     10-15 min   0a6e97e94b616a514066c9e2adb0c97f
   4     leftImg8bit_trainextra.zip    44GB     35 min      9167a331a158ce3e8989e166c95d56d4
   ```

   The general download command is below.

   ```bash
   wget -c -t 0 --load-cookies cookies.txt -O <filename> https://www.cityscapes-dataset.com/file-handling/?packageID=<package_id>
   ```

   These are the specific commands for the 4 files:

   ```bash
   wget -c t 0 --load-cookies cookies.txt -O gtFine_trainvaltest.zip https://www.cityscapes-dataset.com/file-handling/?packageID=1

   wget -c t 0 --load-cookies cookies.txt -O gtCoarse.zip https://www.cityscapes-dataset.com/file-handling/?packageID=2

   wget -c t 0 --load-cookies cookies.txt -O leftImg8bit_trainvaltest.zip https://www.cityscapes-dataset.com/file-handling/?packageID=3

   wget -c -t 0 --load-cookies cookies.txt -O leftImg8bit_trainextra.zip https://www.cityscapes-dataset.com/file-handling/?packageID=4
   ```

5. Unzip the files we downloading using:

   ```bash
   unzip <filename>.zip
   ```
