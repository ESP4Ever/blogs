Summary of developer QA questions

## Where can I download the flash package?

```Bash
[CyberDog2 flash package download] https://s.xiaomi.cn/c/8JEpGDY8
```

## What is the reason for radar self-test failure?

```Bash
Updating NVIDIA official software package may cause radar self-test to fail
1. It is currently known that nvidia's own service nvgetty.service will use the same serial port as the radar, and the two will conflict. We have disabled this service at the factory.
2. You can use the following command to check whether the service is running:
sudo systemctl status nvgetty.service
3. If it is running, use the following command to disable it:
sudo systemctl disable nvgetty.service
Just restart after execution
```

## What should I do if I have trouble compiling and encounter various problems?

``Plain
1. Enter the tools directory and use the Dockerfile file to compile the image.
2. Refer to the Dockerfile document [Docker File Instructions](https://github.com/ESP4Ever/blogs/blob/rolling/docs/en/dockerfile_instructions_en.md) to compile. If you have other questions, you can ask in the issue.
```

## Cyberdog and cyberdog2 corresponding warehouse

```Bash
Cyberdog corresponding warehouse address: https://github.com/MiRoboticsLab/cyberdog_ros2
The corresponding warehouse address of cyberdog2: https://github.com/MiRoboticsLab/cyberdog_ws
```
