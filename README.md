## 👀 cv-model-pt_to_hef Overview  
<h1 align="center">Convert Computer Vision model .pt to .hef for Rasberry Pi 5 Hailo AI HAT</h1>  

## 🔎 Preparation
<details>
<summary>1. Prepare docker enviroment </summary>

Follow these steps:
```bash
sudo systemctl stop docker.socket
sudo systemctl stop docker.service

sudo systemctl status docker
sudo systemctl status docker.socket

sudo mv /var/lib/docker /home/$USER/docker_data
sudo ln -s /home/$USER/docker_data /var/lib/docker

sudo systemctl start docker
sudo systemctl enable docker

Docker Root Dir: /home/$USER/docker_data
```
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

/home/$USER/docker_hailo/hailo8_ai_sw_suite_2025-10_docker.zip
/home/$USER/docker_hailo/train/images/(60%–80% of your photos)
/home/$USER/docker_hailo/shared_with_docker/models/model.onnx
/home/bob/Docker_hailo/shared_with_docker/doc/

## 📦 Setup 
1. `unzip hailo8_ai_sw_suite_2025-10_docker.zip -d /home/$USER/docker_hailo`
2. `Edit your .sh document and delete
3.   -v /etc/timezone:/etc/timezone:ro \
     -v /etc/localtime:/etc/localtime:ro`
4. `./hailo_ai_sw_suite_docker_run.sh --override`

sudo systemctl stop docker.socket
sudo systemctl stop docker.service

sudo systemctl status docker
sudo systemctl status docker.socket

sudo mv /var/lib/docker /home/bob/docker_data
sudo ln -s /home/bob/docker_data /var/lib/docker

sudo systemctl start docker
sudo systemctl enable docker

Docker Root Dir: /home/bob/docker_data





## 🎉 Run  
``
