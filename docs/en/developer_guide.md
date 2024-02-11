# Developer Manual

#Basic abilities

## Sports function

### Servo command issuance

- Specific functions: Continuously accept movement instructions to complete related actions, and can dynamically respond to speed, gait and other parameters, including the following actions:
   - Walking with a slow-frequency gait (maximum longitudinal speed 0.65 (unit m/s, the same below), maximum lateral speed 0.3, maximum angular speed 1.25 (unit rad/s, the same below))
   - Walking with a medium frequency gait (maximum longitudinal speed 1.0, maximum lateral speed 0.3, maximum angular speed 1.25)
   - Walking with a fast gait (maximum longitudinal speed 1.6, maximum lateral speed 0.55, maximum angular speed 2.5)
   - Walking with all four legs off the ground simultaneously (maximum longitudinal speed 0.25, maximum lateral speed 0.1, maximum angular speed 0.5)
   - Walking with the front and rear feet alternately off the ground (maximum longitudinal speed 0.4, maximum lateral speed 0.3, maximum angular speed 0.75)
- Interface form: ros topic
- Interface name: "motion_servo_cmd"
- Message file: protocol/msg/MotionServoCmd.msg
- Message content:


```js
int32 motion_id            # Robot motion control posture
int32 cmd_type             # Instruction type constraints, 1: Data, 2: End;
                           # The Data frame writes the following parameters according to the control requirements, and the End frame does not need to fill in the parameters.
int32 cmd_source           # Command source,
                           # 0: App, 1: Audio, 2: Vis, 3: BluTele 4: Algo
int32 value                # 0, inward gait, 2, vertical gait
float32[3] vel_des         # x y (maximum value 1.5) yaw (maximum value 2.0) speed m/s
float32[3] rpy_des         # Currently not available. roll pitch yaw (maximum value 0.4) rad
float32[3] pos_des         # Currently not open. x y (maximum value 0.2) z (maximum value 0.3) m
float32[3] acc_des         # Currently not available. acc for jump m^2/s
float32[3] ctrl_point      # Currently not available. pose ctrl point m
float32[3] foot_pose       # Currently not open. front/back foot pose x,y,z m
float32[2] step_height     # The leg lifting height when walking, the default can be set to 0.05m
```

### Servo command feedback

- Specific function: Feedback to the user the current servo command execution status (including the current gait and whether the execution result of the servo command is correct)
- Interface form:    ros topic
- Interface name:    "motion_servo_response"
- Message file:      protocol/msg/MotionServoResponse.msg
- Message content:

```js
int32 motion_id
int8 order_process_bar   # Action execution progress
int8 status              # Status of motion control gait switching
bool result              # Whether the action was executed successfully
int32 code               # Error code when exception occurs
```

### Result command

- Specific functions: In the form of client/server, it accepts motion instructions, completes all preset actions, returns the execution results of all actions, and reports all abnormal causes, including motor abnormalities, state machine abnormalities, operation control abnormalities, etc... At the same time, actions can also be customized by users in visual programming. Currently, the customizable parameters are relatively limited, including the foothold position, the flight time of the legs, and the overall speed of the robot dog. After the action is defined, it can be called in visual programming using the same interface as the built-in action. Built-in actions include:

| motion_id | motion name | motion_id | motion name | motion_id | motion name |
| --------- | -------------- | --------- | -------------- | --------- | ---------------- |
| 0 | Emergency stop | 131 | 3D jump and turn right 90 degrees | 144 | Butt circle |
| 101 | High damping lying down | 132 | 3D jump forward 60cm | 145 | Draw a circle with the head |
| 111 | Return to standing | 134 | 3D jump left jump 20cm | 146 | Stretching |
| 118 | Strengthen getting up | 135 | 3D jump right jump 20cm | 151 | Ballet |
| 121 | Backflip | 140 | Dance Set | 152 | Moonwalk |
| 123 | Bow | 141 | Hold left hand | 174 | Push-ups |
| 125 | Walking the dog | 142 | Shaking the right hand | 175 | Showing respect |
| 130 | 3D jump and turn left 90 degrees | 143 | Sit down | | |

- Interface form:  ros service
- Interface name:  "motion_result_cmd"
- Service file:    protocol/srv/MotionResultCmd.srv
- Service Content:

```js
int32 motion_id           # Robot motion control posture
int32 cmd_source          # Command source,
                          # 0: App, 1: Audio, 2: Vis, 3: BluTele 4: Algo
float32[3] vel_des        # Currently not available. x y (maximum value 1.5, m/s) yaw (maximum value 2.0, rad/s)
float32[3] rpy_des        # roll pitch yaw (maximum value 0.4) rad
float32[3] pos_des        # x y (maximum value 0.2) z (maximum value 0.3) m
float32[3] acc_des        # Currently not available. acc for jump m^2/s
float32[3] ctrl_point     # Currently not available. pose ctrl point m
float32[3] foot_pose      # Currently not open. front/back foot pose x,y,z m
float32[2] step_height    # Leg lifting height, currently can be set by 0.05m
int32 duration            # Expectations set when performing incremental position control, incremental force control, and absolute force control posture control
                          # Complete time
---
int32 motion_id           # Robot motion control posture
bool result               # Execution result
int32 code                # module code
```

##Visual function

### Camera Service

**CameraService.srv** 

```Go
uint8 SET_PARAMETERS        = 0
uint8 TAKE_PICTURE          = 1
uint8 START_RECORDING       = 2
uint8 STOP_RECORDING        = 3
uint8 GET_STATE             = 4
uint8 DELETE_FILE           = 5
uint8 GET_ALL_FILES         = 6
uint8 START_LIVE_STREAM     = 7
uint8 STOP_LIVE_STREAM      = 8
uint8 START_IMAGE_PUBLISH   = 9
uint8 STOP_IMAGE_PUBLISH    = 10
uint8 command

# command arguments
string args
uint16 width
uint16 height
uint16 fps

---
uint8 RESULT_SUCCESS         = 0
uint8 RESULT_INVALID_ARGS    = 1
uint8 RESULT_UNSUPPORTED     = 2
uint8 RESULT_TIMEOUT         = 3
uint8 RESULT_BUSY            = 4
uint8 RESULT_INVALID_STATE   = 5
uint8 RESULT_INNER_ERROR     = 6
uint8 RESULT_UNDEFINED_ERROR = 255
uint8 result
string msg
int32 code
```

#### RGB and fisheye cameras

- Turn on and off camera

```Bash
ros2 launch camera_test stereo_camera.py

#View name space
ros2 node list

###If it starts automatically at boot, please add the namespace before the topic.
ros2 lifecycle set /stereo_camera configure
ros2 lifecycle set /stereo_camera activate
ros2 lifecycle set /stereo_camera deactivate
ros2 lifecycle set /stereo_camera cleanup
```

- Get images

Interface form: ros topic
Interface name: camera_server
Topic name:

```Bash
#Use topic to subscribe to rgb camera, left fisheye camera, right fisheye camera

#left eye
/image_left

#right eye
/image_right

#rgbcamera
/image_rgb
```

Image data topic content: sensor_msgs::msg::Image

#### AI Camera

- Turn on and off camera

```Bash
ros2 run camera_test camera_server

##Enable AI camera command
ros2 service call /camera_service protocol/srv/CameraService "{command: 9, width: 640, height: 480, fps: 0}"

##Close camera command
ros2 service call /camera_service protocol/srv/CameraService "{command: 10, args: ''}"
```

- Get images

Interface form: ros topic
Interface name: camera_server
Topic name:

```Bash
#Use topic to subscribe to AI camera

#AIcameratopic
topic: /image
```

Image data topic content: sensor_msgs::msg::Image

 
#### Realsense 

- Control the opening and closing of RealSense data through the lifecycle state machine

```Bash
   #/camera/camera node startup command
   ros2 launch realsense2_camera on_dog.py

   #initialization
   ros2 lifecycle set /camera/camera configure

   #Open data
   ros2 lifecycle set /camera/camera activate

   #Close data
   ros2 lifecycle set /camera/camera deactivate

   #Reset node
   ros2 lifecycle set /camera/camera cleanup

   #/camera/camera_align node startup command
   ros2 launch realsense2_camera realsense_align_node.launch.py

   #initialization
   ros2 lifecycle set /camera/camera_align configure

   #Open data
   ros2 lifecycle set /camera/camera_align activate

   #Close data
   ros2 lifecycle set /camera/camera_align deactivate

   #Reset node
   ros2 lifecycle set /camera/camera_align cleanup
```

- Get images

Interface form: ros topic
Interface name: realsense2_camera_node, realsense_align_node
Topic name:

```Bash
---You need to add a domain name to start automatically at boot.

#lefteyeimage
/camera/infra1/image_rect_raw

#righteyeimage
/camera/infra1/image_rect_raw

#Depth map
/camera/depth/image_rect_raw

#IMUdata
/camera/imu

#aligndepth map
/camera/aligned_depth_to_extcolor/image_raw
```

Image data topic content

```js
# Header (timestamp and frame)
std_msgs/Header header

# image height, that is, number of rows
uint32 height

# image width, that is, number of columns
uint32 width

# Encoding of pixels -- channel meaning, ordering, size
string encoding

# is this data bigendian?
uint8 is_bigendian

# Full row length in bytes
#uint32 step

# actual matrix data, size is (step * rows)
#uint8[] data
```

IMU data topic content

```Bash
#Header (timestamp and frame)
std_msgs/Header header

#orientation
geometry_msgs/Quaternion orientation

#orientation_covariance
double[9] orientation_covariance

#angular_velocity
geometry_msgs/Vector3 angular_velocity

#angular_velocity_covariance
float64[9] angular_velocity_covariance

#linear_acceleration
geometry_msgs/Vector3 linear_acceleration

#linear_acceleration_covariance
float64[9] linear_acceleration_covariance
```

### Face entry and recognition

- Specify the input nickname and input the face through voice interaction.
- At the appropriate angle and distance, faces entered in the database can be recognized.

#### Face entry

- Specific functions: Through voice interaction, enter a face with a specified nickname, and the information entered on the face is unique.
- Interface form: ros service/topic The current face entry module provides a service and a topic. The service is used to activate the face entry function. After activation, within the specified timeout, the node will publish the topic whether the face has been successfully entered. .
- Interface name:

service: "cyberdog_face_entry_srv"
topic: "face_entry_msg"

- Service file: protocol/srv/FaceEntry.srv
- Service file content

```Go
# request
int32 ADD_FACE               = 0    # Add face
int32 CANCEL_ADD_FACE        = 1    # Cancel adding face
int32 CONFIRM_LAST_FACE      = 2    # Confirm the last face
int32 UPDATE_FACE_ID         = 3    # Update face ID
int32 DELETE_FACE            = 4    # Delete face
int32 GET_ALL_FACES          = 5    # Get all faces
int32 command                       # Face input command
string username                     # User name
string origine                      # User’s original name
bool ishost                         # Whether it is the owner
---
int32 RESULT_SUCCESS         = 0    # Request for entry service successful
int32 RESULT_INVALID_ARGS    = 5910 # The input parameters are invalid parameters
int32 RESULT_UNSUPPORTED     = 5908 # Does not support entry
int32 RESULT_TIMEOUT         = 5907 # Input timeout
int32 RESULT_BUSY            = 5911 # Input busy
int32 RESULT_INVALID_STATE   = 5903 # Input invalid status
int32 RESULT_INNER_ERROR     = 5904 # Internal error
int32 RESULT_UNDEFINED_ERROR = 5901 # Unknown error
int32 result                        # Request entry service result
string allfaces                     # Get all faces
```

- Message file: protocol/msg/FaceEntryResult.msg
- Message file content:

```Go
// protocol/msg/FaceEntryResult.msg
int32 RESULT_SUCCESS            = 0    # The entry result is successful
int32 RESULT_TIMEOUT            = 5907 # Input timeout
int32 RESULT_FACE_ALREADY_EXIST = 5921 # Face already exists
int32 result                           # Input result
string username                        # Entered name
```

#### Face recognition

- Specific function: Recognize faces entered in the database.
- Interface form: ros service/topic The current face recognition module provides a service and a topic. The service is used to activate the face recognition function. After activation, within the specified timeout period, the node will publish the topic that detects the face ID. .
- Interface name:

service: "cyberdog_face_recognition_srv"

topic: "face_rec_msg"

- Service file: protocol/srv/FaceRec.srv
- Service file content

```Go
#request
int32 COMMAND_RECOGNITION_ALL    = 0    # Request to identify everyone
int32 COMMAND_RECOGNITION_SINGLE = 1    # Request to identify a person
int32 COMMAND_RECOGNITION_CANCEL = 2    # Cancel face recognition
int32 MAX_TIMEOUT                = 300  # Identify the maximum duration
int32 MIN_TIMEOUT                = 30   # Identify the hour length
int32 DEFAULT_TIMEOUT            = 60   # Default recognition duration
int32 command                           # Request identification command
string username                         # Name of face recognition
string id                               # Identify the id of the face
int32 timeout                           # Valid time 30s～300s, default = 60
                                        # If this field is not added, the default value will be used
---
int32 ENABLE_SUCCESS             = 0    # Request identification successful
int32 ENABLE_FAIL                = 5901 # Request identification failed
int32 result                            # Request identification result
```

- Message file: protocol/msg/FaceRecognitionResult.msg
- Message file content:

```C%2B%2B
// protocol/msg/FaceRecognitionResult.msg
int32 RESULT_SUCCESS              = 0    # Identification successful
int32 RESULT_TIMEOUT              = 5907 # Identification timeout
string username                          # The name of the recognized face
int32 result                             # Recognition result
string id                                # The ID of the face recognized
float32 age                              # Recognized age
float32 emotion                          # Recognized emotion
```

### Dynamic gesture recognition

- Specific functions: Consecutive gesture action recognition, including palm/finger waving to the left, palm/finger waving to the right, palm/finger moving up, palm/finger moving down, palm/finger opening, palm/finger closing.
- Interface form: ros service/topic The gesture recognition module currently provides a service and a topic. The service is used to activate the gesture recognition function. After activation, within the specified timeout, the node will publish the topic of the detected gesture action ID.
- Interface name:

service: "gesture_action_control"

topic: gesture_action_msg

- Service file: protocol/srv/GestureActionControl.srv
- Service file content

```C%2B%2B
// protocol/srv/GestureActionControl.srv
uint8 START_ALGO = 0
uint8 STOP_ALGO = 1
int32 DEFAUT_TIMEOUT = 60 #Algorithm duration defaults to 60s
uint8 command #Open or stop gesture action recognition algorithm
int32 timeout #The algorithm duration is valid for (1s-300s), this keyword is ignored in the request
                                         #The value range is not (1s-300s) and the algorithm duration defaults to 60s.
---
int32 RESULT_SUCCESS = 0 # Request success receipt
int32 RESULT_BUSY = 1 # Repeat request to turn on/off algorithm request receipt
int32 code #request receipt
```

- Message file: protocol/msg/GestureActionResult.msg
- Message file content

```C%2B%2B
// protocol/msg/GestureActionResult.msg
int32 NO_GESTURE =0 #No gesture
int32 PULLING_HAND_OR_TWO_FINGERS_IN =1 #Pull your palm closer
int32 PUSHING_HAND_OR_TWO_FINGERS_AWAY =2 #Push away with palm
int32 SLIDING_HAND_OR_TWO_FINGERS_UP = 3 #Raise hand up
int32 SLIDING_HAND_OR_TWO_FINGERS_DOWN =4 #Hand down
int32 SLIDING_HAND_OR_TWO_FINGERS_LEFT =5 #Push your hand to the left
int32 SLIDING_HAND_OR_TWO_FINGERS_RIGHT =6 #Push your hand to the right
int32 STOP_SIGN =7 #Stop gesture
int32 THUMB_UP =8 #Thumbs up
int32 ZOOMING_IN_WITH_HAND_OR_TWO_FINGERS = 9 #Open your palm or fingers
int32 ZOOMING_OUT_WITH_HAND_OR_TWO_FINGERS =10 #Close the palm or fingers
int32 THUMB_DOWN =11 #Thumbs down
int32 id #gesture recognition result id
```

### Expression recognition

- Specific function: Recognize the expression of the target person in the scene, including five expressions: neutral, smiling, laughing, angry, and crying, as shown below:

| Neutral | Smile | Laugh | Angry | Cry |
| -------------------------------------------------- ---------- | --------------------------------------- ---------------------------- | ---------------------------- ---------------------------------- | ---------------- -------------------------------------------------- | ----- -------------------------------------------------- ----- |
| ![img](./image/developer_guide/image2.png) | ![img](./image/developer_guide/image3.png) | ![img](./image/developer_guide/image4.png) | ! [img](./image/developer_guide/image5.png) | ![img](./image/developer_guide/image6.png) |

- Interface form: ros service + topic Call service to enable the face recognition algorithm. In the request, algo_enable passes in the parameter ALGO_FACE. Obtain the face recognition results through the topic, where the face_info.info[i].emotion of the topic person represents the expression information of the corresponding face.
- Interface name/service file/file location: See target tracking for unified information.

### Age prediction

- Specific function: It can predict the age of the target person in the scene, and the output age range is 0-80.
- Interface form: ros service + topic Call service to enable the face recognition algorithm. In the request, algo_enable passes in the parameter ALGO_FACE. Obtain the face recognition results through the topic, where the face_info.info[i].age of the topic person represents the age information of the corresponding face.
- Interface name/service file/file location: See target tracking for unified information.

### Human skeleton point detection

- Specific functions: detect the key point positions of all people in the scene, and output the coordinates of 17 key points for a single target person, which are: nose, left_eye, right_eye, left_ear, right_ear, left_shoulder, right_shoulder, left_elbow, right_elbow, left_wrist, right_wrist, left_hip, right_hip, left_knee, right_knee, left_ankle, right_ankle, the corresponding key point positions are as follows:

![img](./image/developer_guide/image1.png)

- Interface form: ros service + topic Call service to enable the key point detection algorithm. In the request, algo_enable passes in the parameters ALGO_BODY and ALGO_KEYPOINTS. Obtain the key point detection results through the topic, where body_info.infos[i].keypoints in the topic person is the location information of the key points.
- Interface name/service file/file location: See target tracking for unified information.

### Static gesture recognition

- Specific functions: Recognize the gestures of all people in the scene. A single target person corresponds to a gesture. The gesture categories and definitions are as follows:

  ![image-20230522152113978](./image/developer_guide/image-20230522152113978.png)

- Interface form: ros service + topic Call service to enable the gesture recognition algorithm. In the request, algo_enable passes in the parameters ALGO_BODY and ALGO_GESTURE. Get the gesture recognition result through topic, where body_info.infos[i].gesture in topic person is the result of gesture recognition.
- Interface name/service file/file location: See target tracking for unified information.

### Human body detection

- Specific functions: Detect all human bodies in the scene and give the location and confidence information of each human body.
- Interface form: ros service + topic Call service to enable the human body detection algorithm, and pass in the parameter ALGO_BODY to algo_enable in the request. Obtain the human body detection results through the topic, where body_info.infos[i] in the topic person is the human body detection result.
- Interface name/service file/file location: See target tracking for unified information.

### Target Tracking 

- It can achieve long-term stable tracking of designated targets (human body, basketball, robot dog, toy car, etc.). After the tracking target goes out of view or cross-occlusion reappears, it can be retrieved within a limited time.
- Interface form: ros service + topic Call service to enable the human body tracking algorithm for human body tracking. In the request, algo_enable passes in the parameters ALGO_BODY and ALGO_REID. To track targets other than the human body, call service to enable the tracking algorithm for all objects, and pass in the parameter ALGO_FOCUS in the request algo_enable. Obtain the tracking results of the human body and all things through the topic, where track_res in the topic person is the tracking result of the human body and all things.
- Interface name:

      service: "algo_manager"

      topic: "person"

- File location:

      protocol/srv/AlgoManager.srv

      protocol/msg/AlgoList.msg

      protocol/msg/Person.msg

      protocol/msg/FaceInfo.msg

      protocol/msg/Face.msg

      protocol/msg/BodyInfo.msg

      protocol/msg/Body.msg

      protocol/msg/Keypoint.msg

      protocol/msg/Gesture.msg

      protocol/msg/TrackResult.msg

- document content:

```js
# protocol/srv/AlgoManager.srv

# request

AlgoList[] algo_enable

AlgoList[] algo_disable

# param of face (reserved field, not used yet)

bool open_age

bool open_emotion



---



#response

uint8 ENABLE_SUCCESS = 0

uint8 ENABLE_FAIL = 1

uint8 result_enable



uint8 DISABLE_SUCCESS = 0

uint8 DISABLE_FAIL = 1

uint8 result_disable
# protocol/msg/AlgoList.msg

uint8 ALGO_FACE = 0

uint8 ALGO_BODY = 1

uint8 ALGO_GESTURE = 2

uint8 ALGO_KEYPOINTS = 3

uint8 ALGO_REID = 4

uint8 ALGO_FOCUS = 5



uint8 algo_module
# protocol/msg/Person.msg

# frame header

std_msgs/Header header



# face info (face recognition results)

FaceInfo face_info



# body info

BodyInfo body_info



# auto track

TrackResult track_res
# protocol/msg/FaceInfo.msg

# face info

# frame header

std_msgs/Header header



# number of faces

uint32 count



# face descriptions

Face[] infos
# protocol/msg/Face.msg

# single face info

# face locations in image

sensor_msgs/RegionOfInterest roi



#faceid

string id



#confidence

float32 score



# matching degree

float32 match



#facepose

float32 yaw

float32 pitch

float32 row



#ishost

bool is_host



float32 age

float32 emotion
# protocol/msg/BodyInfo.msg

# body info

# frame header

std_msgs/Header header



# number of bodies

uint32 count



# body descriptions

Body[] infos
# protocol/msg/Body.msg

# single body info

# body locations in image

sensor_msgs/RegionOfInterest roi



# body REID

string reid



#features

float32[] feats



#keypoints

Keypoint[] keypoints



#gesture

Gesture gesture
# protocol/msg/Keypoint.msg

float32x

float32y
# protocol/msg/Gesture.msg

#gesturerect

sensor_msgs/RegionOfInterest roi



#gesturecls

int32 GESTURE_OK = 0

int32 GESTURE_FAST_BACKWARD = 1

int32 GESTURE_FAST_FORWARD = 2

int32 GESTURE_STOP_LEFT = 3

int32 GESTURE_STOP_RIGHT = 4

int32 GESTURE_THUMBS_UP = 5

int32 GESTURE_SHHH = 6

int32 GESTURE_FIST = 7

int32 GESTURE_PALM2FIST = 8

int32 GESTURE_INVALID = 9

int32 cls
# protocol/msg/TrackResult.msg

# frame header

std_msgs/Header header



#rect tracked

sensor_msgs/RegionOfInterest roi
```

## Peripherals and sensors

### touch gesture recognition

- Supports multiple gesture touch recognition such as single click, double click, and long press.
- Based on the value of touch_status, the function of long-pressing to trigger networking and double-clicking to report the battery level is realized.

#### touch gesture acquisition

Specific function: touch gesture recognition acquisition

Interface form: ros topic

Interface name: "touch_status"

Interface content:

```Go
std_msgs/Header header



int32 touch_state // 0x01: single click, 0x03: double click, 0x07: long press

uint64 timestamp // timestamp
```

### Networking function

- Have wireless network card and the ability to access the Internet
- Ability to check network status

### uwbdata

- Initialize UWB sensor firmware
- Enable UWB sensor data collection
- Turn off UWB sensor data collection
- 4 UWB sensor data information

##### uwb data release

- Interface form: ros topic
- Interface name: "uwb_raw"
- Topic file: protocol/msg/UwbRaw
- Topic content:

```js
std_msgs/Header header

         builtin_interfaces/Time stamp

                 int32 sec

                 uint32 nanosec

         string frame_id



float32 dist

float32 angle

float32 n_los

float32 rssi_1

float32 rssi_2
```

### Ultrasound data

- Function description: Obtain the range value and intensity value of the target detected by the ultrasonic sensor.
- Interface form: ros topic
- Interface name: ultrasonic_payload
- Transmission frequency: 10hz
- Interface content:

"sensor_msgs/msg/Range.msg"

```C%2B%2B
# Contains the timestamp and sequence information of the topic. For details, see std_msgs/Header

Header header

    

# Field definitions for specific sensors

uint8 ULTRASOUND=0

uint8 INFRARED=1

# Sensor types include ultrasonic 0 and infrared 1

uint8 radiation_type

# Use the fov of the sensor

float32 field_of_view

# Minimum distance that the sensor can detect [m]

float32 min_range

# Maximum distance that the sensor can detect [m]

float32 max_range

#Ranging value returned by the sensor [m]

float32 range
```

It quotes the message std_msgs/Header that comes with the ros2 system.

```C%2B%2B
# Standard metadata for higher-level stamped data types.

# This is generally used to communicate timestamped data

# in a particular coordinate frame.

#

# sequence ID: consecutively increasing ID

uint32 seq

#Two-integer timestamp that is expressed as:

# * stamp.sec: seconds (stamp_secs) since epoch (in Python the variable is called 'secs')

# * stamp.nsec: nanoseconds since stamp_secs (in Python the variable is called 'nsecs')

# time-handling sugar is provided by the client library

time stamp

#Frame this data is associated with

string frame_id
```

 

### tof data

- Function description: There are four TOF sensors, two each controlled by the head MCU and the tail MCU. A single TOF can obtain 8*8 matrix elevation data
- Interface form: ros topic
- Interface name: topic_name: head_tof_payload, rear_tof_payload
- Transmission frequency: 10hz
- Data description: The currently set effective distance of TOF is 150-660mm. If it is less than 150mm, the displayed value is 0.150; if it is greater than 660mm, the displayed value is the maximum value of 0.660m;
- Interface content:

"HeadTofPayload.msg

```C%2B%2B
# This message is used to describe head tofs



SingleTofPayload left_head

SingleTofPayload right_head
```

"RearTofPayload.msg"

```C%2B%2B
# This message is used to describe rear tofs



SingleTofPayload left_rear

SingleTofPayload right_rear
```

It refers to the customized "SingleTofPayload.msg"

```C%2B%2B
# Contains the timestamp and sequence information of the topic. For details, see std_msgs/Header

# This message is  used to describe single tof payload

# Send frequency: 10

# The effective distance currently set by TOF is 150-660mm, less than 150mm, the displayed value is 150; 

# greater than 660mm, the displayed value is the maximum value of 660mm;

# At present, there is a tof sensor on each of the four legs of the dog, which is used to Obtain the elevation information 



std_msgs/Header header # Header timestamp should be acquisition time of tof data



uint8 LEFT_HEAD=0  #  the macro definition of tof serial number 

uint8 RIGHT_HEAD=1 #  Used to set tof position

uint8 LEFT_REAR=2

uint8 RIGHT_REAR=3

uint8 HEAD=4

uint8 REAR=5



int32 TOF_DATA_NUM=64 # number of tof data 



float32 SCALE_FACTOR=0.001 # factor of tof data used to compute real distance



bool data_available  # A flag that tof data is available



uint8 tof_position   # location of tof sensor 

                     # (LEFT_HEAD, RIGHT_HEAD, LEFT_REAR,RIGHT_REAR) [enum]



float32[] data      # tof data , Unit: m    

float32[] intensity      # tof data intensity    
```

It quotes the message std_msgs/Header that comes with the ros2 system.

```C%2B%2B
# Standard metadata for higher-level stamped data types.

# This is generally used to communicate timestamped data 

# in a particular coordinate frame.

# 

# sequence ID: consecutively increasing ID 

uint32 seq

#Two-integer timestamp that is expressed as:

# * stamp.sec: seconds (stamp_secs) since epoch (in Python the variable is called 'secs')

# * stamp.nsec: nanoseconds since stamp_secs (in Python the variable is called 'nsecs')

# time-handling sugar is provided by the client library

time stamp

#Frame this data is associated with

string frame_id
```

 

### gps data

- Function description: Get the current latitude and longitude data information of the device
- Interface form: ros topic
- Interface name: topic_name: gps_payload
- Transmission frequency: 1hz
- Interface content:

"GpsPayload.msg

```C%2B%2B
#GPS msg

uint32 sec # The seconds component of nv current time.

uint32 nanosec #The nanoseconds component of nv current time .

uint32 itow # the GPS Timestamps

uint8 fix_type #GNSSfix Type:

uint8 num_sv # Number of satellites used (range: 0-12)

float64 lon # longitude

float64 lat # latitude
```

### Battery information

#### Obtain battery status information

- Specific function: Get current battery status
- Interface form: ros topic
- Interface name: "bms_status"
- Message file: protocol/msg/BmsStatus
- Message content:

```js
std_msgs/Header header

uint16 batt_volt               # Voltage
int16 batt_curr                # Current
uint8 batt_soc                 # Power
int16 batt_temp                # Battery temperature
uint8 batt_st                  # Battery status
int8 batt_health               # Battery health
int16 batt_loop_number         # Number of battery cycles
bool power_normal              # Normal mode
bool power_wired_charging      # Wired charging
bool power_finished_charging   # Charging completed
bool power_motor_shutdown      # Motor power down
bool power_soft_shutdown       # Soft shutdown
bool power_wp_place            # Wireless charging in place
bool power_wp_charging         # Wireless charging
bool power_expower_supply      # External power supply
```

 

### Lighting effects

#### Lighting effect settings

- Specific functions: Set the display color and mode of the light strip
- Interface form: ros service
- Interface name: "led_execute"
- Service file: protocol/srv/LedExecute
- Service Content:

```js
# client

string UNDEFINED ="undefined" #Available during debugging, with the highest priority.      

string VP ="vp"

string BMS ="bms"

string CONNECTOR ="connector"

 



# target

uint8 HEAD_LED =1

uint8 TAIL_LED =2

uint8 MINI_LED =3





# mode

uint8 SYSTEM_PREDEFINED =0x01 #Use the system predefined mode. In this mode, users only need to select the effect field from the

                                        #Select the desired lighting effect. The r_value, g_value, and b_value fields are meaningless.

                                        #For example, target = 1 effect = 0xA1 means the headlight is always red.

uint8 USER_DEFINED =0x02 #Use user-defined mode. In this mode, the lighting effect consists of effect, r_value, g_value,

                                        #b_value field is jointly determined. The effect field determines how the light turns on, r_value, g_value, b_value

                                        The # field determines the color of the lighting effect. For example target = 1, effect = 0x01, r_value =255

                                        #g_value =0 b_value=0 means the red light of the headlight is on.



# effect

The basic lighting effects of #HEAD_LED and TAIL_LED (0x01 ~ 0x09) are used in conjunction with r_value, g_value, and b_value when mode = USER_DEFINED.

uint8 RGB_ON =0x01 #Always on

uint8 BLINK =0x02 #Blink

uint8 BLINK_FAST =0x03 #Blink quickly

uint8 BREATH =0x04 #Breathe

uint8 BREATH_FAST =0x05 #Breathe quickly

uint8 ONE_BY_ONE =0x06 #Light up one by one

uint8 ONE_BY_ONE_FAST =0x07 #Light up one by one quickly

uint8 BACK_AND_FORTH =0x08 #Light up one by one in and out

uint8 TRAILING_RACE =0x09 #TRAILING RACE





# HEAD_LED and TAIL_LED system predefined lighting effects (0xA0 ~ 0xB5), valid when mode = SYSTEM_PREDEFINED.

uint8 RGB_OFF =0xA0 #Always off



uint8 RED_ON =0xA1 #The red light is always on

uint8 RED_BLINK =0xA2 #Red light flashes

uint8 RED_BLINK_FAST =0xA3 #Red light flashes quickly

uint8 RED_BREATH =0xA4 #Red light breathing

uint8 RED_BREATH_FAST =0xA5 #Red light breathes quickly

uint8 RED_ONE_BY_ONE =0xA6 #The red lights light up one by one

uint8 RED_ONE_BY_ONE_FAST =0xA7 #Red lights light up quickly one by one



uint8 BLUE_ON =0xA8 #The blue light is always on

uint8 BLUE_BLINK =0xA9 #Blue light flashes

uint8 BLUE_BLINK_FAST =0xAA #Blue light flashes quickly

uint8 BLUE_BREATH =0xAB #Blue light breathing

uint8 BLUE_BREATH_FAST =0xAC #Blue light breathes quickly

uint8 BLUE_ONE_BY_ONE =0xAD #Blue lights light up one by one

uint8 BLUE_ONE_BY_ONE_FAST =0xAE #Blue lights light up one by one quickly



uint8 YELLOW_ON =0xAF #The yellow light is always on

uint8 YELLOW_BLINK =0xB0 #Yellow light flashes

uint8 YELLOW_BLINK_FAST =0xB1 #Yellow light flashes quickly

uint8 YELLOW_BREATH =0xB2 #Yellow light breathing

uint8 YELLOW_BREATH_FAST =0xB3 #Yellow light breathes quickly

uint8 YELLOW_ONE_BY_ONE =0xB4 #Yellow lights light up one by one

uint8 YELLOW_ONE_BY_ONE_FAST =0xB5 #Yellow lights light up quickly one by one



#MINI LED's basic lighting effect (0x30 ~ 0x31), used with r_value, g_value, and b_value when mode = USER_DEFINED.

uint8 CIRCULAR_BREATH = 0x30 #Circular scaling

uint8 CIRCULAR_RING = 0x31 #Draw a circle



# MINI LED system predefined lighting effects (0x32 ~ 0x36), valid when mode = SYSTEM_PREDEFINED.

uint8 MINI_OFF = 0x32 #Always off

uint8 RECTANGLE_COLOR = 0x33 #The square changes color (no need to set r, g, b values)

uint8 CENTRE_COLOR = 0x34 #Center color band (no need to set r, g, b values)

uint8 THREE_CIRCULAR = 0x35 #Three-circle breathing (no need to set r, g, b values)

uint8 COLOR_ONE_BY_ONE = 0x36 #The ribbons light up one by one (no need to set r, g, b values)



# request



string client #Use modules, such as "bms";"vp";"connector" For details, please see the constant definition in the protocol

uint8 target # The light the user wants to use, you can choose HEAD_LED, TAIL_LED, MINI_LED

uint8 mode # The mode adopted by the user (customized USER_DEFINED/preset SYSTEM_PREDEFINED). For the value, see the constant definition in the protocol.

uint8 effect # Lighting effect, see constant definition in the protocol

uint8 r_value # In custom mode, the gray value of the red channel ranges from 0 to 255. It is meaningless in other modes.

uint8 g_value # In custom mode, the gray value of the green channel ranges from 0 to 255. It is meaningless in other modes.

uint8 b_value # In custom mode, the gray value of the blue channel ranges from 0 to 255. It is meaningless in other modes.





---

#response

int32 SUCCEED =0 # The current request parameters are reasonable, the priority is the highest, and the requested lighting effect is successfully executed.

int32 TIMEOUT =1107 # Current request LED hardware response timeout

int32 TARGET_ERROR =1121 # The target parameter of the current request is empty or not in the optional list

int32 PRIORITY_ERROR =1122 # The currently requested client is empty or not in the preset priority list

int32 MODE_ERROR = 1123 # The currently requested mode parameter is empty or not in the optional list

int32 EFFECT_ERROR =1124 #The effect parameter of the current request is empty or not in the optional list

int32 LOW_PRIORITY = 1125 # The current request priority is low and the requested lighting effect cannot be executed immediately



int32 code #execution result
```

- Headlight (led light strip)

Supports a variety of system preset lighting effects, such as red constant light, green breathing, blue gradient, etc.

  Allows users to customize settings, including basic lighting effects (always on, breathing, gradient, flashing, etc.) and customized colors (R, G, B channel values)

- Tail light (led light strip)

         Supports a variety of system preset lighting effects, such as red constant light, green breathing, blue gradient, etc.

          Allows users to customize settings, including basic lighting effects (always on, breathing, gradient, flashing, etc.) and customized colors (R, G, B channel values)

- Eye light (mini led)

          Supports a variety of system preset lighting effects, such as square color changes, middle ribbons, etc.

          Allows users to customize settings, including basic lighting effects (ring, circular zoom) and custom colors (R, G, B channel values)

## Voice function

### Switch and adjustment

#### Volume acquisition

- Specific function: Get the current volume
- Interface form: ros service
- Interface name: "audio_volume_get"
- Service file: protocol/srv/AudioVolumeGet
- Service Content:

```js
# request

---

#response

uint8 volume # Volume size
```

#### Volume setting

- Specific function: Set the specified volume
- Interface form: ros service
- Interface name: "audio_volume_set"
- Service file: protocol/srv/AudioVolumeSet
- Service Content:

```js
# request

uint8 volume # Volume size

---

#response

bool success # true: success; false: failure
```

#### Microphone switch

- Specific functions: turn on/off the microphone, turn on/disable the radio function
- Interface form: ros service
- Interface name: "set_audio_state"
- Service file: protocol/srv/AudioExecute
- Service Content:

```js
# request

string client #Module name for requesting service: "BMS";"BLUETOOTH";"SENSOR" ..;

AudioStatus status # status message

---

#response

bool result # true: success; false: failure
```

- Quote message:
   - Message file: protocol/msg/AudioStatus
   - Message content:

```js
uint8 AUDIO_STATUS_NORMAL = 0 # Open mic status

uint8 AUDIO_STATUS_OFFMIC = 1 # Microphone off status



uint8 state # state value, take the above constant value
```

#### Voice switch

- Specific functions: enable/disable voice control vertical function
- Interface form: ros service
- Interface name: "audio_action_set"
- Service file: std_srvs/srv/SetBool
- Service Content:

```js
# request

bool data # true: on; false: off

---

#response

bool success # true: success; false: failure

string message #Success and failure information
```

### Voice playback

#### Offline/online voice play message

- Specific functions: Offline does not require the ability to access the Internet, online requires the ability to access the Internet. Offline playback is to play preset audio files. Online playback uses the tts (text to speech) process to convert text into speech for playback. The playback process of this function is non-blocking playback. Only one voice can be played at any time. When other messages are received, the currently playing voice will be immediately interrupted, and then the voice corresponding to the latest message will be played.
- Interface form: ros topic
- Interface name: "speech_play_extend"
- Message file: protocol/msg/AudioPlayExtend
- Message content:

```js
string module_name #Player (module) name

bool is_online # true: online; false: offline

AudioPlay speech # Offline playback information

string text # Play text online
```

- Quote message:
   - Message file: protocol/msg/AudioPlay
   - Message content:

```js
uint16 PID_WIFI_ENTER_CONNECTION_MODE_0 = 1 # Enter networking mode

uint16 PID_WIFI_WAIT_FOR_SCAN_CODE_0 = 3 # Wait for scanning QR code (1 time/5s)

uint16 PID_WIFI_SCAN_CODE_SUCCEEDED_0 = 4 # Scan code successfully, network connection is in progress

uint16 PID_WIFI_CONNECTION_SUCCEEDED_0 = 5 # Network connection successful

uint16 PID_WIFI_CONNECTION_FAILED_0 = 7 # The wireless network name is wrong, please modify it and try again

uint16 PID_WIFI_CONNECTION_FAILED_1 = 8 # The wireless network password is wrong, please modify it and try again

uint16 PID_WIFI_CONNECTION_FAILED_2 = 9 # Unable to connect to the network, please check the network status and try again

                                                  # try

uint16 PID_WIFI_EXIT_CONNECTION_MODE_0 = 10 # Exit networking mode

uint16 PID_WIFI_SCAN_CODE_IP_ERROR = 13 # The QR code information is wrong, please use the correct QR code

uint16 PID_WIFI_SCAN_CODE_INFO_ERROR = 14 # The QR code is invalid, please regenerate it

uint16 PID_FACE_ENTRY_ADD_FACE = 21 # Start recording faces. Please face the camera and do not cover it.

                                                  # Block people’s faces and stay stable

uint16 PID_FACE_ENTRY_CANCLE_ADD_FACE = 22 # Cancel face entry

uint16 PID_FACE_ENTRY_CONFIRM_LAST_FACE = 23 # Confirm face entry

uint16 PID_FACE_ENTRY_UPDATE_FACE_ID = 24 # Update the input face

uint16 PID_FACE_ENTRY_DELETE_FACE = 25 # Delete the entered face

uint16 PID_FACE_ENTRY_GET_ALL_FACES = 26 # Get the entered face information

uint16 PID_FACE_ENTRY_FIX_POSE = 27 # Please face the camera

uint16 PID_FACE_ENTRY_FIX_POSE_LEFT = 28 # Please turn your head to the left

uint16 PID_FACE_ENTRY_FIX_POSE_RIGHT = 29 # Please turn your head to the right

uint16 PID_FACE_ENTRY_FIX_POSE_UP = 30 # Please look up

uint16 PID_FACE_ENTRY_FIX_POSE_DOWN = 31 # Please lower your head

uint16 PID_FACE_ENTRY_FIX_DISTANCE_CLOSE = 32 # Please stay closer to the dog’s head

uint16 PID_FACE_ENTRY_FIX_DISTANCE_NEAR = 33 # Please stay away from the dog’s head

uint16 PID_FACE_ENTRY_FIX_STABLE = 34 # Please keep it stable

uint16 PID_FACE_ENTRY_MUTIPLE_FACES = 35 # Multiple faces detected

uint16 PID_FACE_ENTRY_NONE_FACES = 36 # No face detected

uint16 PID_FACE_ENTRY_TIMEOUT = 37 # Entry timed out, please enter again

uint16 PID_FACE_ENTRY_FINISH = 38 # Entry successful

uint16 PID_FACE_RECOGNITION_REQUEST = 39 # To start face recognition, please face the camera and do not cover it.

                                                  # cover people’s faces

uint16 PID_FACE_DEGREE_HEAD_TILT = 40 # Please don’t tilt your head

uint16 PID_FACE_RECGONITION_FINISH = 41 # Face recognition successful

uint16 PID_FACE_RECGONITION_TIMEOUT = 42 # Face recognition has timed out, please try again

uint16 PID_FACE_ALREADY_EXIST = 43 # The face already exists, please do not enter the same face

uint16 PID_CAMERA_START_PIC_TRANSFER = 50 # Start image transmission

uint16 PID_CAMERA_START_PHOTOS = 51 # Take photos

uint16 PID_CAMERA_TAKE_VIDEOS = 52 # Start recording

uint16 PID_CAMERA_VIDEO_RECORDING = 53 # Recording

uint16 PID_BATTERY_CAPICITY_LOW = 101 # The battery is less than 10%

uint16 PID_BATTERY_IN_CHARGING = 102 # Start charging now

uint16 PID_BATTERY_CHARGE_COMPELETE = 104 # Charging completed

uint16 PID_AI_PLEASE_ENABLE = 124 # Please log in to the app account to enable Xiao Ai online function

uint16 PID_AI_ENABLE_SUCCESS = 125 # Xiaoai online function starts

uint16 PID_AI_SERVICE_EXPIRED = 127 # Voice login has expired

uint16 PID_TEST_HARDWARE_AUDIO = 3000 # Currently it is voice hardware test

uint16 PID_TEST_STAGE_ONE = 3001 # The first phase of testing is over. Please review the test results and

                                                  # After recording, perform the second phase of testing

uint16 PID_TEST_STAGE_THREE = 3003 # The third phase of testing is over, please check the test results and

                                                  # Record

uint16 PID_SOUND_EFFECT_READY = 9000 # Machine ready sound effect

uint16 PID_STOP_AUDIO_PLAY = 9999 # Interrupt the currently playing voice



string module_name # module name

uint16 play_id #Play ID, take the above constant value
```

#### Offline/online voice playback service

- Specific functions: Offline does not require the ability to access the Internet, online requires the ability to access the Internet. Offline playback is to play preset audio files. Online playback uses the tts (text to speech) process to convert text into speech for playback. The playback process of this function is blocking playback. Only one voice can be played at any time, blocking other playback requests. If other playback requests are received while the current voice is being played, the currently playing voice will continue until the playback is completed, and then the next voice request will be played.
- Interface form: ros service
- Interface name: "speech_text_play"
- Service file: protocol/srv/AudioTextPlay
- Service Content:

```js
# request

string module_name #Player (module) name

bool is_online # true: online; false: offline

AudioPlay speech # Offline playback information

string text # Play text online

---

#response

uint8 status # 0 Playing completed, 1 Playing failed
```

- Quote message:
   - Message file: protocol/msg/AudioPlay
   - Message content:

```js
uint16 PID_WIFI_ENTER_CONNECTION_MODE_0 = 1 # Enter networking mode

uint16 PID_WIFI_WAIT_FOR_SCAN_CODE_0 = 3 # Wait for scanning QR code (1 time/5s)

uint16 PID_WIFI_SCAN_CODE_SUCCEEDED_0 = 4 # Scan code successfully, network connection is in progress

uint16 PID_WIFI_CONNECTION_SUCCEEDED_0 = 5 # Network connection successful

uint16 PID_WIFI_CONNECTION_FAILED_0 = 7 # The wireless network name is wrong, please modify it and try again

uint16 PID_WIFI_CONNECTION_FAILED_1 = 8 # The wireless network password is wrong, please modify it and try again

uint16 PID_WIFI_CONNECTION_FAILED_2 = 9 # Unable to connect to the network, please check the network status and try again

                                                  # try

uint16 PID_WIFI_EXIT_CONNECTION_MODE_0 = 10 # Exit networking mode

uint16 PID_WIFI_SCAN_CODE_IP_ERROR = 13 # The QR code information is wrong, please use the correct QR code

uint16 PID_WIFI_SCAN_CODE_INFO_ERROR = 14 # The QR code is invalid, please regenerate it

uint16 PID_FACE_ENTRY_ADD_FACE = 21 # Start recording faces. Please face the camera and do not cover it.

                                                  # Block people’s faces and stay stable

uint16 PID_FACE_ENTRY_CANCLE_ADD_FACE = 22 # Cancel face entry

uint16 PID_FACE_ENTRY_CONFIRM_LAST_FACE = 23 # Confirm face entry

uint16 PID_FACE_ENTRY_UPDATE_FACE_ID = 24 # Update the input face

uint16 PID_FACE_ENTRY_DELETE_FACE = 25 # Delete the entered face

uint16 PID_FACE_ENTRY_GET_ALL_FACES = 26 # Get the entered face information

uint16 PID_FACE_ENTRY_FIX_POSE = 27 # Please face the camera

uint16 PID_FACE_ENTRY_FIX_POSE_LEFT = 28 # Please turn your head to the left

uint16 PID_FACE_ENTRY_FIX_POSE_RIGHT = 29 # Please turn your head to the right

uint16 PID_FACE_ENTRY_FIX_POSE_UP = 30 # Please look up

uint16 PID_FACE_ENTRY_FIX_POSE_DOWN = 31 # Please lower your head

uint16 PID_FACE_ENTRY_FIX_DISTANCE_CLOSE = 32 # Please stay closer to the dog’s head

uint16 PID_FACE_ENTRY_FIX_DISTANCE_NEAR = 33 # Please stay away from the dog’s head

uint16 PID_FACE_ENTRY_FIX_STABLE = 34 # Please keep it stable

uint16 PID_FACE_ENTRY_MUTIPLE_FACES = 35 # Multiple faces detected

uint16 PID_FACE_ENTRY_NONE_FACES = 36 # No face detected

uint16 PID_FACE_ENTRY_TIMEOUT = 37 # Entry timed out, please enter again

uint16 PID_FACE_ENTRY_FINISH = 38 # Entry successful

uint16 PID_FACE_RECOGNITION_REQUEST = 39 # To start face recognition, please face the camera and do not cover it.

                                                  # cover people’s faces

uint16 PID_FACE_DEGREE_HEAD_TILT = 40 # Please don’t tilt your head

uint16 PID_FACE_RECGONITION_FINISH = 41 # Face recognition successful

uint16 PID_FACE_RECGONITION_TIMEOUT = 42 # Face recognition has timed out, please try again

uint16 PID_FACE_ALREADY_EXIST = 43 # The face already exists, please do not enter the same face

uint16 PID_CAMERA_START_PIC_TRANSFER = 50 # Start image transmission

uint16 PID_CAMERA_START_PHOTOS = 51 # Take photos

uint16 PID_CAMERA_TAKE_VIDEOS = 52 # Start recording

uint16 PID_CAMERA_VIDEO_RECORDING = 53 # Recording

uint16 PID_BATTERY_CAPICITY_LOW = 101 # The battery is less than 10%

uint16 PID_BATTERY_IN_CHARGING = 102 # Start charging now

uint16 PID_BATTERY_CHARGE_COMPELETE = 104 # Charging completed

uint16 PID_AI_PLEASE_ENABLE = 124 # Please log in to the app account to enable Xiao Ai online function

uint16 PID_AI_ENABLE_SUCCESS = 125 # Xiaoai online function starts

uint16 PID_AI_SERVICE_EXPIRED = 127 # Voice login has expired

uint16 PID_TEST_HARDWARE_AUDIO = 3000 # Currently it is voice hardware test

uint16 PID_TEST_STAGE_ONE = 3001 # The first phase of testing is over. Please review the test results and

                                                  # After recording, perform the second phase of testing

uint16 PID_TEST_STAGE_THREE = 3003 # The third phase of testing is over, please check the test results and

                                                  # Record

uint16 PID_SOUND_EFFECT_READY = 9000 # Machine ready sound effect

uint16 PID_STOP_AUDIO_PLAY = 9999 # Interrupt the currently playing voice



string module_name # module name

uint16 play_id #Play ID, take the above constant value
```

### Voiceprint recognition

- Specific functions: Through the voiceprint registration process on the App side, the owner's nickname and the owner's voiceprint information are associated. When you shout "Tiedan, ironed," to the robot dog, the owner's voiceprint information will be recognized, and then the associated owner's nickname information will be obtained and released to the outside world.
- Interface form: ros topic
- Interface name: "voice_dlg_info"
- Message file: std_msgs/msg/String
- Message content:

```js
string data #Master nickname
```

# Combination capabilities (factory built-in scenes)

## Navigation function

Specific functions: The navigation function includes mapping, AB point navigation, visual following, and UWB following. The startup and shutdown of these functions are managed through a unified interface, and a query interface for the current task status is provided. The following describes the specific content of each function, as well as the specific startup and shutdown methods.

### Common interface

#### Start task

- Interface form: ros action
- Interface name: "start_algo_task"
- Interface file: protocol/ros/srv/Navigation.action
- Interface content:

```js
uint8 nav_type #Task type

geometry_msgs/PoseStamped[] poses # The target point set during AB point navigation

string map_name

bool outdoor # Indoor and outdoor mapping flag

sensor_msgs/RegionOfInterest tracking_roi #roi information when following

bool object_tracking # Everything follows the flag

---

uint8 result # result

---

int32 feedback_code # Status feedback

string feedback_msg # status information
```

#### Close task

- Interface form: ros service
- Interface name: "stop_algo_task"
- Interface file: protocol/ros/srv/StopAlgoTask.srv
- Interface content:

```js
uint8 task_id # ID of the task

---

bool result # true: the task ends normally, false: the task ends abnormally
```

#### Task status query

- Interface form: ros topic
- Interface name: "algo_task_status"
- Interface file: protocol/ros/msg/AlgoTaskStatus.msg
- Interface content:

```Go
uint8 task_status

int32 task_sub_status
```

### Laser mapping

- Laser mapping mainly uses single-line laser data in indoor environments to complete two-dimensional flat map drawing at a height of about 30cm. Flat maps are used for positioning and navigation.
- After completing the map drawing, you can mark point information labels on the map, such as bedroom, living room and other semantic information.
- For the startup method, see the startup task action interface, where the required key goal fields include:

```js
"nav_type: 1" # 1 means to start mapping

"out_door: false" # false means indoor mapping, that is, laser mapping
```

- For the shutdown method, see the shutdown task service interface, where the required key request fields include:

```js
"task_id: 1" # 1 means to turn off mapping
```

### Visual mapping

- Visual mapping is applied in outdoor environments, using camera data to complete simultaneous positioning and mapping functions, and generate a two-dimensional flat map for navigation.
- Relocation function at the initial position, that is, the dog can locate its own position according to the flat map after turning on the phone.
- Like laser maps, semantic information can be annotated on the drawn visual map.
- For the startup method, see the startup task action interface, where the required key goal fields include:

```js
"nav_type: 1" # 1 means to start mapping

"out_door: true" # true means outdoor mapping, that is, visual mapping
```

- For the shutdown method, see the shutdown task service interface, where the required key request fields include:

```js
"task_id: 1" # 1 means to turn off mapping
```

### AB point navigation

- Based on the two-dimensional flat map provided by laser or visual mapping, the dog can be controlled to autonomously navigate to the selected target point based on the drawn semantic labels.
- During the process of autonomous navigation, autonomous obstacle avoidance can be achieved based on laser or visual information.
- For the startup method, see the startup task action interface, where the required key goal fields include:

```js
"nav_type: 5" # 5 means starting AB point navigation

"poses:

  position: ...

  orientation: ..." # The first element of poses is the target point pose
```

 

- For the shutdown method, see the shutdown task service interface, where the required key request fields include:

```js
"task_id: 5" # 5 means turning off point AB navigation
```

 

### Visual follow

Visual following can be divided into human body following and all things following, among which

- Human body following can select the target to follow based on the human body detection results in the image transmission and follow it autonomously
- Everything can be followed in the image transmission by selecting any certain object to follow.
- Autonomous obstacle avoidance can be achieved in both human body following and all things following.
- If the human body or the object being followed deviates from the following field of view, the robot dog will move autonomously (such as rotating in place) to try to find the target. When the target returns to the field of view, it can continue to follow.
- For the startup method, see the startup task action interface, where the required key goal fields include:

```js
"nav_type: 13" # 13 means starting visual following
"object_tracking: true" # true means to start tracking of all things, false means to start following of human body
"relative_pos: 200: # 200: Independently choose to follow the position
                          #201: Follow behind the target
                          #202: Follow left of target
                          #203: Follow on the right side of the target
"keep_distance: 1.2" # Set following distance
```

 

- For the shutdown method, see the shutdown task service interface, where the required key request fields include:

```js
"task_id: 13" # 13 means turning off visual following
```

 

### UWB Follow

- When a person holds a UWB tag, the robot dog can track the person's movements
- UWB following is not restricted by the following field of view. As long as the distance between the person holding the UWB tag and the robot dog is within the UWB detection range, the following state can be maintained.
- In UWB following, you can identify platforms 7-25cm on the path, and realize autonomous jumping up the steps to keep following.
- During UWB following, the target's motion status will be detected. When the target is stationary, the robot dog will simulate the autonomous mode of walking a dog in a real scene, such as twisting its butt, sitting down, lying down, etc., and wait for the owner's action. When the target moves, Continue to follow after recovery
- For the startup method, see the startup task action interface, where the required key goal fields include:

```js
"nav_type: 11" # 11 means to start UWB following
"relative_pos: 200: # 200: Independently choose to follow the position
                          #201: Follow behind the target
                          #202: Follow left of target
                          #203: Follow on the right side of the target
"keep_distance: 1.2" # Set following distance
```

 
- For the shutdown method, see the shutdown task service interface, where the required key request fields include:

```js
"task_id: 11" # 11 means turning off UWB following
```

 

## Image function

### Image transmission

- Images from AI cameras can be transferred to the APP
   - Turn on image transmission
   - Stop image transmission
   - Set the resolution (aspect ratio) to enable image transmission
   - Alignment: Top, Center, Bottom

#### Signaling communication, downlink (APP→NX)

- Interface form: ros topic
- Interface name: "img_trans_signal_in"
- Topic file: std_msgs/msg/String
- Topic content:

```js
string data
```

The content format of data is json, there are two types:

##### offer_sdp

```JSON
{
     "offer_sdp" : {
         "sdp" : "sdp content",
         "type" : "sdp type"
     },
     "uid" : "identification code",
     "height" : "The height of the mobile phone screen",
     "width" : "Width of the mobile phone screen",
     "alignment" : "top or middle or bottom"
}
```

##### ice candidate

```JSON
{
     "c_sdp" : {
         "sdpMid" : "If present, this is the value of the \"a=mid\" attribute of the candidate's m= section in SDP, which identifies the m= section",
         "sdpMLineIndex" : "This indicates the index (starting at zero) of m= section this candidate is associated with. Needed when an endpoint doesn't support MIDs",
         "candidate" : "ice candidate content"
     },
     "uid" : "identification code"
}
```

#### Signaling communication, uplink (NX→APP)

- Interface form: ros topic
- Interface name: "img_trans_signal_out"
- Topic file: std_msgs/msg/String
- Topic content:

```js
string data
```

The content format of data is json, there are two types:

##### answer_sdp

```JSON
{
     "answer_sdp" : {
         "sdp" : "sdp content",
         "type" : "sdp type"
     },
     "uid" : "identification code",
}
```

##### ice candidate

```JSON
{
     "c_sdp" : {
         "sdpMid" : "If present, this is the value of the \"a=mid\" attribute of the candidate's m= section in SDP, which identifies the m= section",
         "sdpMLineIndex" : "This indicates the index (starting at zero) of m= section this candidate is associated with. Needed when an endpoint doesn't support MIDs",
         "candidate" : "ice candidate content"
     },
     "uid" : "identification code"
}
```

### Photograph 

- When image transmission is turned on, take high-resolution photos through the AI camera and transfer them to the mobile phone album
   - Trigger photo
   - If the transfer fails, the photo files will be re-transmitted when reconnecting to the APP
- Interface form: ros service
- Interface name: "camera_service"
- Service file: protocol/srv/CameraService
- Service Content:

```js
uint8 SET_PARAMETERS = 0 #Set internal parameters
uint8 TAKE_PICTURE = 1 #Photography command, take a photo each time, the photo file is saved in /home/mi/Camera
uint8 START_RECORDING = 2 #Start recording command
uint8 STOP_RECORDING = 3 #End recording command, the video file is saved in /home/mi/Camera
uint8 GET_STATE = 4 #Get the status of whether it is currently recording or not
uint8 DELETE_FILE = 5 #Delete the specified photo or video file
uint8 GET_ALL_FILES = 6 #Get the file names of all saved photos and videos
uint8 START_LIVE_STREAM = 7 #Enable image transmission command
uint8 STOP_LIVE_STREAM = 8 #End image transmission command
uint8 START_IMAGE_PUBLISH = 9 #Enable image publishing command, topic name: /image
uint8 STOP_IMAGE_PUBLISH = 10 #Close image publishing instruction
uint8 command

# command arguments
string args
uint16 width
uint16 height
uint16 fps

---
uint8 RESULT_SUCCESS = 0 #Success
uint8 RESULT_INVALID_ARGS = 1 #Invalid parameters
uint8 RESULT_UNSUPPORTED = 2 #Not supported
uint8 RESULT_TIMEOUT = 3 #Timeout
uint8 RESULT_BUSY = 4 #Busy
uint8 RESULT_INVALID_STATE = 5 #Invalid state
uint8 RESULT_INNER_ERROR = 6 #Internal error
uint8 RESULT_UNDEFINED_ERROR = 255 #Undefined error
uint8 result
string msg
```

This service will be called when taking pictures, and the command is TAKE_PICTURE

### Video

- When image transmission is turned on, record high-resolution video through the AI camera and transfer it to the mobile phone album
   - Start recording
   - Stop recording
   - The next video can be started during the transfer process
   - In case of transmission failure, the video file will be re-transmitted when reconnecting to the APP
- Interface form: ros service
- Interface name: "camera_service"
- Service file: protocol/srv/CameraService
- Service Content:

```js
uint8 SET_PARAMETERS = 0 #Set internal parameters
uint8 TAKE_PICTURE = 1 #Photography command, take a photo each time, the photo file is saved in /home/mi/Camera
uint8 START_RECORDING = 2 #Start recording command
uint8 STOP_RECORDING = 3 #End recording command, the video file is saved in /home/mi/Camera
uint8 GET_STATE = 4 #Get the status of whether it is currently recording or not
uint8 DELETE_FILE = 5 #Delete the specified photo or video file
uint8 GET_ALL_FILES = 6 #Get the file names of all saved photos and videos
uint8 START_LIVE_STREAM = 7 #Enable image transmission command
uint8 STOP_LIVE_STREAM = 8 #End image transmission command
uint8 START_IMAGE_PUBLISH = 9 #Enable image publishing command, topic name: /image
uint8 STOP_IMAGE_PUBLISH = 10 #Close image publishing instruction
uint8 command

# command arguments
string args
uint16 width
uint16 height
uint16 fps
---
uint8 RESULT_SUCCESS = 0 #Success
uint8 RESULT_INVALID_ARGS = 1 #Invalid parameters
uint8 RESULT_UNSUPPORTED = 2 #Not supported
uint8 RESULT_TIMEOUT = 3 #Timeout
uint8 RESULT_BUSY = 4 #Busy
uint8 RESULT_INVALID_STATE = 5 #Invalid state
uint8 RESULT_INNER_ERROR = 6 #Internal error
uint8 RESULT_UNDEFINED_ERROR = 255 #Undefined error
uint8 result
string msg
```

## Quick connection

- Trigger networking function through peripheral touch
- RGB camera service to take QR code photos
- AI module parses QR code information
- wifi module connects to the network
- Voice playback of Internet results

### Touch triggers networking

- Specific function: Get touch action, when touch_state=7, perform networking function
- Interface form: ros topic
- Interface name: "touch_status"
- Message file: protocol/msg/TouchStatus
- Message content

```js
std_msgs/Header header # Message header



int32 touch_state # 0x01: single click, 0x03: double click, 0x07: long press

uint64 timestamp # timestamp
```

### Status acquisition

- Specific function: Get the current connection status
- Interface form: ros topic
- Interface name: "connector_status"
- Message file: protocol/msg/ConnectorStatus
- Service Content:

```js
bool is_connected # Whether the network is connected, true: yes; false: no;

bool is_internet # Whether the network is connected, true: yes; false: no;

string ssid #wifi name (if not connected, this field is invalid)

string robot_ip #robot ip

string provider_ip #wifi provider (mobile terminal) IP

uint8 strength # Signal strength, value range 0 - 100, 0: no signal, 100: strongest signal;

int32 code #Execution (status) result, please refer to the cyberdog_system definition document for details
```

### Connect to WiFi and APP devices

- Specific functions: Connect to target WiFi and target device
- Interface form: ros service
- Interface name: "connect"
- Service file: protocol/srv/Connector
- Service Content:

```js
string wifi_name # Target WiFi name

string wifi_password # Target WiFi password

string provider_ip # Target terminal IP

---

bool connected # WiFi is connected successfully
```

### Disconnect the current device

- Specific function: Disconnect the current device
- Interface form: ros topic
- Interface name: "disconnect_app"
- Message file: std_msgs/msg/Bool
- Service Content:

```js
bool data # true: Disconnect the current wifi connection with the current app, otherwise it will be invalid.
```

## Daily funny dog

- Dog enters sport sitting mode
- A human hand touches the dog's chin, and the TOF sensor recognizes the human hand
- Movement for twisting

## Sound event reminder

- Monitor surrounding environment sounds
- Preset behaviors corresponding to ambient sounds
- Dogs react to sounds like knocking, crying, sneezing, etc. that go beyond white noise

# Visual programming

Reference: [Graphical Programming Help Document](https://xiaomi.f.mioffice.cn/docs/dock42yTK5kj89KQZppS8RXFXPd)

## Visual programming interface capabilities

- Operations: including creation, deletion, debugging, running, pausing, continuing, and terminating;
- Attributes: including immediate execution, periodic operation, scheduled operation, function modules, etc.;
- Support operators: arithmetic operators, comparison operators, copy operators, logical operators, bitwise operators;
- Supported data types: integer, floating point, string, list, tuple, dictionary;
- Capability set: basic status information, network module, following module, motion module, navigation module, personnel information module (age, voiceprint, face, etc.), AI human movement module, voice module, Led module, touch pad module, global Positioning system module, laser ranging module, radar module, ultrasonic module, odometer magic module, inertial navigation module;

## Scenario case

By combining a series of capability sets, users can freely create scenarios

For example: getting close to the master:

- Detect voiceprints and faces of family members through the personnel information module;
- Run to the owner by following the module;
- Use the movement module to circle around the owner and perform some cute actions (turning the head, twisting the butt, stretching, dancing, etc.);
- Broadcast some voices through the voice module;
- The AI human action module can detect the owner's gestures and respond accordingly (pull the palm closer: the dog comes; push the palm away: the dog leaves; push the hand to the left: the dog moves left; push the hand to the right: the dog Move the dog to the right; press down with your hand: the dog will pick it off; thumb up: the dog will stand up; etc.);

For example: early morning alarm clock task:

- Create a task;
- Tiedan locates its position through the positioning module;
- Plan the route to your destination through the navigation module;
- The movement process uses radar and vision modules to automatically avoid obstacles;
- Arrive at the destination master bedroom;
- Play music or customize your voice through the voice module;
