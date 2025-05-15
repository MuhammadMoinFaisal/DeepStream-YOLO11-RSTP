# DeepStream-Yolo11

### Update packages
```
sudo apt update
sudo apt upgrade
```
### Install Python 3.8
```
sudo apt install python3.8 python3.8-dev python3.8-distutils
```
### Set it up with update-alternatives
```
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 2
```
Then choose the default version:
```
sudo update-alternatives --config python3
```
You'll be prompted to select a version. Choose Python 3.8

### Install pip for Python 3.8
Download the get-pip.py script:

```
wget https://bootstrap.pypa.io/get-pip.py
```
### Run it with Python 3.8 version
```
sudo python3.8 get-pip.py
```
### Verify pip is working
```
python3.8 -m pip --version
```



### Check the Python Version
```
python3 --version 
```

### Install DeepStream SDK according to the JetPack version
Download the DeepStream File From NVIDIA
* For JetPack 4.6.4, install [`DeepStream 6.0.1`](https://docs.nvidia.com/metropolis/deepstream/6.0.1/dev-guide/text/DS_Quickstart.html)
* For JetPack 5.1.3, install [`DeepStream 6.3`](https://docs.nvidia.com/metropolis/deepstream/6.3/dev-guide/text/DS_Quickstart.html)
* For JetPack 6.1, install [`DeepStream 7.1`](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Installation.html)

### Install DeepStream
```
sudo apt-get install ./deepstream-6.0_6.0.1-1_arm64.deb
```

### Check DeepStream Version
```
deepstream-app --version
```
### Check CUDA Version
```
nvcc --version
```
### Check the TensorRT Version
```
dpkg -l | grep TensorRT
```
### Convert model

#### Download the YOLO11 repo and install the requirements
```
git clone https://github.com/ultralytics/ultralytics.git
cd ultralytics
pip3 install -e .
pip3 install onnx onnxslim onnxruntime
```

#### Download the model

Download Ultralytics YOLO11 detection model (.pt) of your choice from [YOLO11](https://github.com/ultralytics/assets/releases/) releases. Here we use yolo11n.pt.

```
wget https://github.com/ultralytics/assets/releases/download/v8.3.0/yolo11n.pt
```

#### CLone this repository

```
cd ~
https://github.com/MuhammadMoinFaisal/DeepStream-YOLO11-RSTP.git
```

#### Convert model
Copy the export_yolo11.py file from DeepStream-Yolo/utils directory to the ultralytics folder.
Generate the ONNX model file 
```
cd ultralytics
python3 export_yolo11.py -w yolo11n.pt --dynamic
```

#### 6. Compile the lib

1. Open the `DeepStream-Yolo` folder and compile the lib

2. Set the `CUDA_VER` according to your DeepStream version

```
cd ~
cd DeepStream-Yolo
export CUDA_VER=10.2
```

* x86 platform

  ```
  DeepStream 7.1 = 12.6
  DeepStream 7.0 / 6.4 = 12.2
  DeepStream 6.3 = 12.1
  DeepStream 6.2 = 11.8
  DeepStream 6.1.1 = 11.7
  DeepStream 6.1 = 11.6
  DeepStream 6.0.1 / 6.0 = 11.4
  DeepStream 5.1 = 11.1
  ```

* Jetson platform

  ```
  DeepStream 7.1 = 12.6
  DeepStream 7.0 / 6.4 = 12.2
  DeepStream 6.3 / 6.2 / 6.1.1 / 6.1 = 11.4
  DeepStream 6.0.1 / 6.0 / 5.1 = 10.2
  ```

3. Make the lib

```
make -C nvdsinfer_custom_impl_Yolo clean && make -C nvdsinfer_custom_impl_Yolo
```

##

### Edit the config_infer_primary_yolo11 file

Edit the `config_infer_primary_yolo11.txt` file according to your model

```
[property]
...
onnx-file=yolo11n.pt.onnx
...
num-detected-classes=80
...
parse-bbox-func-name=NvDsInferParseYolo
...
```


##

### Edit the deepstream_app_config file

```
...
[primary-gie]
...
config-file=config_infer_primary_yolo11.txt
```

### Create a new Directory by the name "predictions" to save the detection results i.e. bounding box coordinates, class name and confidence score

```
mkdir predictions
```
### RTSP Stream Ingestion
### Dynamic RTSP Stream Configuration without Restart/ Run the Flask API
```
python3 run_deepstream_flask_database.py
```
### Use a POST /start-stream API to start DeepStream dynamically 

```
curl -X POST http://localhost:5000/start-stream      -H "Content-Type: application/json"      -d '{"rtsp_url": "rtsp://rtspstream:-Xb-e9YNYYf7lwpc6n-PR@zephyr.rtsp.stream/people"}'
```
```
curl -X POST http://localhost:5000/start-stream      -H "Content-Type: application/json"      -d '{"rtsp_url": "rtsp://rtspstream:-Xb-e9YNYYf7lwpc6n-PR@zephyr.rtsp.stream/traffic"}'
```
```
curl -X POST http://localhost:5000/start-stream      -H "Content-Type: application/json"      -d '{"rtsp_url": "http://takemotopiano.aa1.netvolante.jp:8190/nphMotionJpeg?Resolution=640x480&Quality=Standard&Framerate=30"}'
```
###  Use a POST /stop-stream API to stop the running stream
```
curl -X POST http://localhost:5000/stop-stream
```

### DeepStream-YOLO11 Docker Guide
This guide helps you build and run the Docker container for DeepStream-YOLO11 on Jetson Nano using DeepStream 6.0.1.

```
sudo apt install docker.io
```
```
sudo apt install nvidia-container-runtime
```

### Build Docker Image
```
sudo docker build -t deepstream-yolo11 .
```

### Run Container
```
sudo docker run -it --rm \
  --runtime nvidia \
  --gpus all \
  -v /home/muhammad/DeepStream-YOLO11:/opt/deepstream-yolo11 \
  deepstream-yolo11
```

### Pull from Docker Hub
```
docker pull muhammadmoin619/deepstream-yolo11:latest

```
#### Then run it like:
```
docker run -it --rm --runtime nvidia --gpus all muhammadmoin619/deepstream-yolo11:latest
```
```
docker run -it --rm \
  --runtime nvidia \
  --gpus all \
  -v /home/muhammad/DeepStream-YOLO11:/opt/deepstream-yolo11 \
  muhammadmoinfaisal/deepstream-yolo11:latest
```

