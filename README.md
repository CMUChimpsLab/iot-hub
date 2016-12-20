# Hub for IoT
Hub for IoT is a centralized system that manages and controls IoT devices. It grants network connection for those devices and controls them to play video and music. It also checks for software update for devices and manages user access. It is implemented as a web app and users can interect with it through web browser. The hub uses EAP-NOOB protocol to grant network access (for more information about EAP-Noob, please take a look at eap-noob.md)

# Instruction to run Authentication server
At location hostapd-2.5/hostapd execute

$ ./hostapd hostapd.conf

# Instruction to run Web server
At location nodejs execute

$ node server.js

# Instruction to run Client
At location wpa_supplicant-2.5/wpa_supplicant execute

$ ./wpa_auto_run.py

### Protocol between server and client:
```
{
    "userID": userID,
    "type": "video" / "update" / "getContent",
    "action": "play",
    "url": "https://www.youtube.com/watch?v=iAmI1ExVqt4",
    "source":"youtube" / "spotify",
    "content": bytestream,
    "software_name": "itunes"
    "source_user_name": ,
    "source_password": ,
}
```

* type: video, music, update, software, volumn
* action: play, pause, up, down   # up/down is for volumn control
* url:
* source: youtube, spotify
* software_list: list of software info
* content: bytestream

### Install packages
* Install PortAudio
```
brew install portaudio
pip install --allow-unverified=pyaudio pyaudio
```

* Install pyspotify
for mac:
```
pip install pyspotify
```
for ubuntu:
```
wget -q -O - https://apt.mopidy.com/mopidy.gpg | sudo apt-key add -
sudo wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/jessie.list
sudo apt-get update
sudo apt-get install python-spotify
```

* Install ws4py
```
pip install ws4py
```

* Install ChromeDriver on Ubuntu
```
sudo apt-get install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

sudo dpkg -i google-chrome*.deb
sudo apt-get install -f
sudo apt-get install xvfb
sudo apt-get install unzip

wget -N http://chromedriver.storage.googleapis.com/2.20/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
chmod +x chromedriver

sudo mv -f chromedriver /usr/local/share/chromedriver
sudo ln -s /usr/local/share/chromedriver /usr/local/bin/chromedriver
sudo ln -s /usr/local/share/chromedriver /usr/bin/chromedriver

pip install pyvirtualdisplay selenium
```

### Database Schema:
```
User
    UserID
    UserName
    Password

Device
    DeviceID
    DeviceState
    DeviceType
    <!-- UserID -->
    DeviceName
    Description
    Image
    SoftwareUpdateURL
    ConnectionID

Notification (software, videolist update)
    NotificationID
    UserID
    DeviceID
    Timestamp
    NotificationType
    Description
```

#### Hardcode insert
ContentList
    ContentID
    UserID
    ContentType
    ContentName
    URL
    Source

AuthorizedUser
    DeviceID
    UserID
    Permission // 0 - admin, 1 - user

## Action on views

```
GET /profile
return
{
    UserID: string userID,
    UserName: string userName,
    NotificationList: [{int NotificationID, int DeviceID, string NotificationType, string Description}, ...],
    Devices: [{int DeviceID, string DeviceName, string ImagePath, string Description}, ...]
}
```

NotificationType: "VideoListUpdate"/"AudioListUpdate"/"SoftwareUpdate"

```
POST /profile?notification=[ID]&Action=(cancel: delete / detail: routing)&Type=[VideoListUpdate]/[AudioListUpdate]
```

```
POST (Working transmit)
/profile?notification=[ID]&Action=(cancel: delete / detail: routing / agree: transmit file)&Type=[SoftwareUpdate]
```

```
GET /devices?userID=[]
{
    UserID: string userID,
    UserName: string userName,
    Devices: [[int DeviceID, string DeviceName, string DeviceState, string Description, string ImagePath], ...]
}
```

DeviceState: "connected"/"disconnected"

```
GET /device?deviceID=[]&userID=[]
{
    UserID: int userID,
    DeviceName: string deviceName,
    DeviceState: string deviceState,
    Description: string description,
    Image: string image,
    NotificationList: [{}, ...]
    ContentList: [] // just empty
    Permission: 0 - highest level
}
// render different ejs (speaker.ejs / display.ejs)
```

```
GET /contentList?userID=[]&contentType=[Video/Audio]&Source=[YouTube]&deviceID=[]
    content type
    content
    render speaker/ejs / display.ejs with specified content
    UserID: int userID,
    DeviceName: string deviceName,
    DeviceState: string deviceState,
    Description: string description,
    Image: string image,
    NotificationList: [{}, ...]
    ContentList: [{ContentID, ContentName, URL}] // with specified source
```

```
POST /control?userID=[]&deviceID=[]&contentType=[]&URL=[]&source=[]&action=[](volume: up/down; click play/resume: change; play: play, with URL)
```

```
POST /checkUpdate?userID=[]
Return list of all the notifications
```

```
GET /authorizedUser?DeviceID=[]&UserID=[]
{
    DeviceID: deviceID,
    UserID: userID,
    // except for himself
    Users: [{
        UserID: id,
        UserName: name
    }, ...]
}
```

```
POST /revokeAuthUser?DeviceID=[]&UserID=[]
    delete it in database
```

```
POST /addAuthUser?DeviceID=[]&Username=[]&Permission=[]
{
    Status: 0 - Success, 1 - User Not Exists, 2 - Failure
    UserID: userID
}
```

// Spotify autentication
POST /getAudio?UserID=[]&ContentType=[]&Source=[]&SourceUserName=[]&SourcePassword=[]&DeviceID=[]
return content list

Client to Server Metadata protocol (through websocket)
```
{
    "Type": "Metadata",
    "DeviceName": e.g "Samsung Display",
    "DeviceType": a string of types separated by semi-colon; e.g "Video;Audio",
    "UpdateSource": "www.abc.com",
    "DeviceDescription": "some description"
}
```
Client to Server GetContent protocol (through websocket)
```
{
    "Type": 'ContentList',
    'UserID': userID,
    'ContentList': [{
        'ContentName': name,
        'ContentType': 'Audio',
        'ContentURL': url,
        'Source': 'Spotify'
    }, ...] // a list of json file, each file is a content
}
```

