Today I want to introduce another tool to help you improve your  productivity: Activity Template in the Android Studio.

## 1. Introduction

![](./_image/2016-08-25 20-18-40.jpg)

This picture is familiar, right? But have you ever wondered what these templates come from when you new a project or an Activity?

Actually, they are come from the directory : "`%Android Studio Installed Directory%`\plugins\android\lib\templates\activities\EmptyActivity".    (`%Android Studio Installed Directory%` is the directory where you install the AndroidStudio.)

Let us open this directory, and take a look:

![](./_image/2016-08-25 20-20-16.jpg)

Yes, now you can find where the "Empty Activity", "Fullscreen Activity", "Basic Activity" come from ; they are all here.

We actually can take advantage of it to make our own Activity template for us, after getting familiar with the Activity Template.

Here is some developer's output:

![](https://i1.kknews.cc/large/a9f00054c5ed33d480d)

(the picture is from: [Hongyang's blog](http://blog.csdn.net/lmj623565791/article/details/51592043) )

Pretty awesome, right?  

## 2. Learning from existing template

Before we make our own template, we should get know how to make a template. And the best way to do that is to learn from the existing template, especially the easiest one. 

In all the templates that already exist in the plugin directory, `EmptyActivity` is the simplest example. Let us have a loot at it.

Open the "EmptyActivity" directory, we found the structure of this directory is this:

![](./_image/2016-08-25 20-34-23.jpg)

### 2.1. template.xml
template.xml is like the layout xml in our Android project. It tells Android Studio how to draw a UI. 

For example, Empty Activity Template looks like this:

![](./_image/2016-08-25 21-20-16.jpg)
You have three EditText for you to input the name, and two CheckBox to select.  And the template will have these five element to draw them. 

A steamlined code of template.xml is like this:
```xml
<?xml version="1.0"?>
<template description="Creates a new empty activity">

    <parameter id="activityClass" name="Activity Name"
        type="string" default="MainActivity" />

    <parameter id="generateLayout" name="Generate Layout File"
        type="boolean" default="true" />

    <parameter id="layoutName" name="Layout Name"
        type="string" constraints="layout|unique|nonempty"
        visibility="generateLayout"/>

    <parameter id="isLauncher" name="Launcher Activity"
        type="boolean" default="false" />
    
    <parameter id="packageName" name="Package name"
        type="string" constraints="package"
        />

    <thumbs>
        <thumb>template_blank_activity.png</thumb>
    </thumbs>

    <globals file="globals.xml.ftl" />
    <execute file="recipe.xml.ftl" />

</template>

```
