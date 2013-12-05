#Moodle for Openshift
Moodle is a well known learning management system (lms) used by many schools and organisations around the world. If you need a personal install or a platform for a small school this Openshift Quickstart might be the right thing for you. Openshift offers a 500MB instance for free which is more than enough if you just need something that works and you still have the options to scale your application if you have to. 

If you're ready to start follow this manual. 

##Current Moodle Version
To see which version of Moodle you're getting with master see the [VERSION](https://github.com/burningTyger/openshift_moodle/blob/master/VERSION) file.

###Requirements
In order to go through this manual you will need a couple of things first:

* [Ruby (1.8.7 and up)](http://ruby-lang.org)
* [Git](http://git-scm.com)

If you need help with this see the Openshift help pages: [https://openshift.redhat.com/community/get-started](https://openshift.redhat.com/community/get-started)


###Getting ready
Make sure you have an account or create a new one at [https://openshift.redhat.com/app/account/new](https://openshift.redhat.com/app/account/new)

Install the rhc gem:

    gem install rhc
    
and set up your environment:

    rhc setup

##Creating the app
To create the new app go to your console and use the rhc gem. In the following examples we're using `moodle` as our app name (you're free to pick any other name you like) and `me` as our username/domain.

    rhc app create moodle php-5.3
    
This will create a new app that will be available as https://moodle-me.rhcloud.com

During the setup process rhc spits out a lot of information. You should copy them into a document so that you can check their values at a later point, e.g. when looging in via SSH.

During the creation of your new app rhc also created a new directory called `moodle` which contains the repository of your app, i.e. it is a clone of your newly created openshift app which, at this point, is still empty.

Since it will later also conflict with the Moodle repository, I recommend to just remove all directories inside the moodle dir and later recreate it:

    rm -rf moodle        

Now that you have the app created you need to add some cartridges to it:

    rhc cartridge add cron-1.4 -a moodle
    rhc cartridge add mysql-5.1 -a moodle
    rhc cartridge add phpmyadmin-4 -a moodle
    
The last one is optional but quite useful if you need to edit your database manually and with a nice GUI.

The first cartridge lets you use cron which you will need for daily cron jobs like sending out notifications and cleaning up the database.

The second cartridge is your database.

###Getting Moodle
Now that you created your app with all the necessary cartridges you need to get the Moodle repository. 

    git clone --origin upstream git://github.com/burningTyger/openshift_moodle.git moodle 
    cd moodle
    
Since you already removed the original repository you need to add your Openshift git repository again. To find out which url you have to use you can use your previous notes or find out again by typing:

    rhc app show moodle
    
Under `Git URL:` you will find the address you need:

    git remote add origin <the Openshift git url>

After this you have everything ready to push your newly created Moodle app to Openshift:

    git push -f origin HEAD:master
    
You need the `-f` option because the new repository differs slightly from the original one created by Openshift. Further pushes should work without the `-f` option.    
    
It will take a while to upload the repository to Openshift and it takes quite some time to load the Moodle repository which is a submodule inside your Openshift repository. Once this is done you can head over to your new Moodle site: https://moodle-me.rhcloud.com (use your own domain here).

You should now be able to install Moodle in your browser and set everything up according to your needs. 

_You made it!_

###Updating Moodle

__Note__ I'm thinking of using different branches for the different versions. For now you will have to use this howto:

Every once in a while you should update/upgrade your moodle install. Be careful though, Moodle has many versions in parallel and I'll try to stick to the latest stable version available. The problem here is you need to be on the latest old stable version in order to upgrade to the latest stable version. So before you in a few weeks upgrade to version 2.5 stable which is currently a dev branch you need to upgrade to the latest 2.4 version first and then switch branches in order to upgrade to 2.5 stable.

There are two possible ways of updating your repository, one is following cloesely my updates on my repository and updating your repository accordingly:

    git pull upstream master

I will tag my updates so you can see if I switch branches to a newer version on Moodle. So usually I will tag my last commit of a branch like `last_23` which you need to pull first, upgrade Moodle by running the update script and then pulling the next commits.

The other way is updating your Moodle submodule yourself:

    cd moodle/php
    git checkout <commit_you_want>
    cd ..
    git commit -am 'updating moodle'
    git push your_os_repo HEAD:master
   
But here's the same idea, you need to make sure you're upgrading correctly. Moodle breaks easily after an incorrect upgrade. If you're unsure make use of Openshift's snapshot utility:

    rhc snapshot save -a moodle

####Known issues
When updating there might be one or the other issue. I'll add some of them here:

#####Too many redirects after an update
This might be due to a missing config load. For some reason not all action hooks were loaded and Moodle is missing some important settings. Try this:

    rhc app reload -a moodle
    rhc app start -a moodle

#####No admin pages in Moodle after an update
You have successfully pushed your update to your Openshift repository and updated the database via Moodle's update view but after that the admin pages stay blank. That seems to be a common problem and can be easily solved by running the cron page:

    https://moodle-your-domain.rhcloud.com/admin/cron.php

Obviously you have to replace the subdomain with yours :)
   
_Good luck!_   
