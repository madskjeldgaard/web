---
title: 'Ambisonics tutorial: Binaural head rotation using Reaper, Hedrot and IEM Plugins'
author: mads
type: post
date: 2019-08-22T08:56:36+00:00
url: /ambisonics-tutorial-binaural-head-rotation-using-reaper-hedrot-and-iem-plugins/
featured_image: /wp-content/uploads/2019/08/headphones.jpg
draft: false
tags:
  - ambisonics
  - tutorial
  - microcontroller
---
[Hedrot][1] is an inexpensive head rotator that you can build yourself and attach to any pair of head phones, based on a small microcontroller (a [Teensy][2]) with an attached sensor board that measures your head&#8217;s rotation, pitch, tilt, etc.

Using the Hedrot, you can monitor a binaural version of an ambisonic mix in a pair of headphones and be able to move your head around inside the sound field.

In this tutorial we will cover how to set up the Hedrot application to send it&#8217;s sensor data via OSC to Reaper to rotate our ambisonic mix with our head movements.

The goal is to setup IEM&#8217;s SceneRotator plugin at the end of our ambisonic signal just before an instance of IEM&#8217;s BinauralDecoder plugin. We will then connect the SceneRotator to our head tracker to move the ambisonic sound field, as we move our head.

PS. Like most things ambisonics, I was introduced to this trick by [Bálint Laczkó][3].

## Prerequisites

Before we begin, make sure you have the following software installed:

  * [Reaper][4]
  * [IEM Plugins][5]
  * [Hedrot][6]

Unfortunately, at the moment Hedrot only works on Windows and MacOS. Albeit if you feel adventurous and want to develop a Hedrot app for Linux, the source code is [available here][6] .

You will also need a pair of studio headphones (not in-ear ones) and  [a built Hedrot tracker][6] .

It is not necessary but advisable that you also read through and follow the [Basic Routing tutorial][7].

## Setting up the hedrotReceiver application

![receiver](/img/small/hedrotReceiver.png)

To get information from the head tracker you need to first connect it to your computer using a usb cable and then open up the hedrotReceiver application.

If it&#8217;s the first time you are using the headtracker, click on &#8220;Headtracker Settings&#8221;. In the new window, click on &#8220;reset all headtracker settings&#8221;

Then, press the &#8220;HEADTRACKER IS OFF&#8221; button to activate the head tracker. Make sure autodetect is on. You should now see the white box at the top of the app display a path to your headtracker.

Before setting anything up, you have to calibrate your head tracker. Click the &#8220;CALIBRATION&#8221; button on the main screen to enter the calibration settings.

### Calibration
![calibration](/img/small/calibrationScreen.png)


At the bottom of this screen, you can see how well your head tracker is performing. The normalized magnetometer and accelerometer readings in the bottom of the screen should ideally be somewhere around the center but they probably aren&#8217;t. If this is the first time you are using your head tracker, then you need to calibrate the accelerometer (only needed this one time) by clicking the &#8220;CALIBRATE ACCELEROMETER&#8221; button. Follow the steps written there and return to the calibration screen. Verify that the reading in the bottom middle of the screen is now somewhere around the middle.

Next, click the &#8220;CALIBRATE MAGNETOMETER&#8221; button to calibrate the magnetometer. Follow the steps in the next screen and return to the main calibration screen when you are done. Verify that the readings in the bottom left are now also around the middle, like your accelerometer.

Every time you attach your headrotator to a new pair of headphones, you need to recalibrate the magnetometer. The accelerometer only needs to be calibrated this one time (normally). That said, make it a happen to check the calibration every time you use the head tracker, often the magnetometer needs to be calibrated even though it hasn&#8217;t left the headphones.

### OSC settings

Now it&#8217;s time to send the tracker&#8217;s data to Reaper. Click the &#8220;OSC settings&#8221; button on the main screen of hedrotReceiver.

The first thing we need to do, is figure where we are sending the data and modify the OSC patterns accordingly in hedrot.

The target for this data will be an instance of IEM&#8217;s SceneRotator plugin which you can [control using OSC][10]. Change the yaw, pitch and roll OSC patterns to the following:

  * yaw: `/SceneRotator/yaw`
  * pitch: `/SceneRotator/pitch`
  * roll: `/SceneRotator/roll`

There is no need to change the scaling/order setting in hedrot, because – as [we can see in the IEM documentation][10] – SceneRotator expects values from -180 to 180 degrees, which is what hedrot puts out normally.

You can ignore the quaternion values in hedrot, as we won&#8217;t be using these.

_Don&#8217;t change the port number just yet._

## Setting up Reaper

As mentioned at the beginning of this tutorial, it is recommended to follow the [basic routing in Reaper tutorial][7].

If you&#8217;re impatient though, just scroll to the bottom of that tutorial and download the Reaper templates.

Open up one of the Reaper templates ( in my case I&#8217;ve opened the third order template ). Now click the `Headphones` track and open it&#8217;s fx chain. Here you will find a BinauralDecoder instance. Choose headphone equalization here, if you need it.

Add an instance of SceneRotator to the `Headphones` just before the BinauralDecoder.

### Set up SceneRotator

In your SceneRotator plugin instance, click the bottom OSC button to set up and open the network port that will receive the data from hedrotReceiver. Set it to 1234 or some other number you like. Click &#8220;open&#8221; to open this port. If you get a &#8220;Connection could not be established!&#8221; error, the port is probably used for something else. Try setting it to a different number and click open again.

_Note: It is advised to always open the port in the plugin before setting it up in hedrot, as this will block the port._

### Start sending OSC data from hedrot

Back to hedrot. Open the OSC settings again and set the port number to 1234 or whatever number you put in as the port in SceneRotator. Click the red &#8220;do not transmit&#8221; buttons to activate transmission. Verify that you see values changing on the right side of your OSC settings window.

![transmitting](/img/small/transmitting.png)


## Conclusion

That&#8217;s it actually. Now you can play ambisonic material on one of the tracks in the `Ambisonic Bus` folder track. If you don&#8217;t have any on hand, you can add the Reaper JS plugin &#8220;Pink Noise Generator&#8221; and a StereoEncoder instance to one of those tracks and play around with it&#8217;s placement.

To save yourself the pain of repeating this laborous setup process, you can save your hedrot settings as a preset file in the hedrotReceiver application. You can also save the SceneRotator&#8217;s settings to make things easier. And finally, save your Reaper Project as a new template that now includes a head rotator.

 [1]: https://abaskind.github.io/hedrot/
 [2]: https://www.pjrc.com/teensy/
 [3]: https://github.com/balintlaczko
 [4]: http://reaper.fm
 [5]: https://plugins.iem.at/
 [6]: https://github.com/abaskind/hedrot
 [7]: https://plugins.iem.at/docs/tutorial_basicrouting/
 [8]: https://www.madskjeldgaard.dk/wp-content/uploads/2019/08/hedrotReceiver.png
 [10]: https://plugins.iem.at/docs/osc/
