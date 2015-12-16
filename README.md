# Alexa-yun-pi-garagedoor
This is a project that will allow you to control an arduino yun with Alexa.  In this project I am controlling a garage door.

    Noisey Youtube video of it in action.  https://www.youtube.com/watch?v=zzA2KRIv-9Q


Physical things that are needed to duplicate this project:
* Amazon Echo
* Raspberry pi (I am running rasbian "jessie" on mine)
* Arduino Yun
* relay

You will also need a Nearbus account (It is easy to sign up and free)

Ok... So how does this work?
Alexa hears a command to "turn on the garage door".  The Echo sends this command to the Raspberry pi that is running an emulation program for Hue.  
The raspberry pi then fires of a url to Nearbus that turns on the Yun's Digital Pin 3 closing the relay that allows the garage door to open/close.

Not so bad.
So here are the steps.

1.  Go here and set up the Raspberry Pi. 
    https://github.com/armzilla/amazon-echo-ha-bridge 

    I found a good walkthrough on how to set up a Pi with this software:
    www.airedalez.net/?p=263
    (the walkthrough is for an old version current version is 0.2.1 So just a heads up)

    Once this is on the Pi and the server is running on it.(Takes a couple minutes to get it going…)You can then access a gui             on it by logging into it by IP from another computer on the lan.

    From there, you can place the custom Url commands for the pi to pick up from Alexa and do stuff.
    
    THIS FIRST STEP IS ESSENTIAL AND SHOULD NOT PROCEED FURTHER UNTIL THIS HAS BEEN COMPLETED!!!!

2.  Next you will need a Nearbus account.
    Go here and set up an account.
    http://nearbus.net/index.php
    Click on Device List and select the HELLO WORLD example or go here:
    http://nearbus.net/wiki/index.php?title=App_Note_1218_-_Hello_World

3.  This is where you will want to take out your arduino yun.
    If this is your first time working with arduino you will want to load the IDE on your PC.
    for that go here. 
    http://arduino.cc/en/Main/Software
    follow the Hello World example (http://nearbus.net/wiki/index.php?title=App_Note_1218_-_Hello_World)
    NOTE in the Hello World example you will download the nearbus lib and I could not get it to compile in the Arduino
    this is because the IDE did not like WProgram.h.  You will need to open the NearbusYun_v15.h file with an editor
    (I use Notepad++)  If you use Notepad++ you can go to Search and select Replace.  Then enter WProgram.h in the "Find                  What" box and then Type Arduino.h in the "Replace with" box, and select replace and save.  Close out the Arduino IDE if      you      have it open, and reopen the program again and it should now compile.

4.  So if you made it this far you have a Raspberry pi that has sort of been sitting on the sideline for now, and an Arduino     Yun      that has the Hello World example sketch running on it.
    
    We need to know that nearbus can see the Yun.  Log into nearbus.net and select DEVICE LIST, and verify that the State is              UP/Green.  If it is you are doing great.  If not, go back and try again.
    You will notice that when you are seeing your device on the nearbus site that there is a drop down menu below it.  Using     the      drop down menu select "EDIT DEVICE".  
    The window that opens up will need a small change.  UNCHECK CONFIGURED AS VMCU!  This will put it in TRNSP mode.  It will     look     Red in the Configure tab.  Do not worry about that.

5.  So now is the time to hook the YUN back up to the PC and open the arduino IDE for one more go. This is where you will        want     to open my sketch and modify your information into it and load on to the YUN.  
    you will want to modify here:
    
    * line 29 char deviceId []    = "Nearbus DEVICE ID"  change to your device id that nearbus has assigned. put your device ID                                          between the ""
    * line 30 char sharedSecret[] = "Shared Secret"  This can be found in the Edit Devices tab on nearbus.  This is not your                                      PIN.  It is the 8 digit number in the Shared Secret space. 
    * line 206 client.get ("http://nearbus.net/v1/api_transp/NB101000?user=nearbus_logon_name&pass=nearbus_logon_password&method=POST&reg00=0");
    * You will want to place your nearbus device ID in place of NB101000, and put in your nearbus                                           login name and password.

    Load sketch to YUN, and connect it to the internet.  Verify that device is in the UP state on the nearbus site.
    at this point open a window in your browser and enter this, but modified with your information 
    
    "http://nearbus.net/v1/api_transp/NB101000?user=nearbus_logon_name&pass=nearbus_logon_password&method=POST&reg00=1"
    
    notice the the last digit has been changed to a 1 make sure you do not forget that.
    
    Hit enter.  you should see the relay close for one second and then open.  Everytime this url is entered the relay should          respond by closing for one second and opening.

6.  So if that URL is closing the relay, it is time to put that url into the raspberry pi.  log into the raspberry pi and        create a switch named "garage door" for the ON url place:

    "http://nearbus.net/v1/api_transp/NB101000?user=nearbus_logon_name&pass=nearbus_logon_password&method=POST&reg00=1"
   
 notice the the last digit has been changed to a 1 make sure you do not forget that.

    for the Off url you can place the same url.  Save and load. 
    
7.  Now you should be able to tell Alexa to turn ON/OFF open/close the garage door, and this should cause the relay to close for one second and open up.  wire the relay into your garage door button/doorbell/momentary switch.

8.  enjoy!  


*When I started with this I had two different urls being sent to the YUN.  one was for open and the other was for close.  This turned problematic because we do not use the echo to open/close the garage all the time.  So there would be times that in order to get the garage to open, I would have to say the close command.  to fix this, I modified the sketch so that when it recieves word from Nearbus to set pin 3 for the relay. it in turn sends the off url for pin 3 so that it will be ready to go again when it hears from alexa.

This was a great way to work with two technologies I don't know much about.  Raspberry pi, and Arduino.  This project could be modified to do many different things.*

*So the next step is to figure out a way to have alexa check the status of the door.  It would be nice to be able to ask if the door is closed as of right now, I am not sure how this could be added.  I think it would have to come from the Arduino side of things, a sensor would be easy to add, but after that, I don't know.

Update 12/16/15
I am currantly working on a garage status check.  What I have done is add another arduino with an ethernet sheild.  I have this new board set up in my house near the amazon echo.  I added another sudo switch to the raspberry pi called "garage status" the url it sends sets the register rg001 to high. The url looks like this:
http://nearbus.net/v1/api_transp/NB101000?user=nearbus_logon_name&pass=nearbus_logon_password&method=POST&reg00=0&reg01=1

So when the Yun gets the message that rg01 is HIGH, it does a digital.Read on a sensor I placed on my garage door.  Once the Yun knows the state of the door, it sends off a url to the new arduino with the ethernet shield that has a new nearbus device ID that sets either rg00 or rg01 to HIGH.  The Yun after a small delay sends the reset url so that the user can ask alexa what the status is again.  The Yun also sends a reset url to the new arduino board aswell so it can be ready for the next time.  

Based on what the new arduino recieves determines what happens next for it. In this case I have an adafruit sound board that plays a custom message stating that the door is Closed, or Open.  

This is still far from being polished and the ideal solution would be a push notification back to the amazon echo to relay the status, but at the time of this writing that is not an option that I am aware of.

If this proves to work well, I will add my updated Yun sketch, with the addition of the new arduino sketch aswell.

B

    
    
                                  
    
    
    
