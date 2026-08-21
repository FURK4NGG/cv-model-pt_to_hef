## 👀 cv-model-pt_to_hef Overview   2/3  
<h1 align="center">Convert Computer Vision model .pt to .hef for Raspberry Pi 5 Hailo AI HAT</h1>  

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
In "(hailo_virtualenv) hailo@user:" terminal:  
1. `git clone https://github.com/LukeDitria/RasPi_YOLO.git`
2.  `cd RasPi_YOLO/` Then you shold be in '/local/workspace/RasPi_YOLO/' directory
3.  `python hailo_calibration_data.py     --data_dir /local/shared_with_docker/train/images/     --target_dir /local/shared_with_docker/doc`
4. `hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib/ --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11l.yaml --classes 7 --hw-arch hailo8`

<details>
<summary>If the process displays <code>Killed</code> after running the command, follow these steps:</summary>

Confirm That the Process Was Terminated Due to Insufficient Memory

Run the following command outside the container, in the normal host terminal:

sudo journalctl -k -b \
  | grep -Ei "out of memory|oom|killed process"

On Arch Linux, you can also use:

sudo dmesg -T \
  | grep -Ei "out of memory|oom|killed process"

If you receive output similar to the following, the diagnosis is confirmed:

Out of memory: Killed process ...
Killed process ... python

Check the available RAM and swap:

free -h
swapon --show

Example output:

Mem:   27Gi
Swap:  32Gi
Check the Docker Memory Limit

Find the container name:

docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

Then use the actual container name:

docker inspect CONTAINER_NAME \
  --format 'Memory={{.HostConfig.Memory}} MemorySwap={{.HostConfig.MemorySwap}}'

If the result is as follows, no custom Docker memory limit is configured:

Memory=0 MemorySwap=0
1. Find the Original YOLO11l ALLS File
find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls"

It should return a result similar to:

/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls/generic/yolov11l.alls
2. Store the Discovered Path in a Variable
DEFAULT_ALLS="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls" -print -quit)"

Verify it:

echo "$DEFAULT_ALLS"

The output must not be empty and should end with:

yolov11l.alls

This variable only stores the file path temporarily. It does not modify any file.

3. Display the Original Finetune Settings
sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' "$DEFAULT_ALLS"

The end of the output should contain something similar to:

epochs=4, batch_size=4)
4. Copy the Original ALLS File to the Shared Directory
cp -f "$DEFAULT_ALLS" /local/shared_with_docker/doc/yolov11l_low_memory.alls

Verify the copied file:

ls -lh /local/shared_with_docker/doc/yolov11l_low_memory.alls

This operation does not modify the original file inside Hailo Model Zoo.

5. Reduce the QAT Batch Size from 4 to 1
sed -i 's/batch_size=4)/batch_size=1)/' /local/shared_with_docker/doc/yolov11l_low_memory.alls

This command changes only the following value:

epochs=4, batch_size=4)

New value:

epochs=4, batch_size=1)

The following value in the calibration line remains unchanged:

model_optimization_config(calibration, batch_size=2)

The calibration setting is not modified.

6. Find the Actual NMS JSON File
NMS_JSON="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg -type f -name "yolov11l_nms_config.json" -print -quit)"

Verify it:

echo "$NMS_JSON"

Expected output:

/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json

Confirm that the file exists:

ls -lh "$NMS_JSON"
7. Replace the Relative NMS Path with an Absolute Path
sed -i "s#../../postprocess_config/yolov11l_nms_config.json#$NMS_JSON#" /local/shared_with_docker/doc/yolov11l_low_memory.alls

This change is required because the custom ALLS file was copied from its original directory to:

/local/shared_with_docker/doc/
8. Verify All Changes
sed -n '1,15p' /local/shared_with_docker/doc/yolov11l_low_memory.alls

Check the following lines in particular:

model_optimization_config(calibration, batch_size=2)
loss_factors=[1,1,1,2,2,2,2,2,2], epochs=4, batch_size=1)
nms_postprocess("/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json", meta_arch=yolov8, engine=cpu)

To display only the relevant section:

sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' /local/shared_with_docker/doc/yolov11l_low_memory.alls

The following command:

grep -nE "finetune|nms_postprocess" /local/shared_with_docker/doc/yolov11l_low_memory.alls

only displays the first line containing the word finetune. Because batch_size=1 is located on the following line, it is not displayed. Use the sed command above to inspect the complete block.

9. Prepare a New Compilation Directory
mkdir -p /local/shared_with_docker/doc/low_memory_compile
cd /local/shared_with_docker/doc/low_memory_compile

To view any existing parsed intermediate files:

ls -lah

A yolov11l.har file may remain from a previous failed attempt. The new command may recreate it; this is not a problem.

10. Start HEF Compilation with the Low-Memory Configuration

As a single-line command:

hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11l.yaml --model-script /local/shared_with_docker/doc/yolov11l_low_memory.alls --classes 7 --hw-arch hailo8

Meaning of the settings:

model.onnx          → Your YOLO11l model
doc/calib           → Calibration data
yolov11l.yaml       → YOLO11l network definition
low_memory.alls     → QAT batch size 1 configuration
--classes 7         → Seven classes
--hw-arch hailo8    → 26 TOPS AI HAT+
11. Confirm That the Batch Size Change Was Applied

When QAT starts, you should no longer see:

Epoch 1/4
1/256

When batch size 1 is applied, you should see something similar to:

Epoch 1/4
1/1024

Because:

1024 images ÷ batch 1 = 1024 steps

This operation takes longer than the previous attempt, but its instantaneous RAM usage is lower.

Find the HEF File After Successful Compilation

Inside the container:

find /local/shared_with_docker/doc/low_memory_compile -type f -name "*.hef"

The expected file should be located approximately at:

/local/shared_with_docker/doc/low_memory_compile/yolov11l.hef

Rename the file:

mv /local/shared_with_docker/doc/low_memory_compile/yolov11l.hef /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

Verify it:

ls -lh /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

On the host system, the file can be found at:

Hailo Suite directory/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

</details>

> ⚠️ **Warning:** The number after --classes must match the number of object classes used in your model
yolov11l.hef  
> For example:Objects[rock,paper,scissors] --> --classes 3  
> ⚠️ **Warning:** You can change your 'yolov11l.yaml' model in your code if there is a different one in that directory:'/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks'  
5. `mv yolov11l.hef /local/shared_with_docker/doc/`

## ✅ Resoults
`Your .hef file in that directory and waiting for you:'/home/$USER/docker_hailo/shared_with_docker/doc/'`
