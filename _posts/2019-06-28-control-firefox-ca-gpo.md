---
layout: post
title:  Configuring Firefox to use the Windows Certificate Store via GPO
date:   2019-07-11 8:30:30 -0600
image:  05.jpg
tags:   [sys-admin]
---

Let's say that you were deploying a Certificate that needed to be trusted in your 1000+ endpoint domain. You deploy it through conventional means, and everything works in IE, Edge, Chrome, even Safari and Opera! But Firefox? Nope, although Firefox does come with the setting to trust the Windows Certificate Store it doesn't come enabled by default. 

To enable this setting the **security.enterprise_roots.enabled** must be set to **true**. To enable this feature on a single computer: 

* In Firefox, type 'about:config' in the address bar  
* If prompted, accept any warnings  
* Right-click to create a new boolean value, and enter 'security.enterprise_roots.enabled' as the Name  
* Set the value to 'true'  

So now we just have to do these steps for the 1000+ computers in your network, and oh btw tons of people work from home on certain days. Good luck!

## Locking Firefox Preferences with files
To enable this feature on multiple computers you will need to use another method to lock the preferences in Firefox. The benefit is that once enabled you can easily manage certificates using group policy in the future. 
 
You can use a preferences file to configure the security.enterprise_roots.enabled setting. To do so use the attached files:
* The 'umbrella.cfg' file must be placed in the root of the Firefox directory. For example: **C:\Program Files\Mozilla Firefox\umbrella.cfg**
* The 'local-settings.js' file must be placed in the \defaults\pref sub-directory. For example: **C:\Program Files\Mozilla Firefox\defaults\pref\local-settings.js**  
The contents of local-settings.js should be as follows:
{% highlight console %}
 pref("general.config.obscure_value", 0); 
 pref("general.config.filename", "windowscerts.cfg"); {% endhighlight %}
 
The contents of the windowscerts.cfg file should be as follows:
{% highlight console %}
 //
 lockPref("security.enterprise_roots.enabled", true);
{% endhighlight %}

**NOTE:  If creating the above files manually, they must be ANSI encoded.**

## Enter GPO  
Group policy can be used to distribute the above files.  Note, this process requires that Firefox is installed to the default location on the client computers.

* Add the files 'windowsstore.cfg' and and 'local-settings.js' to a network share.  Ensure that the share has read permissions for 'Domain Computers'
* Create/Edit a group policy in Group Policy Management
* Edit the settings in 'Computer Configuration > Preferences > Windows Settings > Files'
* Right-click and select 'New File'
* Point the 'Source File' to windowsstore.cfg on the Network Share
* Point the 'Destination' file to be C:\Program Files\Mozilla Firefox\umbrella.cfg and 'Apply'
* Repeat the above step to copy the same file to C:\Program Files (x86)\Mozilla Firefox\windowsstore.cfg
* Repeat these steps to copy 'local-settings.js' to C:\Program Files\Mozilla Firefox\defaults\pref\local-settings.js
* Repeat these steps to copy 'local-settings.js' to C:\Program Files (x86)\Mozilla Firefox\defaults\pref\local-settings.js

## Other methods
Firefox relies on those two files to set the preferences. So any way you can disseminate those files to the correct location would be fine. Whether it's an RMM, GPO, Script or if you place it on your golden image. 

Until next time! - Kevin

[windowsstore.cfg](){:target="_blank"}  
[local-settings.js](){:target="_blank"}  
