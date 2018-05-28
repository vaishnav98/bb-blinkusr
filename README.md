# Install BoneScript on Snappy Ubuntu Core(and generating BoneScript Snaps)


## Introduction 

This is a tutorial to get started with BoneScript on BeagleBone Black running Snappy Ubuntu Core, these same methods can be used for creating
snaps for other ubuntu installations.The example snap was tested on these images:
```
http://people.canonical.com/~ogra/snappy/all-snaps/daily-stable/current/  (Snappy-Core Image)
```
```
https://rcn-ee.net/rootfs/bb.org/testing/2018-05-18/bionic-ros-iot/  (Ubuntu 18.04 ROS Image)
```
The example snap can be installed by running,
```
snap install --edge bb-blinkusr --devmode
snap run bb-blinkusr(for running the blink-script)
```


## Installing BoneScript on Snappy Ubuntu Core 

### Installing Snappy Ubuntu Core on BeagleBone Black

Use Etcher or DD(or similar tools) to flash the above Snappy-Core Image onto a micro-SD card and boot the board from the SD Card.

Please Note that for proceeding with setting up the account you will need to have access to a HDMI Monitor(+Keyboard) connected
to the board(or a Serial Cable), SSH does not work out of the box.After setting up follow the instructions below.

* Create an Ubuntu SSO Account [here](https://login.ubuntu.com/) 
* Generate a Public SSH Key (using ssh-keygen or PuTTYGen) and paste it [here](https://login.ubuntu.com/ssh-keys) 
  (this needs to be done from the device from which you will be ssh logging in to the beaglebone).
* Continue with the configuration steps on the screen,you will be asked to configure the network and enter your ubuntu
  sso account credentials, finally the credentials for ssh login to the device will be displayed.
  
  ``` 
  ssh <Ubuntu SSO user name>@<device IP address>  
  ```
* For making the traditional package management tools like apt and dpkg available in the snap environment, you will 
  have to enable the classic mode by running 
  
  ```
  sudo snap install classic --devmode --edge
  
  sudo classic  (to enter classic mode)   
  ```
 * After entering the classic mode NodeJS , NPM and BoneScript can be installed on the system in the usual way, this is only
   neccessary if you are planning to do BoneScript Development on the System itself, for running snaps which have bonescript
   dependencies it is not required to have BoneScript(or NodeJS) installed on the system becuase all the dependencies required are 
   packaged in the .snap file.
   
 Also note that ,not all of the BoneScript functions are supported in Snappy-Core so it is recommended to try running the Scripts
 before creating a snap(currently PWM and Analog Inputs are not supported in Snappy Core). 
  
### Creating Snaps for BoneScript Projects

For creating Snaps for your BoneScript Projects, it is required to have a package.json file in your project directory,if it 
does not exist you can generate it by running
```
npm init
```
An example package.json file is shown here :
```
{
  "name": "BBUsrLedBlink",
  "version": "1.0.0",
  "preferGlobal": true,
  "description": "Blink BeagleBone User Leds",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Vaishnav M A <mavaishnav007@gmail.com>",
  "license": "MIT",
  "bin" : {
   "blink-usr" : "./index.js"
  },
"dependencies": {
        "bonescript" :"vaishnav98/bonescript#snappy-bonescript"
    }
}
```
Please note that #snappy-bonesccript branch has been listed as a dependency instead of 'bonescript' since there are some changes in 
the snappy-bonescript branch which cannot be merged to the master branch.

index.js contains the code required for blinking the user leds, you can also add multiple entries in "bin" object if required,
keep a note of the command entries in bin object (here 'blink-usr') as it is required when making the snapcraft initialisation.

Source Code : https://github.com/vaishnav98/blink-usr-snappy-bonescript

#### Creating Snaps

The first step for creating snaps is to generate the snapcraft.yaml file , Install SnapCraft by running,

```
sudo apt-get update
sudo apt-get install snapcraft
```
Then make a new project directory and run 
```
snapcraft init
```
on the project directory , this generates a snapcraft.yaml template inside the snap directory, edit the template according to the project details.A sample 
snapcraft.yaml file is shown here.

```
name: bb-blinkusr # you probably want to 'snapcraft register <name>'
version: '0.1' # just for humans, typically '1.2+git' or '1.3.2'
summary: BoneScript Snap to Blink BeagleBone USR Leds # 79 char long summary
description: |
  Blinks all User LEDs Continously for 30s.
grade: devel # must be 'stable' to release into candidate/stable channels
confinement: devmode # use 'strict' once you have the right plugs and slots

parts:
  bb-blinkusr:
    # See 'snapcraft plugins'
    plugin: nodejs
    node-engine: 4.2.6
    source: https://github.com/vaishnav98/blink-usr-snappy-bonescript.git

apps:
  bb-blinkusr:
     command: bin/blink-usr
```

* The parts section defines how the snap is going to be build , for bonescript projects the plugin should be 'nodejs'
  and node-engine specifies the node version on which the snap is going to be build.
  Source points to the source code for the project it can be a Github Repo or a local path.
  
*The apps section consist of the commands and services that are exposed to the user,If the command name matches the snap-name
 user can run the app directly using the snap-name , i.e in this case the app can be run by 
 ```
 snap run bb-blinkusr
 ```
 whereas in cases like this 
 ```
 apps:
  bb-blinkusr1:
     command: bin/blink-usr2
  bb-blinkusr2:
     command: bin/blink-usr2 
 ```
 The corresponding apps can be run using 
 ```
  snap run bb-blinkusr.bb-blinkusr1  (where bb-blinkusr is the snap-name and bb-blinkusr1 (or bb-blinkusr2)  
  snap run bb-blinkusr.bb-blinkusr2   the  app names)
 ```
 after adding the details to the snapcraft.yaml file push the files to a Git Remote Repository for building the snap using 
 [Launchpad](https://launchpad.net/)
 
 #### Build Snaps using Launchpad
 
Follow the instructions [here](https://docs.snapcraft.io/build-snaps/ci-integration)(under Launchpad Section) and choose the following
settings,
```
Processors armv7 (armhf)
```
finally after the build has finished and the project has been uploaded to the edge channel you can install the snaps by running

```
snap install --edge <snap name>
```
 
 Note: For running snaps on other ubuntu installations you will have to install snapd by running 
 
 ```
 sudo apt update 
 sudo apt install snapd 
 ```
 


