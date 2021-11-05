# balena-DonkeyCar-Virtual

A virtual build of a DonkeyCar on a RaspberryPi 4, containerized for balena. No physical hardware needed, just connect to a hosted Simulator.

## Intro

This repo installs the popular DonkeyCar autonomous vehicle / self-driving bot platform into a container, ready to be deployed on a balena device like a Raspberry Pi.

There are several components to cover:
- Hardware
- Software
- Using the Container

Also keep in mind that this Readme only covers the Balena-specific bits, but the proper DonkeyCar documentation covers many more topics, and in much greater detail.

[![balena deploy button](https://www.balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/dtischler/balena-DonkeyCar-Virtual)

Once the contianer is deployed on to your Pi, browse to http://<ip-address-of-your-pi>/drive.

The DonkeyCar can be moved by using the mouse in the contol box. Click and hold, then dragging the mouse up for forward, back for reverse, and left and right to steer.

Alternatively, on the keyboard, I,K,J,L will move forward, backward, left, and right. Individual keystrokes will nudge the value up or down respectively.

Once you get the hang of driving, you can start recording your car's inputs and movements, in order to capture data that will be used in the creation of an AI model. Once that model is built from your source data, your virtual DonkeyCar can attempt to drive itself, without needing your input!

You can watch your car moving around the virtual track, at http://twictch.tv/roboticists 

## TO-DO - Documentation
## Hardware
## Software
## Using the Container
