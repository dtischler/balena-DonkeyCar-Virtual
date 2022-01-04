# Welcome to DonkeyCar on balena!  

A [DonkeyCar](https://www.donkeycar.com) is a small autonomous vehicle project built on an RC Car chassis, that uses computer vision and machine learning techniques to convert the platform into a self-driving vehicle.  It uses a Raspberry Pi 3 or Jetson Nano, a camera, a motor driver, and of course the DonkeyCar application to perform this task.  There is also a "virtual" version that you can use to get started, prior to even building the physical version.  A "virtual" DonkeyCar looks rather like a video game, and it connects to a DonkeyCar Simulator.  Just like the physical version, the virtual Donkey can attempt to drive itself, in the virtual world.

![](https://docs.donkeycar.com/assets/sim_screen_shot.png)

To cover these different scenarios there are 4 main Repositories for the project:

 - [DonkeyCar - Physical Build - Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Physical)
 - [DonkeyCar - Physical Build - UpBoard](https://github.com/dtischler/balena-DonkeyCar-Physical-UpBoard)
 - [DonkeyCar - Virtual Build, Raspberry Pi](https://github.com/dtischler/balena-DonkeyCar-Virtual) 
 - [DonkeyCar Simulator - Intel NUC / x86](https://github.com/dtischler/balena-DonkeyCar-Simulator)

This Readme and Repo cover the **`Virtual Build of a DonkeyCar using a Raspberry Pi 3`** version of the project.  A virtual DonkeyCar needs to connect to a DonkeySim Server, so, this repo is intended to be used in conjunction with the [DonkeyCar Simulator - Intel NUC / x86](https://github.com/dtischler/balena-DonkeyCar-Simulator) repo, or you can connect to any other hosted Donkey Simulator (the DIY Robocars group hosts virtual racers on a regular basis, you could simply connect to their Simulator when it's running).  You can also refer to the other GitHub repos linked above, for other components or flavors of DonkeyCar.

Also note, the DonkeyCar project has their own [detailed documentation available](https://docs.donkeycar.com), that goes into more advanced topics and gives thorough descriptions of the architecture, machine learning and training scenarios, performance and tuning of models, and many more topics.  This Readme is simply intended to make it easy to get started, and once you have the core concepts down, be sure to refer to their documentation for more advanced guidance. 

## Intro

Waymo, Tesla, Cruise, and other companies already have self-driving vehicles deployed out in the world around us.  With this project, it is possible to build a miniature version of an autonomous vehicle, intended to race around a track, which is perfect for learning how the core concepts of computer vision and machine learning play a role in self-driving vehicles!  Keep in mind this Repo is specifically devoted to the "virtual" DonkeyCar, which does not involve any physical construction and does **NOT** require an RC Car Chassis, but will require a secondary DonkeySim Server.  We are going to run the DonkeyCar software stack on the Raspberry Pi, which will then connect up to a DonkeyCar Simulator / Server that is running what ultimately looks like a video game world.  However, our virtual DonkeyCar can be trained and tested just like a "regular" DonkeyCar, and can ultimately attempt to drive itself around the virtual world.  If you are looking to build a "real" DonkeyCar, simply check out [this repo](https://github.com/dtischler/balena-DonkeyCar-Physical) to get started.

The only items required to run DonkeyCar in this repo are a Raspberry Pi 3 or 4, Power Supply, and SD Card (a high quality, fast one is best).  A DonkeySim Server also needs to be running, to connect to.

Before we begin, let's also cover a few basics.  The virtual car can be driven through a web browser, using the keyboard, a Bluetooth gamepad, or even via the accelerometer on your cell phone.  While driving, the DonkeyCar will record data about the car's virtual throttle position, steering position, and what it "sees" in the virtual camera as you make way around the racetrack.  When you are finished recording, all of the data is collected into a folder called a "Tub".  This metadata is then used as the input to create or "train" an AI model.  Once complete, the output of that process (the resulting *model* file) can then be used by the DonkeyCar to attempt to drive itself.  The Raspberry Pi loads the model, and attempts to navigate and move around the virtual racetrack on its own!

The process in the middle - the Training - takes a very long time on a Raspberry Pi.  Analyzing all of the data that got recorded during driving, creating a neural network, and iterating on that data over and over through what are called epochs can easily take hours on the Pi.  And once complete, there is no guarantee the DonkeyCar will even be able to drive safely and accurately!  So, to improve that feedback loop and to reduce the iteration time, most people find it better to drive the DonkeyCar and collect the data, but then transfer that resulting "Tub" of data to a cloud server or a PC with a GPU, and perform the training there.  Once it completes (hopefully much quicker!), the resulting output of the process is the *model* file just like mentioned above, and that model can be transferred back to the Pi.  Then, the DonkeyCar can try to drive using it (hopefully it drives good!).

## Build

As there is no physical construction of the DonkeyCar in this Repo, we can move right into deploying the software stack onto the Raspberry Pi.  This is where we diverge a bit from the official DonkeyCar documentation and process, and "balenafy" the project:

 - In the official DonkeyCar workflow, they begin to walk you through installation of Raspbian, installation of added software and packages such as Tensorflow, PyTorch, OpenCV, Keras, and the rest of the bits necessary for the machine learning functionality.  Then, they have the user install Python and the DonkeyCar application.  At the end of the process, if everything worked, the web interface should load and the car can be driven remotely.
 - However, in this repo, we have bundled all of those bits into a Dockerfile, and instead of performing all of those manual installation steps, you can instead just click this button to launch a build in the cloud, provision a balena device, download an SD Card image, and end up with the same result:

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/dtischler/balena-DonkeyCar-Virtual)

More specifically:
1. Click on the Blue Button just above.
2. Log in to balenaCloud, or create an account if you do not already have one.  (It is free :-) )
3. Create a name for the application in the pop-up modal, and choose the RaspberryPi 3 (or Pi 4 if that's what you are using) from the drop down list.
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


## DonkeyCar Simulator
 
As noted, an active Donkey Simulator is needed for the Virtual DonkeyCar to connect to.  If no public ones are running (about a week before DIY Robocar events, you may find one running at "donkey-sim.roboticist.dev"), then you'll need to build your own, which can be done very easily with this repo:  https://github.com/dtischler/balena-DonkeyCar-Simulator

Head over there, get your Simulator built, then return here once you're ready to move on.  Take note of the IP Address of your finished Simulator, we'll need it in a few steps.

## Drive

With your Simulator up and running, it's time to go for a virtual drive!  In balenaCloud, on the Device Details page, open up an SSH session to the DonkeyCar container with the Terminal interface at the bottom right portion of the screen:

![](/images/img1.png)

Type in `nano mycar/myconfig.py` and press Enter.  You should see an editor, similar to this:

![](/images/img2.png)

Scroll all the way to the bottom, and look for the line that defines the `SIM_HOST`.  Change the value to the IP Address of your Simulator.  As example, mine is `192.168.0.196`, so mine ends up looking like:

![](/images/img3.png)

Press `Control-S` to save the file, then `Control-X` to exit the editor and return to the terminal.

Next, it's time to start the DonkeyCar up.  Type in `cd mycar && python3 manage.py drive` and press Enter. The script will launch, and take a moment to complete, but will eventually reach `Starting vehicle at 20 Hz`. 

![](/images/img4.png)

![](/images/img5.png)

Check the IP address of your Raspberry Pi in the balenaCloud dashboard. Make note of this IP. Open a web browser, and go to `http://ip-address-of-your-pi/drive`. In my example, this would be `http://192.168.0.82/drive`.

![](/images/img6.png)

In the throttle and steering applet on that page, click and drag just a tiny bit up from center, and your virtual car should start to move!  Alternatively, you can use the `I, K, J, and L` keys on the keyboard to nudge the car forward, backward, left, and right.  Remember, this is the exact same software stack as the real DonkeyCar, so if you had built a physical racer, it would start driving as well.  As mentioned earlier, the official DonkeyCar documentation is very thorough, so you can refer to those Docs for more detailed usage. But keep in mind these important details:

- DonkeyCar is meant to be used on a racetrack, and will attempt to learn your driving technique.
- Using the web portal for driving (use a Bluetooth controller for a much better experience than driving via keyboard or phone), drive a few practice laps around the track. When you have a good feel for driving and are ready to capture data, click the "Start Recording" button, and at this point DonkeyCar will begin storing the throttle, steering, and camera feed for later use in the Training process.
- Drive at least 10 laps around the track, preferably more, while Recording.
- Once you have completeled 10 laps (or more), you can stop the Recording.
- Over in the balenaCloud Dashboard, in that Terminal window, press `Control-C` on the keyboard to exit out of the DonkeyCar application.
- You will see all of the data get written out in a table, and the raw files are stored in the `data` directory inside of that `mycar` folder.

![](/images/img7.png)

## Train

Now that we have a bit of sample data recorded and saved, it's time to begin training our model. Remember, as mentioned above, it is NOT very efficient to train directly on the Raspberry Pi, and using a cloud server or a desktop PC with a GPU will be MUCH faster. However, simply for learning purposes and to keep things organized and in one place, we will in this situation train directly on the Pi. It could literally take 8 to 10 hours or more, so, grab a cup of tea, and sip it VERY slowly. Or do something else in the meantime.

Back in the terminal session in balenaCloud, and still within the DonkeyCar container, run `donkey train --tub ./data --model ./models/myawesomepilot.h5`. Now go do something else.

![](/images/img8.png)

Fast forward 10 or so hours, and returning to balenaCloud, you should see that process has completed. The output of all that hard work is the model file. Double check that everything completed successfully, and you should have a file sitting in the `models` directory called `myawesomepilot.h5`

![](/images/img9.png)

### Speeding Up Training

Knowing full well that training on the Pi is not ideal, we simply want to demonstrate functionality in this GitHub repo. If you are interested in offloading the Tub data and training on a PC or Cloud server, have a look at the official DonkeyCar docs here: https://docs.donkeycar.com/guide/train_autopilot/#transfer-data-from-your-car-to-your-computer. That will help immensely. :-)

## Drive Autonomously

With the model now ready (hope you slept well), you can try to let the DonkeyCar now navigate the virtual racetrack autonomously. Fair warning, mine did not drive very well with only 10 laps of data, so, be ready to watch it crash!

- In balenaCloud dashboard, open up a terminal session to the DonkeyCar container.
- Enter `cd mycar && python3 manage.py drive --model models/myawesomepilot.h5`
- Navigate to `http://ip-address-of-your-pi/drive` once again.
- On the left, click the dropdown menu for Mode and Pilot, and choose Local Pilot.
- The DonkeyCar should begin to make it's way around the track (hopefully).

![](/images/img10.png)

## Conclusions

This repo is intended to demonstrate the containerization of DonkeyCar, and help you to quickly deploy the software stack that they are gracious enough to build and maintain. Rather than going through the physical construction of a vehicle, we instead built a virtual clone using the exact same software that the real one would use.  The training process on the Raspberry Pi is not ideal, but there are methods to help accelerate the process by offloading the data.  As you get better at driving and training your model, be sure to also check out and compete in the online races the community hosts: https://www.meetup.com/DIYRobocars/


