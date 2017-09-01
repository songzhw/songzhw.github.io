If you are a Mac/Linux user, you may have no such problem. But if you are a Windows user, you may find sometimes your computer cannot recognize a cellphone in AndroidStudio.  And this post will help you solve this problem

#### Step 01. 
Right click "This PC"(windows 10) or "My Computer"(Windows 7), select "properties"

#### Step 02. 
Select "Device Manager". Then you will see a phone with a yellow exclamatory mark in your phone. Just like this:

![](./_image/2017-08-31-20-22-57.jpg)

#### Step 03.
Make sure you already installed "Google USB Driver" in Android SDK.

![](./_image/2017-08-31-20-26-42.jpg)

#### Step 04. 
Right click this phone icon. Selecte "update Driver Software ..."

![](./_image/2017-08-31-20-27-02.jpg)


#### Step 05. 

![](./_image/2017-08-31-20-27-42.jpg)

#### Step 06. 

![](./_image/2017-08-31-20-27-58.jpg)


#### Step 07. 

![](./_image/2017-08-31-20-28-17.jpg)


#### Step 08. 

![](./_image/2017-08-31-20-28-30.jpg)


![](./_image/2017-08-31-20-28-42.jpg)


#### Step 09.  
select the directory :  `{android_sdk}\extras\google\usb_driver`, and install this driver


![](./_image/2017-08-31-20-28-59.jpg)





#### Step 10. Success


![](./_image/2017-08-31-20-29-37.jpg)


Now your Eclipse or Android Studio can recognize this phone.

Eclipse : 
![](./_image/2017-08-31-20-29-47.jpg)

Android Studio:

![](./_image/2017-08-31-20-33-37.jpg)





  


