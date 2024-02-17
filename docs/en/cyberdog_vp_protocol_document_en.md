**Visual Programming-Protocol and Interface Documentation**

**Overview**

The current protocol is the only legal protocol for visual programming. The protocol belongs to a subset of python3 syntax. Therefore, if you are unclear about any content supported in Sections 1, 2 and 3 of this protocol, you can check it through python books.

**Zero, interaction constraints**

**0.1 ros/grpc format during interaction**

Terminology constraints:

> ‘Front-end’: refers to APP and WEB visual programming control end;
>
> ‘Robot side’: refers to the visual programming engine on the robot side.

**0.1.1 Introduction to GRPC protocol**

> For details about grpc, see Appendix 3.

**0.1.2 Robot-side listening GRPC message constraints**

Message type: std\_msgs/msg/string

Message name: /frontend\_message

Message content: must comply with the json format constraints during interaction in the next section

Other constraints:

<table>
<tbody>
<tr class="odd">
<td>Apache<br />
VISUAL_FRONTEND_MSG = 2002;</td>
</tr>
</tbody>
</table>

**0.1.3 GRPC listening robot side message constraints**

Message type: std\_msgs/msg/string

Message name: /backend\_message

Message content: must comply with the json format constraints during interaction in the next section

Other constraints:

<table>
<tbody>
<tr class="odd">
<td>Apache<br />
VISUAL_BACKEND_MSG = 2001;</td>
</tr>
</tbody>
</table>

**0.2 Task and module functions:**

**0.2.1 \<front-end-robot-side\>/\<robot-side-back-end\> json format during interaction**

**0.2.1.1 \[Front-end-\>Robot-side\]/\[Robot-\>Back-end\]: Send data in json format**

**0.2.1.1.1 Agreement Constraints**

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
{<br />
"type": "type",<br />
"id": "id",<br />
"operate": "operate",<br />
"target_id": [<br />
"target_id_1",<br />
"target_id_2",<br />
"target_id_3",<br />
...<br />
],<br />
"describe": "describe",<br />
"style": "style",<br />
"mode": "mode",<br />
"condition": "condition",<br />
"body": "body"<br />
}</td>
</tr>
</tbody>
</table>

**0.2.1.1.2 Protocol Description**

**0.2.1.1.2.1 Common fields**

**type:** \[Required\] String type, identifying the current json data type.

Legal parameters:

Task:'task'

Module: 'module'

Module:'AI'

Module: ‘SLAM’

**id**: \[Required\] String type, identifying the unique id of the current data frame, used for feedback alignment;

Legal parameter requirements: an id that uniquely identifies the current data frame and conforms to the variable naming rules.

**operate**: \[Required\] String type, identifying the current task operation type.

This field is valid and required only when the 'type' field value is 'task' or 'module'.

Legal parameters:

Save task/module:'save'

Delete tasks/modules: 'delete'

Query task/module/AI:'inquiry'

Debugging task: 'debug'

Run task: 'run'

Terminate task:'shutdown'

Suspend task:'suspend'

Continue task:'recover'

**target\_id**: \[Required\] String array type, used to identify the current operation target, which can be divided into task id and module id according to the scenario.

Legal parameter requirements: an id that uniquely identifies the current data frame and conforms to the variable naming rules.

When the operate field is 'debug', target\_id is fixed to \["debug"\].

Task id, when used to save, delete, query, debug, run, pause, continue, or terminate tasks, the value is the task id to be operated;

Module id, when used to save, delete, query and debug modules, the value is the module id to be operated;

Note: This field is allowed to be empty only when querying the scene. Details are as follows:

When saving and running the scene, if the **target\_id** array is empty**, it will not be executed. Otherwise, only the target constrained by the first target\_id will be operated (that is, only the first id is valid);

When running a task, it will be run according to the saved task attributes;

When pausing, continuing, or terminating task scenarios, if the **target\_id** array is empty**, it will not be executed. Otherwise, only all targets constrained by target\_id will be executed;

When deleting a scene, if the **target\_id** array is empty**, the deletion operation will not be performed. Otherwise, only all targets constrained by target\_id will be deleted;

When querying the scene, if the **target\_id** array is empty**, **all targets constrained by type will be returned, otherwise only all targets constrained by target\_id will be returned;

**describe**: \[optional\] String type, describing the current data frame function, such as task or module scenario.

This field is valid and required only when the 'operate' field value is 'save' or 'debug'.

Legal parameter requirements: It cannot contain three consecutive double quotes (""").

**style**: \[optional\] String type, identifying the current message style, that is, the presentation method of the current execution body content.

This field is valid and required only when the 'operate' field value is 'save' or 'debug'.

**0.2.1.1.2.2 Difference field**

**0.2.1.1.2.2.1 type=task**

**mode**: \[optional\] String type, identifying the current task mode type.

This field is fixed to 'single' when the 'operate' field value is 'debug'

This field is valid and required when the 'operate' field value is 'save'

This field is optional when the 'operate' field value is 'run'. If assigned, the original corresponding field attributes will be modified;

Legal parameters:

Single task: 'single'

Periodic task: 'cycle'

**condition**: \[Optional\] When the current frame is an operation task, identify the task constraints. The details are as follows:

> A string type that conforms to the following constraints and identifies the prerequisites for the execution of the current task.

When the 'operate' field value is 'debug', this field is fixed to 'now';

This field is valid and required when the 'mode' field value is non-empty, and is used to identify the default constraints of the task;

When the 'operate' field value is 'save' or 'debug', this field identifies the default execution constraints of the current task;

When the 'operate' field value is 'run', this field will modify the default execution constraints;

There are two assignment methods: the first is used for single tasks (mode = single), and the second is used for periodic tasks (mode = cycle). The formats are as follows:

The first type: single task (mode = single)

The constraint can be in absolute time or relative time format, if the time has passed then it will be executed at the same time the next day.

The formats are as follows:

Absolute time format:

**'now'**: Use the current time as the constraint time, that is, execute it immediately.

**HH:MM**: Specify the constraint time in the form of "hour:minute".

**HH:MM YYYY-MM-DD**: Specify the constraint time in the form of "hour:minute year-month-day".

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
// <strong>Example</strong><br />
{...,<br />
"condition": "now", //1. Execute the current task immediately<br />
"condition": "16:50", //2. Execute the current task at 16:50 that day<br />
"condition": "00:01 2022-12-01", //3. Execute the current task at 00:01 on December 1, 2022<br />
...}</td>
</tr>
</tbody>
</table>

Relative time format:

**Absolute time + number\[minutes|hours|days|weeks|months|years\]:** Specify the constraint time in the form of "absolute time + n\[minutes|hours|days|weeks|months|years\]" , according to the absolute time format, it can be divided into the following three types:

**now + number\[minutes|hours|days|weeks|months|years\]**

**HH:MM + number\[minutes|hours|days|weeks|months|years\]**

**HH:MM YYYY-MM-DD + number\[minutes|hours|days|weeks|months|years\]**

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
// <strong>Example</strong><br />
// "condition": <strong>Absolute time + number[minutes|hours|days|weeks|months|years]</strong><br />
{...,<br />
"condition": "now + 5minutes", // 1. Execute the current task after 5 minutes<br />
"condition": "16:50 + 5days", // 2. Execute the current task at 16:50 5 days later<br />
"condition": "00:01 2022-12-01 + 5years", // 3. Relative to 00:01 on December 1, 2022, the current task will be executed 5 years later<br />
...}</td>
</tr>
</tbody>
</table>

The second type: periodic task (mode = cycle)

> The format is as follows:

**minute hour day month week**

in:

**minute**: minute (0 - 59)

**hour**: hour (0 - 23)

**day**: day of the month (1 - 31)

**month**: month (1 - 12)

**week**: day of the week (0 - 6)

Remark:

**\*** All numbers within the value range

**/** How many numbers have passed each time?

**-** From X to Z

**, **Hash number

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
// Example<br />
// <strong>minute hour day month week</strong><br />
{...,<br />
"condition": "* * * * *", // 1. Execute the current task once every minute<br />
"condition": "*/1 * * * *", // 2. Execute the current task once every second<br />
"condition": "01 * * * *", // 3. Execute the current task once every hour<br />
"condition": "3,15 * * * *", // 4. Execute the current task at the 3rd and 15th minutes of every hour<br />
"condition": "3,15 8-11 * * *", // 5. Executed at the 3rd and 15th minutes from 8 am to 11 am every day<br />
"condition": "3,15 8-11 */2 * *", // 6. Execute the current task at the 3rd and 15th minutes from 8 am to 11 am every two days<br />
"condition": "3,15 8-11 * * 1", // 7. Execute the current task at the 3rd and 15th minutes from 8 am to 11 am every Monday<br />
"condition": "30 21 * * *", // 8. Execute the current task at 21:30 every night<br />
"condition": "45 4 1,10,22 * *", // 9. Execute the current task at 4:45 on the 1st, 10th, and 22nd of each month<br />
"condition": "0,30 18-23 * * *", // 10. Execute the current task every 30 minutes between 18:00 and 23:00 every day<br />
...}</td>
</tr>
</tbody>
</table>

individual action

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
// Example<br />
{...,<br />
"condition": "@reboot", // 1. Run the current task once every time it is started<br />
...}</td>
</tr>
</tbody>
</table>

**body**: \[optional\] String type, identifying the current message body, that is, the main content of the current execution.

This field is valid and required only when the 'operate' field value is 'save' or 'debug'

The minimum unit of indentation for data fields is 4 spaces.

**0.2.1.1.2.2.2 type=module**

**mode**: \[optional\] String type, identifying the current module mode type.

This field is valid and required only when the 'operate' field value is 'add'

Legal parameters:

Common module: 'common'

Ordinary logic module;

Sequence module: 'sequence'

Module for setting up action sequences.

**condition**: \[optional\] String type, identifying the current module interface.

Interface format constraint: "Function name (parameter)".

This field is valid and required only when the 'operate' field value is 'add'.

It is required to comply with the variable naming rules, and be careful not to conflict with existing module names.

**body**: \[optional\] String type, identifying the body of the current module, that is, the main content of the current execution.

This field is valid and required only when the 'operate' field value is 'add'

The minimum unit of indentation for data fields is 4 spaces, and the later indentation is an integer multiple of 4.

Note: Modules do not support overloading.

**0.2.1.1.2.2.3 type=AI**

**mode**: \[optional\] String type, identifying the current module mode type.

This field is valid and required only when the 'operate' field value is 'inquiry'

Legal parameters:

All AI data: 'all'

Personnel:'personnel'

Face:'face'

Voiceprint:'voiceprint'

Training words:'training\_words'

**0.2.1.1.2.2.4 type=SLAM**

**mode**: \[optional\] String type, identifying the current module mode type.

This field is valid and required only when the 'operate' field value is 'inquiry'

Legal parameters:

Map: 'map', all map information of the current robot.

Preset point: 'preset', a preset point in the map where the current robot is located, which can be used directly for navigation.

**0.2.1.1.3 Protocol Example**

**0.2.1.1.3.1 task protocol example**

**0.2.1.1.3.1.1 Debug code**

Stand first, wait 60 seconds, and then lie down.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["debug"],</strong><br />
<strong>"operate": "debug",</strong><br />
<strong>"mode": "single",</strong><br />
<strong>"condition": "now",</strong><br />
<strong>"body": "</strong><br />
<strong>cyberdog.motion.stand_up()</strong><br />
<strong>time.sleep(60)</strong><br />
<strong>cyberdog.motion.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.2 Save task**

Save an immediate task: stand, wait 60 seconds, then lie down.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"mode": "single",</strong><br />
<strong>"condition": "now",</strong><br />
<strong>"body": "</strong><br />
<strong>cyberdog.motion.stand_up()</strong><br />
<strong>time.sleep(60)</strong><br />
<strong>cyberdog.motion.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Save a scheduled single task:

The current task is required to be executed at the latest 21:30.

Task content: Stand first, wait 5 seconds, and then lie down.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "124",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"mode": "single",</strong><br />
<strong>"condition": "21:30", // 20:47 2022-06-07</strong><br />
<strong>"body": "</strong><br />
<strong>cyberdog.motion.stand_up()</strong><br />
<strong>time.sleep(5)</strong><br />
<strong>cyberdog.motion.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Save a scheduled periodic task:

The current task is required to be executed at 21:30 every day until the task is terminated.

Task content: Stand first, wait 5 seconds, and then lie down.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "125",</strong><br />
<strong>"operate": ["678"],</strong><br />
<strong>"mode": "cycle",</strong><br />
<strong>"condition": "02 15 * * 1,5,7",</strong><br />
<strong>"body": "</strong><br />
<strong>cyberdog.stand_up()</strong><br />
<strong>time.sleep(5)</strong><br />
<strong>cyberdog.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.3 Run task**

Run a saved target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "run"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.4 Pause task**

Pause a running target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "suspend"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Pause multiple running target tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "suspend"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.5 Continue mission**

Resume a paused target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "recover"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Resume multiple paused target tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "recover"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.6 Terminate task**

Terminate a running target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "shutdown"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Terminate multiple running target tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "shutdown"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.7 Delete task**

Delete a saved and terminated target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "delete"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Delete multiple saved and terminated target tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "delete"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.1.8 Query Task**

Query a saved target task.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query multiple saved target tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query all saved tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.2 module protocol example**

**0.2.1.1.3.2.1 Save module**

Save a module with parameters.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "123",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"mode": "common",</strong><br />
<strong>"condition": "usr_behavior_hunger_1(name, size)",</strong><br />
<strong>"body": "</strong><br />
<strong>print("My", name, "Only ", size, "%, lie down for a while.")</strong><br />
<strong>cyberdog.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Save a module without parameters.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "124",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"mode": "common",</strong><br />
<strong>"condition": "usr_behavior_hunger_2()",</strong><br />
<strong>"body": "</strong><br />
<strong>print("I'm hungry")</strong><br />
<strong>cyberdog.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Save a sequence module.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<br />
{<br />
"type": "module",<br />
"id": "124",<br />
"operate": "add",<br />
"mode": "sequence",<br />
"condition": "usr_behavior_sequence_2()",<br />
"body": "<br />
<br />
sequ = MotionSequence()<br />
sequ.name = 'test_sequ'<br />
sequ.describe = 'Test sequence'<br />
<br />
# The original gait configuration file is: ./user_gait_00.toml.<br />
<br />
gait_meta = MotionSequenceGait()<br />
<br />
gait_meta.right_forefoot = 1<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 1<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
gait_meta.right_forefoot = 0<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 0<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
gait_meta.right_forefoot = 1<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 1<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
# The original program configuration file is: ./L91_user80_ballet_full.toml.<br />
<br />
pace_meta = MotionSequencePace()<br />
<br />
pace_meta.twist.linear.x = 0.000000<br />
pace_meta.twist.linear.y = 0.000000<br />
pace_meta.twist.linear.z = 0.000000<br />
pace_meta.centroid.position.x = 0.000000<br />
pace_meta.centroid.position.y = 0.000000<br />
pace_meta.centroid.position.z = 0.020000<br />
pace_meta.centroid.orientation.x = 0.000000<br />
pace_meta.centroid.orientation.y = 0.000000<br />
pace_meta.centroid.orientation.z = 0.000000<br />
pace_meta.weight.linear.x = 50.000000<br />
pace_meta.weight.linear.y = 50.000000<br />
pace_meta.weight.linear.z = 5.000000<br />
pace_meta.weight.angular.x = 10.000000<br />
pace_meta.weight.angular.y = 10.000000<br />
pace_meta.weight.angular.z = 10.000000<br />
pace_meta.right_forefoot.x = 0.030000<br />
pace_meta.right_forefoot.y = 0.050000<br />
pace_meta.left_forefoot.x = -0.030000<br />
pace_meta.left_forefoot.y = -0.050000<br />
pace_meta.right_hindfoot.x = 0.030000<br />
pace_meta.right_hindfoot.y = 0.050000<br />
pace_meta.left_hindfoot.x = -0.030000<br />
pace_meta.left_hindfoot.y = -0.050000<br />
pace_meta.friction_coefficient = 0.400000<br />
pace_meta.right_forefoot.w = 0.030000<br />
pace_meta.left_forefoot.w = 0.030000<br />
pace_meta.right_hindfoot.w = 0.030000<br />
pace_meta.left_hindfoot.w = 0.030000<br />
pace_meta.landing_gain = 1.500000<br />
pace_meta.use_mpc_track = 0<br />
pace_meta.duration = 300<br />
sequ.pace_list.push_back(pace_meta)<br />
<br />
pace_meta.twist.linear.x = 0.000000<br />
pace_meta.twist.linear.y = 0.000000<br />
pace_meta.twist.linear.z = 0.000000<br />
pace_meta.centroid.position.x = 0.000000<br />
pace_meta.centroid.position.y = 0.000000<br />
pace_meta.centroid.position.z = 0.000000<br />
pace_meta.centroid.orientation.x = 0.000000<br />
pace_meta.centroid.orientation.y = 0.000000<br />
pace_meta.centroid.orientation.z = 0.000000<br />
pace_meta.weight.linear.x = 50.000000<br />
pace_meta.weight.linear.y = 50.000000<br />
pace_meta.weight.linear.z = 5.000000<br />
pace_meta.weight.angular.x = 10.000000<br />
pace_meta.weight.angular.y = 10.000000<br />
pace_meta.weight.angular.z = 10.000000<br />
pace_meta.right_forefoot.x = 0.000000<br />
pace_meta.right_forefoot.y = 0.000000<br />
pace_meta.left_forefoot.x = 0.000000<br />
pace_meta.left_forefoot.y = 0.000000<br />
pace_meta.right_hindfoot.x = 0.000000<br />
pace_meta.right_hindfoot.y = 0.000000<br />
pace_meta.left_hindfoot.x = 0.000000<br />
pace_meta.left_hindfoot.y = 0.000000<br />
pace_meta.friction_coefficient = 0.400000<br />
pace_meta.right_forefoot.w = 0.030000<br />
pace_meta.left_forefoot.w = 0.030000<br />
pace_meta.right_hindfoot.w = 0.030000<br />
pace_meta.left_hindfoot.w = 0.030000<br />
pace_meta.landing_gain = 1.500000<br />
pace_meta.use_mpc_track = 0<br />
pace_meta.duration = 300<br />
sequ.pace_list.push_back(pace_meta)<br />
<br />
cyberdog.motion.run_sequence(sequ)<br />
"<br />
}</td>
</tr>
</tbody>
</table>

Call the module just added.

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>usr_behavior_hunger_1('Battery', 16)</strong><br />
<strong>usr_behavior_hunger_2()</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.2.2 Delete module**

Delete a target module that has been saved and is in the terminated state.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "delete"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Delete multiple target modules that have been saved and are in the terminated state.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "delete"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.2.3 Query module**

Query a saved target module.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query multiple saved targets.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query all saved modules.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "module",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.3 AI protocol example**

**0.2.1.1.3.3.1 Query bottom database personnel information**

Query a saved database personnel information.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "AI",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "personnel",</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query multiple saved database personnel information.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "AI",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": ["678","789","891"],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "personnel",</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Query all saved tasks.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "AI",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "personnel",</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.1.3.3.2 Query face information in the base database**

Query all the people whose faces have been entered in the base database.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "AI",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "face",</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Other query categories.

**0.2.1.1.3.3.3 Query the base database voiceprint information**

Query all persons whose voiceprints have been recorded in the base database.

<table>
<tbody>
<tr class="odd">
<td><blockquote>
<p>JSON<br />
<strong>{</strong><br />
<strong>"type": "AI",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "voiceprint",</strong><br />
<strong>}</strong></p>
</blockquote></td>
</tr>
</tbody>
</table>

Other query categories.

**0.2.1.1.3.4 SLAM protocol example**

**0.2.1.1.3.4.1 Query the preset point information in the current map**

Query the preset points in the current map.

<table>
<tbody>
<tr class="odd">
<td><blockquote>
<p>JSON<br />
<strong>{</strong><br />
<strong>"type": "SLAM",</strong><br />
<strong>"id": "xxx",</strong><br />
<strong>"target_id": [],</strong><br />
<strong>"operate": "inquiry",</strong><br />
<strong>"mode": "preset",</strong><br />
<strong>}</strong></p>
</blockquote></td>
</tr>
</tbody>
</table>

**0.2.1.2 \[Robot-\>Front-end\]/ \[Robot-\>Back-end\]: Report data in json format**

**0.2.1.2.1 Agreement Constraints**

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"target_id": "target_id",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "describe"</strong><br />
<strong>},</strong><br />
<strong>"block": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id"</strong><br />
<strong>},</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"dependent": ["","",...],</strong><br />
<strong>"be_depended": ["","",...],</strong><br />
<strong>"describe": "describe"</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>},</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.2.2 Protocol Description**

**0.2.1.2.2.1 General request feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"target_id": "target_id",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "describe"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**feedback:** \[Required\]Request message feedback

**feedback.type**: \[required\] message type

Legal parameters:

Task:'task'

Module: 'module'

**feedback.id**: \[required\] message id

When responding to a request message, it is the original message id;

When feedbacking task status, it is a timestamp.

**feedback.target\_id**: \[required\] message target\_id

When responding to a request message, if the original message target\_id is not empty, it is the first element, otherwise it is empty;

When feedbacking task status, it is task target\_id.

**feedback.operate**: \[required\] message operation

Operations triggered by app:

Debug task/module: 'debug'

Add task/module:'save'

Modify task/module:'run'

Delete tasks/modules: 'delete'

Query task/module:'inquiry'

Terminate task:'shutdown'

Suspend task:'suspend'

Continue task:'recover'

Operations automatically triggered by tasks:

Start task:'start'

Stop the task: 'stop'

Operations that the robot wants to give feedback to the server:

Start task:'start'

**feedback.state**: \[Required\] Original message operation status

Value constraints:

0: The current operation is successful

Non-0: An exception occurs when switching to the currently specified type.

1: non-json format

2: type field, type is invalid

3: id field, id is invalid

4: target\_id field, id is invalid

5: describe field, description is invalid

6: style field, style is invalid

7: operate field, the operation is invalid

8: mode field, mode is invalid

9: condition field, the condition is invalid

10: body field, the body is invalid

21: Unable to create path

22: Unable to open file

23: The body field syntax is abnormal and cannot be constructed.

24: Unable to register

25: Unable to update list

26: Unable to execute

27: The current operation is illegal

28: Request error

29: Other errors

30: Service interrupted

31: Timeout waiting for the service to come online.

31: Request service timeout

**feedback.describe**: \[Required\] Original message operation status description

**Note**: When the requested operations are the following, you will receive two feedbacks.

Terminate task:'shutdown'

Suspend task:'suspend'

Continue task:'recover'

Among them: the main difference between the two feedbacks is: **describe** field:

The first time is when the engine reports that an operation request has been received and the request is legitimate;

The second time is the feedback after the task response request is successful. The **describe** field format is: "Task loop feedback, now state is xxx".

**0.2.1.2.2.2 Feedback during task execution**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"target_id": "target_id",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "describe"</strong><br />
<strong>},</strong><br />
<strong>"block": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**feedback:** \[Required\]Request message feedback

**feedback.type**: \[required\] message type

Legal parameters:

Task:'task'

Module: 'module'

**feedback.id**: \[required\] message id

When responding to a request message, it is the original message id;

When feedbacking task status, it is a timestamp.

**feedback.target\_id**: \[required\] message target\_id

When responding to a request message, if the original message target\_id is not empty, it is the first element, otherwise it is empty;

When feedbacking task status, it is task target\_id.

**feedback.operate**: \[required\] message operation

Operations triggered by app:

Debug task/module: 'debug'

Add task/module:'save'

Modify task/module:'run'

Delete tasks/modules: 'delete'

Query task/module:'inquiry'

Terminate task:'shutdown'

Suspend task:'suspend'

Continue task:'recover'

Operations automatically triggered by tasks:

Start task:'start'

Stop the task: 'stop'

Operations that the robot wants to give feedback to the server:

Start task:'start'

**feedback.state**: \[Required\] Original message operation status

Value constraints:

0: The current operation is successful

Non-0: An exception occurs when switching to the currently specified type.

1: non-json format

2: type field, type is invalid

3: id field, id is invalid

4: target\_id field, id is invalid

5: describe field, description is invalid

6: style field, style is invalid

7: operate field, the operation is invalid

8: mode field, mode is invalid

9: condition field, the condition is invalid

10: body field, the body is invalid

21: Unable to create path

22: Unable to open file

23: The body field syntax is abnormal and cannot be constructed.

24: Unable to register

25: Unable to update list

26: Unable to execute

27: The current operation is illegal

28: Request error

29: Other errors

30: Service interrupted

31: Timeout waiting for the service to come online.

31: Request service timeout

**feedback.describe**: \[Required\] Original message operation status description

**block:** \[optional\] block information

**block.type**：\[required\] message type

Legal parameters:

Start executing target block: 'begin'

End execution target block: 'end'

**block**.**id**: \[required\] block id

**0.2.1.2.2.3 Task and module query feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"dependent": [],</strong><br />
<strong>"be_depended": []</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**response:** \[Optional\]Response information format constraints for query requests

**response.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is a task list: '**task**'

Indicates that the current list type is a module list: '**module**'

**response.id**: \[required\] message id

Legal parameters:

Keep it consistent with the request id.

**response**.**list**：\[Required\] Response list

**response.list\[\].id: **\[Required\] The id of the list element

According to the query scenario, it can be divided into:

task id;

module id;

The field in the data and the delivered data frame must be consistent.

**response.list\[\].describe：**\[Required\] Description of list elements

Task description or module description

The data is consistent with the issued data frame

**response.list\[\].style**: \[optional\] String type, identifying the current message style, that is, the presentation method of the current execution body content.

task style or module style

The field in the data and the delivered data frame must be consistent.

**response.list\[\].operate: **\[Required\] The status of the list element

Task status or module status, the legal value constraints are as follows:

"null" : \[Status\]No state (only saved content)

"error": \[status\]error status

"wait\_run": \[Status\]Waiting for running status

"run\_wait": \[status\]Run wait state

"run": \[status\] running status

"suspend": \[status\]suspended

"shutdown": \[status\] terminated

Module status, legal value constraints are as follows:

"null" : \[Status\]No state (only saved content)

"error": \[status\]error status

"normal": \[Status\]Normal state

**response.list\[\].mode: **\[required\] mode type of list element

Task mode type or module mode type

The field in the data and the delivered data frame must be consistent.

**response.list\[\].condition: **\[Required\] Constraints on list elements

Task execution constraints or module interface constraints

The field in the data and the delivered data frame must be consistent.

**response.list\[\]. dependent\[\]: **\[must\] The dependent module of the list element is used to identify which modules the current task or the current module depends on, that is, which modules are called;

**response.list\[\].be\_depended\[\]: **\[Required\] The dependency of the list element (only modules will be dependent), which may be a task or a module .

**Note:** When the dependent collection is non-empty (there are other tasks or modules that depend on the current module), the module is not allowed to be deleted.

**0.2.1.2.2.4 Personnel information query feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**response:** \[Optional\]Response information format constraints for query requests

**response.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is an artificial intelligence list: 'AI'

**response.id**: \[required\] message id

Legal parameters:

Keep it consistent with the request id.

**response**.**list**：\[Required\] Response list

**response.list\[\].id: **\[Required\] The id of the list element

According to the query scenario, it can be divided into:

Person ID;

The field in the data and the delivered data frame must be consistent.

**response.list\[\].mode: **\[required\] mode type of list element

Person type or face type or voiceprint type

The field in the data and the delivered data frame must be consistent.

**response.list\[\].style:** \[Required\] String type, identifying the current person’s face entry status. Legal values are as follows:

"0": not entered;

"1": has been entered;

"2": logging in;

**response.list\[\].condition:** \[Required\] String type, identifying the current voiceprint entry status of the person. Legal values are as follows:

"0": not entered;

"1": has been entered;

"2": logging in;

**response.list\[\].describe：**\[Required\] Description of list elements

Personnel description

**0.2.1.2.2.5 Training word query feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**response:** \[Optional\]Response information format constraints for query requests

**response.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is an artificial intelligence list: 'AI'

**response.id**: \[required\] message id

Legal parameters:

Keep it consistent with the request id.

**response**.**list**：\[Required\] Response list

**response.list\[\].id: **\[Required\] The id of the list element

Based on the query scenario, the training word id is used to present it to the user (trigger)

**response.list\[\].mode: **\[required\] mode type of list element

Training word type

**response.list\[\].style: **\[required\] string type, identification style (type)

**response.list\[\].condition: **\[required\] string type, identifying training word constraint information (value)

**response.list\[\].describe：**\[Required\] Description of list elements

Training word description

**0.2.1.2.2.6 AI all information query feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**response:** \[Optional\]Response information format constraints for query requests

**response.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is an artificial intelligence list: 'AI'

**response.id**: \[required\] message id

Legal parameters:

Keep it consistent with the request id.

**response**.**list**：\[Required\] Response list

**response.list\[\].id: **\[Required\] The id of the list element

Based on the query scenario, the training word id is used to present it to the user (trigger)

**response.list\[\].mode: **\[required\] mode type of list element

personnel: personnel type

training\_words: training word type

**response.list\[\].style: **\[Required\] Style (type) of list elements

When mode is personnel: identifies the current person's face entry status. Legal values are as follows:

"0": not entered;

"1": has been entered;

"2": logging in;

When mode is training\_words: identifies the current training word style (type)

**response.list\[\]. condition: **\[required\] Constraints on list elements

When mode is personnel: identifies the current person's voiceprint entry status. Legal values are as follows:

"0": not entered;

"1": has been entered;

"2": logging in;

When mode is training\_words: identifies the current training word operation (value)

**response.list\[\].describe：**\[Required\] Description of list elements

Training word description

**0.2.1.2.2.7 Map preset point query feedback**

<table>
<tbody>
<tr class="odd">
<td>Python<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"mode": "mode"</strong><br />
<strong>"style": "style",</strong><br />
<strong>"condition": "condition"</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>},</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**response:** \[Optional\]Response information format constraints for query requests

**response.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is an artificial intelligence list: 'SLAM'

**response.id**: \[required\] message id

Legal parameters:

Keep it consistent with the request id.

**response**.**list**：\[Required\] Response list

**response.list\[\].id: **\[Required\] The id of the list element

According to the query scenario, it can be divided into:

Preset point id;

The field in the data and the delivered data frame must be consistent.

**response.list\[\].describe：**\[Required\] Description of list elements

Preset point description

**response.list\[\].mode: **\[required\] mode type of list element

Preset point type

The field in the data and the delivered data frame must be consistent.

**response.list\[\].style: **\[required\] style type of list element

Store the preset point coordinates, the data format is as follows:

"\[x, y, z\]"

**response.list\[\].condition: **\[required\] constraint type of list elements

Map name where the preset point is located

**0.2.1.2.2.1 Protocol Description**

**0.2.1.2.3 Protocol Example**

Send a save scheduled single task:

The current task is required to be executed once every day at 21:30, but it will be destroyed after the first execution and will not be executed again.

Task content: Stand first, wait 5 seconds, and then lie down.

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "123",</strong><br />
<strong>"target_id": ["678"],</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"mode": "single",</strong><br />
<strong>"condition": "now",</strong><br />
<strong>"body": "</strong><br />
<strong>cyberdog.block('block_01')</strong><br />
<strong>cyberdog.stand_up()</strong><br />
<strong>cyberdog.block('block_02')</strong><br />
<strong>time.sleep(60)</strong><br />
<strong>cyberdog.block('block_03')</strong><br />
<strong>cyberdog.get_down()</strong><br />
<strong>"</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.2.3.1 Reporting task process**

Report when the save is successful (take the success scenario as an example):

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "0123",</strong><br />
<strong>"target_id": "678",</strong><br />
<strong>"operate": "save",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "save task"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Report when the task starts (taking a successful scenario as an example):

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "1123",</strong><br />
<strong>"target_id": "678",</strong><br />
<strong>"operate": "start",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "start task"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.2.3.2 Reported when starting execution of current task block 1**

Reported when starting to execute the current task block 1:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "2123",</strong><br />
<strong>"target_id": "678",</strong><br />
<strong>"operate": "run",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "run task"</strong><br />
<strong>},</strong><br />
<strong>"block": {</strong><br />
<strong>"type": "begin",</strong><br />
<strong>"id": "block_01"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

The rest of the blocks are similar.

**0.2.1.2.3.3 Reported when the execution of current task block 1 ends**

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"feedback": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"id": "5123",</strong><br />
<strong>"target_id": "678",</strong><br />
<strong>"operate": "run",</strong><br />
<strong>"state": 0,</strong><br />
<strong>"describe": "run task"</strong><br />
<strong>},</strong><br />
<strong>"block": {</strong><br />
<strong>"type": "end",</strong><br />
<strong>"id": "block_01"</strong><br />
<strong>}</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

The rest of the blocks are similar.

**0.2.1.2.3.4 Query task list**

Return value when module list is empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"list": []</strong><br />
<strong>},</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Return value when the module list is not empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "module",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition"</strong><br />
<strong>},</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition"</strong><br />
<strong>},</strong><br />
<strong>...</strong><br />
<strong>]</strong><br />
<strong>},</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.2.3.5 Query module list**

Return value when module list is empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "module",</strong><br />
<strong>"list": []</strong><br />
<strong>},</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Return value when the module list is not empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"response": {</strong><br />
<strong>"type": "task",</strong><br />
<strong>"list": [</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition"</strong><br />
<strong>},</strong><br />
<strong>{</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition"</strong><br />
<strong>},</strong><br />
<strong>...</strong><br />
<strong>]</strong><br />
<strong>},</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.3 Backend-\>Robot side: Report data in json format**

**0.2.1.3.1 Agreement Constraints**

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"code": "code",</strong><br />
<strong>"message": "message",</strong><br />
<strong>"request_id": "request_id",</strong><br />
<strong>"data": [</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>},</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>}</strong><br />
...<br />
<strong>]</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.3.2 Protocol Description**

**code**: \[Required\] String type, identifying the current frame status code;

200 success

400 access timeout

403 Access Denied

401 Unauthorized access

404 Resource does not exist

405 The current request method is not supported

500 Server is running abnormally

10001 Parameter cannot be empty

**message**: \[required\] String type, identifying the current frame status code description;

**request\_id**: \[required\] String type, identifying the current frame id;

**data:** \[Optional\] block information, when responding to the query request, it is an incremental query, that is, only the collection content that has not been synchronized to the robot is returned.

**data.type**：\[required\] message type

Legal parameters:

Indicates that the current list type is a task list: '**task**'

Indicates that the current list type is a module list: '**module**'

Indicates that the current list type is a record list: '**record**'

**data**.**list**: \[required\] response list

**data.list\[\].id: **\[Required\] The id of the list element

task id or module id

The field in the data and the delivered data frame must be consistent.

**data.list\[\].describe：**\[Required\] Description of list elements

Task description or module description

The data is consistent with the issued data frame

**data.list\[\].style**: \[optional\] String type, identifying the current message style, that is, the presentation method of the current execution body content.

task style or module style

The field in the data and the delivered data frame must be consistent.

**data.list\[\].operate: **\[Required\] Status of list elements

Task status or module status

The data is the current real state

**data.list\[\].mode: **\[Required\] Mode type of list elements

Task mode type or module mode type

The field in the data and the delivered data frame must be consistent.

**data.list\[\].condition: **\[Required\] Constraints on list elements

Task execution constraints or module interface constraints

The field in the data and the delivered data frame must be consistent.

**data.list\[\].body: **\[required\] script data of list elements

task script or module script

The field in the data and the delivered data frame must be consistent.

**data.list\[\].state: **\[Required\] The script state of the list element. The legal values are as follows:

Task status or module status, the legal value constraints are as follows:

"null" : \[Status\]No state (only saved content)

"error": \[status\]error status

"wait\_run": \[Status\]Waiting for running status

"run\_wait": \[status\]Run wait state

"run": \[status\] running status

"suspend": \[status\]suspended

"shutdown": \[status\] terminated

Module status, legal value constraints are as follows:

"null" : \[Status\]No state (only saved content)

"error": \[status\]error status

"normal": \[Status\]Normal state

**0.2.1.3.3 Protocol Example**

**0.2.1.3.3.1 Query task list**

Return value when module list is empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"code": "code",</strong><br />
<strong>"message": "message",</strong><br />
<strong>"request_id": "request_id",</strong><br />
<strong>"data": []</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Return value when the module list is not empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"code": "code",</strong><br />
<strong>"message": "message",</strong><br />
<strong>"request_id": "request_id",</strong><br />
<strong>"data": [</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>},</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>}</strong><br />
<strong>...</strong><br />
<strong>]</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.1.3.3.2 Query module list**

Return value when module list is empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"code": "code",</strong><br />
<strong>"message": "message",</strong><br />
<strong>"request_id": "request_id",</strong><br />
<strong>"data": []</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

Return value when the module list is not empty:

<table>
<tbody>
<tr class="odd">
<td>JSON<br />
<strong>{</strong><br />
<strong>"code": "code",</strong><br />
<strong>"message": "message",</strong><br />
<strong>"request_id": "request_id",</strong><br />
<strong>"data": [</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>},</strong><br />
<strong>{</strong><br />
<strong>"type": "type",</strong><br />
<strong>"id": "id",</strong><br />
<strong>"describe": "describe",</strong><br />
<strong>"style": "style",</strong><br />
<strong>"operate": "operate",</strong><br />
<strong>"mode": "mode",</strong><br />
<strong>"condition": "condition",</strong><br />
<strong>"body": "body"</strong><br />
<strong>}</strong><br />
<strong>...</strong><br />
<strong>]</strong><br />
<strong>}</strong></td>
</tr>
</tbody>
</table>

**0.2.2 Robot-side status transfer relationship**

**0.2.2.1 Robot-side task status transfer relationship**

| | | | | | | | | | | |
| --------------- | ----------- | ----------- | ---------- - | ----------- | ------- | ----------- | ----------- | ---- -------- | --------- | ----------- |
| Status transfer relationship table | | | | | | | | | | |
| Current status | Current operation request | | | | | | | | Task automatically triggered | |
| | inquiry | save | delete | debug | run | suspend | recover | shutdown | start | stop ) |
| Empty (null) | Empty status | Error status, waiting for running status | Empty status | Error status, running waiting status | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation |
| Error (error) | Error status | Error status, waiting for running status | Empty status | Error status, running waiting status | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation |
| Waiting to run (wait\_run) | Waiting for running status | Error status, waiting for running status | Empty status | Error status, running waiting status | Running waiting status | Illegal operation | Illegal operation | Termination status | Illegal operation | Illegal operation |
| Run wait (run\_wait) | Run wait status | Illegal operation | Illegal operation | Error status, run wait status | Illegal operation | Illegal operation | Illegal operation | Termination status | Running status | Illegal operation |
| Run (run) | Running status | Illegal operation | Illegal operation | Error status, running waiting status | Running status | Pause status | Running status | Running waiting status, termination status | Running status | Running waiting status, termination status |
| Suspend (suspend) | Suspension status | Illegal operation | Illegal operation | Error status, running waiting status | Illegal operation | Suspended status | Running status | Running waiting status, termination status | Illegal operation | Running waiting status, termination status |
| Termination (shutdown) | Termination status | Error status, waiting for running status | Empty status | Error status, running waiting status | Running waiting status | Illegal operation | Illegal operation | Termination status | Illegal operation | Illegal operation |

As shown in the table above, the flow relationship between the seven task states under the eight task operations is clearly visible. The eight task operations and seven task states are explained as follows:

**Eight types of task operations**

Save task: Construct a task with the current ID. If the task ID already exists, overwrite it, check whether the task syntax is compliant, and feedback the operation results.

Run task: Run the task corresponding to the current ID. If the task corresponding to the task ID does not exist or the syntax status is wrong, it will not be executed and the operation result will be fed back.

Query task: Query the task corresponding to the current ID and feedback the operation results.

Delete task: Delete the task corresponding to the current ID and feedback the operation results.

Pause task: Pause the task corresponding to the current ID and feedback the operation results.

Continue task: Continue the task corresponding to the current ID and feedback the operation results.

Terminate task: Terminate the task corresponding to the current ID and feedback the operation results.

Debugging task: Based on the current debugging ID, save, review and run the logic carried by the current frame, and feedback the operation results.

**Seven mission statuses**

Empty state: This state refers to any state without a current task. The essence is that the current task does not exist. That is to say, the state of any unrecorded task is an empty state.

Error status: This status means that the current task does not comply with the grammatical rules. The essence is that the current task is not compliant, which means that the current task cannot be run and can only be edited again.

Waiting to run state: This state means that the current task can be run but has not yet been added to the task registry. It is essentially the syntax rule of the current task, which means that the current task is waiting for the user to confirm the running state.

Running waiting state: This state refers to the state in which the current task has been added to the task registry, but has not yet met the running conditions. It is essentially the syntax rules of the current task, and is waiting to be run immediately after the running conditions are met.

Running state: This state refers to the state in which the current task has met the execution conditions and is executing internal logic.

Paused state: This state means that the current task is in a state of suspended execution. At this time, the task process is still there. It can be paused at a breakpoint or the user manually pauses the executing task.

Termination status: This status means that the current task is terminated and there is no task process at this time. It can be the end of normal execution or the task is forced to terminate.

**Note**: The current task status is based on the programming logic within the task and does not consider the task execution constraints (considered by the caller):

When the programming logic within a task passes the review, the task status is the wait\_run status. If the task execution constraint is a timed single execution, the caller needs to determine whether the current task's constraint time is based on the timing constraint. It has expired. Start the task again when it has not expired. Of course, you can ignore it and issue the open request directly, but it will be regarded as an illegal request and the operation will fail, which is not friendly to the user's operating experience.

**0.2.2.2 Robot module status transfer relationship**

| | | | |
| ---------- | ---------- | ---------- | ---------- |
| Status transfer relationship table | | | |
| Current status | Current operation request | | |
| | query | save | delete |
| Empty (null) | Empty status | Error status, normal status | Illegal operation |
| Error (error) | Error status | Error status, normal status | Empty status |
| Normal (normal) | Normal state | Error state, normal state | Empty state, normal state |

As shown in the table above, the flow relationship between the three module states under the three module operations is clear at a glance. The three module operations and three module states are explained as follows:

**Three types of module operations**

Save task: Build the module with the current ID. If the module ID already exists, overwrite it, check whether the task syntax is compliant, and feedback the operation results.

Query task: Query the module corresponding to the current ID and feedback the operation results.

Delete task: Review whether the module corresponding to the current ID can be deleted. If so, delete the module corresponding to the current ID and feedback the operation results.

**Three module states**

Empty state: This state refers to any state without the current module. The essence is that the current module does not exist, which means that the state of any unrecorded module is empty.

Error status: This status means that the current module does not comply with the grammatical rules, which means that the current module cannot be run and can only be edited again.

Normal state: This state means that the current module complies with the grammar rules, which means that the current module can be called.

**0.2.2.3 Operation constraints on the robot side in each mode**

> Illegal operations will be rejected, legal operations will be adopted, and the status will be fed back to the requesting party regardless of whether it is illegal or legal.

| | | | | | | | | | | |
| ------------ | ------------- | ----------------------- --- | ----------- | ---------- | --------- | ------- | ----- ------ | ----------- | ------------ | --------- | -------- |
| Operation constraint table in each mode of the robot | | | | | | | | | | | |
| Current mode | | User query operations on tasks, modules, AI capabilities, and SLAM capabilities | User operations on tasks or modules | | User operations on tasks | | | | | Automatic triggering of tasks | |
| Schema name | Schema identifier | query | save | delete | debug | run | suspend | continue (recover) | terminate (shutdown) | start ) | stop |
| Uninitialized | Uninitialized | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation |
| Resource loading mode | SetUp | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation |
| Resource release mode | TearDown | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation |
| Self-check mode | SekfCheck | Legal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Legal operation | Illegal operation | Legal operation | Illegal operation | Legal operation |
| Active mode | Active | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation | Legal operation |
| Silent Mode | DeActive | Legal Operation | Legal Operation | Legal Operation | Illegal Operation | Illegal Operation | Legal Operation | Illegal Operation | Legal Operation | Illegal Operation | Legal Operation |
| Low Power Mode | Protected | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation | Legal Operation |
| Low Power Consumption Mode | LowPower | Legal Operation | Legal Operation | Legal Operation | Illegal Operation | Illegal Operation | Legal Operation | Illegal Operation | Legal Operation | Illegal Operation | Legal Operation |
| Remote upgrade mode | OTA | Legal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Legal operation | Illegal operation | Legal operation | Illegal operation | Legal operation |
| Error mode | Error | Legal operation | Illegal operation | Illegal operation | Illegal operation | Illegal operation | Legal operation | Illegal operation | Legal operation | Illegal operation | Legal operation |

As shown in the table above, there are eight user operations on tasks or modules in ten robot modes, one user operation on AI capabilities, and two operation constraints triggered by tasks themselves. The constraint indicators in each mode are mainly considered. as follows:

In the three modes of Uninitialized, SetUp, and TearDown, each functional module of the robot is in a state where it cannot work normally, so all operations are restricted;

In SelfCheck, OTA and Error modes, each functional module of the robot is in a closed state, so all operations that may cause new processes in graphical programming are restricted, and only existing processes are allowed to automatically stop (stop) or the user manually suspend (suspend) or terminate. (shutdown) Existing processes, allowing inquiry (inquiry) are intended to provide users with display tasks, modules and AI information, provide a basis for users to suspend (suspend) or terminate, and also support the function of secondary editing tasks or modules, improving Asynchronous experience during the interaction between user and robot (will not block user programming due to self-test), this idea is also applicable to Active, DeActive, Protected, LowPower, OTA and Error modes;

In Active mode, all functional modules of the robot are in a stage where they can work normally, so all operations are open;

In Protected mode, most of the robot's functional modules are in a stage where they can work normally, so all operations are open. The restricted functions in this mode are as follows:

Movement module: all result commands except standing and lying down;

LED module: BMS will seize the LED device.

In DeActive and LowPower modes, each functional module of the robot is in a dormant state. The constraints of inquiry, stop, suspend or shutdown are the same as those in SekfCheck mode. For save and delete ) operation is considered the same as query. It also wants to provide users with display tasks, modules and AI information, provide a basis for users to suspend or terminate, and also support the function of secondary editing tasks or modules to improve the interaction between users and robots. Asynchronous experience during the process (no self-test blocking user programming);

<table>
<tbody>
<tr class="odd">
<td><p>Low battery mode</p>
<p>Enter:</p>
<p>When the battery is lower than 20%, it will enter automatically;</p>
<p>Exit:</p>
<p>When the battery level is greater than or equal to 20%, it will automatically exit;</p>
<p>Low power mode</p>
<p>Enter:</p>
<p>Automatically enter low power consumption when the battery level is less than 5%;</p>
<p>Lie down for more than 30 seconds and enter low power consumption;</p>
<p>After waking up and exiting the low-power mode, if no motion control is performed within 30 seconds, the low-power mode will be entered again;</p>
<p>Exit:</p>
<p>Voice wake-up: "Iron egg, iron egg";</p>
<p>Click on the app side to exit low power consumption;</p>
<p>Double-click the dog head to exit low power consumption;</p></td>
</tr>
</tbody>
</table>

**1. Basic constraints**

**1.1 Reserved words**

| | | | | | | | | | |
| ------ | ------- | ---- | -------- | ------ | ------ | ---- | - ----- | ----- | ------ |
| and | exec | not | class | from | print | del | import | try | except |
| assert | finally | or | continue | global | raise | elif | in | while | lambda |
| break | for | pass | def | if | return | else | is | with | yield |

**1.2 Operators**

**1.2.1 Arithmetic operators**

<table>
<tbody>
<tr class="odd">
<td><strong>Operator</strong></td>
<td><strong>Description</strong></td>
<td><strong>Example (a=10, b=20)</strong></td>
</tr>
<tr class="even">
<td><strong>+</strong></td>
<td>Add: Add two objects</td>
<td>a + b outputs 30</td>
</tr>
<tr class="odd">
<td><strong>-</strong></td>
<td>Subtraction: Get a negative number or subtract one number from another number</td>
<td>a - b output result -10</td>
</tr>
<tr class="even">
<td><strong>*</strong></td>
<td>Multiplication: Multiply two numbers or return a string repeated several times</td>
<td>a * b output result 200</td>
</tr>
<tr class="odd">
<td><strong>/</strong></td>
<td>Division: x divided by y</td>
<td>b / a output result 2</td>
</tr>
<tr class="even">
<td><strong>%</strong></td>
<td>Modulo: Return the remainder of division</td>
<td>b % a output result 0</td>
</tr>
<tr class="odd">
<td><strong>**</strong></td>
<td>Power: Returns the y power of x</td>
<td>a**b is 10 raised to the 20th power, and the output result is 100000000000000000000</td>
</tr>
<tr class="even">
<td><strong>//</strong></td>
<td>Round and divide: Return the integer part of the quotient (rounded down)</td>
<td><p>9//2 output result 4</p>
<p>-9//2 output result -5</p></td>
</tr>
</tbody>
</table>

**1.2.2 Comparison operators**

All comparison operators return 1 for true and 0 for false. These are equivalent to the special variables True and False respectively.

| | | |
| ------- | ---------------- | ------------------- |
| **Operator** | **Description** | **Example (a=10, b=20)** |
| **==** | Equals: Compares whether objects are equal | (a == b) Returns false. |
| **\!=** | Not equal to: Compares whether two objects are equal | (a \!= b) returns true. |
| **\>** | Greater than: returns whether x is greater than y | (a \> b) returns false. |
| **\<** | Less than: Returns whether x is less than y. | (a \< b) returns true. |
| **\>=** | Greater than or equal to: Returns whether x is greater than or equal to y. | (a \>= b) returns false. |
| **\<=** | Less than or equal to: Returns whether x is less than or equal to y. | (a \<= b) returns true. |

**1.2.3 Assignment operator**

| | | |
| --------- | -------- | ---------------------------- |
| **Operator** | **Description** | **Example (a=10, b=20)** |
| **=** | Simple assignment operator | c = a + b assigns the result of a + b to c |
| **+=** | Addition assignment operator | c += a is equivalent to c = c + a |
| **-=** | Subtractive assignment operator | c -= a is equivalent to c = c - a |
| **\*=** | Multiplicative assignment operator | c \*= a is equivalent to c = c \* a |
| **/=** | Division assignment operator | c /= a is equivalent to c = c / a |
| **%=** | Modulo assignment operator | c %= a is equivalent to c = c % a |
| **\*\*=** | Power assignment operator | c \*\*= a is equivalent to c = c \*\* a |
| **//=** | Integer division assignment operator | c //= a is equivalent to c = c // a |

**1.2.4 Logical operators**

| | | | |
| ------- | --------- | ------------------------------- --------------------- | --------------------- |
| **Operator** | **Logical expression** | **Description** | **Example (a=10, b=20)** |
| **and** | x and y | Boolean AND - x and y returns False if x is False, otherwise it returns the computed value of y. | (a and b) returns 20. |
| **or** | x or y | Boolean OR - if x is non-zero, it returns the computed value of x, otherwise it returns the computed value of y. | (a or b) returns 10. |
| **not** | not x | Boolean "not" - Returns False if x is True. If x is False, it returns True. | not(a and b) returns False |

**1.2.5 Member Operator**

| | | |
| ------- | ---------------------------------- | ------- -------------------------- |
| **Operator** | **Description** | **Example** |
| in | Returns True if a value is found in the specified sequence, False otherwise. | x is in the y sequence, Returns True if x is in the y sequence. |
| not in | Returns True if the value is not found in the specified sequence, False otherwise. | x is not in the y sequence, returns True if x is not in the y sequence. |

**1.2.6 Identity Operator**

| | | |
| ------- | ----------------------- | --------------- -------------------------------------------------- ---------- |
| **Operator** | **Description** | **Example** |
| is | is is used to determine whether two identifiers refer to the same object | **x is y**, similar to **id(x) == id(y)**, if they refer to the same object, it will be returned True, otherwise returns False |
| is not | is not is to determine whether two identifiers refer to different objects | **x is not y**, similar to **id(a) \!= id(b)**. Returns True if the referenced object is not the same, False otherwise. |

**1.2.7 Bit Operators**

| | | |
| -------- | ---------------------------------------- ------------------ | ---------------------------------- ------------------ |
| **Operator** | **Description** | **Example (a=60, b=13)** |
| **&** | Bitwise AND operator: Two values participating in the operation, if the two corresponding bits are 1, the result of the bit is 1, otherwise it is 0 | (a & b) The output result is 12, Binary interpretation: 0000 1100 |
| **|** | Bitwise OR operator: As long as one of the two corresponding binary bits is 1, the result bit will be 1. | (a | b) Output result 61, binary interpretation: 0011 1101 |
| **^** | Bitwise XOR operator: When the two corresponding binary bits are different, the result is 1 | (a ^ b) The output result is 49, binary interpretation: 0011 0001 |
| **\~** | Bitwise negation operator: negates each binary bit of the data, that is, changes 1 to 0 and 0 to 1. **\~x** Similar to **-x-1** | (\~a ) Output result -61 , binary interpretation: 1100 0011, in the two's complement form of a signed binary number. |
| **\<\<** | Left shift operator: All binary bits of the operand are shifted to the left by a certain number of bits. The number on the right of **\<\<** specifies the number of bits to move. The high bits are discarded and the low bits are discarded. Fill with 0. | a \<\< 2 output result 240, binary interpretation: 1111 0000 |
| **\>\>** | Right shift operator: Shift all the binary digits of the operand to the left of "\>\>" to the right by a certain number of bits, **\>\>** The number on the right specifies the move Number of digits | a \>\> 2 The output result is 15, binary interpretation: 0000 1111 |

**1.2.8 Operator precedence**

| | |
| ---------------------------- | ---------------------------- ------------- |
| **operator** | **description** |
| \*\* | index (highest priority) |
| \~ + - | Bitwise flip, unary plus and minus signs (the last two methods are named +@ and -@) |
| \* / % // | Multiplication, division, modulo and integer division |
| \+ - | Addition and subtraction |
| \>\> \<\< | Right shift, left shift operator |
| & | bit 'AND' |
| ^ | | bitwise operator |
| \<= \< \> \>= | Comparison operator |
| \<\> == \!= | Equality operator |
| \= %= /= //= -= += \*= \*\*= | Assignment operator |
| is is not | identity operator |
| in not in | member operator |
| not and or | logical operator |

**1.2 Code block constraints**

Like Python, code blocks do not use curly braces **{}** to control classes, functions and other logical judgments. Use indentation to write modules.

The amount of indented whitespace is variable, but all code block statements must contain the same amount of indented whitespace, and this must be strictly enforced.

**1.3 variable**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example 1</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
key <strong>=</strong> value</td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
size = 10</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**1.4 function**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example 1</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>def</strong> functionname<strong>(</strong><br />
parameters<br />
<strong>):</strong><br />
function_code<br />
<strong>return [</strong>expression<strong>]</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Python<br />
def print_msg(msg):<br />
print(msg)<br />
return</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**1.5 Built-in functions**

The functions involved in Sections 4 to 9 are all built-in functions and can be called directly. In addition, there are the following internal value functions.

| | | |
| -- | -- | -- |
| Function | Function | Example |
| | | |

**2. Type constraints**

**2.1 Numbers**

**2.1.1 int (signed integer type)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example 1</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>key = &lt;int&gt;</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>size_a = 1</strong><br />
<strong>size_b = -1</strong><br />
<strong>size_c = 0x01</strong><br />
<strong>size_d = -0x01</strong><br />
...</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**2.1.2 float (floating point type)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example 1</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>key = &lt;float&gt;</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>size_a = 0.1</strong><br />
<strong>size_b = -0.1</strong><br />
<strong>size_c = 0.1e+18</strong><br />
<strong>size_d = 0.1</strong><br />
...</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**2.1.3 complex**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example 1</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>key = &lt;complex&gt;</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>size_a = 3.14j</strong><br />
<strong>size_b =</strong> 3e+26J<br />
<strong>size_c =</strong> 0.1e-7j<br />
size_d = -.6545+0J</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**2.2 String**

[Reference link](https://docs.python.org/zh-cn/3/tutorial/index.html)

**2.3 List**

[Reference link](https://www.runoob.com/python/python-lists.html)

**2.4 Tuple**

[Reference link](https://www.runoob.com/python/python-tuples.html)

**2.5 Dictionary**

[Reference link](https://www.runoob.com/python/python-dictionary.html)

**3. Logic control**

**3.1 if**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if</strong> conditional_expression_A:<br />
function_code_A<br />
<strong>elif</strong> conditional_expression_B:<br />
function_code_B<br />
<strong>else</strong>:<br />
function_code_C</td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if</strong> 1 &lt; 2:<br />
print('1&lt;2')<br />
<strong>elif</strong> 1 &gt; 2:<br />
print('1&gt;2')<br />
<strong>else</strong>:<br />
print('1==2')</td>
</tr>
</tbody>
</table>
<p>Will output:</p>
<table>
<tbody>
<tr class="odd">
<td><br />
1&lt;2</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**3.2 for**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>for</strong> iterating_var <strong>in</strong> sequence<strong>:</strong><br />
function_code</td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Perl<br />
<strong>for</strong> now_char <strong>in</strong> 'hello!':<br />
<strong>print(</strong>'Current character:', <strong>now_char)</strong></td>
</tr>
</tbody>
</table>
<p>Will output:</p>
<table>
<tbody>
<tr class="odd">
<td><br />
Current character: h<br />
Current character: e<br />
Current character: l<br />
Current character: l<br />
Current character: o<br />
Current character: !</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**3.3 while**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>while</strong> conditional_expression<strong>：</strong><br />
function_code</td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Swift<br />
now_num = 0<br />
while now_num &lt; 2:<br />
<strong>print(</strong>'Current number:', now_num<strong>)</strong><br />
now_num = now_num + 1</td>
</tr>
</tbody>
</table>
<p>Will output:</p>
<table>
<tbody>
<tr class="odd">
<td><br />
Current number: 0<br />
Current number: 1</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**3.4 break**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example (with the help of while explanation)</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>break</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Swift<br />
now_num = 0<br />
while true:<br />
<strong>print(</strong>'Current number:', now_num<strong>)</strong><br />
now_num = now_num + 1<br />
break<br />
<br />
<strong>print(</strong>'Final number:', now_num<strong>) </strong></td>
</tr>
</tbody>
</table>
<p>Will output:</p>
<table>
<tbody>
<tr class="odd">
<td><br />
Current number: 0<br />
Final number: 1</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**3.5 continue**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Example (with the help of while explanation)</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>continue</strong></td>
</tr>
</tbody>
</table></td>
<td><table>
<tbody>
<tr class="odd">
<td>Swift<br />
now_num = 0<br />
while now_num &lt; 2:<br />
now_num = now_num + 1<br />
continue<br />
<strong>print(</strong>'Current number:', now_num<strong>)</strong><br />
<br />
<strong>print(</strong>'Final number:', now_num<strong>) </strong></td>
</tr>
</tbody>
</table>
<p>Will output:</p>
<table>
<tbody>
<tr class="odd">
<td><br />
Final number: 2</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4. Cyberdog ability set (Iron Egg ability set)**

Here, in order for Tiedan to better serve customers, we have abstracted most of Tiedan's capabilities into a collection of easy-to-understand functional interfaces for users to call directly, which is the main content of this section. Before that, we have The effective values of iron egg general parameters are subject to the following constraints:

**4.0 Iron Egg Coordinate System**

<table>
<tbody>
<tr class="odd">
<td>Right-hand criterion coordinate system</td>
<td>Right-hand criterion rotation direction</td>
</tr>
<tr class="even">
<td><p>The finger pointing direction is the positive direction of the axis, which is:</p>
<p>X-axis speed direction: front positive and back negative</p>
<p>Y-axis speed direction: left positive, right negative</p>
<p>Z-axis speed direction: left positive, right negative</p></td>
<td><p>The finger pointing direction is the positive direction of the axis, which is:</p>
<p>X-axis rotation direction: left negative and right positive</p>
<p>Y-axis rotation direction: up negative and down positive</p>
<p>Z-axis rotation direction: left positive, right negative</p></td>
</tr>
<tr class="odd">
<td>Euler angles</td>
<td></td>
</tr>
<tr class="even">
<td><a href="https://quaternions.online/">Explanation of Euler angles and quaternions</a></td>
<td></td>
</tr>
</tbody>
</table>

As shown above, the iron egg conforms to the right-hand lateral coordinate system,

**4.1 Tiedan ability set parameter constraint table**

<table>
<tbody>
<tr class="odd">
<td>Serial number</td>
<td>Classification</td>
<td>Parameters</td>
<td>Variables</td>
<td>Unit</td>
<td><p>Resolution</p>
<p>(Control granularity)</p></td>
<td>Numeric type</td>
<td>Semantic Type</td>
<td>Minimum value</td>
<td>Default</td>
<td>Maximum value</td>
<td>Direction</td>
<td>Remarks</td>
</tr>
<tr class="even">
<td>1</td>
<td>Sports</td>
<td>Center of mass X-axis constraint</td>
<td>centroid_x</td>
<td>m</td>
<td>0.001m</td>
<td>Floating point numbers</td>
<td><del>Absolute (not open)</del></td>
<td>0</td>
<td>0</td>
<td>0</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.4</td>
<td>0</td>
<td>0.4</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>2</td>
<td></td>
<td>Center of mass Y-axis constraint</td>
<td>centroid_y</td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td><del>Absolute (not open)</del></td>
<td>0</td>
<td>0</td>
<td>0</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.3</td>
<td>0</td>
<td>0.3</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>3</td>
<td></td>
<td>Center of mass Z axis constraint</td>
<td>zcentroid_z</td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>Absolutely</td>
<td>0.1</td>
<td>0</td>
<td>0.3</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.2</td>
<td>0</td>
<td>0.2</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>4</td>
<td></td>
<td>Pivot X-axis constraint</td>
<td>fulcrum_x</td>
<td>m</td>
<td>0.005m</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.5</td>
<td>0</td>
<td>0.5</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>5</td>
<td></td>
<td>Fulcrum Y-axis constraint</td>
<td>fulcrum_y</td>
<td>m</td>
<td>0.005m</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.5</td>
<td>0</td>
<td>0.5</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>6</td>
<td></td>
<td>Pivot Z axis constraint</td>
<td>fulcrum_z</td>
<td>m</td>
<td>0.005m</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.5</td>
<td>0</td>
<td>0.5</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>7</td>
<td></td>
<td>Body roll: X axis</td>
<td>roll</td>
<td>deg</td>
<td>0.1</td>
<td>Floating point numbers</td>
<td>Absolutely</td>
<td>-0.45*57.3=-25.8</td>
<td>0</td>
<td>0.45*57.3=25.8</td>
<td>Direction: left negative, right positive</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.9*57.3=-51.6</td>
<td>0</td>
<td>0.9*57.3=51.6</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>8</td>
<td></td>
<td>Body pitch: Y axis</td>
<td>pitch</td>
<td>deg</td>
<td>0.1</td>
<td>Floating point numbers</td>
<td>Absolutely</td>
<td>-0.45*57.3=-25.8</td>
<td>0</td>
<td>0.45*57.3=25.8</td>
<td>Direction: up negative and down positive</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.9*57.3=-51.6</td>
<td>0</td>
<td>0.9*57.3=51.6</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>9</td>
<td></td>
<td>Body yaw: Z axis</td>
<td>yaw</td>
<td>deg</td>
<td>0.1</td>
<td>Floating point numbers</td>
<td>Absolutely</td>
<td>-0.45*57.3=-25.8</td>
<td>0</td>
<td>0.45*57.3=25.8</td>
<td>Direction: left positive, right negative</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>Floating point numbers</td>
<td>relative</td>
<td>-0.9*57.3=-51.6</td>
<td>0</td>
<td>0.9*57.3=51.6</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>10</td>
<td></td>
<td>x-axis walking speed</td>
<td>x_velocity</td>
<td>m/s</td>
<td>0.01</td>
<td>Floating point numbers</td>
<td></td>
<td>-1.6</td>
<td>0</td>
<td>1.6</td>
<td>Direction: forward positive and back negative</td>
<td></td>
</tr>
<tr class="even">
<td>11</td>
<td></td>
<td>y-axis walking speed</td>
<td>y_velocity</td>
<td>m/s</td>
<td>0.01</td>
<td>Floating point numbers</td>
<td></td>
<td>-1.2</td>
<td>0</td>
<td>1.2</td>
<td>Direction: left positive, right negative</td>
<td></td>
</tr>
<tr class="odd">
<td>12</td>
<td></td>
<td>z-axis walking speed</td>
<td>z_velocity</td>
<td>deg/s</td>
<td>0.1</td>
<td>Floating point numbers</td>
<td></td>
<td>-2.5*57.3=-114.6</td>
<td>0</td>
<td>2.5*57.3= 114.6</td>
<td>Direction: left positive, right negative</td>
<td></td>
</tr>
<tr class="even">
<td>13</td>
<td></td>
<td>x-axis jumping speed</td>
<td>x_jump_velocity</td>
<td>m/s</td>
<td>0.01</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.25</td>
<td>0</td>
<td>0.15</td>
<td>Direction: forward positive and back negative</td>
<td></td>
</tr>
<tr class="odd">
<td>14</td>
<td></td>
<td>y-axis jumping speed</td>
<td>y_jump_velocity</td>
<td>m/s</td>
<td>0.01</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.1</td>
<td>0</td>
<td>0.1</td>
<td>Direction: left positive, right negative</td>
<td></td>
</tr>
<tr class="even">
<td>15</td>
<td></td>
<td>z-axis jump speed</td>
<td>z_jump_velocity</td>
<td>deg/s</td>
<td>0.1</td>
<td>Floating point numbers</td>
<td></td>
<td>-0.5* 57.3=-28.65</td>
<td>0</td>
<td>0.5* 57.3=28.65</td>
<td>Direction: left positive, right negative</td>
<td></td>
</tr>
<tr class="odd">
<td>16</td>
<td></td>
<td>Leg lift height of front legs</td>
<td>front_leg_lift</td>
<td>m</td>
<td>0.001</td>
<td>Floating point numbers</td>
<td></td>
<td>0.005</td>
<td>0.03</td>
<td>0.1</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>17</td>
<td></td>
<td>Leg lift height of front legs</td>
<td>back_leg_lift</td>
<td>m</td>
<td>0.001</td>
<td>Floating point numbers</td>
<td></td>
<td>0.005</td>
<td>0.03</td>
<td>0.1</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>18</td>
<td></td>
<td>Expected distance</td>
<td>distance</td>
<td>m</td>
<td>0.01</td>
<td>Floating point numbers</td>
<td></td>
<td>0</td>
<td>0</td>
<td>10</td>
<td></td>
<td>0: Invalid field</td>
</tr>
<tr class="even">
<td>19</td>
<td></td>
<td>Expected time consumption</td>
<td>duration</td>
<td>s</td>
<td>2</td>
<td>Floating point numbers</td>
<td></td>
<td>0</td>
<td>1</td>
<td>6</td>
<td></td>
<td>0: Invalid field</td>
</tr>
<tr class="odd">
<td>20</td>
<td>Voice</td>
<td>Volume</td>
<td>volume</td>
<td>%</td>
<td>1</td>
<td>Integer type</td>
<td></td>
<td>0</td>
<td>-1</td>
<td>100</td>
<td>The larger the value, the louder the volume</td>
<td>-1: Invalid field</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

**4.2 Common type constraint table of Tiedan ability set**

**[Time](https://docs.ros2.org/latest/api/builtin_interfaces/msg/Time.html): timestamp**

**[Header](https://docs.ros2.org/latest/api/std_msgs/msg/Header.html)：Frame header**

**[LaserScan](https://docs.ros2.org/latest/api/sensor_msgs/msg/LaserScan.html): Radar message**

**[Range](https://docs.ros2.org/latest/api/sensor_msgs/msg/Range.html): Ultrasonic message**

**[Odometry](https://docs.ros2.org/latest/api/nav_msgs/msg/Odometry.html): Odometry message**

**[Imu](https://docs.ros2.org/latest/api/sensor_msgs/msg/Imu.html): Inertial navigation message**

**[Point](https://docs.ros2.org/latest/api/geometry_msgs/msg/Point.html)：Point message**

**[Quaternion](https://docs.ros2.org/latest/api/geometry_msgs/msg/Quaternion.html): Quaternion messages**

**[Pose](https://docs.ros2.org/latest/api/geometry_msgs/msg/Pose.html): Inertial navigation message**

**[Vector3](https://docs.ros2.org/latest/api/geometry_msgs/msg/Vector3.html): Inertial navigation message**

**[Twist](https://docs.ros2.org/latest/api/geometry_msgs/msg/Twist.html): Inertial navigation message**

**[PoseWithCovariance](https://docs.ros2.org/latest/api/geometry_msgs/msg/PoseWithCovariance.html): Inertial navigation message**

**[TwistWithCovariance](https://docs.ros2.org/latest/api/geometry_msgs/msg/TwistWithCovariance.html): Inertial navigation message**

**BmsStatus: Battery Management System Message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://docs.ros2.org/latest/api/std_msgs/msg/Header.html">BmsStatus</a></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>header</td>
<td><a>Header</a></td>
<td>Message header</td>
</tr>
<tr class="odd">
<td></td>
<td>batt_volt</td>
<td>int</td>
<td>Voltage - mV</td>
</tr>
<tr class="even">
<td></td>
<td>batt_curr</td>
<td>int</td>
<td>Current - mA</td>
</tr>
<tr class="odd">
<td></td>
<td>batt_soc</td>
<td>int</td>
<td>Remaining battery</td>
</tr>
<tr class="even">
<td></td>
<td>batt_temp</td>
<td>int</td>
<td>Temperature - C</td>
</tr>
<tr class="odd">
<td></td>
<td>batt_st</td>
<td>int</td>
<td>Battery mode: bit0 - normal mode; bit1 - charging; bit2 - charging completed; bit3 - motor power down; bit4 - soft shutdown</td>
</tr>
<tr class="even">
<td></td>
<td>key_val</td>
<td>int</td>
<td>Shutdown signal: 1 - shutdown; 0 - normal/no shutdown</td>
</tr>
<tr class="odd">
<td></td>
<td>disable_charge</td>
<td>int</td>
<td>Disable charging</td>
</tr>
<tr class="even">
<td></td>
<td>power_supply</td>
<td>int</td>
<td>Power supply</td>
</tr>
<tr class="odd">
<td></td>
<td>buzz</td>
<td>int</td>
<td>Beep</td>
</tr>
<tr class="even">
<td></td>
<td>status</td>
<td>int</td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>batt_health</td>
<td>int</td>
<td>Battery Health</td>
</tr>
<tr class="even">
<td></td>
<td>batt_loop_number</td>
<td>int</td>
<td>Battery cycle count</td>
</tr>
<tr class="odd">
<td></td>
<td>powerboard_status</td>
<td>int</td>
<td>Power board status: bit0 - serial port error {1 - error; 0 - no error}</td>
</tr>
<tr class="even">
<td></td>
<td>power_normal</td>
<td>bool</td>
<td>Normal mode</td>
</tr>
<tr class="odd">
<td></td>
<td>power_wired_charging</td>
<td>bool</td>
<td>Wired charging</td>
</tr>
<tr class="even">
<td></td>
<td>power_finished_charging</td>
<td>bool</td>
<td>Charging completed</td>
</tr>
<tr class="odd">
<td></td>
<td>power_motor_shutdown</td>
<td>bool</td>
<td>Motor power out</td>
</tr>
<tr class="even">
<td></td>
<td>power_soft_shutdown</td>
<td>bool</td>
<td>Soft shutdown</td>
</tr>
<tr class="odd">
<td></td>
<td>power_wp_place</td>
<td>bool</td>
<td>Wireless charging in place</td>
</tr>
<tr class="even">
<td></td>
<td>power_wp_charging</td>
<td>bool</td>
<td>Wireless charging</td>
</tr>
<tr class="odd">
<td></td>
<td>power_expower_supply</td>
<td>bool</td>
<td>External power supply</td>
</tr>
</tbody>
</table>

**TouchStatus: Touchpad message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/TouchStatus.msg">TouchStatus</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>header</td>
<td><a>Header</a></td>
<td>Message header</td>
</tr>
<tr class="odd">
<td></td>
<td>touch_state</td>
<td>int</td>
<td>Touchpad status {1: single click, 2: double click, 3: long press}</td>
</tr>
</tbody>
</table>

**GpsPayload: Global Positioning System Message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/GpsPayload.msg">GpsPayload</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>sec</td>
<td>int</td>
<td>Seconds</td>
</tr>
<tr class="odd">
<td></td>
<td>nanosec</td>
<td>int</td>
<td>Nanosecond</td>
</tr>
<tr class="even">
<td></td>
<td>itow</td>
<td>int</td>
<td>GPS timestamp</td>
</tr>
<tr class="odd">
<td></td>
<td>fix_type</td>
<td>int</td>
<td>GNSS type</td>
</tr>
<tr class="even">
<td></td>
<td>num_sv</td>
<td>int</td>
<td>Current number of satellite search satellites</td>
</tr>
<tr class="odd">
<td></td>
<td>lon</td>
<td>int</td>
<td>Longitude</td>
</tr>
<tr class="even">
<td></td>
<td>lat</td>
<td>int</td>
<td>Latitude</td>
</tr>
</tbody>
</table>

**SingleTofPayload: Single Tof data message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/SingleTofPayload.msg">SingleTofPayload</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>header</td>
<td><a>Header</a></td>
<td>Message header</td>
</tr>
<tr class="odd">
<td></td>
<td>data_available</td>
<td>bool</td>
<td>Is the data available</td>
</tr>
<tr class="even">
<td></td>
<td>tof_position</td>
<td>int</td>
<td>The position of the sensor (front left: 0, front right: 1, rear left: 2, rear right: 3)</td>
</tr>
<tr class="odd">
<td></td>
<td>data</td>
<td>float</td>
<td>Sensor data[m]</td>
</tr>
</tbody>
</table>

**HeadTofPayload: Head Tof data message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/HeadTofPayload.msg">HeadTofPayload</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>left_head</td>
<td><a>SingleTofPayload</a></td>
<td>Left side of head Tof</td>
</tr>
<tr class="odd">
<td></td>
<td>right_head</td>
<td><a>SingleTofPayload</a></td>
<td>Tof on the right side of the head</td>
</tr>
</tbody>
</table>

**RearTofPayload: Rear Tof data message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/RearTofPayload.msg">RearTofPayload</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>left_rear</td>
<td><a>SingleTofPayload</a></td>
<td>Tail left Tof</td>
</tr>
<tr class="odd">
<td></td>
<td>right_rear</td>
<td><a>SingleTofPayload</a></td>
<td>Tail right Tof</td>
</tr>
</tbody>
</table>

**TofPayload: Laser ranging message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>TofPayload</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>head</td>
<td><a>HeadTofPayload</a></td>
<td>Head tof</td>
</tr>
<tr class="odd">
<td></td>
<td>rear</td>
<td><a>RearTofPayload</a></td>
<td>Tail tof</td>
</tr>
</tbody>
</table>

**StateCode: status code**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Enumeration</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>StateCode</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>value</p>
<p>About</p>
<p>Bundle</p></td>
<td>key</td>
<td>value</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>invalid</td>
<td>-1</td>
<td>Invalid</td>
</tr>
<tr class="odd">
<td></td>
<td>success</td>
<td>0x00</td>
<td>Success</td>
</tr>
<tr class="even">
<td></td>
<td>fail</td>
<td>0x01</td>
<td>Failed</td>
</tr>
<tr class="odd">
<td></td>
<td>no_data_update</td>
<td>0x20</td>
<td>No data updated</td>
</tr>
<tr class="even">
<td></td>
<td>command_waiting_execute</td>
<td>0x30</td>
<td>An error occurred while waiting to be executed</td>
</tr>
<tr class="odd">
<td></td>
<td>service_client_interrupted</td>
<td>0x40</td>
<td>The client was interrupted when the requested service occurred</td>
</tr>
<tr class="even">
<td></td>
<td>service_appear_timeout</td>
<td>0x41</td>
<td>Timeout waiting for the service to appear (start)</td>
</tr>
<tr class="odd">
<td></td>
<td>service_request_interrupted</td>
<td>0x42</td>
<td>Request service interruption</td>
</tr>
<tr class="even">
<td></td>
<td>service_request_timeout</td>
<td>0x43</td>
<td>Request service timeout/delay</td>
</tr>
<tr class="odd">
<td></td>
<td>spin_future_interrupted</td>
<td>0x50</td>
<td>Request service interruption</td>
</tr>
<tr class="even">
<td></td>
<td>spin_future_timeout</td>
<td>0x51</td>
<td>Request service timeout/delay</td>
</tr>
</tbody>
</table>

**State: state**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>State</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>code</td>
<td><a>StateCode</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>describe</td>
<td>string</td>
<td>Status description</td>
</tr>
</tbody>
</table>

**DefaultAndMaximum: Default type of motion parameters**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>DefaultAndMaximum</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>minimum_value</td>
<td>double</td>
<td>Minimum value</td>
</tr>
<tr class="odd">
<td></td>
<td>default_value</td>
<td>double</td>
<td>Default value</td>
</tr>
<tr class="even">
<td></td>
<td>maximum_value</td>
<td>double</td>
<td>Maximum value</td>
</tr>
<tr class="odd">
<td></td>
<td>unit</td>
<td>string</td>
<td>Unit</td>
</tr>
</tbody>
</table>

**MotionParams: Motion parameter message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionParams</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>centroid_x</td>
<td><a>DefaultAndMaximum</a></td>
<td>Center of mass X-axis constraint</td>
</tr>
<tr class="odd">
<td></td>
<td>centroid_y</td>
<td><a>DefaultAndMaximum</a></td>
<td>Center of mass Y-axis constraint</td>
</tr>
<tr class="even">
<td></td>
<td>centroid_z</td>
<td><a>DefaultAndMaximum</a></td>
<td>Center of mass Z axis constraint</td>
</tr>
<tr class="odd">
<td></td>
<td>fulcrum_x</td>
<td><a>DefaultAndMaximum</a></td>
<td>Pivot X-axis constraint</td>
</tr>
<tr class="even">
<td></td>
<td>fulcrum_y</td>
<td><a>DefaultAndMaximum</a></td>
<td>Fulcrum Y-axis constraint</td>
</tr>
<tr class="odd">
<td></td>
<td>fulcrum_z</td>
<td><a>DefaultAndMaximum</a></td>
<td>Pivot Z axis constraint</td>
</tr>
<tr class="even">
<td></td>
<td>roll</td>
<td><a>DefaultAndMaximum</a></td>
<td>Body roll</td>
</tr>
<tr class="odd">
<td></td>
<td>pitch</td>
<td><a>DefaultAndMaximum</a></td>
<td>Fuselage pitch</td>
</tr>
<tr class="even">
<td></td>
<td>yaw</td>
<td><a>DefaultAndMaximum</a></td>
<td>Fuselage yaw</td>
</tr>
<tr class="odd">
<td></td>
<td>x_velocity</td>
<td><a>DefaultAndMaximum</a></td>
<td>X-axis speed</td>
</tr>
<tr class="even">
<td></td>
<td>y_velocity</td>
<td><a>DefaultAndMaximum</a></td>
<td>Y-axis speed</td>
</tr>
<tr class="odd">
<td></td>
<td>z_velocity</td>
<td><a>DefaultAndMaximum</a></td>
<td>Z-axis angular velocity</td>
</tr>
<tr class="even">
<td></td>
<td>front_leg_lift</td>
<td><a>DefaultAndMaximum</a></td>
<td>Leg lift height of front legs</td>
</tr>
<tr class="odd">
<td></td>
<td>back_leg_lift</td>
<td><a>DefaultAndMaximum</a></td>
<td>Rear leg lift height</td>
</tr>
<tr class="even">
<td></td>
<td>distance</td>
<td><a>DefaultAndMaximum</a></td>
<td>Expected distance</td>
</tr>
<tr class="odd">
<td></td>
<td>duration</td>
<td><a>DefaultAndMaximum</a></td>
<td>Expected time</td>
</tr>
<tr class="even">
<td></td>
<td>delta</td>
<td><a>DefaultAndMaximum</a></td>
<td>Variation</td>
</tr>
</tbody>
</table>

**MotionResultServiceResponse: motion result service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionResultServiceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>motion_id</td>
<td>int32</td>
<td>Robot movement control attitude status</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>result</td>
<td>bool</td>
<td>Execution results</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>code</td>
<td>int32</td>
<td>Standard error codes</td>
</tr>
</tbody>
</table>

**MotionServoCmdResponse: motion servo command feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionServoCmdResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>motion_id</td>
<td>int32</td>
<td>Robot movement control attitude status</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>result</td>
<td>bool</td>
<td>Execution results</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>code</td>
<td>int32</td>
<td>Standard error codes</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>order_process_bar</td>
<td>int8</td>
<td>Order Process</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>status</td>
<td>int32</td>
<td>Status</td>
</tr>
</tbody>
</table>

**MotionSequenceServiceResponse: Sequence motion service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionSequenceServiceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>result</td>
<td>bool</td>
<td>Execution results</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>code</td>
<td>int32</td>
<td>Status code</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>describe</td>
<td>string</td>
<td>Status code description</td>
</tr>
</tbody>
</table>

**LedConstraint: LED constraint**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td></td>
<td></td>
<td>Enumeration</td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td></td>
<td></td>
<td>LedConstraint</td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>value</p>
<p>About</p>
<p>Bundle</p></td>
<td>Scenario</td>
<td></td>
<td>Key value</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>Control target</td>
<td></td>
<td>target_head</td>
<td>Headlight</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>target_tail</td>
<td>Tail lights</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>target_mini</td>
<td>Eye lamp</td>
</tr>
<tr class="odd">
<td></td>
<td><p>Header</p>
<p>Tail</p>
<p>Light</p>
<p>With</p></td>
<td><p>Lighting effects</p>
<p>(Support RGB color correction)</p></td>
<td>effect_line_on</td>
<td>Always on</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>effect_line_blink</td>
<td>Flash</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>effect_line_blink_fast</td>
<td>Fast flashing</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>effect_line_breath</td>
<td>Breathe</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>effect_line_breath_fast</td>
<td>Breathe quickly</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>effect_line_one_by_one</td>
<td>Light up one by one</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>effect_line_one_by_one_fast</td>
<td>Light up quickly one by one</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td><p>System lighting effects</p>
<p>(RGB color correction is not supported)</p></td>
<td>system_effect_line_off</td>
<td>Always off</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_on</td>
<td>The red light is always on</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_blink</td>
<td>Red light flashes</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_blink_fast</td>
<td>The red light flashes quickly</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_breath</td>
<td>Red light breathing</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_breath_fast</td>
<td>Breathe quickly at red light</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_one_by_one</td>
<td>The red lights light up one by one</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_red_one_by_one_fast</td>
<td>The red lights turn on quickly one by one</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_on</td>
<td>The blue light is always on</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_blink</td>
<td>Blue light flashes</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_blink_fast</td>
<td>The blue light flashes quickly</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_breath</td>
<td>Blue light breathing</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_breath_fast</td>
<td>Breathe quickly with blue light</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_one_by_one</td>
<td>The blue lights light up one by one</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_blue_one_by_one_fast</td>
<td>The blue lights turn on quickly one by one</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_on</td>
<td>The yellow light is always on</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_blink</td>
<td>Yellow light flashes</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_blink_fast</td>
<td>The yellow light flashes quickly</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_breath</td>
<td>Yellow light breathing</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_breath_fast</td>
<td>Breathe quickly with yellow light</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_one_by_one</td>
<td>The yellow lights light up one by one</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_line_yellow_one_by_one_fast</td>
<td>The yellow lights turn on quickly one by one</td>
</tr>
<tr class="even">
<td></td>
<td><p>Eye</p>
<p>Light</p></td>
<td><p>Lighting effects</p>
<p>(Support RGB color correction)</p></td>
<td>effect_mini_circular_breath</td>
<td>Circular Zoom</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>effect_mini_circular_ring</td>
<td>Draw a circle</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td><p>System lighting effects</p>
<p>(RGB color correction is not supported)</p></td>
<td>system_effect_mini_off</td>
<td>Always off</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_mini_rectangle_color</td>
<td>The blocks change color</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_mini_centre_color</td>
<td>Middle ribbon</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>system_effect_mini_three_circular</td>
<td>Three-circle breathing</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>system_effect_mini_one_by_one</td>
<td>The ribbons light up one by one</td>
</tr>
</tbody>
</table>

**LedSeviceResponse: LED service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>LedSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>code</td>
<td>int32</td>
<td><p>Standard error codes</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
<br />
int32 SUCCEED =0 # The current request parameters are reasonable, the priority is the highest, and the requested lighting effect is executed successfully<br />
int32 TIMEOUT =1107 # Current request LED hardware response timeout<br />
int32 TARGET_ERROR =1121 # The target parameter of the current request is empty or not in the optional list<br />
int32 PRIORITY_ERROR =1122 # The currently requested client is empty or not in the preset priority list<br />
int32 MODE_ERROR = 1123 # The currently requested mode parameter is empty or not in the optional list<br />
int32 EFFECT_ERROR =1124 # The effect parameter of the current request is empty or not in the optional list<br />
int32 LOW_PRIORITY = 1125 # The current request priority is low and the requested lighting effect cannot be executed immediately</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**ConnectorStatus: Network connection message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td><a href="https://git.n.xiaomi.com/MiRoboticsLab/rop/bridges/-/blob/dev/protocol/ros/msg/ConnectorStatus.msg">ConnectorStatus</a></ td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>is_connected</td>
<td>bool</td>
<td>Whether to connect to wifi</td>
</tr>
<tr class="odd">
<td></td>
<td>is_internet</td>
<td>bool</td>
<td>Can you access the external network</td>
</tr>
<tr class="even">
<td></td>
<td>ssid</td>
<td>string</td>
<td>wifi name</td>
</tr>
<tr class="odd">
<td></td>
<td>robot_ip</td>
<td>string</td>
<td>Robot IP</td>
</tr>
<tr class="even">
<td></td>
<td>provider_ip</td>
<td>string</td>
<td>wifi provider/mobile IP</td>
</tr>
<tr class="odd">
<td></td>
<td>strength</td>
<td>int</td>
<td>wifi signal strength</td>
</tr>
<tr class="even">
<td></td>
<td>code</td>
<td>int</td>
<td>Standard error codes</td>
</tr>
</tbody>
</table>

**Vector\<xxx\>: list**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>List</td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>vector&lt;xxx&gt;</td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="odd">
<td></td>
<td>empty();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>capacity();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>clear();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>push_back();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>pop_back();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>at();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>front();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>back();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

**MotionSequenceGait: sequence gait message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionSequenceGait</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>right_forefoot</td>
<td>bool</td>
<td>Right forefoot: Is it touching the ground? (Default: True)</td>
</tr>
<tr class="odd">
<td></td>
<td>left_forefoot</td>
<td>bool</td>
<td>Left forefoot: Is it touching the ground? (Default: True)</td>
</tr>
<tr class="even">
<td></td>
<td>right_hindfoot</td>
<td>bool</td>
<td>Right rear foot: Is it touching the ground? (Default: True)</td>
</tr>
<tr class="odd">
<td></td>
<td>left_hindfoot</td>
<td>bool</td>
<td>Left rear foot: Is it touching the ground? (Default: True)</td>
</tr>
<tr class="even">
<td></td>
<td>duration</td>
<td>int</td>
<td>Current gait duration (milliseconds) (default: 1000)</td>
</tr>
</tbody>
</table>

**MotionSequencePace: sequence pace message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionSequencePace</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>Words</p>
<p>paragraph</p>
<p>About</p>
<p>Bundle</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>twist</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Twist.html">geometry_msgs/msg/Twist.msg</a></td>
<td><p>Speed (default: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
linear.x // X-axis linear speed (m/s)<br />
linear.y // Y-axis linear speed (m/s)<br />
linear.z // z-axis angular velocity (°/s)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td></td>
<td>centroid</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Pose.html">geometry_msgs/msg/Pose.msg</a></td>
<td><p>Center of mass (default: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
position.x // X-axis center of mass offset (m)<br />
position.y // Y-axis center of mass offset (m)<br />
position.z //Z-axis center of mass height (m)<br />
orientation.x // X-axis attitude: R(°)<br />
orientation.y // Y-axis attitude: P (°)<br />
orientation.z // Z-axis attitude: Y (°)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="even">
<td></td>
<td>weight</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Twist.html">geometry_msgs/msg/Twist.msg</a></td>
<td><p>6 degrees of freedom weight</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
linear.x // X-axis weight<br />
linear.y // Y-axis weight<br />
linear.z // Z-axis weight<br />
angular.x // X-R axis weight<br />
angular.y // Y-P axis weight<br />
angular.z // Z-Y axis weight</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td></td>
<td>right_forefoot</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Quaternion.html">geometry_msgs/msg/Quaternion.msg</a></td>
<td><p>Right forefoot information (default: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
x // Foothold position: X-axis offset (m)<br />
y // Foothold position: Y-axis offset (m)<br />
z // Foothold position: Z-axis offset (m)<br />
w // Foot lifting height (m)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="even">
<td></td>
<td>left_forefoot</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Point.html">geometry_msgs/msg/Quaternion.msg</a></td>
<td><p>Left forefoot information (default: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
x // Foothold position: X-axis offset (m)<br />
y // Foothold position: Y-axis offset (m)<br />
z // Foothold position: Z-axis offset (m)<br />
w // Foot lifting height (m)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td></td>
<td>right_hindfoot</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Point.html">geometry_msgs/msg/Quaternion.msg</a></td>
<td><p>Right rear foot information (default value: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
x // Foothold position: X-axis offset (m)<br />
y // Foothold position: Y-axis offset (m)<br />
z // Foothold position: Z-axis offset (m)<br />
w // Foot lifting height (m)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="even">
<td></td>
<td>left_hindfoot</td>
<td><a href="https://docs.ros2.org/latest/api/geometry_msgs/msg/Point.html">geometry_msgs/msg/Quaternion.msg</a></td>
<td><p>Left rear foot information (default: all 0.0)</p>
<table>
<tbody>
<tr class="odd">
<td>C++<br />
x // Foothold position: X-axis offset (m)<br />
y // Foothold position: Y-axis offset (m)<br />
z // Foothold position: Z-axis offset (m)<br />
w // Foot lifting height (m)</td>
</tr>
</tbody>
</table></td>
</tr>
<tr class="odd">
<td></td>
<td>use_mpc_track</td>
<td>bool</td>
<td>Whether to use MPC trajectory</td>
</tr>
<tr class="even">
<td></td>
<td>landing_gain</td>
<td>float</td>
<td>Floor coefficient [0,1], default value: 1.0</td>
</tr>
<tr class="odd">
<td></td>
<td>friction_coefficient</td>
<td>float</td>
<td>Friction coefficient [0.1 1.0] (default: 0.8)</td>
</tr>
<tr class="even">
<td></td>
<td>duration</td>
<td>int</td>
<td>Current parameter duration (milliseconds) (default: 1000)</td>
</tr>
</tbody>
</table>

**MotionSequence: Motion sequence message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MotionSequence</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>name</td>
<td>string</td>
<td>[Required] Motion sequence name (conforms to variable naming rules)</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>describe</td>
<td>string</td>
<td>Motion sequence description</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>gait_list</td>
<td><a>MotionSequenceGait</a>&gt;</td>
<td>Gait list</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>pace_list</td>
<td><a>MotionSequencePace</a>&gt;</td>
<td>Step list</td>
<td></td>
</tr>
</tbody>
</table>

**AudioPlaySeviceResponse: Voice playback service feedback message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>AudioPlaySeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>status</td>
<td>int32</td>
<td>0 played completed, 1 failed to play</td>
</tr>
</tbody>
</table>

**AudioGetVolumeSeviceResponse: Voice get volume service feedback message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>AudioGetVolumeSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>volume</td>
<td>int32</td>
<td>The volume of the currently playing voice</td>
</tr>
</tbody>
</table>

**AudioSetVolumeSeviceResponse: Voice set volume service feedback message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>AudioSetVolumeSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>success</td>
<td>bool</td>
<td>Whether the volume setting was successful</td>
</tr>
</tbody>
</table>

**FaceRecognitionResult: Face recognition result information**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>FaceRecognitionResult</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>result</td>
<td>int</td>
<td>Personnel results</td>
</tr>
<tr class="odd">
<td></td>
<td>username</td>
<td>string</td>
<td>Personnel name</td>
</tr>
<tr class="even">
<td></td>
<td>age</td>
<td>float</td>
<td>Age of personnel</td>
</tr>
<tr class="odd">
<td></td>
<td>emotion</td>
<td>float</td>
<td>Personnel sentiment</td>
</tr>
</tbody>
</table>

**FaceRecognizedSeviceResponse: Face recognition feedback message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>FaceRecognizedSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td><a>State</a></td>
<td>Status</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>list</td>
<td>list&lt;<a>FaceRecognitionResult</a>&gt;</td>
<td>Face recognition list</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>list</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>capacity();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>append();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>pop();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>at();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>front();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>back();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>dictionary</td>
<td>dictionary&lt;string, <a>FaceRecognitionResult</a>&gt;</td>
<td>Face recognition dictionary</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>Dictionary</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
</tbody>
</table>

**VoiceprintRecognized: Voiceprint recognition message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>VoiceprintRecognized</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>list</td>
<td>vector &lt;string&gt;</td>
<td>List of recognized persons</td>
</tr>
<tr class="even">
<td></td>
<td>data</td>
<td>bool</td>
<td>Identification successful</td>
</tr>
</tbody>
</table>

**MsgPreset: Preset point information**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MsgPreset</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>label_name</td>
<td>string</td>
<td>Preset point name</td>
</tr>
<tr class="odd">
<td></td>
<td>physic_x</td>
<td>float</td>
<td>X-axis coordinate</td>
</tr>
<tr class="even">
<td></td>
<td>physic_y</td>
<td>float</td>
<td>Y-axis coordinate</td>
</tr>
</tbody>
</table>

**MapPresetSeviceResponse: Map preset point service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MapPresetSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td><a>State</a></td>
<td>Status</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>map_name</td>
<td>string</td>
<td>Map name</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>is_outdoor</td>
<td>bool</td>
<td>Whether it is an outdoor map</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>list</td>
<td>list&lt;<a>MsgPreset</a>&gt;</td>
<td>Face recognition list</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>list</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>capacity();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>append();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>pop();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>at();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>front();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>back();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>dictionary</td>
<td>dictionary&lt;string, <a>MsgPreset</a>&gt;</td>
<td>Face recognition dictionary</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>Dictionary</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
</tbody>
</table>

**NavigationActionResponse: navigation action feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>NavigationActionResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>result</td>
<td>int</td>
<td><p>Service action feedback:</p>
<p>0: Success;</p>
<p>1: Accept;</p>
<p>2: Not available;</p>
<p>3: Failure;</p>
<p>4: Reject;</p>
<p>5: Cancel. </p></td>
</tr>
</tbody>
</table>

**GestureType: Gesture recognition type**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Enumeration</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>GestureType</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>value</p>
<p>About</p>
<p>Bundle</p></td>
<td>key</td>
<td>value</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>no_gesture</td>
<td>0</td>
<td>No gestures</td>
</tr>
<tr class="odd">
<td></td>
<td>pulling_hand_or_two_fingers_in</td>
<td>1</td>
<td>Pull your palms closer</td>
</tr>
<tr class="even">
<td></td>
<td>pushing_hand_or_two_fingers_away</td>
<td>2</td>
<td>Push away with your palms</td>
</tr>
<tr class="odd">
<td></td>
<td>sliding_hand_or_two_fingers_up</td>
<td>3</td>
<td>Raise your hand upward</td>
</tr>
<tr class="even">
<td></td>
<td>sliding_hand_or_two_fingers_down</td>
<td>4</td>
<td>Press down with your hand</td>
</tr>
<tr class="odd">
<td></td>
<td>sliding_hand_or_two_fingers_left</td>
<td>5</td>
<td>Push your hand to the left</td>
</tr>
<tr class="even">
<td></td>
<td>sliding_hand_or_two_fingers_right</td>
<td>6</td>
<td>Push your hand to the right</td>
</tr>
<tr class="odd">
<td></td>
<td>stop_sign</td>
<td>7</td>
<td>Stop gesture</td>
</tr>
<tr class="even">
<td></td>
<td>thumb_down</td>
<td>8</td>
<td>Thumbs down</td>
</tr>
<tr class="odd">
<td></td>
<td>thumb_up</td>
<td>9</td>
<td>Thumbs up</td>
</tr>
<tr class="even">
<td></td>
<td>zooming_in_with_hand_or_two_fingers</td>
<td>10</td>
<td>Open your palms or fingers</td>
</tr>
<tr class="odd">
<td></td>
<td>zooming_out_with_hand_or_two_fingers</td>
<td>11</td>
<td>Close palms or fingers</td>
</tr>
</tbody>
</table>

**GestureData: gesture data**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>GestureType</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>pulling_hand_or_two_fingers_in</td>
<td>bool</td>
<td>Pull your palms closer</td>
</tr>
<tr class="odd">
<td></td>
<td>pushing_hand_or_two_fingers_away</td>
<td>bool</td>
<td>Push away with your palms</td>
</tr>
<tr class="even">
<td></td>
<td>sliding_hand_or_two_fingers_up</td>
<td>bool</td>
<td>Raise your hand upward</td>
</tr>
<tr class="odd">
<td></td>
<td>sliding_hand_or_two_fingers_down</td>
<td>bool</td>
<td>Press down with your hand</td>
</tr>
<tr class="even">
<td></td>
<td>sliding_hand_or_two_fingers_left</td>
<td>bool</td>
<td>Push your hand to the left</td>
</tr>
<tr class="odd">
<td></td>
<td>sliding_hand_or_two_fingers_right</td>
<td>bool</td>
<td>Push your hand to the right</td>
</tr>
<tr class="even">
<td></td>
<td>stop_sign</td>
<td>bool</td>
<td>Stop gesture</td>
</tr>
<tr class="odd">
<td></td>
<td>thumb_down</td>
<td>bool</td>
<td>Thumbs down</td>
</tr>
<tr class="even">
<td></td>
<td>thumb_up</td>
<td>bool</td>
<td>Thumbs up</td>
</tr>
<tr class="odd">
<td></td>
<td>zooming_in_with_hand_or_two_fingers</td>
<td>bool</td>
<td>Open your palms or fingers</td>
</tr>
<tr class="even">
<td></td>
<td>zooming_out_with_hand_or_two_fingers</td>
<td>bool</td>
<td>Close palms or fingers</td>
</tr>
</tbody>
</table>

**GestureRecognizedSeviceResponse: Gesture recognition service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>GestureRecognizedSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>code</td>
<td>int</td>
<td>Service request status, 0: success; 1: failure. </td>
</tr>
</tbody>
</table>

**GestureRecognizedMessageResponse: gesture recognition message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>GestureRecognizedMessageResponse</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>data</td>
<td><a>GestureData</a></td>
<td>Gesture recognition status data</td>
</tr>
</tbody>
</table>

**SkeletonType: Skeleton point recognition type**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Enumeration</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>SkeletonType</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>value</p>
<p>About</p>
<p>Bundle</p></td>
<td>key</td>
<td>value</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>squat</td>
<td>1</td>
<td>Squat</td>
</tr>
<tr class="odd">
<td></td>
<td>highknees</td>
<td>2</td>
<td>Raise your legs high</td>
</tr>
<tr class="even">
<td></td>
<td>situp</td>
<td>3</td>
<td>Sit-ups</td>
</tr>
<tr class="odd">
<td></td>
<td>pressup</td>
<td>4</td>
<td>Push-ups</td>
</tr>
<tr class="even">
<td></td>
<td>plank</td>
<td>5</td>
<td>Plank</td>
</tr>
<tr class="odd">
<td></td>
<td>jumpjack</td>
<td>6</td>
<td>Jumping jacks</td>
</tr>
</tbody>
</table>

**SkeletonRecognizedSeviceResponse: Skeleton point recognition service feedback message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>SkeletonRecognizedSeviceResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>result</td>
<td>int</td>
<td>Service feedback status, 0: success, 1: failure</td>
</tr>
</tbody>
</table>

**SkeletonRecognizedMessageResponse: Skeleton point recognition message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>SkeletonRecognizedMessageResponse</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Combined</p>
<p>法</p>
<p>Words</p>
<p>paragraph</p>
<p>About</p>
<p>Bundle</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>algo_switch</td>
<td>int</td>
<td>Algorithm switch, 0: on, 1: off</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>sport_type</td>
<td>int</td>
<td>Algorithm recognition type, see SkeletonType</td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>counts</td>
<td>int</td>
<td>Motion counting, starting from 1</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>duration</td>
<td>int</td>
<td>Exercise duration (plank timing length)</td>
</tr>
</tbody>
</table>

**MsgTrainingWords: training word information**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>MsgTrainingWords</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>trigger</td>
<td>string</td>
<td>Keywords</td>
</tr>
<tr class="odd">
<td></td>
<td>type</td>
<td>string</td>
<td>Type</td>
</tr>
<tr class="even">
<td></td>
<td>value</td>
<td>string</td>
<td>value</td>
</tr>
</tbody>
</table>

**TrainingWordsRecognizedSeviceResponse: Training word service feedback**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>TrainingWordsRecognizedSeviceResponse</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td></td>
<td>Type</td>
<td>Meaning</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td></td>
<td><a>State</a></td>
<td>Status</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td>training_set</td>
<td>list&lt;<a>MsgTrainingWords</a>&gt;</td>
<td>Face recognition list</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>list</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>capacity();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>append();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>pop();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>at();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>front();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>back();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td>dictionary</td>
<td></td>
<td>dictionary&lt;string, <a>MsgTrainingWords</a> &gt;</td>
<td>Face recognition dictionary</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>Available operations</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>Interface</td>
<td>Description</td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>empty();</td>
<td>See <a>Dictionary</a></td> for details
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>size();</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td></td>
<td></td>
<td>max_size();</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td></td>
<td></td>
<td>clear();</td>
<td></td>
</tr>
</tbody>
</table>

**TrainingWordsRecognizedMessageResponse: training word recognition message**

<table>
<tbody>
<tr class="odd">
<td>Category</td>
<td>Structure</td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Type</td>
<td>TrainingWordsRecognizedMessageResponse</td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><p>Legal</p>
<p>Field</p>
<p>Constraints</p></td>
<td>Field</td>
<td>Type</td>
<td>Meaning</td>
</tr>
<tr class="even">
<td></td>
<td>state</td>
<td><a>State</a></td>
<td>Status</td>
</tr>
<tr class="odd">
<td></td>
<td>response</td>
<td><a>MsgTrainingWords</a></td>
<td>Training words</td>
</tr>
</tbody>
</table>

**4.3 Tiedan capability set interface constraint table**

| | | |
| -------------------------------------------------- -------------------------- | -- | -------------------------- ------------- |
| Legend | Meaning | Description |
| [🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) | Field | Identifies the interface as a variable and can directly take the value. |
| [🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) | Function | Identifies the interface as a function. The main function ends after the function exits, and the execution status is judged based on the return value. |
| 🟡 | Handle | Identifies the interface as an object and cannot be called directly. Only the fields and functions under it can be called. |

**4.4 Tiedan capability set interface constraints**

**4.4.01 🟡cyberdog (iron egg)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.state</strong>: cyberdog module status acquisition interface name. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) set\_log (setting log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.set_log</strong>: Set cyberdog module log. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.set_log(False).code == StateCode.success:</strong><br />
<strong>print('cyberdog module log has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) shutdown**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.shutdown()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.shutdown</strong>: Exit cyberdog. </p>
<p>Parameters: none</p>
<p>Return value: None</p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.shutdown()</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.02 🟡network (network module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.network.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.network.state</strong>: Get the interface name of the network module status under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.network.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.network.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.network.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.network.data</strong>: Get the interface name of the network module status under cyberdog. </p>
<p>Type: <a>ConnectorStatus</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.network.data.is_connected:</strong><br />
<strong>print('The current robot is connected to', cyberdog.network.data.ssid, 'WiFi network')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) set\_log (setting log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.network.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.network.set_log</strong>: Set the network module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.network.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The network module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.03 🟡follow (follow module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.state</strong>: Get the interface name by following the module status in cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.follow.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) set\_log (setting log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.set_log</strong>: Set the follow module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The follow module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) add\_personnel**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.add_personnel(</strong><br />
<strong>string preset_name)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.add_personnel</strong>: Add follower information. </p>
<p>Parameters:</p>
<p><strong>preset_name</strong>: Personnel name, string type;</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.add_personnel('Zhang San').code == StateCode.success:</strong><br />
<strong>print('Add Zhang San to the follower module successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) delete\_personnel (delete person)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.delete_personnel(</strong><br />
<strong>string preset_name)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.add_personnel</strong>: Delete follower information. </p>
<p>Parameters:</p>
<p><strong>preset_name</strong>: Personnel name, string type;</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.delete_personnel('Zhang San').code == StateCode.success:</strong><br />
<strong>print('Delete Zhang San from the following module successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) follow\_personnel**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.follow_personnel(</strong><br />
<strong>string preset_name,</strong><br />
<strong>double intimacy)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.follow_personnel</strong>: Add follower information. </p>
<p>Parameters:</p>
<p><strong>preset_name</strong>: Personnel name, string type;</p>
<p><strong>intimacy</strong>: following spacing, floating point type;</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.follow_personnel('张三', 1.0).code == StateCode.success:</strong><br />
<strong>print('Start following Zhang San successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) cancel\_follow (cancel follow)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.follow.cancel_follow(</strong><br />
<strong>string preset_name)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.follow.cancel_follow</strong>: Delete follower information. </p>
<p>Parameters:</p>
<p><strong>preset_name</strong>: Personnel name, string type;</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.follow.cancel_follow('Zhang San').code == StateCode.success:</strong><br />
<strong>print('Cancel following Zhang San successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.04 🟡motion (motion module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.state</strong>: The name of the interface for obtaining the status of the motion module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.motion.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟣 params**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.params</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.params</strong>: The default parameter acquisition interface name of the motion module under cyberdog. </p>
<p>Type: <a>MotionParams</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.motion.params)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) set\_log (setting log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.set_log</strong>: Set the motion module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The motion module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) emergency\_stop**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.emergency_stop()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.emergency_stop</strong>: Causes the robot to stop urgently. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.emergency_stop().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot emergency stop successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_down（Lie down）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.get_down()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.get_down</strong>: Makes the robot lie down. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.get_down().state.code == StateCode.success:</strong><br />
<strong>print('Making the robot lie down successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) resume\_standing（standing）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.resume_standing()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.resume_standing</strong>: Makes the robot stand again. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.resume_standing().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot stand successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) back\_flip**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.back_flip()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.back_flip</strong>: Makes the robot do a backflip. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.back_flip().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot backflip successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) front\_flip（front flip）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.front_flip()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.front_flip</strong>: Makes the robot do a front flip. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.front_flip().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot do a front flip successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) bow**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.bow()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.bow</strong>: Makes the robot bow. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.bow().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot bow successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) roll\_left (recover after lying on the left side)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.roll_left()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.roll_left</strong>: Makes the robot lie on the left side and recover. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.roll_left().state.code == StateCode.success:</strong><br />
<strong>print('The robot recovered successfully after lying on the left side')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) walk\_the\_dog**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.walk_the_dog(</strong><br />
<strong>double front_leg_lift</strong>,<br />
<strong>double</strong> <strong>back_leg_lift)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.walk_the_dog</strong>: Makes the robot enter dog walking mode. </p>
<p>Parameters:</p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.walk_the_dog(0.03, 0.03).state.code == StateCode.success:</strong><br />
<strong>print('The robot entered dog walking mode successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump\_stair（jump up the steps）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump_stair()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump_stair</strong>: Makes the robot jump up the stairs. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump_stair().state.code == StateCode.success:</strong><br />
<strong>print('The robot jumped up the steps successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) right\_somersault (right somersault)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.right_somersault()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.right_somersault</strong>: Makes the robot somersault right. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.right_somersault().state.code == StateCode.success:</strong><br />
<strong>print('Made the robot somersault successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) left\_somersault (left somersault)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.left_somersault()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.left_somersault</strong>: Makes the robot somersault left. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.left_somersault().state.code == StateCode.success:</strong><br />
<strong>print('Made the robot somersault successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) run\_and\_jump\_front\_flip (running jump front flip)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.run_and_jump_front_flip()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.run_and_jump_front_flip</strong>: Makes the robot run, jump and do a front flip. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.run_and_jump_front_flip().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot run, jump and front flip successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_left90deg（3D jump: turn left 90 degrees）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump3d_left90deg()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_left90deg</strong>: Makes the robot 3D jump: turn left 90 degrees. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_left90deg().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot 3D jump: turn left 90 degrees successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_right90deg（3D jump: turn right 90 degrees）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_right90deg()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_right90deg</strong>: Makes the robot 3D jump: turn right 90 degrees. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_right90deg().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot 3D jump: turn right 90 degrees successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_forward60cm (3D jump: jump 60cm forward)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_forward60cm()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_forward60cm</strong>: Makes the robot jump 3D: forward 60cm. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_forward60cm().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump in 3D: jump 60cm forward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_forward30cm (3D jump: jump forward 30cm)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_forward30cm()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_forward30cm</strong>: Makes the robot jump 3D: forward 30cm. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_forward30cm().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump in 3D: jump 30cm forward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_left20cm（3D jump: left jump 20cm）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_left20cm()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_left20cm</strong>: Makes the robot 3D jump: jump 20cm to the left. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_left20cm().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump in 3D: jump 20cm to the left successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_right20cm（3D jump: right jump 20cm）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_right20cm()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_right20cm</strong>: Makes the robot 3D jump: jump 20cm to the right. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_right20cm().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump in 3D: jump 20cm to the right successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_up30cm（3D jump: up 30cm）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td>Apache<br />
<strong>cyberdog.motion.jump3d_up30cm()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_up30cm</strong>: Makes the robot jump 3D: up 30cm. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_up30cm().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot 3D jump: 3D jump upward: 30cm successful')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump3d\_down\_stair（3D jump: jump down the stairs）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump3d_down_stair()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump3d_down_stair</strong>: Makes the robot 3D jump: jump down the stairs. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump3d_down_stair().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump in 3D: jump down the steps successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) roll\_right (recover after lying on the right side)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.roll_right()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.roll_right</strong>: Makes the robot lie down on the right side and recover. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.roll_right().state.code == StateCode.success:</strong><br />
<strong>print('The robot recovered successfully after lying on the right side')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) dance\_collection (dance collection)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.dance_collection()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.dance_collection</strong>: Makes a collection of robot dances. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.dance_collection().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot perform the dance collection successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) hold\_right\_hand（hold left hand）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.hold_left_hand()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.hold_left_hand</strong>: Makes the robot hold its left hand. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.hold_left_hand().state.code == StateCode.success:</strong><br />
<strong>print('Making the robot hold its left hand successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) hold\_right\_hand（hold the right hand）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.hold_right_hand()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.hold_right_hand</strong>: Makes the robot hold its right hand. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.hold_right_hand().state.code == StateCode.success:</strong><br />
<strong>print('Making the robot hold its right hand successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) sit\_down (sit down)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.sit_down()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.sit_down</strong>: Makes the robot sit down. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.sit_down().state.code == StateCode.success:</strong><br />
<strong>print('Made the robot sit down successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) butt\_circle (butt circle)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.butt_circle()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.butt_circle</strong>: Makes the robot's butt circle. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.butt_circle().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's butt draw a circle successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) head\_circle (head circle)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.head_circle()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.head_circle</strong>: Makes the robot head draw a circle. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.head_circle().state.code == StateCode.success:</strong><br />
<strong>print('The robot head draws a circle successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) stretch\_the\_body (stretch the body)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.stretch_the_body()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.stretch_the_body</strong>: Makes the robot stretch its body. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.stretch_the_body().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot stretch its body successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) shake\_ass\_left（Shake your butt to the left）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.shake_ass_left()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.shake_ass_left</strong>: Makes the robot shake its butt to the left. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p>
<p>Note: Must be done after sitting down. </p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.shake_ass_left().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot shake its butt to the left successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) shake\_ass\_right（Shake your butt to the right）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.shake_ass_right()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.shake_ass_right</strong>: Makes the robot shake its butt to the right. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p>
<p>Note: Must be done after sitting down. </p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.shake_ass_right().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot shake its butt to the right successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) shake\_ass\_from\_side\_to\_side (shake your butt left and right)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.shake_ass_from_side_to_side()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.shake_ass_from_side_to_side</strong>: Makes the robot shake its butt from side to side. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p>
<p>Note: Must be done after sitting down. </p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.shake_ass_from_side_to_side().state.code == StateCode.success:</strong><br />
<strong>print('Successfully made the robot shake its butt left and right')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) ballet**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.ballet()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.ballet</strong>: Makes a robot ballet. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.ballet().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot ballet successful')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) space\_walk**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.space_walk()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.space_walk</strong>: Makes the robot spacewalk. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.space_walk().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot spacewalk successful')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) front\_leg\_jumping（front leg jumping jacks）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.front_leg_jumping()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.front_leg_jumping</strong>: Makes the robot's front legs jumping jacks. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.front_leg_jumping().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's front legs jump successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) hind\_leg\_jumping（hind leg jumping jacks）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.hind_leg_jumping()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.hind_leg_jumping</strong>: Makes the robot do jumping jacks on its hind legs. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.hind_leg_jumping().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot successfully jump on its hind legs')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) lift\_the\_left\_leg\_and\_nod (lift the left leg and nod)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.lift_the_left_leg_and_nod()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.lift_the_left_leg_and_nod</strong>: Makes the robot lift its left leg and nod. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.lift_the_left_leg_and_nod().state.code == StateCode.success:</strong><br />
<strong>print('The robot raised its left leg and nodded successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) lift\_the\_right\_leg\_and\_nod (lift the right leg and nod)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.lift_the_right_leg_and_nod()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.lift_the_right_leg_and_nod</strong>: Makes the robot lift its right leg and nod. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.lift_the_right_leg_and_nod().state.code == StateCode.success:</strong><br />
<strong>print('The robot raised its right leg and nodded successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) left\_front\_right\_back\_legs\_apart（Left left front right back spread legs）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.left_front_right_back_legs_apart()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.left_front_right_back_legs_apart</strong>: Makes the robot spread its legs to the left, front, right and back. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.left_front_right_back_legs_apart().state.code == StateCode.success:</strong><br />
<strong>print('The robot spread its legs to the left, front, right and back successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) right\_front\_left\_back\_legs\_apart (spread the legs on the right front, left and back)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.right_front_left_back_legs_apart()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.right_front_left_back_legs_apart</strong>: Makes the robot's legs spread apart from the right front to the left. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.right_front_left_back_legs_apart().state.code == StateCode.success:</strong><br />
<strong>print('The robot spread its legs to the right, front, left and back successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) walk\_nodding**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.walk_nodding()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.walk_nodding</strong>: Makes the robot walk and nod. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.walk_nodding().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot walk and nod successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) walking\_with\_divergence\_and\_adduction\_alternately (walking alternately with divergence and adduction)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.walking_with_divergence_and_adduction_alternately()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.walking_with_divergence_and_adduction_alternately</strong>: Makes the robot walk alternately with divergence and adduction. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.walking_with_divergence_and_adduction_alternately().state.code == StateCode.success:</strong><br />
<strong>print('The robot was able to walk alternately between adduction and divergence')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) nodding\_in\_place（nodding in place）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.nodding_in_place()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.nodding_in_place</strong>: Makes the robot stand still and nod. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.nodding_in_place().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot stand still and nod successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) front\_legs\_jump\_back\_and\_forth (front legs jump back and forth)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.front_legs_jump_back_and_forth()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.front_legs_jump_back_and_forth</strong>: Makes the robot's front legs jump forward and backward. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.front_legs_jump_back_and_forth().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's front legs jump back and forth successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) hind\_legs\_jump\_back\_and\_forth (hind legs jump back and forth)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.hind_legs_jump_back_and_forth()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.hind_legs_jump_back_and_forth</strong>: Makes the robot's hind legs jump forward and backward. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.hind_legs_jump_back_and_forth().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's hind legs jump back and forth successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) alternately\_front\_leg\_lift (front legs alternately lift)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.alternately_front_leg_lift()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.alternately_front_leg_lift</strong>: Makes the robot's front legs alternately lift. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.alternately_front_leg_lift().state.code == StateCode.success:</strong><br />
<strong>print('The robot's front legs were lifted alternately successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) alternately\_hind\_leg\_lift (hind legs alternately lift)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.alternately_hind_leg_lift()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.alternately_hind_leg_lift</strong>: Makes the robot's hind legs alternately lift. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.alternately_hind_leg_lift().state.code == StateCode.success:</strong><br />
<strong>print('The robot's hind legs were lifted alternately successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump\_collection**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump_collection()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump_collection</strong>: Makes the robot jump collection. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump_collection().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) stretching\_left\_and\_right (stretch your legs left and right)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.stretching_left_and_right()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.stretching_left_and_right</strong>: Makes the robot stretch its legs left and right. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.stretching_left_and_right().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot extend its legs left and right and step successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump\_forward\_and\_backward (swinging legs forward and backward)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump_forward_and_backward()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump_forward_and_backward</strong>: Makes the robot jump by swinging its legs forward and backward. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump_forward_and_backward().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump forward and backward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) step\_left\_and\_right (swing your legs left and right)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.step_left_and_right()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.step_left_and_right</strong>: Makes the robot step by swinging its legs left and right. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.step_left_and_right().state.code == StateCode.success:</strong><br />
<strong>print('The robot swings its legs left and right and steps successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) right\_leg\_back\_and\_forth\_stepping (right leg stepping forward and backward)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.right_leg_back_and_forth_stepping()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.right_leg_back_and_forth_stepping</strong>: Makes the robot's right leg step forward and backward. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.right_leg_back_and_forth_stepping().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's right leg step forward and backward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) left\_leg\_back\_and\_forth\_stepping（Left leg stepping forward and backward**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.left_leg_back_and_forth_stepping()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.left_leg_back_and_forth_stepping</strong>: Makes the robot's left leg step forward and backward. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.left_leg_back_and_forth_stepping().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot's left leg step forward and backward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) squat\_down\_on\_all\_fours**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.squat_down_on_all_fours()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.squat_down_on_all_fours</strong>: Makes the robot squat on all fours. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.squat_down_on_all_fours().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot squat on all fours successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) push\_ups（Push-ups）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.push_ups()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.push_ups</strong>: Makes the robot do push-ups. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.push_ups().state.code == StateCode.success:</strong><br />
<strong>print('Make the robot push-up successful')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) bow\_to\_each\_other（zuobixin）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.bow_to_each_other()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.bow_to_each_other</strong>: Makes the robot bow to each other. </p>
<p>Parameters: none</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.bow_to_each_other().state.code == StateCode.success:</strong><br />
<strong>print('Made the robot to do the bidding successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) absolute\_attitude (absolute attitude)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.absolute_attitude(</strong><br />
<strong>double centroid_z,</strong><br />
<strong>double roll,</strong><br />
<strong>double pitch,</strong><br />
<strong>double yaw,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.absolute_attitude</strong>: Makes the robot absolutely force-control the attitude. </p>
<p>Parameters:</p>
<p><strong>centroid_z:</strong>Height of center of mass, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>roll</strong>: The body rolls, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>pitch</strong>: fuselage pitch, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>yaw</strong>: fuselage yaw, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration:</strong>The expected time to change the posture is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value 0</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.absolute_attitude(0.2,5,5,5,1).state.code == StateCode.success:</strong><br />
<strong>print('Controlling the absolute posture of the robot successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) relatively\_force\_control\_attitude (relative force control attitude)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.relatively_force_control_attitude(</strong><br />
<strong>double centroid_x,</strong><br />
<strong>double centroid_y,</strong><br />
<strong>double centroid_z,</strong><br />
<strong>double roll,</strong><br />
<strong>double pitch,</strong><br />
<strong>double yaw,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.relatively_force_control_attitude</strong>: Makes the robot perform force-controlled posture actions relative to the current posture. </p>
<p>Parameters:</p>
<p><strong>centroid_x:</strong>The x-axis coordinate of the center of mass, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>centroid_y:</strong>The y-axis coordinate of the center of mass, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>centroid_z:</strong>The z-axis coordinate of the center of mass, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>roll</strong>: The body rolls, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>pitch</strong>: fuselage pitch, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>yaw</strong>: fuselage yaw, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration:</strong>The expected time to change the posture is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value 1</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.relatively_force_control_attitude(0.2,0.2,0.2,5,5,5,1).state.code == StateCode.success:</strong><br />
<strong>print('The relative force control posture of the robot was successfully controlled')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) transition\_standing (transition standing)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.transition_standing(</strong><br />
<strong>double centroid_z,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.transition_standing</strong>: Makes the robot transition to standing. </p>
<p>Parameters:</p>
<p><strong>centroid_z:</strong>Height of center of mass, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration:</strong>The expected time to change the posture is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value 1</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.transition_standing(0.2,1).state.code == StateCode.success:</strong><br />
<strong>print('Control the robot to transition to stand successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) relatively\_position\_control\_attitude (relative position control attitude)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.relatively_position_control_attitude(</strong><br />
<strong>double centroid_x,</strong><br />
<strong>double centroid_y,</strong><br />
<strong>double centroid_z,</strong><br />
<strong>double roll,</strong><br />
<strong>double pitch,</strong><br />
<strong>double yaw,</strong><br />
<strong>double fulcrum_x,</strong><br />
<strong>double fulcrum_y,</strong><br />
<strong>double fulcrum_z,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.relatively_position_control_attitude</strong>: Makes the robot relative force control attitude. </p>
<p>Parameters:</p>
<p><strong>centroid_x</strong>: center of mass x-axis coordinate, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>centroid_y</strong>: center of mass y-axis coordinate, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>centroid_z</strong>: center of mass z-axis coordinate, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>roll</strong>: The body rolls, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>pitch</strong>: fuselage pitch, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>yaw</strong>: fuselage yaw, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>fulcrum_x</strong>: The x-axis coordinate of the fulcrum, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>fulcrum_y</strong>: y-axis coordinate of the fulcrum, type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>fulcrum_z</strong>: The z-axis coordinate of the fulcrum, the type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration:</strong>The expected time to change the posture is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value 1</p>
<p>Return value: <a>MotionResultServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.relatively_position_control_attitude(0.2,0.2,0.2,5,5,5,0,0,0,1).state.code == StateCode.success:</strong><br />
<strong>print('Controlling the relative position control attitude of the robot successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) jump\_back\_and\_forth (jump forward and backward)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.jump_back_and_forth(</strong><br />
<strong>double x_jump_velocity,</strong><br />
<strong>double y_jump_velocity,</strong><br />
<strong>double z_jump_velocity,</strong><br />
<strong>double front_leg_lift,</strong><br />
<strong>double back_leg_lift,</strong><br />
<strong>double distance,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.jump_back_and_forth</strong>: Makes the robot jump forward and backward. </p>
<p>Parameters:</p>
<p><strong>x_jump_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>y_jump_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>z_jump_velocity</strong>: Angular velocity in degrees per second (<a href="https://zh.wikipedia.org/wiki/°">°</a>/s), type Floating point type. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Remarks:</p>
<p>Distance and duration are mutually exclusive. When they appear at the same time, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump_back_and_forth(0.03, 0.03).state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump forward and backward successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) small\_jump\_walking**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.small_jump_walking(</strong><br />
<strong>double x_jump_velocity</strong>,<br />
<strong>double y_jump_velocity</strong>,<br />
<strong>double z_jump_velocity</strong>,<br />
<strong>double front_leg_lift</strong>,<br />
<strong>double back_leg_lift</strong>,<br />
<strong>double distance</strong>,<br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.small_jump_walking</strong>: Makes the robot walk with a small jump. </p>
<p>Parameters</p>
<p><strong>x_jump_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>y_jump_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>z_jump_velocity</strong>: Angular velocity in degrees per second (<a href="https://zh.wikipedia.org/wiki/°">°</a>/s), type Floating point type. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.jump_back_and_forth(0.2，0, 0.0, 0.02, 0.02，0，1).state.code == StateCode.success:</strong><br />
<strong>print('Make the robot jump and walk successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) trot\_walking (slow walking)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.trot_walking(</strong><br />
<strong>double x_velocity</strong>,<br />
<strong>double y_velocity</strong>,<br />
<strong>double z_velocity</strong>,<br />
<strong>double front_leg_lift</strong>,<br />
<strong>double back_leg_lift</strong>,<br />
<strong>double distance</strong>,<br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.trot_walking</strong>: Makes the robot walk slowly. </p>
<p>Parameters</p>
<p><strong>x_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>y_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>z_velocity</strong>: Angular velocity in degrees per second (<a href="https://zh.wikipedia.org/wiki/°">°</a>/s), type Floating point type. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.trot_walking(0.2, 0.3, 0.0, 0.02, 0.02).state.code == StateCode.success:</strong><br />
<strong>print('Make the robot walk slowly')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) automatic\_frequency\_conversion\_walking (automatic frequency walking)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.automatic_frequency_conversion_walking(</strong><br />
<strong>double x_velocity</strong>,<br />
<strong>double y_velocity</strong>,<br />
<strong>double z_velocity</strong>,<br />
<strong>double front_leg_lift</strong>,<br />
<strong>double back_leg_lift</strong>,<br />
<strong>double distance</strong>,<br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.automatic_frequency_conversion_walking</strong>: Makes the robot automatically walk with frequency conversion. </p>
<p>Parameters</p>
<p><strong>x_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>y_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>z_velocity</strong>: Angular velocity in degrees per second (<a href="https://zh.wikipedia.org/wiki/°">°</a>/s), type Floating point type. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. 0.03</p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. 0.03</p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.automatic_frequency_conversion_walking(0.2, 0.3, 0.0, 0.02, 0.02).state.code == StateCode.success:</strong><br />
<strong>print('The robot automatically walks with variable frequency successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) run\_fast\_walking**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.run_fast_walking(</strong><br />
<strong>double x_velocity</strong>,<br />
<strong>double y_velocity</strong>,<br />
<strong>double z_velocity</strong>,<br />
<strong>double front_leg_lift</strong>,<br />
<strong>double back_leg_lift</strong>,<br />
<strong>double distance</strong>,<br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.run_fast_walking</strong>: Makes the robot run fast. </p>
<p>Parameters</p>
<p><strong>x_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>y_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>z_velocity</strong>: Angular velocity in degrees per second (<a href="https://zh.wikipedia.org/wiki/°">°</a>/s), type Floating point type. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>front_leg_lift</strong>: Front leg lift height, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>back_leg_lift</strong>: Front leg lift height, unit second (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.run_fast_walking(0.2, 0.3, 0.0, 0.02, 0.02).state.code == StateCode.success:</strong><br />
<strong>print('Make the robot run successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.turn(</strong><br />
<strong>double</strong> <strong>angle</strong>,<br />
<strong>double</strong> <strong>duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.turn</strong>: Control the robot's steering. </p>
<p>Parameters</p>
<p><strong>angle</strong>: angle, unit degree (<a href="https://zh.wikipedia.org/wiki/°">°</a>), type is floating point; </p>
<p>With the current attitude as 0 degrees, clockwise is negative</p>
<p>Valid range: [-360, 360). </p>
<p>Default: 0</p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: 1</p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.turn(10).state.code == StateCode.success:</strong><br />
<strong>print('Successfully turned the robot 10 degrees to the left')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) go\_straight (go straight)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.go_straight(</strong><br />
<strong>double x_velocity</strong>,<br />
<strong>double</strong> <strong>distance</strong>,<br />
<strong>double</strong> <strong>duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog. motion.go_straight</strong>: Control the robot to go straight. </p>
<p>Parameters</p>
<p><strong>x_velocity</strong>: Longitudinal linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: 1</p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.go_straight(0.3, 0, 10).state.code == StateCode.success:</strong><br />
<strong>print('Successfully made the robot move forward at a speed of 0.3m/s for 10 seconds')</strong><br />
<strong>if cyberdog.motion.go_straight(-0.3, 0, 10).state.code == StateCode.success:</strong><br />
<strong>print('Successfully made the robot move backward for 10 seconds at a speed of 0.3m/s')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) lateral\_movement**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.lateral_movement(</strong><br />
<strong>double y_velocity</strong>,<br />
<strong>double</strong> <strong>distance</strong>,<br />
<strong>double</strong> <strong>duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.lateral_movement</strong>: The name of the lateral movement interface, which controls the lateral movement of the robot. </p>
<p>Parameters</p>
<p><strong>y_velocity</strong>: Transverse linear velocity, unit is meters per second (m/s), type is floating point. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>distance</strong>: desired distance, distance, unit meter (m), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: 1</p>
<p>Remarks:</p>
<p><strong>distance</strong> and <strong>duration</strong> are mutually exclusive. When they appear together, they are constrained by the first non-zero value. </p>
<p>Return value: <a>MotionServoCmdResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.motion.lateral_movement(0.3, 0, 10).state.code == StateCode.success:</strong><br />
<strong>print('Successfully made the robot move left at a speed of 0.3m/s for 10 seconds')</strong><br />
<strong>if cyberdog.motion.lateral_movement(-0.3, 0, 10).state.code == StateCode.success:</strong><br />
<strong>print('Successfully made the robot move to the right at a speed of 0.3m/s for 10 seconds')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) run\_sequence (run sequence)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.motion.run_sequence(</strong><br />
<strong>MotionSequence sequence</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.motion.run_sequence</strong>: Run sequence interface name to make the robot run the current sequence. </p>
<p>Parameters</p>
<p><strong>sequence</strong>: Sequence of type <a>MotionSequence</a>. </p>
<p>Return value: <a>MotionSequenceServiceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td>Python<br />
<br />
sequ = MotionSequence()<br />
sequ.name = 'test_sequ'<br />
sequ.describe = 'Test sequence'<br />
<br />
gait_meta = MotionSequenceGait()<br />
<br />
gait_meta.right_forefoot = 1<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 1<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
gait_meta.right_forefoot = 0<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 0<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
gait_meta.right_forefoot = 1<br />
gait_meta.left_forefoot = 1<br />
gait_meta.right_hindfoot = 1<br />
gait_meta.left_hindfoot = 1<br />
gait_meta.duration = 1<br />
sequ.gait_list.push_back(gait_meta)<br />
<br />
pace_meta = MotionSequencePace()<br />
<br />
pace_meta.twist.linear.x = 0.000000<br />
pace_meta.twist.linear.y = 0.000000<br />
pace_meta.twist.linear.z = 0.000000<br />
pace_meta.centroid.position.x = 0.000000<br />
pace_meta.centroid.position.y = 0.000000<br />
pace_meta.centroid.position.z = 0.020000<br />
pace_meta.centroid.orientation.x = 0.000000<br />
pace_meta.centroid.orientation.y = 0.000000<br />
pace_meta.centroid.orientation.z = 0.000000<br />
pace_meta.weight.linear.x = 50.000000<br />
pace_meta.weight.linear.y = 50.000000<br />
pace_meta.weight.linear.z = 5.000000<br />
pace_meta.weight.angular.x = 10.000000<br />
pace_meta.weight.angular.y = 10.000000<br />
pace_meta.weight.angular.z = 10.000000<br />
pace_meta.right_forefoot.x = 0.030000<br />
pace_meta.right_forefoot.y = 0.050000<br />
pace_meta.left_forefoot.x = -0.030000<br />
pace_meta.left_forefoot.y = -0.050000<br />
pace_meta.right_hindfoot.x = 0.030000<br />
pace_meta.right_hindfoot.y = 0.050000<br />
pace_meta.left_hindfoot.x = -0.030000<br />
pace_meta.left_hindfoot.y = -0.050000<br />
pace_meta.friction_coefficient = 0.400000<br />
pace_meta.right_forefoot.w = 0.030000<br />
pace_meta.left_forefoot.w = 0.030000<br />
pace_meta.right_hindfoot.w = 0.030000<br />
pace_meta.left_hindfoot.w = 0.030000<br />
pace_meta.landing_gain = 1.500000<br />
pace_meta.use_mpc_track = 0<br />
pace_meta.duration = 300<br />
sequ.pace_list.push_back(pace_meta)<br />
<br />
pace_meta.twist.linear.x = 0.000000<br />
pace_meta.twist.linear.y = 0.000000<br />
pace_meta.twist.linear.z = 0.000000<br />
pace_meta.centroid.position.x = 0.000000<br />
pace_meta.centroid.position.y = 0.000000<br />
pace_meta.centroid.position.z = 0.000000<br />
pace_meta.centroid.orientation.x = 0.000000<br />
pace_meta.centroid.orientation.y = 0.000000<br />
pace_meta.centroid.orientation.z = 0.000000<br />
pace_meta.weight.linear.x = 50.000000<br />
pace_meta.weight.linear.y = 50.000000<br />
pace_meta.weight.linear.z = 5.000000<br />
pace_meta.weight.angular.x = 10.000000<br />
pace_meta.weight.angular.y = 10.000000<br />
pace_meta.weight.angular.z = 10.000000<br />
pace_meta.right_forefoot.x = 0.000000<br />
pace_meta.right_forefoot.y = 0.000000<br />
pace_meta.left_forefoot.x = 0.000000<br />
pace_meta.left_forefoot.y = 0.000000<br />
pace_meta.right_hindfoot.x = 0.000000<br />
pace_meta.right_hindfoot.y = 0.000000<br />
pace_meta.left_hindfoot.x = 0.000000<br />
pace_meta.left_hindfoot.y = 0.000000<br />
pace_meta.friction_coefficient = 0.400000<br />
pace_meta.right_forefoot.w = 0.030000<br />
pace_meta.left_forefoot.w = 0.030000<br />
pace_meta.right_hindfoot.w = 0.030000<br />
pace_meta.left_hindfoot.w = 0.030000<br />
pace_meta.landing_gain = 1.500000<br />
pace_meta.use_mpc_track = 0<br />
pace_meta.duration = 300<br />
sequ.pace_list.push_back(pace_meta)<br />
<br />
cyberdog.motion.run_sequence(sequ)</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.05 🟡navigation (navigation module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.state</strong>: The interface name for obtaining the status of the navigation module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.set_log</strong>: Set the navigation module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The navigation module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_preset (get preset point)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.get_preset(</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.get_preset</strong>: Get the preset point. </p>
<p>Parameters</p>
<p>Return value: <a>MapPresetSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.get_preset().state.code == StateCode.success:</strong><br />
<strong>print('Add kitchen preset point successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) to\_preset（Navigate to the preset point）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.to_preset(</strong><br />
<strong>string preset_name,</strong><br />
<strong>bool assisted_relocation,</strong><br />
<strong>bool interact,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.to_preset</strong>: Navigate to the preset point. </p>
<p>Parameters</p>
<p><strong>preset_name</strong>: Preset point name, type is string. </p>
<p><strong>assisted_relocation</strong>: Whether to enable the assisted relocation function, the type is Boolean. </p>
<p>False: off (default value)</p>
<p>True: On</p>
<p><strong>interact</strong>: Whether to enable the auxiliary relocation interaction function, the type is Boolean. </p>
<p>False: off (default value)</p>
<p>True: On</p>
<p><strong>volume</strong>: Auxiliary relocation interaction volume, type is integer, default is 50. </p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.to_preset('kitchen', True , True , 50).response.result == 0:</strong><br />
<strong>print('Navigate to kitchen preset point successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_on\_navigation (turn on (enter) navigation mode)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.turn_on_navigation(</strong><br />
<strong>bool outdoor,</strong><br />
<strong>bool assisted_relocation,</strong><br />
<strong>bool interact,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.turn_on_navigation</strong>: Turn on navigation mode. </p>
<p>Parameters</p>
<p><strong>outdoor</strong>: Whether it is an outdoor environment, the type is Boolean. </p>
<p>False: Indoor (default)</p>
<p>True: Outdoor</p>
<p><strong>assisted_relocation</strong>: Whether to enable the assisted relocation function, the type is Boolean. </p>
<p>False: off (default value)</p>
<p>True: On</p>
<p><strong>interact</strong>: Whether to enable the auxiliary relocation interaction function, the type is Boolean. </p>
<p>False: off (default value)</p>
<p>True: On</p>
<p><strong>volume</strong>: Auxiliary relocation interaction volume, type is integer, default is 50. </p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.turn_on_navigation(False, True , True , 50).response.result == 0:</strong><br />
<strong>print('Turn on navigation mode')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_off\_relocation (turn off (exit) navigation mode)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.turn_off_navigation(</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.turn_off_navigation</strong>: Turn off navigation mode. </p>
<p>Parameters: none</p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.turn_off_navigation().response.result == 0:</strong><br />
<strong>print('Close navigation mode')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) navigation\_to\_preset（Navigate to the preset point）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.navigation_to_preset(</strong><br />
<strong>string preset_name</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.navigation_to_preset</strong>: Navigate to the preset point. </p>
<p>Parameters</p>
<p><strong>preset_name</strong>: Preset point name, type is string. </p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.navigation_to_preset('kitchen').response.result == 0:</strong><br />
<strong>print('Navigate to kitchen preset point successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) navigation\_to\_coordinates**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.navigation_to_coordinates(</strong><br />
<strong>double x,</strong><br />
<strong>double y,</strong><br />
<strong>double z,</strong><br />
<strong>double roll,</strong><br />
<strong>double pitch,</strong><br />
<strong>double yaw</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.navigation_to_coordinates</strong>: Navigate to coordinates. </p>
<p>Parameters</p>
<p><strong>x</strong>: The x coordinate of the target point, the type is floating point. </p>
<p><strong>y</strong>: The y coordinate of the target point, the type is floating point. </p>
<p><strong>z</strong>: The z coordinate of the target point, the type is floating point. </p>
<p><strong>roll</strong>: Target point posture <strong>roll</strong>, type is floating point. </p>
<p><strong>pitch</strong>: Target point pose <strong>pitch</strong>, type is floating point. </p>
<p><strong>yaw</strong>: Target point attitude <strong>yaw</strong>, type is floating point. </p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.navigation_to_coordinates(0,0,0,0,0,0).response.result == 0:</strong><br />
<strong>print('Navigate to target point successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) cancel\_navigation**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.navigation.cancel_navigation()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.navigation.cancel_navigation</strong>: Cancel navigation. </p>
<p>Parameters: none</p>
<p>Return value: <a>NavigationActionResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.navigation.cancel_navigation().response.result == 0:</strong><br />
<strong>print('Cancel navigation successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.06 🟡task (task module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.state</strong>: The interface name for obtaining the status of the task module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.task.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.set_log</strong>: Set the task module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The task module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) start（Start task）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.start()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.start</strong>: Start the task. </p>
<p>Parameters: none</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.start().state.code == StateCode.fail:</strong><br />
<strong>print('Start task successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) stop（End task）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.stop()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.stop</strong>: Cancel navigation. </p>
<p>Parameters: none</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.stop().state.code == StateCode.fail:</strong><br />
<strong>print('End task successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) block (ordinary block)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.block(</strong><br />
<strong>string id</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.block</strong>: Set the code block. </p>
<p>Parameters:</p>
<p><strong>id</strong>: Current block id, string type. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.block('abc').state.code == StateCode.fail:</strong><br />
<strong>print('Trigger abc block successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) breakpoint\_block (breakpoint block)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.breakpoint_block(</strong><br />
<strong>string id</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.breakpoint_block</strong>: Set breakpoint code block. </p>
<p>Note: When the program executes this block, the following logical functions will be blocked:</p>
<p>Normal block: block</p>
<p>Servo command related blocks:</p>
<p>Turn: turn</p>
<p>Go straight: go_straight</p>
<p>Lateral movement: lateral_movevment</p>
<p>Jump forward and backward: jump_back_and_forth</p>
<p>small jump walking: small_jump_walking</p>
<p>Automatic frequency walking: automatic_frequency_conversion_walking</p>
<p>Slow walking: trot_walking</p>
<p>Run and walk quickly: run_fast_walking</p>
<p>Here, the program will not continue execution until the user clicks the APP continue button, or calls the continue task interface. </p>
<p>Parameters:</p>
<p><strong>id</strong>: Current block id, string type. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.breakpoint_block('abc').state.code == StateCode.fail:</strong><br />
<strong>print('Trigger abc block successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) recover（continue task）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.task.recover()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.task.recover</strong>: When the task is paused, continue the task directly. </p>
<p>Explanation: This function will break out of the breakpoint blocking, allowing the program to continue running. </p>
<p>Parameters: none</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.task.recover().state.code == StateCode.fail:</strong><br />
<strong>print('Continue task successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.07 🟡personnel (personnel module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.personnel.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.personnel.state</strong>: The interface name for obtaining the status of the personnel module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.personnel.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.personnel.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.personnel.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.personnel.set_log</strong>: Set the personnel module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.personnel.set_log(False).code == StateCode.success:</strong><br />
<strong>print('personnel module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) face\_recognized (recognized target person’s face)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.personnel.face_recognized(</strong><br />
<strong>list personnel_ids,</strong><br />
<strong>bool and_operation,</strong><br />
<strong>double duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.personnel.face_recognized</strong>: The face of the target person is recognized. </p>
<p>Parameters:</p>
<p><strong>names</strong>: Nickname of the currently identified target person, list type. </p>
<p>When the list is empty, identifying one of the people in the bottom library is successful. </p>
<p><strong>and_operation</strong>: AND operation, valid when the <strong>names</strong> field is not empty;</p>
<p>The default is False, which is an OR operation;</p>
<p>True: Success will be returned only after all target persons are currently identified. </p>
<p>False: Return success if one of the target persons is currently identified. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [30,300]. </p>
<p>Return value: <a>FaceRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.personnel.face_recognized(['Zhang San']).state.code == StateCode.success:</strong><br />
<strong>print('Zhang San recognized')</strong><br />
<br />
<strong>if cyberdog.personnel.face_recognized(['Zhang San','李思']).state.code == StateCode.success:</strong><br />
<strong>print('Zhang San or Li Si recognized')</strong><br />
<br />
<strong>if cyberdog.personnel.face_recognized(['Zhang San','李思'], True).state.code == StateCode.success:</strong><br />
<strong>print('Zhang San and Li Si were recognized')</strong><br />
<br />
<strong>ret</strong> = <strong>cyberdog.personnel.face_recognized(['Zhang San','Li Si'], True)</strong><br />
<strong>if ret.state.code == StateCode.success:</strong><br />
<strong>print('Zhang San and Li Si were recognized')</strong><br />
<strong>if</strong> <strong>ret.dictionary.has_key('Zhang San') and ret.dictionary['Zhang San'].age &lt; 12:</strong><br />
<strong>cyberdog.audio.play('Please', ret.response['Zhang San'].username, 'Classmate goes home to do homework')</strong><br />
if <strong>ret.dictionary.has_key('Zhang San') and ret.response['Zhang San'].emotion &gt; 1:</strong><br />
<strong>cyberdog.audio.play(ret.response['Zhang San'].username, 'You look very happy')</strong></td>
</tr>
</tbody>
</table></td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) voiceprint\_recognized (the target person’s voiceprint is recognized)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.personnel.voiceprint_recognized(</strong><br />
<strong>list names,</strong><br />
<strong>bool and_operation,</strong><br />
<strong>double duration,</strong><br />
<strong>int sensitivity</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.personnel.voiceprint_recognized</strong>: The target person’s voiceprint is recognized. </p>
<p>Parameters:</p>
<p><strong>names</strong>: Nickname of the currently identified target person, list type. </p>
<p>When the list is empty, identifying one of the people in the bottom library is successful. </p>
<p><strong>and_operation</strong>: AND operation, valid when the <strong>names</strong> field is not empty;</p>
<p>The default is False, which is an OR operation;</p>
<p>True: Success will be returned only after all target persons are currently identified. </p>
<p>False: Return success if one of the target persons is currently identified. </p>
<p><strong>duration</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [1, 30]. </p>
<p><strong>sensitivity</strong>: sensitivity, unit second (s), type is integer; </p>
<p>Those identified in the recent period are all valid. </p>
<p>Return value: <a>VoiceprintRecognized</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.personnel.voiceprint_recognized(['Zhang San']).data:</strong><br />
<strong>print('Zhang San recognized')</strong><br />
<br />
<strong>if cyberdog.personnel.voiceprint_recognized(['Zhang San','Li Si']).data:</strong><br />
<strong>print('Zhang San or Li Si recognized')</strong><br />
<br />
<strong>if cyberdog.personnel.voiceprint_recognized(['Zhang San','Li Si'], True).data:</strong><br />
<strong>print('Zhang San and Li Si were recognized')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.08 🟡audio (voice module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.state</strong>: The interface name for obtaining the module status of the voice module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.audio.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.set_log</strong>: Set the audio module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.set_log(False).code == StateCode.success:</strong><br />
<strong>print('Audio module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) play (play voice)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.play(</strong><br />
<strong>string message,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.play</strong>: Play voice. This mode is blocking mode, that is, the next step will not be continued until the voice is played. </p>
<p>Parameters:</p>
<p><strong>message</strong>: Current broadcast message, single-line string type (cannot contain newline characters, or escape with \). </p>
<p><strong>volume</strong>: current and future broadcast volume, integer. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: -1</p>
<p>Return value: <a>AudioPlaySeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.play('Love me China').state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong><br />
<strong>if cyberdog.audio.play('Love me China\</strong><br />
<strong>').state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) instantly\_play (play voice immediately)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.instantly_play(</strong><br />
<strong>string message,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio. instantly_play</strong>: Play the voice immediately. This mode is non-blocking and preemptive mode, that is, while playing the voice, continue to execute downwards. If the voice is currently playing, the current voice will be interrupted and Play new voice. </p>
<p>Parameters:</p>
<p><strong>message</strong>: Current broadcast message, single-line string type (cannot contain newline characters, or escape with \). </p>
<p><strong>volume</strong>: current and future broadcast volume, integer. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: -1</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.instantly_play('Love me China').state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong><br />
<strong>if cyberdog.audio.instantly_play('Love me China\</strong><br />
<strong>').state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) offline\_play（Play offline voice）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.offline_play(</strong><br />
<strong>int audio_id,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.offline_play</strong>: Play offline voice. This mode is blocking mode, that is, the next step will not be continued until the voice is played. </p>
<p>Parameters:</p>
<p><strong>audio_id</strong>: Current broadcast message, integer type, only supports the following values:</p>
<p>4000: Woof woof (soft voice)</p>
<p>4001: Woof woof (rough voice)</p>
<p>6000: Music 1</p>
<p>6001: Music 1</p>
<p><strong>volume</strong>: current and future broadcast volume, integer. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: -1</p>
<p>Return value: <a>AudioPlaySeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.offline_play(4000).state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) offline\_instantly\_play (play offline voice immediately)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.offline_instantly_play(</strong><br />
<strong>int audio_id,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.offline_instantly_play</strong>: Play offline voice immediately. This mode is non-blocking and preemptive mode, that is, while playing the voice, continue to execute downwards. If the voice is currently playing, the current voice will be interrupted. and play new voices. </p>
<p>Parameters:</p>
<p><strong>audio_id</strong>: Current broadcast message, integer type, only supports the following values:</p>
<p>4000: Woof woof (soft voice)</p>
<p>4001: Woof woof (rough voice)</p>
<p>6000: Music 1</p>
<p>6001: Music 1</p>
<p><strong>volume</strong>: current and future broadcast volume, integer. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: -1</p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.offline_instantly_play(4000).state.code == StateCode.success:</strong><br />
<strong>print('Report voice successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_volume (get playback volume)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.get_volume()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.get_volume</strong>: Get the playback volume. </p>
<p>Parameters: none</p>
<p>Return value: <a>AudioGetVolumeSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>audio_volume = cyberdog.audio.get_volume()</strong><br />
<strong>if audio_volume.state.code == StateCode.success:</strong><br />
<strong>print('Getting the volume successfully, the current volume is:', audio_volume.response.volume)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) set\_volume (set playback volume)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.audio.set_volume(</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.audio.set_volume</strong>: Set playback volume. </p>
<p>Parameters:</p>
<p><strong>volume</strong>: current and future broadcast volume, integer. </p>
<p>Legal value constraints: <a>Iron Egg ability set parameter constraint table</a>. </p>
<p>Default value: -1</p>
<p>Return value: <a>AudioSetVolumeSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.audio.set_volume(10).response.success:</strong><br />
<strong>print('Set broadcast volume successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.09 🟡led (led module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.led.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.led.state</strong>: Get the interface name of the LED module status under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.led.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.led.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.led.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.led.set_log</strong>: Set the LED module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.led.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The led module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) play (play system lighting effect)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.led.play(</strong><br />
<strong>int target,</strong><br />
<strong>int effect</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.led.play</strong>: Plays the system lighting effect. Once triggered, the lighting effect will be maintained until the next lighting effect request comes. </p>
<blockquote>
<p><strong>Note:</strong></p>
</blockquote>
<p>When LED is called by a high-level module (cyberdog_manager, low power consumption, low battery, etc.), the visual programming request for light effects will fail. </p>
<p>When the task ends, the LED control authority will be released and handed over to other modules of the system for control. </p>
<p>Parameters:</p>
<p><strong>target</strong>: The target light currently controlled, integer. </p>
<p>LedConstraint.target_*. </p>
<p>For details of constraints, see <a>LED Parameter Constraints</a>. </p>
<p><strong>effect</strong>: Current lighting effect, integer, legal value is as follows (given in hexadecimal):</p>
<p>LedConstraint.system_effect_line_* or</p>
<p>LedConstraint.system_effect_mini_*. </p>
<p>For details of constraints, see <a>LED Parameter Constraints</a>. </p>
<p>Return value: <a>LedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.led.play(LedConstraint.target_head, LedConstraint.system_effect_line_red_on).state.code == StateCode.success:</strong><br />
<strong>print('Controlling headlights to stay on successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) play\_rgb (play RGB lighting effects)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.led.play_rgb(</strong><br />
<strong>int target,</strong><br />
<strong>int effect,</strong><br />
<strong>int r,</strong><br />
<strong>int g,</strong><br />
<strong>int b</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.led.play</strong>: Plays the RGB lighting effect. Once triggered, the lighting effect will be maintained until the next lighting effect request comes. </p>
<blockquote>
<p><strong>Note:</strong></p>
</blockquote>
<p>When LED is called by a high-level module (cyberdog_manager, low power consumption, low battery, etc.), the visual programming request for light effects will fail. </p>
<p>When the task ends, the LED control authority will be released and handed over to other modules of the system for control. </p>
<p>Parameters: For details, see <a>LED Parameter Constraints</a>. </p>
<p><strong>target</strong>: The target light currently controlled, integer. </p>
<p>LedConstraint.target_*. </p>
<p>For details of constraints, see <a>LED Parameter Constraints</a>. </p>
<p><strong>effect</strong>: Current lighting effect, integer, legal value is as follows (given in hexadecimal):</p>
<p>LedConstraint.effect_line_* or</p>
<p>LedConstraint.effect_mini_*. </p>
<p>For details of constraints, see <a>LED Parameter Constraints</a>. </p>
<p><strong>r</strong>: The gray value of the current red channel, integer, value range [0,255]</p>
<p><strong>g</strong>: The gray value of the current green channel, integer, value range [0,255]</p>
<p><strong>b</strong>: The gray value of the current blue channel, integer, value range [0,255]</p>
<p>Return value: <a>LedSeviceResponse</a></p>
<p>Remarks: For color swatches, see <a href="https://www.rapidtables.com/web/color/RGB_Color.html">RGB</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.led.play_rgb(LedConstraint.target_head, LedConstraint.effect_line_breath_fast, 255, 255, 255).state.code == StateCode.success:</strong><br />
<strong>print('Control the headlight to display rapid breathing success in white')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) freed (release control of the specified device)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.led.freed(</strong><br />
<strong>int target</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.led.freed</strong>: Release control of the specified device. </p>
<p>Parameters:</p>
<p><strong>target</strong>: The target light currently controlled, integer. </p>
<p>LedConstraint.target_*. </p>
<p>For details of constraints, see <a>LED Parameter Constraints</a>. </p>
<p>Return value: <a>LedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.led.freed(LedConstraint.target_head).state.code == StateCode.success:</strong><br />
<strong>print('Release control of headlight strip successfully')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.10 🟡bms (Battery Management System Module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.bms.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.bms.state</strong>: The name of the interface for obtaining the status of the battery management system module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.bms.state.code == StateCode.fail &amp;&amp; :</strong><br />
<strong>print(cyberdog.bms.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.bms.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.bms.data</strong>: The name of the interface for obtaining the status of the battery management system module under cyberdog. </p>
<p>Type: <a>BmsStatus</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.bms.data.batt_soc &lt; 20:</strong><br />
<strong>print('The current battery power of the robot is less than 20%')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.bms.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.bms.set_log</strong>: Set the bms module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.bms.set_log(False).code == StateCode.success:</strong><br />
<strong>print('BMS module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.bms.get_data(</strong><br />
<strong>int</strong> <strong>timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.bms.get_data</strong>: Get the latest battery data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>BmsStatus</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.bms.get_data().batt_soc &lt; 20:</strong><br />
<strong>print('The current battery power of the robot is less than 20%')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.11 🟡touch (touchpad module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.touch.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.touch.state</strong>: The name of the interface for obtaining the status of the touchpad module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.touch.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.touch.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.touch.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.touch.data</strong>: The name of the interface for obtaining the status of the touchpad module under cyberdog. </p>
<p>Type: <a>TouchStatus</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.touch.data.touch_state:</strong><br />
<strong>print('The current robot touchpad has been triggered')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.touch.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.touch.set_log</strong>: Set the touch module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.touch.set_log(False).code == StateCode.success:</strong><br />
<strong>print('touch module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.touch.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.touch.get_data</strong>: Get the latest touch data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>TouchStatus</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.touch.get_data().touch_state:</strong><br />
<strong>print('The current robot touchpad has been triggered')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.12 🟡gps (global positioning system module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gps.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gps.state</strong>: cyberdog module status acquisition interface name. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gps.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.gps.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gps.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gps.data</strong>: The name of the interface for obtaining the global positioning system module status under cyberdog. </p>
<p>Type: <a>GpsPayload</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gps.data.num_sv &gt; 3:</strong><br />
<strong>print('The current latitude and longitude of the robot is:', cyberdog.network.data.lon, ',', cyberdog.network.data.lat)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gps.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gps.set_log</strong>: Set the touch module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gps.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The gps module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gps.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gps.get_data</strong>: Get the latest <strong>gps</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>GpsPayload</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gps.get_data().num_sv &gt; 3:</strong><br />
<strong>print('The current latitude and longitude of the robot is:', cyberdog.network.data.lon, ',', cyberdog.network.data.lat)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.13 🟡tof (laser ranging module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.tof.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.tof.state</strong>: The name of the interface for obtaining the status of the laser ranging module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.tof.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.tof.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.tof.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.tof.data</strong>: The name of the laser ranging module status acquisition interface under cyberdog. </p>
<p>Type: <a>TofPayload</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.tof.data)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.tof.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.tof.set_log</strong>: Set the tof module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.tof.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The tof module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.tof.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.tof.get_data</strong>: Get the latest <strong>tof</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>TofPayload</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.tof.get_data())</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.14 🟡lidar (radar module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.lidar.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.lidar.state</strong>: The name of the interface for obtaining the status of the radar module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.lidar.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.lidar.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.lidar.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.lidar.data</strong>: The name of the radar module status acquisition interface under cyberdog. </p>
<p>Type: <a>LaserScan</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.lidar.data)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.lidar.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.lidar.set_log</strong>: Set the lidar module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.lidar.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The lidar module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.lidar.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.lidar.get_data</strong>: Get the latest <strong>lidar</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>LaserScan</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.lidar.get_data())</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.15 🟡ultrasonic (ultrasonic module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.ultrasonic.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.ultrasonic.state</strong>: The name of the ultrasonic module status acquisition interface under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.ultrasonic.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.ultrasonic.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.ultrasonic.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.ultrasonic.data</strong>: The name of the ultrasonic module status acquisition interface under cyberdog. </p>
<p>Type: <a>Range</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.ultrasonic.data)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.ultrasonic.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.ultrasonic.set_log</strong>: Set the ultrasonic module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.ultrasonic.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The ultrasonic module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.ultrasonic.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.ultrasonic.get_data</strong>: Get the latest <strong>ultrasonic</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>Range</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.ultrasonic.get_data())</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.16 🟡odometer (odometer module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.odometer.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.odometer.state</strong>: The name of the interface to obtain the status of the odometer module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.odometer.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.odometer.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.odometer.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.odometer.data</strong>: The name of the interface for obtaining the odometer module status under cyberdog. </p>
<p>Type: <a>Odometry</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.odometer.data)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

🟢 set\_log (set log)

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.odometer.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.odometer.set_log</strong>: Set the odometer module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.odometer.set_log(False).code == StateCode.success:</strong><br />
<strong>print('Odometer module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.odometer.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.odometer.get_data</strong>: Get the latest <strong>odometer</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>Range</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.odometer.get_data())</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.17 🟡imu (inertial navigation module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.imu.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.imu.state</strong>: The name of the interface for obtaining the status of the inertial navigation module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.imu.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.imu.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) data**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.imu.data</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.imu.data</strong>: The name of the interface for obtaining the status of the inertial navigation module under cyberdog. </p>
<p>Type: <a>Imu</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.imu.data)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.imu.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.imu.set_log</strong>: Set the imu module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.imu.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The imu module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_data (get the latest data)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.imu.get_data(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.imu.get_data</strong>: Get the latest <strong>imu</strong> data. </p>
<p>Parameters:</p>
<p><strong>timeout:</strong>The current acquisition action timeout limit (unit: seconds), integer. </p>
<p>Default value: 5. </p>
<p>Return value: <a>Imu</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print(cyberdog.imu.get_data())</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.18 🟡gesture (gesture recognition module)**

Gesture recognition function performance reference [Continuous Gesture Recognition Test Method](https://xiaomi.f.mioffice.cn/docx/doxk4yqqAvVDsdaNnRmbTXKmqxb) document.

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture.state</strong>: The name of the interface to obtain the status of the gesture recognition module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.gesture.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture.set_log</strong>: Set the gesture module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The gesture module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) recognized (starts to recognize and recognizes any gesture)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.recognized(</strong><br />
<strong>int duration,</strong><br />
<strong>int sensitivity</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture.recognized</strong>: Gesture recognized. </p>
<p>Parameters:</p>
<p><strong>duration</strong>: expected time in seconds (s), type is integer;</p>
<p>Legal value constraints: [30,300]. </p>
<p><strong>sensitivity</strong>: sensitivity, unit second (s), type is integer; </p>
<p>Those identified in the recent period are all valid. </p>
<p>Return value: <a>GestureRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.recognized().data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Recognize the palm close gesture of the warehouse staff')</strong><br />
<br />
<strong>if cyberdog.gesture.recognized().data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Palm pull gesture recognized') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_on\_recognition (turn on gesture recognition function)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.turn_on_recognition(</strong><br />
<strong>int duration</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture.turn_on_recognition</strong>: Turn on gesture recognition function. </p>
<p>Parameters:</p>
<p><strong>duration</strong>: expected time in seconds (s), type is integer;</p>
<p>Legal value constraints: [30,300]. </p>
<p>Return value: <a>GestureRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.turn_on_recognition(300).response.code == 0:</strong><br />
<strong>print('Gesture recognition function turned on successfully') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_off\_recognition (turn off gesture recognition function)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.turn_off_recognition(</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture. turn_off_recognition</strong>: Turn off gesture recognition function. </p>
<p>Parameters: none</p>
<p>Return value: <a>GestureRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.turn_off_recognition().response.code == 0:</strong><br />
<strong>print('Failed to turn on gesture recognition function') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) recognized\_designated\_gesture**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.recognized_designated_gesture(</strong><br />
<strong>int timeout,</strong><br />
<strong>int gesture_type</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture.recognized_designated_gesture</strong>: The specified gesture is recognized. </p>
<p>Parameters:</p>
<p><strong>timeout</strong>: timeout time, unit is seconds (s), type is integer; </p>
<p>Legal value constraints: [1,300]. </p>
<p><strong>gesture_type</strong>: Gesture type, type is integer. For details, see <a>GestureType</a>;</p>
<p>Specify the recognition target gesture. </p>
<p>Return value: <a>GestureRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
// Take the recognition of palm closeness as an example:<br />
<strong>if cyberdog.gesture.recognized_designated_gesture(60, 1).data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Recognize the palm close gesture of the warehouse staff')</strong><br />
// Equivalent to the following calling method<br />
<strong>if cyberdog.gesture.recognized_designated_gesture(60, GestureType.pulling_hand_or_two_fingers_in).data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Recognize the palm close gesture of the warehouse staff')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) recognized\_any\_gesture**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.gesture.recognized_any_gesture(</strong><br />
<strong>int timeout,</strong><br />
<strong>int sensitivity</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.gesture. recognized_any_gesture</strong>: Any gesture recognized. </p>
<p>Parameters:</p>
<p><strong>timeout</strong>: timeout time, unit is seconds (s), type is integer; </p>
<p>Legal value constraints: [30,300]. </p>
<p><strong>sensitivity</strong>: sensitivity, unit second (s), type is integer; </p>
<p>Those identified in the recent period are all valid. </p>
<p>Return value: <a>GestureRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.gesture.recognized_any_gesture(60, 1).data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Recognize the palm close gesture of the warehouse staff')</strong><br />
<br />
<strong>if cyberdog.gesture.recognized_any_gesture(60, 1).data.pulling_hand_or_two_fingers_in:</strong><br />
<strong>print('Palm pull gesture recognized') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.19 🟡skeleton (skeleton point identification module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton.state</strong>: The name of the interface for obtaining the status of the skeleton point recognition module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.skeleton.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton.set_log</strong>: Set the skeleton module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.set_log(False).code == StateCode.success:</strong><br />
<strong>print('The skeleton module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) sports\_recognition (sports recognition)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.sports_recognition(</strong><br />
<strong>int sport_type,</strong><br />
<strong>int counts,</strong><br />
<strong>int timeout,</strong><br />
<strong>bool interact,</strong><br />
<strong>bool instantly,</strong><br />
<strong>int volume</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton. sports_recognition</strong>: Sports recognition. </p>
<p>Parameters:</p>
<p><strong>sport_type</strong>: Identification type, the type is integer, see <a>SkeletonType</a> for constraints;</p>
<p>1 # Squat</p>
<p>2 # Lift your legs high</p>
<p>3 # sit-ups</p>
<p>4 # push-ups</p>
<p>5 # Plank</p>
<p>6 # jumping jacks</p>
<p><strong>counts</strong>: number of actions, type is integer;</p>
<p>The number of actions requested starts from 1. </p>
<p><strong>timeout</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [5,300]. </p>
<p><strong>interact</strong>: Whether to enable the interactive function, the type is Boolean. </p>
<p>False: Close</p>
<p>True: on (default value)</p>
<p><strong>instantly</strong>: Whether to enable the immediate interaction function, the type is Boolean. </p>
<p>False: Close</p>
<p>True: on (default value)</p>
<p><strong>volume</strong>: Auxiliary relocation interaction volume, type is integer, default is 50. </p>
<p>Return value: <a>DefineSkeletonRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.sports_recognition(SkeletonType.squat, 10, 60, True, True, 50).response.result == 0:</strong><br />
<strong>print('The detection and recognition function of 10 bone point squats was successfully turned on') </strong></td>
</tr>
</tbody>
</table>
<table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.sports_recognition(1, 10, 60, True, True, 50).response.result == 0:</strong><br />
<strong>print('The detection and recognition function of 10 bone point squats was successfully turned on') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_on\_recognition (turn on the bone point recognition function)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.turn_on_recognition(</strong><br />
<strong>int sport_type,</strong><br />
<strong>int counts,</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton. turn_on_recognition</strong>: Turn on the skeleton (point) recognition function. </p>
<p>Parameters:</p>
<p><strong>sport_type</strong>: Identification type, the type is integer, see <a>SkeletonType</a> for constraints;</p>
<p>1 # Squat</p>
<p>2 # Lift your legs high</p>
<p>3 # sit-ups</p>
<p>4 # push-ups</p>
<p>5 # Plank</p>
<p>6 # jumping jacks</p>
<p><strong>counts</strong>: number of actions, type is integer;</p>
<p>The number of actions requested starts from 1. </p>
<p><strong>timeout</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [5,300]. </p>
<p>Return value: <a>DefineSkeletonRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.turn_on_recognition(SkeletonType.squat, 10, 60).response.result == 0:</strong><br />
<strong>print('The detection and recognition function of 10 bone point squats was successfully turned on') </strong></td>
</tr>
</tbody>
</table>
<table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.turn_on_recognition(1, 10, 60).response.result == 0:</strong><br />
<strong>print('The detection and recognition function of 10 bone point squats was successfully turned on') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) turn\_off\_recognition (turn off the function of recognizing bone points)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.turn_off_recognition()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton. turn_off_recognition</strong>: Turn off the skeleton (point) recognition function. </p>
<p>Parameters: none</p>
<p>Return value: <a>DefineSkeletonRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.skeleton.turn_off_recognition().response.result == 0:</strong><br />
<strong>print('Skeleton point recognition function turned off successfully') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) blocking\_recognized**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.blocking_recognized(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton.blocking_recognized</strong>: Turn on the skeleton (point) recognition function. </p>
<p>Parameters:</p>
<p><strong>timeout</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [5,300]. </p>
<p>Return value: <a>SkeletonRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
// Interaction, <strong>print transposition voice interface can achieve interaction</strong><br />
<strong>print('Current movement count is', cyberdog.skeleton.blocking_recognized().response.counts,')</strong><br />
<br />
<strong>print('The current movement duration is', cyberdog.skeleton.blocking_recognized().response.duration, 'seconds') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) instant\_recognized**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.skeleton.instant_recognized()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.skeleton.instant_recognized</strong>: Turn on the skeleton (point) recognition function. </p>
<p>Parameters: none</p>
<p>Return value: <a>SkeletonRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
// Determine whether the current algorithm is enabled<br />
if (<strong>cyberdog.skeleton.instant_recognized().response.algo_switch == 0</strong>) // Turn on<br />
if (<strong>cyberdog.skeleton.instant_recognized().response.algo_switch == 1</strong>) // Close</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.4.20 🟡train (training module)**

**[🟣](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#Qp608Q) state**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.train.state</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.train.state</strong>: The name of the interface to obtain the status of the training module under cyberdog. </p>
<p>Type: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.train.state.code == StateCode.fail:</strong><br />
<strong>print(cyberdog.train.state.describe)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**🟢 set\_log (set log)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.train.set_log(</strong><br />
<strong>bool log)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.train.set_log</strong>: Set the train module log under cyberdog. </p>
<p>Parameters:</p>
<p><strong>log</strong>: Set log status, Boolean type;</p>
<p>True: enable cyberdog module log;</p>
<p>False: Turn off the cyberdog module log. </p>
<p>Return value: <a>State</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>if cyberdog.train.set_log(False).code == StateCode.success:</strong><br />
<strong>print('Train module log under cyberdog has been closed')</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) get\_training\_words\_set (get training word set)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.train.get_training_words_set(</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.train.get_training_words_set</strong>: Get the training word set. </p>
<p>Parameters: none</p>
<p>Return value: <a>TrainingWordsRecognizedSeviceResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog. train.get_training_words_set()</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) training\_words\_recognized (recognized training words)**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>cyberdog.train.training_words_recognized(</strong><br />
<strong>int timeout</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>cyberdog.train.get_training_words_set</strong>: Get the training word set. </p>
<p>Parameters:</p>
<p><strong>timeout</strong>: expected time in seconds (s), type is floating point;</p>
<p>Legal value constraints: [5,300]. </p>
<p>Return value: <a>TrainingWordsRecognizedMessageResponse</a></p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>print('Training words recognized:', cyberdog. train.training_words_recognized().response.trigger)</strong><br />
<br />
if <strong>cyberdog. train.training_words_recognized().response.trigger == 'Zhang San':</strong><br />
<strong>print('The training word is recognized as: Zhang San') </strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.5 Built-in capability interface constraints**

**4.5.1 🟡choreographer (choreographer module)**

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) moonwalk**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>choreographer.moonwalk(</strong><br />
<strong>float x_velocity,</strong><br />
<strong>float y_velocity,</strong><br />
<strong>float stride,</strong><br />
<strong>int number</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>choreographer.moonwalk</strong>: Choreograph the moonwalk. </p>
<p>Parameters:</p>
<p><strong>x_velocity</strong>: X-axis velocity, unit is meters per second (m/s), type is floating point;</p>
<p>Legal value constraints: [-0.08, 0.08]. </p>
<p><strong>y_velocity</strong>: Y-axis velocity, unit is meters per second (m/s), type is floating point;</p>
<p>Legal value constraints: [-0.05, 0.05]. </p>
<p><strong>stride</strong>: No width, unit is meter (m), type is floating point;</p>
<p>Legal value constraints: (0,0.065].</p>
<p><strong>number</strong>: number of moonwalk repetitions, type is integer;</p>
<p>Legal value constraints: [1,10]. </p>
<p>Return value: None</p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>choreographer.moonwalk(0.05, 0.03, 0.03, 2)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) push\_up（Push-ups）**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>choreographer.push_up(</strong><br />
<strong>int frequency,</strong><br />
<strong>int number</strong><br />
<strong>)</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>choreographer.moonwalk</strong>: Choreograph the moonwalk. </p>
<p>Parameters:</p>
<p><strong>frequency</strong>: No frequency, unit times per minute, type is integer;</p>
<p>Legal value constraints: [10, 60]. </p>
<p><strong>number</strong>: Number of push-up repetitions, type is integer;</p>
<p>Legal value constraints: [2,10]. </p>
<p>Return value: None</p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>choreographer.push_up(10, 3)</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**4.5.2 🟡dancer（Dancer module）**

**[🟢](https://xiaomi.f.mioffice.cn/docs/dock4nUNWHVve526QMxnT4b0taf#DTXadn) dance**

<table>
<tbody>
<tr class="odd">
<td>Agreement Constraints</td>
<td>Protocol Description</td>
<td>Examples</td>
</tr>
<tr class="even">
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>dancer.dance()</strong></td>
</tr>
</tbody>
</table></td>
<td><p>Interface name</p>
<p><strong>dancer. dance</strong>: Dance according to the choreography. </p>
<p>Parameters: none</p>
<p>Return value: None</p></td>
<td><table>
<tbody>
<tr class="odd">
<td><br />
<strong>dancer.dance()</strong></td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

**5. System capability set**

Currently, it only supports other functions of the python3 time module. In theory, it supports all other functions of python3, but it requires fine-tuning in the graphical programming engine.
