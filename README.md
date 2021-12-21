# Welcome to DonkeyCar on balena!  

A [DonkeyCar](https://www.donkeycar.com) is a small autonomous vehicle project built on an RC Car chassis, that uses computer vision and machine learning techniques to convert the platform into a self-driving vehicle.  It uses a Raspberry Pi 3 or Jetson Nano, a camera, a motor driver, and of course the DonkeyCar application to perform this task.  There is also a "virtual" version that you can use to get started, prior to even building the physical version.  A "virtual" DonkeyCar looks rather like a video game, and it connects to a DonkeyCar Simulator.  Just like the physical version, the virtual Donkey can attempt to drive itself, in the virtual world.

![](https://docs.donkeycar.com/assets/sim_screen_shot.png)

To cover these different scenarios there are 4 main Repositories for the project:

 - [DonkeyCar - Physical Build - Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Physical)
 - [DonkeyCar - Physical Build - UpBoard](https://github.com/dtischler/balena-DonkeyCar-Physical-UpBoard)
 - [DonkeyCar - Virtual Build, Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Virtual) 
 - [DonkeyCar Simulator - Intel NUC / x86](https://github.com/dtischler/balena-DonkeyCar-Simulator)

This Readme and Repo cover the **`Virtual Build of a DonkeyCar using a Raspberry Pi 3`** version of the project.  You can refer to the other GitHub repos linked above, for other components or flavors of DonkeyCar.

Also note, the DonkeyCar project has their own [detailed documentation available](https://docs.donkeycar.com), that goes into more advanced topics and gives thorough descriptions of the architecture, machine learning and training scenarios, performance and tuning of models, and many more topics.  This Readme is simply intended to make it easy to get started, and once you have the core concepts down, be sure to refer to their documentation for more advanced guidance. 

## Intro

Waymo, Tesla, Cruise, and other companies already have self-driving vehicles deployed out in the world around us.  With this project, it is possible to build a miniature version of an autonomous vehicle, intended to race around a track, which is perfect for learning how the core concepts of computer vision and machine learning play a role in self-driving vehicles!  Keep in mind this Repo is specifically devoted to the "virtual" DonkeyCar, which does not involve any physical construction and does **NOT** require an RC Car Chassis.  We are going to run the DonkeyCar software stack, but it will connect up to a DonkeyCar Server that is running what ultimately looks like a video game world.  However, our virtual DonkeyCar can be trained and tested just like a "regular" DonkeyCar, and can ultimately attempt to drive itself around the virtual world.  If you are looking to build a "real" DonkeyCar, simply check out [this repo](https://github.com/dtischler/balena-DonkeyCar-Physical) to get started.

The only items required to run DonkeyCar in this repo are a Raspberry Pi 3 or 4, Power Supply, and SD Card (a high quality, fast one is best).

Before we begin, let's also cover a few basics.  The virtual car can be driven through a web browser, using the keyboard, a Bluetooth gamepad, or even via the accelerometer on your cell phone.  While driving, the DonkeyCar will record data about the car's throttle position, steering position, and what it "sees" in the virtual camera as you make way around the racetrack.  When you are finished recording, all of the data is collected into a folder called a "Tub".  This metadata is then used as the input to create or "train" an AI model.  Once complete, the output of that process (the resulting *model* file) can then be used by the DonkeyCar to attempt to drive itself.  The Raspberry Pi loads the model, and attempts to navigate and move around the track on its own!

The process in the middle - the Training - takes a very long time on a Raspberry Pi.  Analyzing all of the data that got recorded during driving, creating a neural network, and iterating on that data over and over through what are called epochs can easily take hours on the Pi.  And once complete, there is no guarantee the DonkeyCar will even be able to drive safely and accurately!  So, to improve that feedback loop and to reduce the iteration time, most people find it better to drive the DonkeyCar and collect the data, but then transfer that resulting "Tub" of data to a cloud server or a PC with a GPU, and perform the training there.  Once it completes (hopefully much quicker!), the resulting output of the process is the *model* file just like mentioned above, and that model can be transferred back to the Pi.  Then, the DonkeyCar can try to drive using it (hopefully it drives good!).

## Build

As there is no physical construction of the DonkeyCar in this Repo, we can move right into deploying the software stack onto the Raspberry Pi.  This is where we diverge a bit from the official DonkeyCar documentation and process, and "balenafy" the project:

 - In the official DonkeyCar workflow, they begin to walk you through installation of Raspbian, installation of added software and packages such as Tensorflow, PyTorch, OpenCV, Keras, and the rest of the bits necessary for the machine learning functionality.  Then, they have the user install Python and the DonkeyCar application.  At the end of the process, if everything worked, the web interface should load and the car can be driven remotely.
 - However, in this repo, we have bundled all of those bits into a Dockerfile, and instead of performing all of those manual installation steps, you can instead just click this button to launch a build in the cloud, provision a balena device, download an SD Card image, and end up with the same result:

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/dtischler/balena-DonkeyCar-Virtual)

More specifically:
1. Click on the Blue Button just above.
2. Log in to balenaCloud, or create an account if you do not already have one.  (It is free :-) )
3. Create a name for the application in the pop-up modal, and choose the RaspberryPi 3 from the drop down list.
4. Click Create and deploy.
5. Click Add Device.
6. You will come to the Summary page.  Here, click "Add Device".
7. While developing, it is probably best to choose a Development variant of the operating system, and you can enter your WiFi credentials as well.
8. At the bottom of the modal, click "Download balenaOS".
9. After download completes, flash the file to your SD Card with Etcher.
10. Insert the SD Card into the Raspberry Pi, plug in power, and wait a few minutes for it to register itself with balenaCloud and come online.
11. After another moment, the Pi will begin downloading the pre-built DonkeyCar container, which will take some time.  Get a cup of tea while this occurs.

Once the Pi has finished downloading the container, we have two quick settings we need to add in the balenaCloud dashboard.  After each entry, the Pi will reboot, so after you get the first entry in, it will take a moment before you enter the second variable we need to alter.  So, you'll just have to watch closely, but not a big deal.

First, click on Device Configuration on the left in the balenaCloud dashboard, and look for "Define device GPU memory in megabytes".  It is likely set to `16`.  Click on the pencil icon to edit it, and change the value to `128`.  Click "Save".  This is going to trigger the first reboot.  Click on Summary on the left navigation, and watch for a moment as the device shuts down, then in a moment comes back online.  Once it is back "Online", we can enter the second setting.  Click on Device Configuration again, and this time scroll down to "Custom Configuration Variable" section.  Click the blue "Add Custom Variable" button.  In the name, enter `BALENA_HOST_CONFIG_start_x` and in the value, just enter a `1`.  Click Save.  This will again trigger a reboot. 

We're ready to drive now, so, it is time to move on to the next section!


## Drive - TODO TODO TODO
 

