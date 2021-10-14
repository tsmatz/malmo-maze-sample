# Reinforcement Learning in Minecraft (Project Malmo Example)

This sample code trains an agent in Minecraft with reinforcement learning. (Here I used RLlib.)<br>
In this example, a maze (in which, the path is randomized) is given and the agent will learn to reach to a goal block using observed frame pixels (84 x 84 x 3 channels).

See [here](https://tsmatz.wordpress.com/2020/07/09/minerl-and-malmo-reinforcement-learning-in-minecraft/) for the tutorial of Project Malmo, which is a modded Minecraft built by Microsoft Research.

Follow this description to run this sample.

## 1. Setup prerequisite environment ##

In this example, I assume Ubuntu 18.04 with real monitor (which is used to show Minecraft UI) to run the training. (I have tried and run these steps on Ubuntu 18.04 in Microsoft Azure.)<br>
You can also join into the same game with your own Minecraft PC client, if needed. (See [here](https://tsmatz.wordpress.com/2020/07/09/minerl-and-malmo-reinforcement-learning-in-minecraft/) for details.)

Now let's start to set up your Ubuntu environment.

<blockquote>
Note : I have run and trained the agent in GPU utilized VM (instance).<br>
When you run on NVIDIA GPU-utilized instance to speed up, please setup drivers and libraries as follows.<br>
In this setup, I use TensorFlow 2.4.1 and please then install corresponding version of drivers (cuda 11.0) and libraries (cuDNN 8.0). See https://www.tensorflow.org/install/source#gpu for details about compatible drivers in TensorFlow.

For preparation, install ```gcc``` and ```make``` tools.

```
sudo apt-get update
sudo apt install gcc
sudo apt-get install make
```

Install cuda 11.0 by the following command. (After installation, make sure that cuda exists in ```/usr/local``` directory.)

```
wget http://developer.download.nvidia.com/compute/cuda/11.0.2/local_installers/cuda_11.0.2_450.51.05_linux.run
sudo sh cuda_11.0.2_450.51.05_linux.run
```

Install cuDNN 8.0.<br>
First, download this library (runtime, dev, and docs) from [NVIDIA developer site](https://developer.nvidia.com/cudnn).

Install downloaded packages as follows.

```
sudo dpkg -i libcudnn8_8.0.5.39-1+cuda11.0_amd64.deb
sudo dpkg -i libcudnn8-dev_8.0.5.39-1+cuda11.0_amd64.deb
sudo dpkg -i libcudnn8-samples_8.0.5.39-1+cuda11.0_amd64.deb
```

Setup is done.

Make sure to install ```tensorflow-gpu==2.4.1``` instead of ```tensorflow==2.4.1``` and train with ```--num_gpu``` option in the following step.
</blockquote>

First, make sure that Python 3 is installed on Ubuntu. (If not, please install Python 3 on Ubuntu.)

```
python3 -V
```

Install and upgrade pip3.

```
sudo apt-get update
sudo apt-get install -y python3-pip
sudo -H pip3 install --upgrade pip
```

Install X remote desktop, and start RDP service.<br>
After this setting, restart your computer.

```
sudo apt-get update
sudo apt-get install -y lxde
sudo apt-get install -y xrdp
/etc/init.d/xrdp start  # password is required
```

Allow (Open) inbound port 3389 (default RDP port) in network settings to allow your client to connect.

> Note : When you want to join into the same game with your own Minecraft client remotely, please open Minecraft port too.

Install and setup Java (JDK) as follows.

```
sudo apt-get install -y openjdk-8-jdk
echo -e "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc
source ~/.bashrc
```

## 2. Install and Setup Project Malmo ##

Install Malmo binaries, which is a modded Minecraft built by [Microsoft Research](https://www.microsoft.com/en-us/research/project/project-malmo/).

```
# install prerequisite packages
pip3 install gym lxml numpy pillow
# install malmo
pip3 install --index-url https://test.pypi.org/simple/ malmo==0.36.0
```

Expand Malmo bootstrap files as follows.<br>
All files will be deployed on ```./MalmoPython``` folder.

```
python3 -c "import malmo.minecraftbootstrap; malmo.minecraftbootstrap.download();"
```

Set ```MALMO_XSD_PATH``` environment variable as follows.

```
echo -e "export MALMO_XSD_PATH=$PWD/MalmoPlatform/Schemas" >> ~/.bashrc
source ~/.bashrc
```

Set the Malmo version in ```./MalmoPlatform/Minecraft/src/main/resources/version.properties``` file by the following command.

```
cd MalmoPlatform/Minecraft
(echo -n "malmomod.version=" && cat ../VERSION) > ./src/main/resources/version.properties
cd ../..
```

## 3. Install Ray and RLlib framework ##

Install ray framework (which is used to run a trained AI agent) with dependencies as follows.

```
pip3 install tensorflow==2.4.1 ray[default]==1.6.0 ray[rllib]==1.6.0 ray[tune]==1.6.0 attrs==19.1.0 pandas
```

> Note : When you run on GPU, install ```tensorflow-gpu==2.4.1``` instead of ```tensorflow==2.4.1```.

## 4. Run Minecraft with Project Malmo ##

Login Ubuntu using remote desktop client.<br>
Run the following commands **on monitor-attached shell** (such as, LXTerminal) to launch Minecraft with Project Malmo mod.

For the first time to run, all dependencies (including Project Malmo mod) are built and installed in Minecraft. Hence it will take a while to start for the first time. (Please be patient to wait.)

```
cd MalmoPlatform/Minecraft
./launchClient.sh -port 9000
```

> Note : When you have any trouble to use the monitor, see "Trouble Shooting" in the appendix below. (See below if compilation fails.)

## 5. Train an agent (Deep Reinforcement Learning) ##

Now let's start training.<br>
Before starting, make sure that Minecraft is running with malmo port 9000. (See above.)

First, clone this repository.

```
git clone https://github.com/tsmatz/minecraft-rl-example
cd minecraft-rl-example
```

Run the training script (train.py) as follows.<br>
Note that this command is not needed to be run on the monitor attached shell. (This process will connect to the running Malmo instance with port 9000.)

```
python3 train.py /YOUR_HOME_DIR/minecraft-rl-example/lava_maze_malmo.xml
```

<blockquote>
Note : When you run on GPU-utilized worker to speed up, specify --num_gpus option as follows.

```
python3 train.py /YOUR_HOME_DIR/minecraft-rl-example/lava_maze_malmo.xml --num_gpus 1
```
</blockquote>

When you start above training code, you will see 84 x 84 Minecraft's screen running. This frame pixels are used by agent to learn.<br>
This frame size (84 x 84 x channel size) is supported for RLlib built-in convolutional network (ConvNet), and no custom model is then needed in this code. (Otherwise, create your own model and configure to use the custom model.)<br>
See the source code ([visionnet.py](https://github.com/ray-project/ray/blob/master/rllib/models/tf/visionnet.py)) for the RLlib built-in convolutions.

This training will take a day and a half long for completion, when running on GPU.

> Note : You can also run training on multiple workers in Ray cluster to speed up training. In this case, each workers in a cluster should be configured to use a virtual monitor, because it will run as a batch in backgroud.<br>
> Using [Azure Machine Learning](https://tsmatz.wordpress.com/2018/11/20/azure-machine-learning-services/), you can quickly configure cluster (multiple workers) with built-in RL estimator.

## 6. Run pre-trained agent

This repository also includes pre-trained checkpoint (```checkpoint/checkpoint-622```) and you can then check the result soon.

After running Minecraft with malmo port 9000 (see above), run this pre-trained agent as follows.

```
python3 run_agent.py /YOUR_HOME_DIR/minecraft-rl-example/lava_maze_malmo.xml
```

If you have your own trained checkpoint, you can also run and simulate your own agent as follows.

```
python3 run_agent.py /YOUR_HOME_DIR/minecraft-rl-example/lava_maze_malmo.xml --checkpoint_file YOUR_OWN_CHECKPOINT_FILE_PATH
```

![Simulate a trained agent](https://tsmatz.files.wordpress.com/2020/07/20200717_rollout_capture.gif)

After you have learned, please proceed to the scalable distributed RL with Ray cluster. (See [here](https://github.com/tsmatz/minecraft-rl-on-ray-cluster).)

## Appendix : Trouble Shooting

**Xrdp won't accept a special character for password.**

Please create a new user with a simple password.

**Minecraft compilation errors**

If ```forgeSrc.jar``` (```forgeSrc-1.11.2-13.20.0.2228.jar```) could not be found and the error occurs in Minecraft compilation, please download and use the successful cache [malmo-gradle-caches.zip](https://1drv.ms/u/s!AuopXnMb-Aqcgc9v58e-4x8hhQCFDg?e=EZtHP9).

```
mv ~/.gradle/caches ~/.gradle/caches-org
sudo apt-get install zip unzip
unzip malmo-gradle-caches.zip -d ~/.gradle/caches
```

**Error for connecting to Minecraft client**

When you have an error "Failed to find an available client for this mission" in ```startMission()```, please check as follows.

- Minecraft client is correctly running
- Not setting ```LC_ALL``` environment variable as follows (This will prevent from connecting)<br>
```export LC_ALL=C.UTF-8```

**Error in desktop session start**

See error details in ```~/.xsession-errors```, when it has some toubles to start xrdp (X remote desktop) session. Set ```mate-session``` in ```~/.xsession``` to fix, if needed.

**Azure DSVM or ML compute**

When you use data science virtual machine (DSVM) or [AML](https://tsmatz.wordpress.com/2018/11/20/azure-machine-learning-services/) compute in Azure :

- Deactivate conda environment, since MineRL cannot be installed with conda.

```
echo -e "conda deactivate" >> ~/.bashrc
source ~/.bashrc
```

- It will include NVidia cuda, even when you run on CPU VM. This will cause a driver error ("no OpenGL context found in the current thread") when you run Minecraft java server with malmo mod.<br>
  Thereby, please ensure to uninstall cuda.

```
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean
sudo rm -rf /usr/local/cuda*
```

**Errors for display setting (monitor)**

When your application cannot detect your display (monitor), please ensure to set ```DISPLAY``` as follows.<br>
(The error message "MineRL could not detect a X Server, Monitor, or Virtual Monitor" will show up.)

```
# check your display id
ps -aux | grep vnc
# set display id (when your display id is 10)
export DISPLAY=:10
```

When you cannot directly show outputs in your physical monitor, please divert outputs through a virtual monitor (xvfb).<br>
For instance, the following will show outputs (Minecraft game) on your own VNC viewer window through a virtual monitor (xvfb).

```
# install components
sudo apt-get install xvfb
sudo apt-get install x11vnc
sudo apt-get install xtightvncviewer
# generate xvfb monitor (99) and bypass to real monitor (10)
/usr/bin/Xvfb :99 -screen 0 768x1024x24 &
/usr/bin/x11vnc -rfbport 5902 -forever -display :99 &
DISPLAY=:10 /usr/bin/vncviewer localhost:5902 &
# run program
export DISPLAY=:99
python3 train.py
```
