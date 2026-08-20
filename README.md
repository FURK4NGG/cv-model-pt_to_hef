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
In "(hailo_virtualenv) hailo@user:" terminal:  
1. `git clone https://github.com/LukeDitria/RasPi_YOLO.git`
2.  `cd RasPi_YOLO/` Then you shold be in '/local/workspace/RasPi_YOLO/' directory
3.  `python hailo_calibration_data.py     --data_dir /local/shared_with_docker/train/images/     --target_dir /local/shared_with_docker/doc`
4. `hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib/ --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11n.yaml --classes 2 --hw-arch hailo8`

<details>
<summary>Eger calistirdiktan sonra killed yazarsa sunlari yapabilirsin:</summary>
Container dışında, normal host terminalinde:
sudo journalctl -k -b \
  | grep -Ei "out of memory|oom|killed process"

Arch Linux’ta ayrıca:
sudo dmesg -T \
  | grep -Ei "out of memory|oom|killed process"

Şuna benzer bir çıktı çıkarsa teşhis kesindir:

>Out of memory: Killed process ...
>Killed process ... python



free -h
swapon --show
>Mem:   27Gi
>Swap:  32Gi



Docker bellek sınırını kontrol et
Container adını öğren:
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

Ardından gerçek container adını kullan:
docker inspect CONTAINER_ADI \
  --format 'Memory={{.HostConfig.Memory}} MemorySwap={{.HostConfig.MemorySwap}}'

Şöyle çıkarsa özel sınır yoktur:
>Memory=0 MemorySwap=0


1. Orijinal YOLO11l ALLS dosyasını bul
find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls"

Örneğin şu sonucu vermeli:

/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls/generic/yolov11l.alls
2. Bulunan yolu değişkene kaydet
DEFAULT_ALLS="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/alls -type f -name "yolov11l.alls" -print -quit)"

Kontrol et:

echo "$DEFAULT_ALLS"

Çıktı boş olmamalı ve sonunda şu bulunmalı:

yolov11l.alls

Bu değişken yalnızca dosya yolunu geçici olarak saklıyor. Herhangi bir dosyayı değiştirmiyor.

3. Orijinal finetune ayarlarını görüntüle
sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' "$DEFAULT_ALLS"

Çıktının sonunda yaklaşık olarak şu bulunmalı:

epochs=4, batch_size=4)
4. Orijinal ALLS dosyasını ortak klasöre kopyala
cp -f "$DEFAULT_ALLS" /local/shared_with_docker/doc/yolov11l_low_memory.alls

Kontrol et:

ls -lh /local/shared_with_docker/doc/yolov11l_low_memory.alls

Bu işlem Hailo Model Zoo içindeki orijinal dosyayı değiştirmez.

5. QAT batch size değerini 4’ten 1’e düşür
sed -i 's/batch_size=4)/batch_size=1)/' /local/shared_with_docker/doc/yolov11l_low_memory.alls

Bu komut yalnızca şu değeri değiştirir:

epochs=4, batch_size=4)

Yeni değer:

epochs=4, batch_size=1)

Calibration satırındaki:

model_optimization_config(calibration, batch_size=2)

değişmez. Ona dokunmuyoruz.

6. Gerçek NMS JSON dosyasını bul
NMS_JSON="$(find /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg -type f -name "yolov11l_nms_config.json" -print -quit)"

Kontrol et:

echo "$NMS_JSON"

Beklenen çıktı:

/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json

Dosyanın bulunduğunu doğrula:

ls -lh "$NMS_JSON"
7. Göreli NMS yolunu mutlak yolla değiştir
sed -i "s#../../postprocess_config/yolov11l_nms_config.json#$NMS_JSON#" /local/shared_with_docker/doc/yolov11l_low_memory.alls

Bu gerekli çünkü özel ALLS dosyasını orijinal klasöründen çıkarıp şuraya koyduk:

/local/shared_with_docker/doc/
8. Yapılan bütün değişiklikleri kontrol et
sed -n '1,15p' /local/shared_with_docker/doc/yolov11l_low_memory.alls

Özellikle şu satırları kontrol et:

model_optimization_config(calibration, batch_size=2)
loss_factors=[1,1,1,2,2,2,2,2,2], epochs=4, batch_size=1)
nms_postprocess("/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/postprocess_config/yolov11l_nms_config.json", meta_arch=yolov8, engine=cpu)

Yalnızca ilgili bölümü görmek için:

sed -n '/post_quantization_optimization(finetune/,/nms_postprocess/p' /local/shared_with_docker/doc/yolov11l_low_memory.alls

Senin kullandığın şu komut:

grep -nE "finetune|nms_postprocess" /local/shared_with_docker/doc/yolov11l_low_memory.alls

yalnızca finetune kelimesinin bulunduğu ilk satırı gösterir. batch_size=1 sonraki satırda olduğu için görünmez. Bu nedenle yukarıdaki sed komutuyla bütün bloğu kontrol et.

9. Yeni derleme klasörünü hazırla
mkdir -p /local/shared_with_docker/doc/low_memory_compile
cd /local/shared_with_docker/doc/low_memory_compile

Mevcut parse edilmiş ara dosyaları görmek için:

ls -lah

Önceki başarısız denemeden kalan yolov11l.har bulunabilir. Yeni komut bunu yeniden oluşturabilir; sorun değildir.

10. Düşük bellek ayarıyla HEF derlemesini başlat

Tek satır hâlinde:

hailomz compile --ckpt /local/shared_with_docker/models/model.onnx --calib-path /local/shared_with_docker/doc/calib --yaml /local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks/yolov11l.yaml --model-script /local/shared_with_docker/doc/yolov11l_low_memory.alls --classes 7 --hw-arch hailo8

Ayarların anlamı:

model.onnx          → YOLO11l modelin
doc/calib           → Calibration verileri
yolov11l.yaml       → YOLO11l ağ tanımı
low_memory.alls     → QAT batch size 1 ayarı
--classes 7         → Yedi class
--hw-arch hailo8    → 26 TOPS AI HAT+
11. Batch size değişikliğinin uygulandığını kontrol et

QAT başladığında önceki gibi:

Epoch 1/4
1/256

görmemelisin.

Batch size 1 uygulandığında yaklaşık şöyle görünmeli:

Epoch 1/4
1/1024

Çünkü:

1024 görüntü ÷ batch 1 = 1024 adım

Bu işlem önceki denemeden daha uzun sürer fakat anlık RAM kullanımı daha düşük olur.


İşlem başarıyla tamamlanırsa HEF’i bul

Container içinde:

find /local/shared_with_docker/doc/low_memory_compile -type f -name "*.hef"

Beklenen dosya yaklaşık olarak:

/local/shared_with_docker/doc/low_memory_compile/yolov11l.hef

Adını değiştir:

mv /local/shared_with_docker/doc/low_memory_compile/yolov11l.hef /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

Kontrol et:

ls -lh /local/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

Host sisteminde dosya şurada bulunur:

Hailo Suite klasörü/shared_with_docker/doc/wildlife_yolo11l_hailo8.hef

</details>

> ⚠️ **Warning:** The number after --classes must match the number of object classes used in your model
yolov11n.hef  
> For example:Objects[rock,paper,scissors] --> --classes 3  
> ⚠️ **Warning:** You can change your 'yolov11n.yaml' model in your code if there is a different one in that directory:'/local/workspace/hailo_model_zoo/hailo_model_zoo/cfg/networks'  
5. `mv yolov11n.hef /local/shared_with_docker/doc/`

## ✅ Resoults
`Your .hef file in that directory and waiting for you:'/home/$USER/docker_hailo/shared_with_docker/doc/'`
