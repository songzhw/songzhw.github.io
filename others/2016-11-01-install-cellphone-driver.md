If you are a Mac/Linux user, you may have no such problem. But if you are a Windows user, you may find sometimes your computer cannot recognize a cellphone in AndroidStudio.  And this post will help you solve this problem

#### Step 01. 
Right click "This PC"(windows 10) or "My Computer"(Windows 7), select "properties"

#### Step 02. 
Select "Device Manager". Then you will see a phone with a yellow exclamatory mark in your phone. Just like this:
![](./_image/2016-11-01 20-16-18.jpg)
#### Step 03.
Make sure you already installed "Google USB Driver" in Android SDK.
![](./_image/2016-11-01 20-17-33.jpg)


#### Step 04. 
Right click this phone icon. Selecte "update Driver Software ..."
![](./_image/2016-11-01 20-17-10.jpg)

#### Step 05. 
![](./_image/2016-11-01 20-19-11.jpg)


#### Step 06. 
![](./_image/2016-11-01 20-19-16.jpg)


#### Step 07. 
![](./_image/2016-11-01 20-19-26.jpg)



#### Step 08. 
![](./_image/2016-11-01 20-19-37.jpg)

![](./_image/2016-11-01 20-20-36.jpg)
#### Step 09.  
select the directory :  `{android_sdk}\extras\google\usb_driver`, and install this driver

![](./_image/2016-11-01 20-21-27.jpg)

![](./_image/2016-11-01 20-21-47.jpg)


#### Step 10. Success
![](./_image/2016-11-01 20-21-57.jpg)

Now your Eclipse or Android Studio can recognize this phone.
![](./_image/2016-11-01 20-22-28.jpg)


  


