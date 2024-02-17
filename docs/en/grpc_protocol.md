#cyberdog_grpcprotocol

## Summary

### Noun convention

- Downward command/download: refers to the command or direction sent by the APP to the NV board;
- Uplink command/uplink: refers to the command or direction sent by the NV board to the APP;

### Design

[cyberdog_grpc design document](cyberdog_grpc_en.md)


## sport control

### Servo control (joystick function)

####Download command

ros interface: topic, type: protocol/msg/MotionServoCmd, topic name: "motion_servo_cmd"

```Nginx
nameCode = 1002;

params
{
     int32 motion_id; # Please refer to the above interface document
     int32 cmd_type; # Instruction type constraints 0 or 1: Data, 2: End; (2 is sent in the last frame)
     int32 cmd_source; #Command source, app fills in 0
     float32 vel_des[3]; # x direction, y direction, rotation, "vel_des": [x, y, z]
     float32 pos_des[3]; # [1] value is valid, "pos_des": [0, value, 0]
     float32 rpy_des[3]; # [2] value is valid, "rpy_des": [0, 0, value]
     float32 step_height[2]; # Valid when walking, recommended value [0.05, 0.05]
     int32 value; # 0 - inner eight gait, 2 - vertical gait, 4 - movement control start calibration
}
```

#### Uplink data

ros interface: topic, "motion_servo_response"

```Nginx
nameCode = 1003

params
{
     int32 motion_id # Return the issued motion_id to the original path
     int32 cmd_id # If this field is delivered, return this field
     bool result # true: Normal execution; false: Execution failed, and the delivery will not be executed if it continues.
     int32 code
}
```


### Result command control

####Download command

ros interface: service, "motion_result_cmd"

```Nginx
nameCode = 1004

params
{
     int32 motion_id;
     int32 cmd_source; #Command source, app fills in 0
     float32 vel_des[3]; # x direction, y direction, rotation, "vel_des": [x, y, z]
     float32 pos_des[3]; # [1] value is valid, "pos_des": [0, value, 0]
     float32 rpy_des[3]; # [2] value is valid, "rpy_des": [0, 0, value]
     float32 ctrl_point[3]; # Currently not open. pose ctrl point m
     float32 foot_pose[3]; # Currently not open. front/back foot pose x,y,z m
     float32 step_height[3]; # Leg lifting height, the maximum value of the operation control group is uncertain, currently it can be set to 0.06m
     int32 contact;
     int32 duration # execution time
     int32 value
}
```

#### Return results

```Nginx
nameCode = 1004

data
{
     int32 motion_id # Return the issued motion_id to the original path
     bool result # true: execution successful; false: execution failed;
     int32 code # The result code of the motion module, used for debugging and quickly locating problems
}
```

## OTA

### Status reporting

####Download command

```Nginx
nameCode = 5001

params
{}
```



#### Uplink command

ros interface: topic, "ota_status"? ? ?

```Nginx
nameCode = 5001

params
{
     string status # idle: idle status
                      # update: Upgrading status
                      # download: downloading status
     int32 code # 0 is normal, other values are error codes
}
```


### start download 

####Download command

ros interface: service, "ota_service_start_download"

```C%23
nameCode = 5003

params
{
     {
             "modules": [{
                 "product": "l91",
                 "device": "l91.NX",
                 "module": "l91.NX.OTA",
                 "current": {
                     "version": null,
                     "versionSerial": null,
                     "packages": null,
                     "logInfo": null,
                     "historyLogInfo": null,
                     "forceUpdateFlag": false
                 },
                 "latest": {
                     "version": "1.0.0.4",
                     "versionSerial": 1655173627000,
                     "packages": [{
                         "version": "1.0.0.4",
                         "type": "FULL_PACKAGE",
                         "md5": "b7e2a750b0",
                         "fileSize": 0,
                         "fileName": "L91%2FNX%2Fdaily-release%2F2022-06-14%2F122441%2Fcarpo_galactic_V1.0.0.1.20220614.102737_release_b7e2a750b0.tgz",
                         "description": null,
                         "downloadOption": {
                             "mirrors": ["https://cnbj2m.fds.api.xiaomi.com/version-release/L91%2FNX%2Fdaily-release%2F2022-06-14%2F122441%2Fcarpo_galactic_V1.0.0.1.20220614.102737_release_b7e2a750b0.tgz? Expires=1970540922760&GalaxyAccessKeyId=AKKF3F5Z2NCBBWT7IL&Signature=vPoyrIZTvJbpsRtS4PKFfrBJXlI="]
                         }
                     }],
                     "logInfo": null,
                     "historyLogInfo": [],
                     "forceUpdateFlag": false
                 }
             }]
         }
}
```



#### Uplink command

```Nginx
nameCode = 5003

params
{
     bool response # true: OTA executes the download command successfully
                     # false: OTA failed to execute the start download command
     int32 code # 0 is normal, other values ​​are error codes
}
```

### Start upgrading

####Download command

ros interface: service, "ota_service_start_upgrade"

```Nginx
nameCode = 5004

params
{
     "md5":xxxxxx
}
```



#### Uplink command

```Nginx
nameCode = 5004

params
{
     bool response # true: OTA executes the start upgrade command successfully
                     # false: OTA failed to execute the start upgrade command
     int32 code # 0 is normal, other values are error codes
}
```



### Progress reporting

####Download command

ros interface: service, "ota_service_process"

```Nginx
nameCode = 5005

params
{
}
```

#### Uplink command

```Nginx
nameCode = 5005

params
{
     int32 download_progress # Progress download failed: -1, remaining space is less than 2G: -2
     int32 upgrade_progress # Upgrade upgrade failed: -1, remaining space is less than 2G: -2
     int32 code # 0 is normal, other values are error codes
}
```

### Estimated upgrade time query

####Download command

ros interface: topic, "ota_topic_estimate_upgrade_time"

```Nginx
nameCode = 5006

params
{
}
```



#### Uplink command

```Nginx
nameCode = 5006

params
{
     int32 time # Estimated total upgrade time -1: Calculation failed -2: Current status cannot be obtained
}
```



### NX restart

#### Uplink command (active reporting)

```Nginx
nameCode = 5007

params
{
}
```

### Upgrade results

#### Uplink command (active reporting)

```Nginx
nameCode = 5008

params
{
     bool success # true: success; false: failure
     int32 code # Error code, 0-no error
}
```

## AUDIO

### Authentication request

####Download command

ros interface: service, "get_authenticate_didsn"

```Bash
nameCode = 3001

params
{
}
```

#### Uplink command

```Bash
nameCode = 3001

params
{
     string did
     string sn
}
```

### Authentication reply

####Download command

ros interface: service, "set_authenticate_token"

```Bash
nameCode = 3002

params
{
     uint64 uid
     string token_access
     string token_fresh
     string token_expires_in
}
```

#### Uplink command

```Bash
nameCode = 3002
params
{
     bool result
}
```

### Voiceprint training begins

####Download command

```Bash
nameCode = 3011

params
{
     string nick_name
     string voiceprint_id
}
```

#### Uplink command

```Bash
nameCode = 3011
params
{
}
```

### Voiceprint training canceled

####Download command

```Bash
nameCode = 3012

params
{
}
```

#### Uplink command

```Bash
nameCode = 3012

params
{
}
```

### Voiceprint training result notification

#### Uplink command

```Bash
nameCode = 3013

params
{
     int32 code # 0 is normal, non-0 is abnormal
     string voiceprint_id # Voiceprint id
}
```

### Voiceprint information query

#### Uplink command

```Bash
nameCode = 3014

params
{
}
```

####Download command

```Bash
nameCode = 3014

params
{
     string voiceprints_data
}
```

## Robot status

### Heartbeat

```ProtoBuf
Ticks
{
     string ip = 1;
     fixed32 wifi_strength = 2;
     fixed32 battery_power = 3;
     bool internet = 4;
     string sn = 5;
     MotionStatus motion_status = 6;
     TaskStatus task_status = 7;
     SelfCheckStatus self_check_status = 8;
     StateSwitchStatus state_switch_status = 9;
     ChargingStatus charging_status = 10;
     bool audio = 11;
}
```

### Information query

#####Download command

```C%2B%2B
nameCode = 1001

params
{
     bool is_version # Whether to return the current version information, true: yes; false: no;
     bool is_sn # Whether to return sn, true: yes; false: no;
     bool is_nick_name # Whether to return the nickname, true: yes; false: no;
     bool is_volume # Whether to return the volume, true: yes; false: no;
     bool is_mic_state # Whether to return MIC state, true: yes; false: no;
     bool is_voice_control # Whether to return voice control status, true: yes; false: no;
     bool is_wifi # Whether to return wifi information
     bool is_bat_info # Whether to return battery information, true: yes; false: no;
     bool is_motor_temper # Whether to return the motor temperature
     bool is_audio_state # Whether to return the speaker board activation state
     bool is_device_model # Whether to return the device model
     bool is_stand # Whether to stand
     bool is_lowpower_control # Whether to return to low power state
     string uid # Current user uid
}

# When configuring is_audio_state to obtain the activation status of the speaker board, pass the uid data
```

##### Uplink command

```C%2B%2B
nameCode = 1001

params # json string.
{
     string version #Current version information. Contains the current version information of all board software and the latest version information of the server
     string sn # machine sn
     string nick_name # Nickname
     int volume # volume
     bool mic_state # Current mic status. true: enable; false: disable;
     bool voice_control # Voice control status. true: enable; false: disable;
     bool wifi # WiFi information
     string bat_info #Battery related information
     string motor_temper # Motor temperature
     bool audio_state # Whether the current speaker board is activated. true: activated; false: not activated
     string device_model #Device model, string
     bool stand # Whether to stand
     bool lowpower_control # Low power switch status. true: enable; false: disable;
}
example:
{
     # The specific contents of the current version are as follows:
     "version": {
           "new_version":"1.0.0.0.2020202",
           "nx_debs": {
             "sw-version" : "1.0.6" # NX version
           },
           "nx_mcu" : {
             "MCU_HEAD_MINILED" : "0.0.0.1_20220712", #mini-led version
             "MCU_HEAD_TOF" : "0.0.0.1_20220712", #TOF version of the head
             "MCU_IMU" : "0.0.0.1_20220712", #imu version
             "MCU_POWER" : "0.0.0.1_20220712", #Power version
             "MCU_REAR_TOF" : "0.0.0.1_20220712" #Tail tof version
           },
           "r329" : {
             "l91_audio" : "1.0.0.1.20220712.112619" #Voice board version
           },
           "mr813" : {
             "l91_loco" : "1.0.0.1.20220712.022049" #Operation control board version
           },
           "mr813_mcu" : {
             "MCU_SPIE0" : "0.0.0.1_20220712", #
             "MCU_SPIE1" : "0.0.0.1_20220712" #
           },
           "motors" : {
             "MCU_Motor" : "0.2.1.2_20220606" #Motor version
           }
     },
     "sn": "zxw88****166",
     "nick_name": {
           "default_name" : "Iron Egg",
           "current_name" : "Wangcai"
     },
     "volume": 50,
     "mic_state": true，
     "voice_control": true，
     "wifi": {
         "name": "***",
         "ip": "***",
         "mac": "***",
         "strength": 50
     },
     "bat_info": {
         "capacity": 97, # battery capacity
         "power": 97, # battery power
         "voltage": 220, # battery voltage
         "temperature": 100 # Battery temperature
         "is_charging": true #Charging status, true: charging; false: not charging;
         "discharge_time": 120 # Discharge time, in minutes.
     },
     "motor_temper": {
         "hip": [97, 97, 97, 97], # [Hip left front, hip right front, hip left back, hip right back]
         "thigh": [97, 97, 97, 97], # [front left thigh, front right thigh, rear left thigh, right rear thigh]
         "crus": [97, 97, 97, 97] # [Left front of calf, Front right of calf, Back left of calf, Back right of calf]
     },
     "audio_state": false，
     "device_model": "MS2241CN"，
     "stand": true，
     "lowpower_control": true
}
```

### Nickname switch

#####Download command

```C%2B%2B
nameCode = 1010

params
{
     bool enable #On: true, off: false
}
```

##### Uplink command

```C%2B%2B
nameCode = 1010

params
{
     bool success # Success: true, failure: false
}
```

### Set nickname

#####Download command

```C%2B%2B
nameCode = 1011

params
{
     string nick_name # Nickname
     string wake_name # Wake word. If the nickname is 2 or 3 characters, the wake-up word = nickname*2; if the nickname is more than 3 characters, the wake-up word = nickname.
}
```

##### Uplink command

```C%2B%2B
nameCode = 1011

params
{
     bool success # Success: true, failure: false
}
```

### Set volume

#####Download command

```C%2B%2B
nameCode = 1012

params
{
     int volume # volume
}
```

##### Uplink command

```C%2B%2B
nameCode = 1012

params
{
     bool success # Success: true, failure: false
}
```

### Set microphone status

#####Download command

```C%2B%2B
nameCode = 1013

params
{
     bool enable # true: enable; false: disable;
}
```

##### Uplink command

```C%2B%2B
nameCode = 1013

params
{
     bool success # Success: true, failure: false
}
```

### Set voice control switch

#####Download command

```C%2B%2B
nameCode = 1014

params
{
     bool enable # true: enable; false: disable;
}
```

##### Return results

```C%2B%2B
nameCode = 1014

data
{
     bool success # Success: true, failure: false
}
```

### Set up the eye light

####Download command

```C%2B%2B
nameCode = 1015

params
{
     int eyelight # brightness
}
```

#### Return results

```C%2B%2B
nameCode = 1015

data
{
     bool success # Success: true, failure: false
}
```



### Member addition

#####Download command

```C%2B%2B
nameCode = 1020

params
{
     string member
}
```

##### Uplink command

```C%2B%2B
nameCode = 1020

params
{
     bool success # Success: true, failure: false
}
```

### Member query

#####Download command

```C%2B%2B
nameCode = 1021
params
{
     string account # Empty: return all member information; specific member: this member information;
}
```

##### Uplink command

```Bash
nameCode = 1021

params
{
     string accounts
}

{
     "code": 0,
    
     "accounts": [
         {
             "name": "xiao_ming",
             "face_state": 0, # 0-not entered, 1-entered, 2-in progress
             "voice_state" 0 # 0-not entered, 1-entered, 2-in progress
         },
         {
             "name": "da_qiang",
             "face_state": 0,
             "voice_state" 0
         }
     ]
}

# code = 0 query successful; code = 121 user query failed, the current user does not exist, please delete and re-add the user
# "accounts": Queryed user information
```

### Member deletion

#####Download command

```C%2B%2B
nameCode = 1022

params
{
     string member
}
```

##### Uplink command

```C%2B%2B
nameCode = 1022

params
{
     bool success # Success: true, failure: false
}
```

### Member synchronization

nameCode = 1023

### Member changes nickname

#####Download command

```C%2B%2B
nameCode = 1024

params
{
     string pre_name
     string new_name
}
```

##### Uplink command

```C%2B%2B
nameCode = 1024

params
{
     bool success # Success: true, failure: false
}
```

### Stop sound playback

ros interface: service, type: std_srvs/srv/Empty, service name: "stop_play"

####Download command

```Nginx
nameCode = 1025

params
{
}
```

#### Return results

```Nginx
nameCode = 1025

data
{
}
```

## Image transmission

### Start image transmission

webrtc signaling handshake

grpc service: GrpcApp.sendMsg

ros interface: topic, type: std_msgs/msg/String, upstream name: "img_trans_signal_out", downstream name: "img_trans_signal_in"

####Download command

sdp:

```Nginx
nameCode = 4001

params
{
     string offer_sdp or answer_sdp #Contains type and sdp fields
     string uid #An identification name on the app side. If you only connect to one app, you don’t need this item.
     int32 height #The mobile phone resolution is high, if not filled in, the default is 1280
     int32 width #Mobile phone resolution width, if not filled in, the default is 720
     string alignment #Alignment method, top top alignment, middle horizontal midline alignment, bottom bottom alignment, if not filled in, the default is horizontal midline alignment
}
```

##### offer_sdp and answer_sdp

```Nginx
string type
string sdp
```

ice candidate:

```Nginx
nameCode = 4001

params
{
     string c_sdp #Contains sdpMid, sdpMLineIndex and candidate fields
     string uid #An identification name on the app side. If you only connect to one app, you don’t need this item.
}
```

##### c_sdp

```Nginx
string sdpMid
int32 sdpMLineIndex
string candidate
```

#### Uplink command

The same as the uplink command, if the uid item is not set on the app side, the uid of the uplink command is "default_uid"

### Stop image transmission

grpc service: GrpcApp.sendMsg

ros interface: topic, type: std_msgs/msg/String, upstream name: "img_trans_signal_out", downstream name: "img_trans_signal_in"

####Download command

```Nginx
nameCode = 4001

params
{
     string uid #An identification name on the app side. If you only connect to one app, you don’t need this item.
}
```

#### Return results

```Nginx
nameCode = 4001

data
{
     bool is_closed #Whether PeerConnection has been disconnected
     string uid #An identification name on the app side. If you only connect to one app, you don’t need this item.
}
```

## Photograph

grpc service: GrpcApp.getFile

ros interface: service, type protocol/srv/CameraService, name: "camera_service"

####Download command

```Nginx
nameCode = 4002

params
{
     uint8 command; #1 is to take pictures
}
```

#### Return results

```Nginx
fixed32 error_code #Error code, 0 is normal, other values ​​are errors
string file_name #File name
fixed32 file_size #File size (bytes)
bytes buffer #File data block (maximum 4MB)
```

## Video

ros interface: service, type protocol/srv/CameraService, name: "camera_service"

### Start recording

grpc service: GrpcApp.sendMsg

####Download command

```Nginx
nameCode = 4002

params
{
     uint8 command; #2 is to start recording
}
```

#### Return results

```Nginx
nameCode = 4002

data
{
     uint8 result #Start recording result, 0 means success, other values mean failure
}
```

### Stop recording

grpc service: GrpcApp.getFile

####Download command

```Nginx
nameCode = 4002

params
{
     uint8 command; #3 is to stop recording
}
```

#### Return results

```Nginx
fixed32 error_code #Error code, 0 is normal, other values ​​are errors
string file_name #File name
fixed32 file_size #File size (bytes)
bytes buffer #File data block (maximum 4MB)
```

### Unfinished transfer of video and photo files

If grpc interrupts the connection during file transfer, nx will send this request after reconnecting, and the app needs to follow this request to re-obtain the file.

grpc service: GrpcApp.sendMsg

#### Uplink command

```Nginx
nameCode = 4003

params
{
     string file_name[] # List of files that need to be re-uploaded
}
```

### Get saved video and photo files

If the APP receives a 4003 message after the connection is successful, it means that there are untransmitted files in the last connection or files that were automatically saved by the app during the recording process and need to be retransmitted. At this time, each file calls this request.

grpc service: GrpcApp.getFile

####Download command

```Nginx
nameCode = 4004

params
{
     string file_name # File that needs to be re-uploaded
}
```

#### Return results

```Nginx
fixed32 error_code #Error code, 0 is normal, other values ​​are errors
string file_name #File name
fixed32 file_size #File size (bytes)
bytes buffer #File data block (maximum 4MB)
```

### Insufficient space warning

When the disk space of nx board is lower than a certain preset value (tentatively 500MB), nx will send this prompt. When receiving this prompt, the app should prompt the user or stop recording immediately.

grpc service: GrpcApp.sendMsg

#### Uplink command

```Nginx
nameCode = 4005

params
{
     int remain_size #remaining space bytes
}
```

## AI

### Face input request

#####Download command

```C%23
nameCode = 7001 #FACE_ENTRY_REQUEST

params
{
     int32 command
     string username //Current name
     string origine // old name
     bool ishost //Is it the owner?
}
/* int32 command
int32 ADD_FACE = 0
int32 CANCEL_ADD_FACE = 1
int32 CONFIRM_LAST_FACE = 2
int32 UPDATE_FACE_ID = 3
int32 DELETE_FACE = 4
int32 GET_ALL_FACES = 5
*/
```

##### Uplink command

```C%2B%2B
nameCode = 7001
params
{
     int32 result
     string allfaces // temporary
}
/*
int32 RESULT_SUCCESS = 0 // Except for add, the rest are the real return results
int32 RESULT_INVALID_ARGS = 5910 //The parameters are illegal
int32 RESULT_UNSUPPORTED = 5908 //Parameters not supported
int32 RESULT_TIMEOUT = 5907
int32 RESULT_BUSY = 5911 //The simultaneous face entry and face recognition functions are not supported. Please turn off the face recognition function in graphical programming.
int32 RESULT_INVALID_STATE = 5903
int32 RESULT_INNER_ERROR = 5904
int32 RESULT_UNDEFINED_ERROR = 5901
*/
```

### Face entry results

#### Uplink command

```C%2B%2B
nameCode = 7003 #FACE_ENTRY_RESULT_PUBLISH
params
{
     int32 result
     string username //Add the name of the face
}
/*
int32 RESULT_SUCCESS =0
int32 RESULT_TIMEOUT =5907
int32 RESULT_FACE_ALREADY_EXIST = 5921
*/
```

### Face recognition request

####Download command

```Go
nameCode = 7002 #FACE_RECORDING_REQUEST

params
{
     int32 command
     string username //Name of person to be identified
     int32id
     int32 timeout #valid time 1～300, default = 60
}
/* int32 command
int32 COMMAND_RECOGNITION_ALL = 0
int32 COMMAND_RECOGNITION_SINGLE = 1
*/
```

#### Uplink command

```C%2B%2B
nameCode = 7002
params
{
     int32 result
}
/*
int32 ENABLE_SUCCESS = 0
int32 ENABLE_FAIL = 5901
*/
```

### Face recognition results

#### Uplink command

```Go
nameCode = 7004 #FACE_RECOGNITION_RESULT_PUBLISH
params
{
     string username
     int32 result
     int32id
     float32 age
     float32 emotion
}
/*
int32 RESULT_SUCCESS =0
int32 RESULT_TIMEOUT =5907
*/
```

## SLAM related

Unable to copy loading content

### Get map list

![](./image/grpc_protocol/grpc_slam.svg)

### Map data reporting

ros interface: topic, type: nav_msgs/msg/OccupancyGrid, name: "map"

#### Uplink command

```Nginx
nameCode = 6001

params
{
     float32 resolution #map resolution
     uint32 width #map width
     uint32 height #map height
     float64 px #Map origin x
     float64 py #map origin y
     float64 pz #Map origin z
     float64 qx #Map origin quaternion x
     float64 qy #Map origin quaternion y
     float64 qz #Map origin quaternion z
     float64 qw #Map origin quaternion w
     int8 data[] #map data
}
```

### Set label

ros interface: service, type: protocol/srv/SetMapLabel, name: "set_label"

####Download command

```Nginx
nameCode = 6002

params
{
   string mapName #map name
   uint64 map_id # Map unique identification code
   bool is_outdoor #Whether it is outdoor mapping
   label locationLabelInfo[] #label list
}
```

##### label structure

```Nginx
string labelName #label name
float64 physicX #X coordinate
float64 physicY #Y coordinate
```

#### Return results

```Nginx
NameCode=6002

data
{
   uint8 success #0 means success, 1 means failure
}
```

### Delete map

ros interface: service, type: protocol/srv/SetMapLabel, name: "set_label"

####Download command

```Nginx
nameCode = 6002

params
{
   string mapName #map name
   uint64 map_id
   bool only_delete #Delete the target map when this value is true
}
```

#### Return results

```Nginx
NameCode=6002

data
{
   uint8 success #0 means success, 1 means failure
}
```

### Delete tag

ros interface: service, type: protocol/srv/SetMapLabel, name: "set_label"

####Download command

```Nginx
nameCode = 6002

params
{
   string mapName #map name
   uint64 map_id
   label locationLabelInfo[] #List of labels to be deleted. If you only want to delete one, fill in one. If not, it will be considered as deleting the map and its label.
   bool only_delete #Delete the target label when this value is true
}
```

#### Return results

```Nginx
NameCode=6002

data
{
   uint8 success #0 means success, 1 means failure
}
```

### Get labels and map data

ros interface: service, type: protocol/srv/GetMapLabel, name: "get_label"

####Download command

```Nginx
nameCode = 6003

params
{
   string mapName #map name
   uint64 map_id
}
```

#### Return results

```Nginx
nameCode = 6003

data
{
     string mapName #map name
     uint8 success #0 means success, 1 means failure
     bool is_outdoor #Whether it is outdoor mapping
     label locationLabelInfo[] #Label list, the label structure is the same as above
     Map map
}
```

##### Map structure

```Nginx
float32 resolution #map resolution
uint32 width #map width
uint32 height #map height
float64 px #Map origin x
float64 py #map origin y
float64 pz #Map origin z
float64 qx #Map origin quaternion x
float64 qy #Map origin quaternion y
float64 qz #Map origin quaternion z
float64 qw #Map origin quaternion w
int8 data[] #map data
```

The label structure is the same as above

### Mapping task

Start building a map, but the results will not be returned immediately, there will be intermediate feedback

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

All tasks started by 6004 can be manually stopped by 6009

####Download command

```Nginx
NameCode=6004

params
{
     uint8 type # 5 starts mapping
     bool outdoor #When starting mapping, determine whether it is outdoor mapping. true means outdoor mapping, false means indoor mapping.
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

These three feedback codes apply to all 6004 tasks: 1000 is activating the dependent node, 1001 activates the dependent node successfully, and 1002 fails to activate the dependent node.

```Nginx
NameCode=6004

data
{
   int32 feedback_code #Feedback code, 6 starts successfully, 7 starts fails, 8 relocation succeeds, 9 relocation fails
   string feedback_msg # Additional message, currently redundant
}
```

#### Return results

```Nginx
NameCode=6004

data
{
     uint8 result #Mapping result, 0 success, non-0 failure
}
```

### Robot dog position reporting

ros interface: topic, type: geometry_msgs/msg/PoseStamped, name: "dog_pose"

#### Uplink command

```Nginx
nameCode = 6005

params
{
   float64 px #Position x coordinate
   float64 py #Position y coordinate
   float64 pz #Position z coordinate
   float64 qx #position quaternion x
   float64 qy #position quaternion y
   float64 qz #position quaternion z
   float64 qw #position quaternion w
}
```

## Navigation

![](./image/grpc_protocol/grpc_nav.svg)

### Relocation task

All tasks started by 6004 can be manually stopped by 6009

This request must be called before using AB point navigation (that is, when entering the navigation page). The result will not be returned immediately after calling this request. The robot dog will enable the relocation function. Feedback information may be sent during the relocation process. After the relocation is completed, the requested result will be returned. If the relocation is successful, the positioning function will be enabled and will continue. Continue until you exit the navigation page.

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6004

params
{
   uint8 type #7 is to start the relocation task
   bool outdoor #Whether it is outdoor
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

These three feedback codes apply to all 6004 tasks: 1000 is activating the dependent node, 1001 activates the dependent node successfully, and 1002 fails to activate the dependent node.

```Nginx
NameCode=6004

data
{
   int32 feedback_code #Feedback code, 0 success, 100 failure, continue to try, the remote control dog moves forward for a certain distance, 200 failure, prompting that it is not in the map or the map is wrong, 8 sensors are turned on successfully, 9 sensors fail to turn on
   string feedback_msg # Additional message, currently redundant
}
```

Map check service feedback_code:

- Checking map: 3100
- Map check successful: 3101
- Map check is unavailable, please rebuild the map: 3102
- The map background is under construction, please wait: 3110
- The map check service is abnormal, please restart the robot dog: 3111

Start visual SLAM related service feedback_code:

- Starting relocation dependent services: 3103
- Successfully started the relocation dependent service: 3104
- Failed to start the relocation dependent service, please try the navigation function again: 3105

Start dependent node feedback_code:

- Starting sensor dependent nodes: 1000
- Started sensor dependent node successfully: 1001
- Failed to start sensor dependent node: 1002

Relocation feedback_code:

- The relocation function timed out, please try the navigation function again: 3106
- If the relocation function fails, try again and the remote-controlled dog moves forward a certain distance: 3107
- The relocation function is successfully located. Click on the map or the label in the map to navigate: 3108
- The relocation function failed to locate, please try the navigation function again: 3109

#### Return results

```Nginx
NameCode=6004

data
{
   uint8 result #Start navigation result, that is, relocation result, 0 success, non-0 failure
}
```

### AB point navigation task

All tasks started by 6004 can be manually stopped by 6009

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6004

params
{
   uint8 type #1 is AB point navigation
   float64 goalX #X coordinate
   float64 goalY #Y coordinate
   float64 theta #Yaw angle [-π, π), if this target point has no angle, do not appear in this field
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

These three feedback codes apply to all 6004 tasks: 1000 is activating the dependent node, 1001 activates the dependent node successfully, and 1002 fails to activate the dependent node.

```Nginx
NameCode=6004

data
{
   int32 feedback_code #Feedback code, success, failure, running (continuous sending), path planning failure
   string feedback_msg # Additional message, currently redundant
}
```

- success 
   - The navigation is started successfully, the target point is set successfully, and the path is being planned: 300
   - Navigating: 307
   - Target point reached: 308
- fail
   - Underlying navigation failed:
     - The underlying navigation function service connection failed, please resend the target: 302
     - Failed to send target point, please resend target: 303
     - The underlying navigation function failed, please resend the target: 304
     - The target point is empty, please select another target: 305
     - Path planning failed, please select the target again: 306
- Map check service feedback_code:
   - Checking map: 309
   - Map check successful: 310
   - The map does not exist, please create it again: 311



#### Return results

```Nginx
NameCode=6004

data
{
   uint8 result #Navigation result, 0 success, 3 failure, 4 rejection, 5 cancellation, 2 not obtained, 10 timeout and not reaching the end point
}
```

### Path upload

The APP is used to display the planned path during AB point navigation.

ros interface: topic, type: nav_msgs/msg/Path, name: "plan"

#### Uplink command

```Nginx
nameCode = 6006

params
{
   Point2D path_point[] #Array of two-dimensional points
}
```

Point2D structure

```Nginx
float64 px #Position x coordinate
float64 py #Position y coordinate
```

### Laser point reporting

ros interface: topic, type: sensor_msgs/msg/LaserScan, name: "scan"

#### Uplink command

```Nginx
nameCode = 6011

params
{
   Point2D laser_point[] #array of two-dimensional points
}
```

## Charge

### Automatic staking tasks

All tasks started by 6004 can be manually stopped by 6009

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6004

params
{
   uint8 type #9 is automatic staking
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

It is temporarily undefined. It may feedback the intermediate status and stage information of automatic staking.

#### Return results

```Nginx
NameCode=6004

data
{
   uint8 result #Stake result, 0 success, non-0 failure
}
```

## Visual follow

![](./image/grpc_protocol/grpc_vision_tracking.svg)

### Visual following task

All tasks started by 6004 can be manually stopped by 6009

It is divided into human body following and all things following. There is feedback in the middle, and the result will never be returned during normal operation.

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6004

params
{
   uint8 type #13 is to start visual recognition
   bool object_tracking #true means everything follows, false means human targets follow
   uint8 relative_pos # Relative orientation, 200 chooses the following position independently, 201 follows behind the target, 202 follows on the left side of the target, 203 follows on the right side of the target
   float32 keep_distance #The distance to keep from the following target
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

These three feedback codes apply to all 6004 tasks: 1000 is activating the dependent node, 1001 activates the dependent node successfully, and 1002 fails to activate the dependent node.

```Nginx
NameCode=6004

data
{
   int32 feedback_code #Feedback code, indicating the status of the following task, 500 means that the visual following function is started, 501 means that human body recognition has been started, waiting for the user to choose to follow the target, 502 means that the following target is started, 503 means following, 504 means that the target is temporarily lost , try to find the original target, 504 Human Body and All Things Follow the target and lose it, find the target, 505 Human Body and All Things follow the target and completely lose it, restart following
   string feedback_msg # Additional message, currently redundant
}
```

#### Return results

```Nginx
NameCode=6004

data
{
   uint8 result #Visual following task result, 0 means success (will not appear), non-0 means other
}
```

### Follow target box list

After the human body recognition task request is successfully started, the following target list will be posted (it will not be posted when everything is following). After selecting the target, the selected target box will be posted during the following process.

#### Coordinate conversion

The coordinates of the frame are based on 640×480 as the full size of the image, which is different from the image transmission and mobile phone resolutions and needs to be converted.

ros interface: topic, type: protocol/msg/Person, name: "person"

#### Uplink command

```Nginx
NameCode=6007

param
{
   SelectTrackObject body_info_roi[] #List of human body candidate target boxes for visual human tracking
   RegionOfInterest track_res_roi #Target frame for visual following. If the target is not successfully selected, the height and width will be 0.
}
```

##### SelectTrackObject structure

```Nginx
RegionOfInterest roi #Identified frame
string reid # identification number
```

##### RegionOfInterest structure

```Nginx
uint32 x_offset # The starting pixel coordinate x of the box
uint32 y_offset # The starting pixel coordinate y of the box
uint32 height # Pixel height of the box
uint32 width # Pixel width of the box
```

### Check the follow target box

6007 During the uploading process and the selection has not yet been successful, this request is called. The parameters of the target box are consistent with the uploaded parameters.

ros interface: service, type: protocol/srv/BodyRegion, name: "tracking_object_srv"

####Download command

```Nginx
NameCode=6008

param
{
   RegionOfInterest roi #Selected visual following target
}
```

#### Return results

```Nginx
NameCode=6008

data
{
   bool success #Whether the selection is successful, it will take about 20~30 seconds
}
```

## UWB Follow

![](./image/grpc_protocol/grpc_uwb.svg)

### UWB follow mission

All tasks started by 6004 can be manually stopped by 6009

This request is called after the robot dog is connected to the bracelet (the UWB of the bracelet will automatically connect to the UWB on the robot dog after Bluetooth connection)

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6004

params
{
   uint8 type #11 is to start UWB following
   uint8 relative_pos # Relative orientation, 200 chooses the following position independently, 201 follows behind the target, 202 follows on the left side of the target, 203 follows on the right side of the target
   float32 keep_distance #The distance to keep from the following target
}
```

#### Request accepted

```Nginx
NameCode=6004

data
{
   uint8 accepted #When the request is accepted, it is 1 and the task continues to run; when the request is not accepted, it is 2 and the task is terminated
}
```

#### Intermediate feedback

These three feedback codes apply to all 6004 tasks: 1000 is activating the dependent node, 1001 activates the dependent node successfully, and 1002 fails to activate the dependent node.

```Nginx
NameCode=6004

data
{
   int32 feedback_code #Feedback code, indicating the status of the following task, 10 is normal, 11 detector is abnormal, 12 coordinate conversion is abnormal, 13 planner is abnormal, 14 controller is abnormal, 15 the target is empty when UWB is started, 16 the step jump is triggered during UWB following, 17UWB following triggered autonomous behavior, 18UWB following autonomous behavior was abnormal
   string feedback_msg # Additional message, currently redundant
}
```

#### Return results

```Nginx
NameCode=6004

data
{
   uint8 result #Visual following task result, 0 means success (will not appear), non-0 means other
}
```

## Access ongoing tasks (access tasks started by 6004)

For tasks started by 6004, if the intermediate APP exits and the task is still in progress when re-entering, you can access through this request. The interface is similar to 6004.

ros interface: action, type: protocol/action/Navigation, name: "start_algo_task"

####Download command

```Nginx
NameCode=6010

params
{
   uint8 type #type of task
}
```

#### Request accepted

```Nginx
NameCode=6010

data
{
   uint8 accepted #1 is accepted. If it is 3, it means that the task has ended when connecting, or this type of task is not recorded (it may have ended in the previous app connection state)
}
```

#### Intermediate feedback

Consistent with the task performed

```Nginx
NameCode=6010

data
{
   int32 feedback_code
   string feedback_msg
}
```

#### Return results

Consistent with the task performed

```Nginx
NameCode=6010

data
{
   uint8 result
}
```

## Stop the task (stop the task started by 6004)

All tasks started by 6004 can be manually stopped by 6009

ros interface: service, type: protocol/srv/StopAlgoTask, name: "stop_algo_task"

### Stop the current task

####Download command

```Nginx
NameCode=6009

params
{
   uint8 type # is consistent with the type when the corresponding task is started. If all tasks are stopped, fill in 0
   string map_name #If it is a mapping task, the map name needs to be written when stopping.
}
```

Type definition reference: [Algorithm Task Management Interface](https://xiaomi.f.mioffice.cn/docs/dock4nE0rvOLin8zA5xjN1MSlpc)

#### Return results

```Nginx
NameCode=6009

data
{
   int8 result #Whether the stop command is accepted, 0 has stopped, 1 stops failed, 10 request timed out and no response
}
```

## Bluetooth and UWB

The UWB connection needs to be established through Bluetooth. The operations on Bluetooth are provided here. Bluetooth connection and interruption will include the corresponding operations of UWB.

### Scan for Bluetooth devices

Scan low-power Bluetooth devices and return the device information list (devices without names will not be returned). When the Bluetooth connected state is lowered, this request will return the results of the last scan (because experiments have shown that scanning in the connected state will cause Bluetooth to be disconnected later. exception)

ros interface: service, type: protocol/srv/BLEScan, name: "scan_bluetooth_device"

####Download command

```Nginx
NameCode=8001

param
{
   float64 scan_seconds #Scan time (seconds)
}
```

#### Return results

```Nginx
NameCode=8001

data
{
   DeviceInfo device_info_list[] #Scan result information list
}
```

##### DeviceInfo structure

```Nginx
string mac #mac address
string name #Bluetooth device name
string addr_type #mac address type, may be random or public
uint8 device_type #uwb device type, 0 unknown, 16 bracelet, 17 charging pile, unconnected device type unknown
string firmware_version #Firmware version, connected devices have correct version readings
float battery_level #Battery power, connected devices can have correct readings, range: 0~1
```

### Obtain the information of connected Bluetooth devices

ros interface: service, type: protocol/srv/BLEScan, name: "scan_bluetooth_device"

####Download command

```Nginx
NameCode=8001

param
{
   float64 scan_seconds #Fill in 0 to obtain history records
}
```

#### Return results

```Nginx
NameCode=8001

data
{
   DeviceInfo device_info_list[] #Historical device information list
}
```

### Connect Bluetooth device

After scanning the Bluetooth device, connect the selected Bluetooth device. After connecting the Bluetooth device, the UWB device connection will be made.

ros interface: service, type: protocol/srv/BLEConnect, name: "connect_bluetooth_devices"

####Download command

```Nginx
NameCode=8002

param
{
   DeviceInfo selected_device #Selected device name
}
```

#### Return results

```Nginx
NameCode=8002

data
{
   uint8 result #Connection result, 0 successful, 1 Bluetooth connection failed, 2 UWB connection failed, 3 failed to obtain UWB mac and session id
}
```

### Disconnect Bluetooth device

Disconnecting the Bluetooth device will also disconnect the corresponding UWB device.

ros interface: service, type: protocol/srv/BLEConnect, name: "connect_bluetooth_device"

####Download command

```Nginx
NameCode=8002

param
{
  #Empty means disconnecting the device
}
```

#### Return results

```Nginx
NameCode=8002

data
{
   uint8 result #Disconnection result, 0 success, 1 Bluetooth communication error, 2UWB disconnection failure, 3 current Bluetooth is not connected
}
```

### Connection status update

Abnormal disconnection and automatic reconnection of Bluetooth peripherals are reported.

ros interface: topic, type: std_msgs/msg/Bool, name: "bluetooth_disconnected_unexpected"

#### Upload instructions

```Nginx
NameCode=8003

param
{
     bool disconnected #true means abnormal disconnection, false means automatic reconnection is successful
}
```

### Get the currently connected device

ros interface: service, type: protocol/srv/BLEScan, name: "get_connected_bluetooth_info"

####Download command

```Nginx
NameCode=8004

param
{
}
```

#### Return results

```Nginx
NameCode=8004

data
{
   DeviceInfo device_info_list[] #Currently only supports connecting to one device, so this list has only one or zero elements.
}
```

### Get the currently connected device firmware version

ros interface: service, type: std_srvs/srv/Trigger, name: "ble_device_firmware_version"

####Download command

```Nginx
NameCode=8005

param
{
}
```

#### Return results

```Nginx
NameCode=8005

data
{
   bool success #true means successful acquisition, false means the device is not connected
   string message #version number
}
```

### Get the current power of the connected device

ros interface: service, type: protocol/srv/GetBLEBatteryLevel, name: "ble_device_battery_level"

####Download command

```Nginx
NameCode=8006

param
{
}
```

#### Return results

```Nginx
NameCode=8006

data
{
   bool connected #true means the device is connected, false means the device is not connected
   float32 presentage #Battery: 0.0~1.0
}
```

### Delete historical connection records

ros interface: service, type: nav2_msgs/srv/SaveMap, name: "delete_ble_devices_history"

####Download command

```Nginx
NameCode=8007

param
{
     string mac #To delete the mac address of the device, if not filled in, all records will be deleted
}
```

#### Return results

```Nginx
NameCode=8007

data
{
   bool success #Whether the deletion was successful
}
```

### Bluetooth firmware upgrade tips

When the Bluetooth peripheral is connected and the mobile APP is connected, it will check whether there is adapted firmware, and if so, this prompt will be sent.

ros interface: topic, type: std_msgs/msg/String, name: "ble_firmware_update_notification"

#### Uplink command

```Nginx
NameCode=8008

param
{
   string data # The format is "old version new version", separated by spaces.
}
```

### Update Bluetooth firmware

After receiving the prompt, you can issue an upgrade confirmation and start the upgrade process.

ros interface: service, type: std_srvs/srv/Trigger, name: "update_ble_firmware"

####Download command

```Nginx
NameCode=8009

param
{
}
```

#### Return results

```Nginx
NameCode=8009

data
{
   bool success # Whether the upgrade was successful
   string message # Error message
}
```

### Bluetooth firmware update progress

Continue to send this information during the Bluetooth firmware upgrade process

ros interface: topic, type: protocol/msg/BLEDFUProgress, name: "ble_dfu_progress"

#### Uplink command

```Nginx
NameCode=8010

param
{
   uint8 status # Status: 0 is being upgraded, 1 is upgraded successfully, 2 is upgraded failed, 3 is decompressing the file, 4 fails to decompress the file, 5 is sending the initialization package, 6 fails to send the initialization package, 7 is sending the firmware image, 8 fails to send the firmware image , 9 is updating firmware and waiting for restart, 10 failed to restart successfully
   float64 progress # Progress: 0~1
   string message # Status information, status-related messages, such as: specific upgrade steps, failure reasons, etc. You can ignore it if not needed
}
```

### Bluetooth remote control speed gear related

#### Set the remote control speed gear

ros interface: topic, type: std_msgs/msg/Int8, name: "set_bluetooth_tread"

#####Download command

```Nginx
NameCode = 8011

param
{
     int8 data #0 low gear, 1 mid gear, 2 high gear
}
```

#### Notification of remote control gear change report

ros interface: topic, type: std_msgs/msg/Int8, name: "update_bluetooth_tread"

##### Uplink command

```Nginx
NameCode=8012

param
{
     int8 data #0 low gear, 1 mid gear, 2 high gear
}
```

#### Query the speed gear of the remote control

ros interface: service, type: std_srvs/srv/Trigger, name: "get_bluetooth_tread"

#####Download command

```Nginx
NameCode=8013

param
{
}
```

##### Return results

```Nginx
NameCode=8013

data
{
     int8 data #0 low gear, 1 mid gear, 2 high gear
}
```

## Robot status

### Status Query

The status included in the heartbeat can be obtained in the form of query through this interface. Please refer to the format of the heartbeat.

This request only involves grpc nodes, so there is no ros interface

####Download command

```Nginx
NameCode=10001

param
{
}
```

#### Return results

```Nginx
NameCode=10001

data
{
   MotionStatus motion_status
   TaskStatus task_status
   SelfCheckStatus self_check_status
   StateSwitchStatus state_switch_status
   ChargingStatus charging_status
}
```

in:

##### MotionStatus structure

```Nginx
int32 motion_id
```

##### TaskStatus structure

```Go
uint32 task_status
int32 task_sub_status
```

##### SelfCheckStatus structure

```Go
int32 code
string description
```

##### StateSwitchStatus structure

```Go
int32 state
int32 code
```

##### ChargingStatus structure

```C%2B%2B
bool wired_charging
bool wireless_charging
```

### Low power consumption

#### Exit low power consumption

APP can manually exit low power consumption without waking up. Note: Low power consumption will completely exit when this request returns. It is recommended to set the timeout to 15 seconds.

ros interface: service, type: std_srvs/srv/Trigger, name: "low_power_exit"

#####Download command

```Nginx
NameCode=10002

param
{
}
```

##### Return results

```Nginx
NameCode=10002

data
{
     bool success #Whether the exit was successful
}
```

#### Turn on/off automatic entry into low power consumption

After shutting down, the robot dog will not automatically enter low-power mode when lying down.

ros interface: service, type: std_srvs/srv/SetBool, name: "low_power_onoff"

#####Download command

```Nginx
NameCode=10003

param
{
     bool data #true means on, false means off
}
```

##### Return results

```Nginx
NameCode=10003

data
{
     bool success #Whether the setting is successful
}
```

## Environment switching

ros interface: service, type: bridge/protocol/ros/srv/Trigger.srv, name: "set_work_environment"

###Download command

```Nginx
NameCode=10004

param
{
     string data #Production environment: pro; test environment: test
}
```

### Return results

```Nginx
NameCode=10004

data
{
     bool success # true: success; false: failure
     string message
}
```

## Syslog log reporting

ros interface: service, type: bridge/protocol/ros/srv/BesHttpSendFile.srv, name: "upload_syslog"

###Download command

```Nginx
NameCode=10005

param
{
}
```

### Return results

```Nginx
NameCode=10005

data
{
     bool success # true: success; false: failure
}
```

## Protocol for testing

### Disconnect app

ros interface: topic, type: std_msgs/msg/Bool, name: "disconnect_app"

####Download command

```Nginx
NameCode = 55001

params
{
}
```

#### Return results

```Nginx
NameCode = 55001

data
{
}
```

## Developer permissions

### Unlock command

####Download command

```C%2B%2B
NameCode=9030

params
{
    string=httplink
}
```

#### Return results

```C%2B%2B
NameCode=9030

params
{
    int result;
}
```

### Restart command

####Download command

```C%2B%2B
NameCode = 9031

params
{
    string = SystemRestart
}
```

#### Return results

```C%2B%2B
NameCode = 9031

params
{
    int result; #
                 #255 failed
}
```

### Shutdown and restart instructions

ros interface: service, type: "std_srvs/srv/SetBool", name: "enable_elec_skin"


## Dogleg calibration

The user manually calibrates the dogleg, first lies down and cuts off the power, then straightens it and then powers on and stands up.

ros interface: service, type: "std_srvs/srv/SetBool", name: "dog_leg_calibration"

###Download command

```YAML
NameCode = 12001

param
{
     bool data # false lie down and power off true power on and stand up
}
```

### Return results

```Go
NameCode = 12001

data
{
     bool success
     string message
}
```
