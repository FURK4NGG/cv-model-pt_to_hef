## 👀 cv-model-pt_to_hef Overview   2/3  
<h1 align="center">Convert Computer Vision model .pt to .hef for Raspberry Pi 5 Hailo AI HAT</h1>  

## 🔎 Preparation
<details>
<summary>1. Prepare docker enviroment </summary>

Follow these steps:
<details>
<summary>Linux</summary>

<details>
<summary>Arch Setup</summary>

```bash
# Update the system
sudo pacman -Syu

# Install Docker, Buildx and Compose
sudo pacman -S docker docker-buildx docker-compose

# Start Docker and enable it at boot
sudo systemctl enable --now docker

# Check Docker
docker --version

# Test Docker
sudo docker run hello-world


# ----------------------------------------------------------
# Move Docker data to:
# /home/$USER/docker_data
# ----------------------------------------------------------

# Stop Docker
sudo systemctl stop docker.socket
sudo systemctl stop docker.service

# Create the new Docker data directory
sudo mkdir -p "/home/$USER/docker_data"

# Copy existing Docker data
sudo rsync -aP \
    /var/lib/docker/ \
    "/home/$USER/docker_data/"

# Configure Docker
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "data-root": "/home/$USER/docker_data"
}
EOF

# Start Docker
sudo systemctl enable --now docker

# Verify Docker Root Dir
docker info | grep "Docker Root Dir"


# Expected:
# Docker Root Dir: /home/YOUR_USERNAME/docker_data


# ----------------------------------------------------------
# Optional: allow Docker without sudo
# ----------------------------------------------------------

sudo usermod -aG docker "$USER"

# Log out and log back in before using Docker without sudo.

# Test:
docker run hello-world
```
</details>

<details>
<summary>Ubuntu Setup</summary>
      
```bash
# Update the system
sudo apt update
sudo apt upgrade -y

# Install required packages
sudo apt install -y ca-certificates curl

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL \
    https://download.docker.com/linux/ubuntu/gpg \
    -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker's official repository
sudo tee /etc/apt/sources.list.d/docker.sources > /dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

# Update package lists
sudo apt update

# Install Docker Engine and related components
sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

# Start Docker and enable it at boot
sudo systemctl enable --now docker

# Check Docker
docker --version

# Test Docker
sudo docker run hello-world


# ----------------------------------------------------------
# Move Docker data to:
# /home/$USER/docker_data
# ----------------------------------------------------------

# Stop Docker
sudo systemctl stop docker.socket
sudo systemctl stop docker.service

# Create the new Docker data directory
sudo mkdir -p "/home/$USER/docker_data"

# Copy existing Docker data
sudo rsync -aP \
    /var/lib/docker/ \
    "/home/$USER/docker_data/"

# Configure Docker
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "data-root": "/home/$USER/docker_data"
}
EOF

# Start Docker
sudo systemctl enable --now docker

# Verify Docker Root Dir
docker info | grep "Docker Root Dir"


# Expected:
# Docker Root Dir: /home/YOUR_USERNAME/docker_data
```
</details>

<details>
<summary>Fedora Setup</summary>
   
```bash
# Update the system
sudo dnf upgrade -y

# Add Docker's official repository
sudo dnf config-manager addrepo \
    --from-repofile \
    https://download.docker.com/linux/fedora/docker-ce.repo

# Install Docker Engine and related components
sudo dnf install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin

# Start Docker and enable it at boot
sudo systemctl enable --now docker

# Check Docker
docker --version

# Test Docker
sudo docker run hello-world


# ----------------------------------------------------------
# Move Docker data to:
# /home/$USER/docker_data
# ----------------------------------------------------------

# Stop Docker
sudo systemctl stop docker.socket
sudo systemctl stop docker.service

# Create the new Docker data directory
sudo mkdir -p "/home/$USER/docker_data"

# Copy existing Docker data
sudo rsync -aP \
    /var/lib/docker/ \
    "/home/$USER/docker_data/"

# Configure Docker
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "data-root": "/home/$USER/docker_data"
}
EOF

# Start Docker
sudo systemctl enable --now docker

# Verify Docker Root Dir
docker info | grep "Docker Root Dir"


# Expected:
# Docker Root Dir: /home/YOUR_USERNAME/docker_data


# ----------------------------------------------------------
# Optional: allow Docker without sudo
# ----------------------------------------------------------

sudo usermod -aG docker "$USER"

# Log out and log back in.

# Test:
docker run hello-world
```   
</details>

```bash
============================================================
             FINAL VERIFICATION — LINUX
============================================================

# Check Docker service
sudo systemctl status docker

# Check Docker version
docker --version

# Check Docker Root Dir
docker info | grep "Docker Root Dir"

# Check Docker storage
docker info

# Test container
docker run hello-world
```
</details>

<details>
<summary>Windows Setup</summary>

```bash
# ----------------------------------------------------------
# Run these commands in PowerShell as Administrator
# ----------------------------------------------------------

# Install WSL 2
wsl --install

# Update WSL
wsl --update

# Verify WSL
wsl --version

# Set WSL 2 as the default
wsl --set-default-version 2


# ----------------------------------------------------------
# Install Docker Desktop
# ----------------------------------------------------------

# Download and install Docker Desktop for Windows.
#
# During installation select:
#
#     Use WSL 2 instead of Hyper-V
#
# WSL 2 is the recommended backend for the normal
# Docker Desktop Linux-container workflow.


# ----------------------------------------------------------
# Optional command-line installation
# ----------------------------------------------------------

# If the Docker Desktop installer is located in the
# current directory:

Start-Process `
    ".\Docker Desktop Installer.exe" `
    -Wait `
    -ArgumentList "install", "--backend=wsl-2"


# ----------------------------------------------------------
# Set the Docker Desktop WSL data location
# ----------------------------------------------------------

# Example:
# Move Docker Desktop's WSL data to:
#
#     D:\DockerData
#
# The Docker Desktop installer supports:
#
#     --wsl-default-data-root=<path>

Start-Process `
    ".\Docker Desktop Installer.exe" `
    -Wait `
    -ArgumentList `
        "install",
        "--backend=wsl-2",
        "--wsl-default-data-root=D:\DockerData"


# ----------------------------------------------------------
# If Docker Desktop is already installed
# ----------------------------------------------------------
#
# Open:
#
# Docker Desktop
#     -> Settings
#     -> Resources
#     -> Advanced
#
# Change the Docker/WSL disk image location to:
#
#     D:\DockerData
#
# ----------------------------------------------------------


# Start Docker Desktop manually from Windows if necessary.


# ----------------------------------------------------------
# Test Docker from PowerShell
# ----------------------------------------------------------

docker --version

docker run hello-world


# ----------------------------------------------------------
# Test Docker from WSL
# ----------------------------------------------------------

wsl

docker --version

docker run hello-world


# Docker commands should now work from the integrated
# WSL distribution as well.


============================================================
              FINAL VERIFICATION — WINDOWS
============================================================

# PowerShell:

docker --version

docker info

docker run hello-world

# WSL:

wsl

docker info

docker run hello-world
```
</details>
</details>

2. `Go to the`[`HAILO AI DEVELOPER ZONE`](https://hailo.ai/developer-zone/software-downloads/)`adress and download .zip file with this configration: Software Package[AI Software Suite], Software Sub-Package[AI Software Suite], Architecture[x86], OS[Linux], Python Version[3.8]`
3. `In model training, place 60%–80% of the photos you use into the train/images folder. The photos should be raw, unlabelled images — there should be no labeling process applied to them. You don’t need labels or a classes.txt file; only the images folder is required.`
<details>
<summary>3. Get your .pt model and convert it to .onnx</summary>

1. Run this .py code at the same directory with your .pt model:
```bash
!pip install ultralytics
from ultralytics import YOLO

model = YOLO("model.pt")
model.export(format="onnx")
```
</details>


## 📦 Setup  

1. `unzip hailo8_ai_sw_suite_2025-10_docker.zip -d /home/$USER/docker_hailo`
2. `cd /home/$USER/docker_hailo/`
3. `Edit your .sh document and delete these lines:`
   > -v /etc/timezone:/etc/timezone:ro \
   > -v /etc/localtime:/etc/localtime:ro`
4. `./hailo_ai_sw_suite_docker_run.sh --override`
   > *If you want to continue with your already configured project:*./hailo_ai_sw_suite_docker_run.sh --resume 
5. /home/$USER/docker_hailo/shared_with_docker/train/images/(2.5% – 5% of your photos)  
   /home/$USER/docker_hailo/shared_with_docker/models/model.onnx  


## 🎉 Run 
In "(hailo_virtualenv) hailo@user:" terminal:  
1. `git clone https://github.com/LukeDitria/RasPi_YOLO.git`
2.  `cd RasPi_YOLO/` Then you shold be in '/local/workspace/RasPi_YOLO/' directory
3.  `python hailo_calibration_data.py     --data_dir /local/shared_with_docker/train/images/     --target_dir /local/shared_with_docker/doc`
4. `hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib/ --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11l.yaml --classes 7 --hw-arch hailo8`

<details>
<summary>If the process displays <code>Killed</code> after running the command, follow these steps:</summary>

### Confirm That the Process Was Terminated Due to Insufficient Memory

Run the following command outside the container, in the normal host terminal:

```bash
sudo journalctl -k -b | grep -Ei "out of memory|oom|killed process"
```

On Arch Linux, you can also use:

```bash
sudo dmesg -T | grep -Ei "out of memory|oom|killed process"
```

If you receive output similar to the following, the diagnosis is confirmed:

> Out of memory: Killed process ...
> Killed process ... python

Check the available RAM and swap:

```bash
free -h
```

```bash
swapon --show
```

Example output:

> Mem:   27Gi
> Swap:  32Gi

### Check the Docker Memory Limit

Find the container name:

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

Then use the actual container name:

```bash
docker inspect CONTAINER_NAME --format 'Memory={{.HostConfig.Memory}} MemorySwap={{.HostConfig.MemorySwap}}'
```

If the result is as follows, no custom Docker memory limit is configured:

> Memory=0 MemorySwap=0

### 1. Find the Original YOLO11l ALLS File

```bash
find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls"
```

It should return a result similar to:

> /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls/generic/yolov11l.alls

### 2. Store the Discovered Path in a Variable

```bash
DEFAULT_ALLS="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls" -print -quit)"
```

Verify it:

```bash
echo "$DEFAULT_ALLS"
```

The output must not be empty and should end with:

> yolov11l.alls

This variable only stores the file path temporarily. It does not modify any file.

### 3. Display the Original Finetune Settings

```bash
sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' "$DEFAULT_ALLS"
```

The end of the output should contain something similar to:

> epochs=4, batch_size=4)

### 4. Copy the Original ALLS File to the Shared Directory

```bash
cp -f "$DEFAULT_ALLS" /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

Verify the copied file:

```bash
ls -lh /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

This operation does not modify the original file inside Hailo Model Zoo.

### 5. Reduce the QAT Batch Size from 4 to 1

```bash
sed -i 's/batch_size=4)/batch_size=1)/' /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

This command changes only the following value:

> epochs=4, batch_size=4)

New value:

> epochs=4, batch_size=1)

The following value in the calibration line remains unchanged:

> model_optimization_config(calibration, batch_size=2)

The calibration setting is not modified.

### 6. Find the Actual NMS JSON File

```bash
NMS_JSON="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg -type f -name "yolov11l_nms_config.json" -print -quit)"
```

Verify it:

```bash
echo "$NMS_JSON"
```

Expected output:

> /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json

Confirm that the file exists:

```bash
ls -lh "$NMS_JSON"
```

### 7. Replace the Relative NMS Path with an Absolute Path

```bash
sed -i "s#../../postprocess_config/yolov11l_nms_config.json#$NMS_JSON#" /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

This change is required because the custom ALLS file was copied from its original directory to:

> /local/shared_with_docker/doc/

### 8. Verify All Changes

```bash
sed -n '1,15p' /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

Check the following lines in particular:

> model_optimization_config(calibration, batch_size=2)
> loss_factors=[1,1,1,2,2,2,2,2,2], epochs=4, batch_size=1)
> nms_postprocess("/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json", meta_arch=yolov8, engine=cpu)

To display only the relevant section:

```bash
sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

The following command:

```bash
grep -nE "finetune|nms_postprocess" /local/shared_with_docker/doc/yolov11l_low_memory.alls
```

only displays the first line containing the word `finetune`. Because `batch_size=1` is located on the following line, it is not displayed. Use the `sed` command above to inspect the complete block.

### 9. Prepare a New Compilation Directory

```bash
mkdir -p /local/shared_with_docker/doc/low_memory_compile
```

```bash
cd /local/shared_with_docker/doc/low_memory_compile
```

To view any existing parsed intermediate files:

```bash
ls -lah
```

A `yolov11l.har` file may remain from a previous failed attempt. The new command may recreate it; this is not a problem.

### 10. Start HEF Compilation with the Low-Memory Configuration

As a single-line command:

```bash
hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11l.yaml --model-script /local/shared_with_docker/doc/yolov11l_low_memory.alls --classes 7 --hw-arch hailo8
```

Meaning of the settings:

> `model.onnx` → Your YOLO11l model
> `doc/calib` → Calibration data
> `yolov11l.yaml` → YOLO11l network definition
> `low_memory.alls` → QAT batch size 1 configuration
> `--classes 7` → Seven classes
> `--hw-arch hailo8` → 26 TOPS AI HAT+

### 11. Confirm That the Batch Size Change Was Applied

When QAT starts, you should no longer see:

> Epoch 1/4
> 1/256

When batch size 1 is applied, you should see something similar to:

> Epoch 1/4
> 1/1024

Because:

> 1024 images ÷ batch 1 = 1024 steps

This operation takes longer than the previous attempt, but its instantaneous RAM usage is lower.

### Find the HEF File After Successful Compilation

Inside the container:

```bash
find /local/shared_with_docker/doc/low_memory_compile -type f -name "*.hef"
```

The expected file should be located approximately at:

> /local/shared_with_docker/doc/low_memory_compile/yolov11l.hef

Move and rename the file:

```bash
mv /local/shared_with_docker/doc/low_memory_compile/yolov11l.hef /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef
```

Verify it:

```bash
ls -lh /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef
```

On the host system, the file can be found at:

> Hailo Suite directory/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

</details>

> ⚠️ **Warning:** The number after --classes must match the number of object classes used in your model
yolov11l.hef  
> For example:Objects[rock,paper,scissors] --> --classes 3  
> ⚠️ **Warning:** You can change your 'yolov11l.yaml' model in your code if there is a different one in that directory:'/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks'  
5. `mv yolov11l.hef /local/shared_with_docker/doc/`

## ✅ Resoults
`Your .hef file in that directory and waiting for you:'/home/$USER/docker_hailo/shared_with_docker/doc/'`
