# Steps for Object Detection on Raspberry-pi using Edge Impulse platform

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Install Raspberry Pi OS on Raspberry Pi 4 model B using Raspberry Pi Imager

* Download Raspberry Pi Imager from [Rasberry Pi Imager for Windows](https://www.raspberrypi.com/software/) to your system.
* Install it to SD card with SD card reader.


## Connect Raspberry Pi to your Internet (Wifi/Mobile Data)

* You can use some IP scanner tool to scan and check if your Raspi is connected to your internet
* You can download [Angry IP Scanner](https://angryip.org/download/#windows) from here.
* Scan using IP scanner and identify IP address of RasPi

## Access Raspberry-pi Graphical Desktop remotely

* To access graphical desktop of RasPi from your laptop, download [vnc viewer](https://www.realvnc.com/en/connect/download/viewer/) from here.
* Add new connection in VNC viewer.
* Enter IP address of RasPi and password set for Raspi which was set when OS was installed on RasPi using Raspi Imager.
* Now RasPi Desktop will be visible on your laptop screen.
* Sometimes, it may not be visible in first attempt and can show following possible errors:

##### 1. 'The connection was refused by the computer'

* Open PUTTY
* Type IP address of Raspi in Host Name or IP address text box
* Enter username and password that you have set for Raspi while flashing OS on SD card
* Now you are logged in to raspi terminal
* Type command ‘sudo raspi-config’ on Raspi Terminal
* Go to ‘Interface’ option-->> vnc --> enable -->ok
* Now connect from vnc viewer again

##### 2. 'Timed out waiting for a response from the computer'

* This issue can be resolved by configuring firewall to allow VNC Viewer port
* Search Windows Firewall in the search box and then select "Windows Firewall with advanced security" to open Port 5900.
* Select Inbound Rules from the left sidebar. Click New Rule in the right sidebar
* Select Port and click Next.
* Select TCP, enter 5900 in the Specific local port field, then click Next.
* Choose Allow the connection and click Next. Choose Domain and click Next. Enter a name for the rule, such as “vnc", and then click Finish.
* Restart the VNC Viewer

## Connecting to Raspberry Pi Bullseye Camera

* Once you are able to access Raspi remotely from laptop, next step is to connect to Raspi camera
* Open Raspi terminal
* Type command `raspistill` on terminal
* Following possible error may occur:

##### 1. 'Can not currently show Desktop'

* To resolve this issue, connect to Raspi through ssh
* i.e. Open command prompt
* Type command `ssh@ip_address_of_Raspi`
* Enter password set for Raspi while installing OS on it. 
* At terminal type command `sudo nano  /boot/config.txt`
* Config.txt file will be opened
* Do following modifications in `config.txt` file
    1. Uncomment `hdmi_force_hotplug=1`
    2. Uncomment `hdmi_group=1``
    3. Modify and uncomment `hdmi_model=1` to `hdmi_mode=16`
    4. Under [pi4] title comment the first line.
    5. Save `config.txt` using `ctrl+s` and exit using `ctrl+x`
* Reboot the raspi using sudo reboot command
* Go to Raspi terminal and use command `raspistill`

## Connecting raspberry Pi to Edge Impulse

* Install dependencies by running following commands
* `sudo apt update`
* `curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -`
* `sudo apt install -y gcc g++ make build-essential nodejs sox gstreamer1.0-tools gstreamer1.0-plugins-good gstreamer1.0-plugins-base gstreamer1.0-plugins-base-apps`
* `npm config set user root && sudo npm install edge-impulse-linux -g --unsafe-perm`
* Connect to edge impulse by running the command `edge-impulse-linux`
* This will start a wizard which will ask you to log in, and choose an Edge Impulse project
* Raspberry Pi is now connected to Edge Impulse. To verify this, go to your Edge Impulse project, and   click Devices. The device will be listed here.
* Once the device is connected to Edge Impulse Platform, below sequence of steps are followed which includes steps from data collection to live inference.

#####   1. Data Acquisition

* Click on `Data acquisition` tab in edge impulse and capture images using Raspberry Pi camera
* Divide captured images into two sets. 80% images in training set and 20% images in test set

#####   2. Impulse Design (Data acquisition, Data processing and Model training)

* Click on `Impulse design` tab. 
* An impulse takes raw data, uses signal processing to extract features, and then uses a learning block to classify new data.
* A complete Impulse will consist of 3 main building blocks: input block, processing block and a learning block.
* Click on `create impulse` option. Input block indicates type of input data you are training your model with. For Image input in our case, we use square image size. 360x360. Select this image size in input block.
* A processing block is basically a feature extractor. Select `pre-process and normalize images` option from available options.
* After adding processing block, next step is to add learning block. A learning block is simply a neural network that is trained to learn on your data. Learning blocks vary depending on what you want your model to do and the type of data in your training dataset. Select fine tuned pretrained model for object detection. It is recommended option by Edge Impulse.
* Save the created impulse.
* Click on `Image` tab under `Impulse Design` tab.
* Under `Parameters` tab click on `Save parameters`
* Under `Generate Features` tab click on `Generate Features`. It will generate learned features.
* Now click on `Object Detection` tab under `Impulse Design` tab. Here you can customize the neural network model setting that you have selected in learning block. You can customize the parameters like `Number of training cycles`, `learning rate`, `validation set size` etc. 
* Also, you can change the model if you want to and start training the model.
* Once the model is trained, it will give the performance of training model in the form of accuracy or precision score.
* It also displays `inference time` and `flash usage` for selected type of output device which is Raspberry Pi4 in this case
* You can fine tune neural network settings to get improved performance.

##### Live Inference

* Once model is trained for its best performance, we can capture image from Raspberry Pi4 camera and classify it using our trained model i.e. live inference
* To get live inference by trained model, click on `Live Classification` model.
* Select `Device` as `Raspberry Pi`.
* Select `Sensor` as `camera` and click on `Start Sampling`.
* This will capture live image. Model immediately detects the object in current frame and labels it.
* It shows bounding box around the object with label and confidence level.

##### Deploying the model back to device

* Once the model is trained and tested for live inference, the trained model is deployed on the Raspberry Pi. It will run your impulse locally.
* Type command `edge-impulse-linux-runner` on Raspberry Pi terminal to run the impulse locally.
* This will automatically compile your model with full hardware acceleration, download the model to your Raspberry Pi, and then start object detection.
* It gives an URL on terminal. If you want to see the feed of camera and live object detection in browser, then open this url in browser and live object detection can be seen in browser.