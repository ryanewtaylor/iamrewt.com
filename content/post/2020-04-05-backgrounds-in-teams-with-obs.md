---
title: "Custom Video Backgrounds In Microsoft Teams With OBS Studio"
date: 2020-04-05T20:55:19-04:00
categories:
  - blog
tags:
  - microsoft-teams
  - open-broadcast-studio
keywords:
  - tech
draft: true
#thumbnailImage: //example.com/image.jpg
---

Do you use Microsoft Teams for your video calls? Are you jealous of Zoom users
and Microsoft Teams insiders with custom backgrounds in their video calls? Can't
wait until Microsoft Teams releases their custom background options?

Well you're in luck. You don't need to wait until Teams adds custom backgrounds.
With free software, some green cloth, and appropriate lighting, you can add
custom video backgrounds to your Microsoft Teams calls today!

## Set Up the Green Screen

Rather than purchase a green screen I "built" my own in order to save on cost.
The materials used were:  
&nbsp;

- Green Cloth (several yards)
- Scissors
- Fabric Glue
- Paper Clamps
- One Box Spring

Using fabric glue I cut and stitched together the cloth to form a 5' x 7' sheet
of fabric. Originally I pinned the cloth to the wall opposite my desk. However
the door of my current home office is directly behind me so that is no longer an
option.

To avoid blocking the doorway I draped the sheet over a old box spring I have
yet to remove. I pulled the sheet around the edges of the box sprint to remove
as many wrinkles and folds as I could. The paper clamps were used to hold the
fabric in place.

Finally, I placed this Frankenstein like creation about 1' behind me and
standing on end so as to take up the largest field of view in my camera.

Elegant. No. Portable. No. Functional. Definitely!

## Install Open Broadcast Studio and a Virtual Web Camera

1. Download and install
   [Open Broadcaster Software (OBS) Studio](https://obsproject.com/)
1. Download and install
   [OBS-VirtualCam](https://github.com/CatxFish/obs-virtual-cam/releases)
1. Launch OBS Studio

OBS Studio should look something like this.

<!-- ![OBS Studio](/images/2020-04-05/01-launch-obs.png) -->

## Enable the Virtual Webcam

In order to export your scene as a Virtual Webcam you first need to enable it.  
&nbsp;

1. In the menu select `Tools > VirtualCam`
1. Check `AutoStart`
1. Click `Start`
1. Close the window

<!-- ![VirtualCam Settings](/images/2020-04-05/02-enable-virtual-camera.gif) -->

## Add a Background Image

Find a background image you'd like to use. Choose one close to the same
resolution as your web camera for best results. Save it to disk and and follow
the instructions below.  
&nbsp;

1. Under `Sources` click the `plus button`
1. Select `Image`
1. Accept the default name `Image` and click `OK`
1. Select `Browse`
1. Select the image you saved earlier
1. Click `OK`
1. Under `Sources` right click `Image`
1. Select `Transform` > `Stretch to Screen`

<!-- ![Add Background Image](/images/2020-04-05/03-add-background-image.gif) -->

If `Stretch to Screen` doesn't look quite right. Try some of the other
transforms. You can also manually resize the image by clicking and dragging the
image handles on the image preview itself.

## Add Your Webcam

My web camera is a Logitech HD Pro Webcam C920 and as such the instructions
below are specific to that camera. These settings are what I have found work
well enough for my use. If you have a different camera you may need to
experiment with different settings.  
&nbsp;

1. Under `Sources` click the `plus button`
1. Select `Video Capture Device`
1. Accept the default name `Video Capture Device` and click `OK`
1. Set the following properties:
   1. Device: Logitech HD Pro Webcam C920
   1. Resolution/FPS Type: Custom
   1. Resolution: 1920 x 1080
   1. FPS: Match Output FPS
1. Click OK

<!-- ![Add Background Image](/images/2020-04-05/04-add-web-camera.mp4) -->

## Configure Chroma Key (Green Screen) Filter

Here's where the magic happens. Using a Chroma Key filter we replace the green
screen with the background of your choice.  
&nbsp;

1. Under `Sources` right click `Video Capture Device`
1. Select `Filters`
1. Under `Effect Filters` click the `plus button`
1. Select `Chroma Key`
1. Accept the default name `Chroma Key` and click `OK`
1. Move the `Filters for 'Video Capture Device` window to view the preview
1. For `Key Color Type` select `Green` if not already selected
1. Adjust `Similarity` only you and your background are visible

The quality of the Chroma Key filtering is very dependent on how well your
environment is illuminated. I have found that illuminating directly from the
front -- that is light should be shining on you and the green screen -- greatly
improves the quality of the filtering and scene in general.

## Configure Microsoft Teams Video Source

Almost there. One last step!  
&nbsp;

1. Keep OBS Studio Open
1. Open Microsoft Teams
1. Select `Profile` (your avatar)
1. Select `Settings`
1. Select `Devices`
1. Under `Camera` select `OBS-Camera`
1. Close `Settings`

That's it. Done! Microsoft Teams will now use your OBS Studio Virtual Camera as
its video source for your video calls. Should Teams revert to your previous
configuration - such as when you close OBS Studio - repeat these steps with OBS
Studio open to re-establish the virtual web camera.

Enjoy!
