## 👀 cv-model-pt_to_hef Overview   2/3  
<h1 align="center">Convert Computer Vision model .pt to .hef for Rasberry Pi 5 Hailo AI HAT</h1>  

## 🔎 Preparation
<details>
<summary>1. Prepare docker enviroment </summary>

Follow these steps:
```bash
sudo apt update
sudo apt upgrade -y

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

sudo systemctl start docker
sudo systemctl enable docker

docker --version
sudo docker run hello-world




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


## 📦 Setup  

1. `unzip hailo8_ai_sw_suite_2025-10_docker.zip -d /home/$USER/docker_hailo`
2. `cd /home/$USER/docker_hailo/`
3. `Edit your .sh document and delete these lines:`
   > -v /etc/timezone:/etc/timezone:ro \
   > -v /etc/localtime:/etc/localtime:ro`
4. `./hailo_ai_sw_suite_docker_run.sh --override`
   > *If you want to continue with your already configured project:*./hailo_ai_sw_suite_docker_run.sh --resume 
5. /home/$USER/docker_hailo/shared_with_docker/train/images/(60%–80% of your photos)  
   /home/$USER/docker_hailo/shared_with_docker/models/model.onnx  


## 🎉 Run 
1. `git clone https://github.com/LukeDitria/RasPi_YOLO.git`
2.  `cd RasPi_YOLO/` Then you shold be in '/local/workspace/RasPi_YOLO/' directory
3.  `python hailo_calibration_data.py     --data_dir /local/shared_with_docker/train/images/     --target_dir /local/shared_with_docker/doc`
4. `hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib/ --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11n.yaml --classes 2 --hw-arch hailo8`
> ⚠️ **Warning:** The number after --classes must match the number of object classes used in your model
yolov11n.hef  
> For example:Objects[rock,paper,scissors] --> --classes 3  
> ⚠️ **Warning:** You can check and choose different yolo models in this directory '/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks'  
5. `mv yolov11n.hef /local/shared_with_docker/doc/`

## ✅ Resoults
`Your .hef file in that directory and waiting for you:'/home/$USER/docker_hailo/shared_with_docker/doc/'`
