# GoPro to ORB-SLAM3 Pipeline

This repository provides a streamlined process to use GoPro videos with ORB-SLAM3. Follow the steps below to set up and run the pipeline.

---

## Steps to Use

### 1. Clone the Repository and Build the CUDA Container
Clone this repository and build the provided CUDA container. The container includes all necessary packages.

---

### 2. Copy GoPro Videos to the Container
Transfer your GoPro videos from the host machine to the Docker container:

```bash
docker cp /path/to/video_on_host /path/to/video_in_container
```

---

### 3. Generate Euroc Format Data
Run the following command to convert your GoPro video to the required Euroc format:

```bash
roslaunch gopro_ros gopro_to_asl.launch gopro_video:=/ORB_SLAM3/catkin_ws/Data/GX010024.MP4 rosbag:=/ORB_SLAM3/catkin_ws/Data/GX010024.bag asl_dir:=/ORB_SLAM3/catkin_ws/Data/GX010024/
```

---

### 4. Extract IMU Parameters
Use the following command to find the IMU parameters:
fisrt you need to create a config file like 
```bash
imu_topic: '/gopro/imu'
imu_rate: 200
measure_rate: 200 # Rate to which imu data is subsampled
sequence_time: 40 # 3 hours in seconds
```
```bash
rosrun allan_variance_ros allan_variance /ORB_SLAM3/catkin_ws/Data/ /ORB_SLAM3/catkin_ws/Data/imu_config.yaml
```
if nan in allan_variance.csv

```bash
sed -i '1201,$d' allan_variance.csv
rosrun allan_variance_ros analysis.py --data /ORB_SLAM3/catkin_ws/Data/allan_variance.csv

```
---

### 5. Prepare the `.txt` File
Run the following commands to create a correctly formatted `.txt` file:

```bash
cp data.csv data.txt
sed -i '1d' data.txt
sed -i 's/^\([^,]*\),[^,]*/\1/' data.txt
```

---

### 6. Launch ORB-SLAM3
Use this command to launch the SLAM process:

```bash
./Monocular-Inertial/mono_inertial_euroc ../Vocabulary/ORBvoc.txt /ORB_SLAM3/catkin_ws/Data/mono_inertial_EuRoC.yaml /ORB_SLAM3/catkin_ws/Data/GOPRO/ /ORB_SLAM3/catkin_ws/Data/GOPRO/mav0/cam0/data.txt
```

---

### 7. Troubleshooting: Reducing Image Resolution
If you encounter the issue "waiting for images," you may need to reduce the image resolution. Use the following Python script to resize images in your dataset:

```python
import os
from PIL import Image
from tqdm import tqdm

def resize_images_in_folder(folder_path, output_size=(960, 540)):
    # List all PNG files in the folder
    files = [f for f in os.listdir(folder_path) if f.endswith('.png')]
    total_files = len(files)

    if total_files == 0:
        print("No PNG files found in the specified folder.")
        return

    print(f"Resizing {total_files} PNG files to {output_size}...")

    for i, file_name in enumerate(tqdm(files, desc="Resizing", unit="file")):
        file_path = os.path.join(folder_path, file_name)
        
        # Open and resize the image
        with Image.open(file_path) as img:
            resized_img = img.resize(output_size, Image.ANTIALIAS)

            # Save the resized image (overwrites the original)
            resized_img.save(file_path)

    print("All images resized successfully!")

# Example usage
folder_path = "data"  # Replace with the path to your folder
resize_images_in_folder(folder_path)
```

# ORB_SLAM3 docker

This docker is based on <b>Ros Noetic Ubuntu 20</b>. If you need melodic with ubuntu 18 checkout #8fde91d

There are two versions available:
- CPU based (Xorg Nouveau display)
- Nvidia Cuda based. 

To check if you are running the nvidia driver, simply run `nvidia-smi` and see if get anything.

Based on which graphic driver you are running, you should choose the proper docker. For cuda version, you need to have [nvidia-docker setup](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) on your machine.

---

## Compilation and Running

Steps to compile the Orbslam3 on the sample dataset:

- `./download_dataset_sample.sh`
- `build_container_cpu.sh` or `build_container_cuda.sh` depending on your machine.

Now you should see ORB_SLAM3 is compiling. 
- Download A sample MH02 EuRoC example and put it in the `Datasets/EuRoC/MH02` folder
```
mkdir -p Datasets/EuRoC 
wget -O Datasets/EuRoC/MH_02_easy.zip http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_02_easy/MH_02_easy.zip
unzip Datasets/EuRoC/MH_02_easy.zip -d Datasets/EuRoC/MH02
```
To run a test example:
- `docker exec -it orbslam3 bash`
- `cd /ORB_SLAM3/Examples && bash ./euroc_examples.sh`
It will take few minutes to initialize. Pleasde Be patient.
---

You can use vscode remote development (recommended) or sublime to change codes.
- `docker exec -it orbslam3 bash`
- `subl /ORB_SLAM3`
