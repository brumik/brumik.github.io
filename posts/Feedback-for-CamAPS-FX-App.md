---
title: Feedback for CamAPS FX
date: 2024-05-12
permalink: /blog/2024/05/12/feedback-for-camaps-fx
tags: feedback application app diabetes 
---

CamAPS FX is an application for people with Diabetes who are using a pump and a sensor to manage their blood sugar levels. If you are not familiar with it, there are some screenshots on the [Google Play page](https://play.google.com/store/apps/details?id=com.camdiab.fx_alert.mgdl).

Using the CamAPS FX application day to day basis to manage my closed (hybrid) loop for diabetes allowed me to notice so if it's shortcomings and be able to provide feedback from a user perspective.

## State

Currently the application is really stable and reliable. The communication with the sensors are fast, and connecting to the pump is also relatively fast (usually under 5s). It is not overusing the phone's resources, for which battery life is thankful.

The application has all the functions it needs, as long as you are able to find them and figure out which one does what. 

## Day to day Usage

When the most comfort (or discomfort) comes in an application is how it is intended/used on a day to day basis. In an application like this, we need to interact with it quite a few times and these times are better be efficient. The situations where you check and use it are often social, just before a food or while drinking beer. It is the most rare situation where you sit back on your couch, open up you phone and comfortably look at your CamAPS FX. This is not leisure time.

Currently what I use and it is recommended to use on a day to day basis are:
1. Go to the bolus calculator and set how much food you will eat, give direct insulin.
2. Add a snack when you are not sure how much you will eat or it is just a small bite (often used when drinking).
3. Add long lasting food (like meat or pasta) on top of the bolus (used together often with the first option).
4. Lastly you wanna be able to see the trend in your blood sugar (but since it is a closed loop and you have notifications when something bad is happening, this is the least of your interactions)

Let's look at how in CamAPS you can do the things (after you had your app opened):
1. Bolus:
	- Click on the middle tab on the top of the screen
	- Wait for the app to connect to the pump
	- Select (quick select or type) the amount of food
	- Click next
	- Click next to confirm insulin size
	- Wait 5s
	- Wait for the insulin to be delivered
	- **Summary**: You need 4 clicks and you have minimum 10s time when you wait for things to happen
	- **Optimization**:
		- Food size available from home screen which takes you directly to confirm insulin
		- No wait time until it connects to a pump. If cannot connect to the pump we can still get a notification about that something went wrong
		- Let the button be on the bottom of the screen to be easily reachable
2. Snack
	- Swipe right (which is hard on new androids) or click the top left hamburger menu 
	- Press the top (first) option "Add meal"
	- Click into the invite field (not focused by default)
	- Type in the size in grams (no quick select)
	- Click confirm
	- **Summary**: You need 4 clicks.
	- **Optimization**: Move the button to the home screen to be reachable easier.
3. Long lasting food
	-  Swipe right (which is hard on new androids) or click the top left hamburger menu 
	- Press the top (first) option "Add meal"
	- Click into the invite field (not focused by default)
	- Type in the size in grams (no quick select)
	- Click on the "Slowly absorbed meal" radio button
	- Click confirm
	- **Summary**: 5 clicks. 
	- **Optimization**: Allow the specification of slowly absorbed meal when adding bolus since this is hardly used standalone
4. Seeing your blood sugar is the default, good points here. 

### Some ideas how the app could be aligned

Considerations:
- Least clicks when doing day to day operations on the phone
- No unused space (nobody turns off Auto mode and it takes up 1/6th of the screen)
- The "Add meal" and "Bolus calculator/add insulin" should be one thing
- Add to the home screen the select food portion buttons/slider

## Abusing of notifications

The CamAPS FX has, as every other app on which your life is actually dependent a notification system where it can send you notifications even when your phone is mute or you set the `do not disturb` mode on. This is great, since I may not want random calls and messages to disturb me in a meeting, I may be happy if the phone "speaks up" if I am going to have near death experience. 

This feature is working somewhat awkwardly in CampAPS FX. As long as I was able to figure out how it works:
- If it is in `mute` you still get notifications for `soon low` and `urgent low` 
- If the phone is do not disturb you only get the `urgent low` notifications

Then there is the "announcement" system that they have and which they are abusing. This system is using a in app notification that does override `mute` (could not test do not disturb). These notifications are in most cases completely useless for me as an user and could be a mail.

Such notifications are:
- We updated the documentation
- "US Only" we fixed an outage (multiple times within few days)
- Do not update your android (this is funny since the supported phones comes with newer android that they actually support, but it works anyways)

Understandably these notifications that ring on the loudest settings as if it was life threatening even on mute is an abuse of the customer trust. When I allow the application to be loud, no matter what I expect the app to take into consideration when to use this privilege. Announcing news, even local ones in a world wide loud notification is not the way to go.

## Changing sensors is confusing

The CamAPS FX supports multiple sensors like Dexcom or Libre. I understand that their app is not the intended experience how the user should set up these devices, but the complete lack of UX design is a bit too much.

**Good things first**: For Dexcom I like the fact that once you figured out how it works you can set up a new sensor in just 3 clicks, while in the official application it takes you through all the instructions with infinite scrolling and way more clicks while denying that you do not want to use the camera to enter 4 digit number. In CamAPS you click the "start sensor" button, select the text input, type the number and press continue. That's all, fast, accessible from the home screen, no pain at all. 

**Could be improved**: Changing sensor Bluetooth receiver is completely different experience. This is the "brain" of the sensor and needs to be changed once every 3 months. The place to change is:
- Open the side menu
- Click on the "Dexcom G6" which looks the same as all the following non clickable entries
- Select the CGM device (Libre or Dexcom) like you setting it up the first time
- Enter the number of the sensor
- Do not forget to actively watch your phone for the next 4 minutes for a android system notification to pair with a new device.

The design is so confusing that using the app for nearly a year now I am still in doubt when I need to change the receiver. This part could be a bit more explicit communicating to the user what is a button and what you need to press when needed. If you already have a cluttered side menu I am sure one more button fits there.

## Summary

I could not find any person to give feedback on the application. To record my frustration and to give my voice a chance to reach anybody I am publishing this article. In general, the app works as expected, only that these expectations are not really high.
