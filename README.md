# Set up alexa with asterisk

## The setup of Asterisk for Alexa involves 3 main steps.
* Install Asterisk Server with extensions.
* Install Alexa on Asterisk Server.
* Test the call

## Step 1: Install Asterisk Server

### 1. Install an Asterisk PBX on a LInux system (We used Ubuntu 16.04):
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install asterisk
```

### 2. Add a SIP Phone to your Asterisk. 
Since there are many ways to do this, I am showing a generic
configuration. Each phone has different options, but this should be all that is needed here.
This is adding an extension number 5310 to Asterisk. Remember, this does NOT include inbound or outbound calling. Only internal calls. Backup your /etc/asterisk/*.conf files before editing!
```
$git clone https://github.com/rgrokett/RaspiAsteriskAlexa.git
$cd config
$sudo cp /etc/asterisk/sip.conf /etc/asterisk/sip.conf.bak
$sudo cat sip.conf >> /etc/asterisk/sip.conf
$sudo cp /etc/asterisk/extensions.conf /etc/asterisk/extensions.conf.bak
$sudo cat extensions.conf >> /etc/asterisk/extensions.conf
$cd config
$sudo cp /etc/asterisk/sip.conf /etc/asterisk/sip.conf.bak
$sudo cat sip.conf >> /etc/asterisk/sip.conf
$sudo cp /etc/asterisk/extensions.conf /etc/asterisk/extensions.conf.bak
$sudo cat extensions.conf >> /etc/asterisk/extensions.conf
```
### 3. Configure your SIP Phone with the options above. Most other phone options are not used here, so can usually be ignored.

### 4. Restart system to save the changes.


## Step 2: Install Alexa on Asterisk
### 1. Install packages and files
```
sudo cp alexa.agi /usr/share/asterisk/agi-bin/alexa.agi
sudo cp avs_audio.json /etc/
sudo cp sounds/alexa*.sln /usr/share/asterisk/sounds/custom/
sudo chown asterisk:asterisk /usr/share/asterisk/agi-bin/alexa.agi
sudo chown asterisk:asterisk /etc/avs_audio.json
sudo chown asterisk:asterisk /usr/share/asterisk/sounds/custom/alexa*.sln
cp token.pl /home/pi
sudo rm /tmp/*
sudo rm /tmp/token.* /tmp/avs*
```

#### Install other dependencies
```
sudo apt-get install sox
sudo apt-get install libsox-fmt-mp3
sudo apt-get install libwww-perl libjson-perl
sudo apt-get install flac
sudo curl -L http://cpanmin.us | perl - --sudo App::cpanminus
sudo cpanm IO::Socket::SSL --force
sudo perl -MCPAN -e 'install JSON'
sudo apt-get install libjson-pp-perl
```

### 2. Verify the Asterisk /etc/asterisk/extensions.conf contains the below and append if needed:
```
$ sudo tail -30 /etc/asterisk/extensions.conf
```

/etc/asterisk/extensions.conf
```
; AMAZON ALEXA VOICE
[alexa_tts]
exten => 5555,1,Answer()
; Get an AWS Token
exten => 5555,n,System(/home/pi/token.pl)
; Play prompts
exten => 5555,n,Playback(./custom/alexa_hello)
exten => 5555,n,Playback(./custom/alexa_example)
; Alexa API integration
exten => 5555,n(record),agi(alexa.agi,en-us)
; Loop
exten => 5555,n,Playback(./custom/alexa_another)
exten => 5555,n,goto(record)
; These are not used currently
exten => 5555,n(goodbye),Playback(vm-goodbye)
exten => 5555,n,Hangup()
```

### 3. Add the following line to extensions.conf so that the extension is dialable locally.
#### a. Edit /etc/asterisk/extensions.conf
#### b. Locate the section called [local]
#### c. Add a line “include => alexa_tts”
```
/etc/asterisk/extensions.conf
[local]
ignorepat => 9
include => default
include => trunklocal
include => iaxtel700
include => trunktollfree
include => iaxprovider
include => alexa_tts
```
### 4. Reboot system

## Step 3: Test the Call

Create Extension 5310: Append to /etc/asterisk/sip.conf
```
[5310]
type=friend
username=5310
fromuser=5310
host=dynamic
context=local
insecure=port
qualify=500
dtmfmode=rfc2833
disallow=all
allow=ulaw
obtained
progressinband=no
nat=no
mailbox=5310
callerid=5310

```
Append to /etc/asterisk/extensions.conf
```
; Basic SIP Phone
exten => 5310,1,Dial(SIP/5310,15)
exten => 5310,2,Voicemail(5310,u)
exten => 5310,3,Hangup
exten => 5310,102,Voicemail(5310,b)
exten => 5310,103,Hangup
```


### Test the call:

1. Configure a softphone(SIP Phone) using the extension 5310.
2. SIP Phone options for 5310: Download a softphone like Zoiper and configure the following:
	SIP Proxy Server: 192.168.1.XX <- The IP Address of your server
	Domain/Realm/Registration: <same IP as above>
	Username: 5310
	Password: alexa5310
3. Dial 5555 from the softphone
6. After the alexa greeting message a beep will sound. Now, you may ask alexa help for various queries, like:
    * what is the best time to grow rice?
    * what is the current market rate of wheat?
    * and so on

