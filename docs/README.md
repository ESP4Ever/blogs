<h1 align="center">
   Quadruped Robot Development Guide
</h1>


## Introduction

This article provides software development guidance based on Xiaomi's four-legged development platform.
[Table of Contents](https://github.com/ESP4Ever/blogs/blob/rolling/docs/_table_of_contents.md)


## Frame

**Develop overall framework diagram**

![dev](./image/dev_en.png)

**Development Method**

1. Write a program based on PC and schedule the robot through grpc or ros.
2. Write a scheduler based on NX and call the existing service class ros interface (recommended)
3. Rewrite the program based on the NX side to conduct secondary development of existing scheduling or service businesses

**Development Process**

1. Apply for developer permission through mobile APP

2. Establish a connection with the robot (either wireless or wired)

    - Wired login method, USB-Type data cable connects the PC and the robot Download port:

      `ssh mi@192.168.55.1`

      `passwd: "123"`

    - Wirelessly, connect the robot to the Internet through the app, and log in using the wireless network IP (which can be viewed in the app)

    ![connect](./image/connect.png)

3. Just start the way you want



## resource

- [Core code open source](https://github.com/MiRoboticsLab/cyberdog_ws)
- [Developer manual (API, 1st and 2nd development methods above)](https://github.com/ESP4Ever/blogs/blob/rolling/docs/en/developer_guide.md)
- Programming (source code, the third development method mentioned above): application documents, operation control documents, slam documents and perception documents;
- [Flash](https://github.com/ESP4Ever/blogs/blob/rolling/docs/en/cyberdog_flash.md)
- Operation consultation: [mi-cyberdog@xiaomi.com](mailto:mi-cyber@xiaomi.com)
