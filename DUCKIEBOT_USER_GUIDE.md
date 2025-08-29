# 🤖 DuckieBot Object Following System - User Guide

## 📋 **Table of Contents**
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Setup Instructions](#setup-instructions)
4. [Running the System](#running-the-system)
5. [Customization Options](#customization-options)
6. [Troubleshooting](#troubleshooting)
7. [Advanced Features](#advanced-features)

---

## 🎯 **Overview**

This system enables your DuckieBot to autonomously follow objects (especially tennis balls) using advanced computer vision and AI. The system features:

- **🎾 YOLOv8-based Tennis Ball Detection** via API
- **🧠 Enhanced PID Control** for smooth following
- **📊 Real-time Performance Monitoring**
- **🛡️ Obstacle Avoidance & Safety Features**
- **🔄 Kalman Filter Tracking** for robust object following

---

## ✅ **Prerequisites**

### **Hardware Requirements:**
- ✅ DuckieBot with camera functionality
- ✅ Network connectivity (WiFi)
- ✅ Tennis ball or red object for testing

### **Software Requirements:**
- ✅ DuckieBot with `dt-duckiebot-interface` container running
- ✅ Server machine with YOLOv8 API running (for CNN-based detection)
- ✅ SSH access to your DuckieBot

### **Network Setup:**
- ✅ DuckieBot and server machine on the same network
- ✅ Know your server machine's IP address

---

## 🚀 **Setup Instructions**

### **Step 1: Connect to Your DuckieBot**
```bash
ssh duckie@blueduckie.local
```
*Replace `blueduckie` with your DuckieBot's name*

### **Step 2: Check Running Containers**
```bash
docker ps
```
Look for the container with image `duckietown/dt-duckiebot-interface`

### **Step 3: Get Container ID**
Copy the **Container ID** of the `duckietown/dt-duckiebot-interface` container from the previous command output.

### **Step 4: Enter the DuckieBot Container**
```bash
docker exec -it [CONTAINER_ID] /bin/bash
```
*Replace `[CONTAINER_ID]` with the actual container ID from Step 3*

Example:
```bash
docker exec -it a1b2c3d4e5f6 /bin/bash
```

### **Step 5: Clone the Project Repository**
```bash
git clone https://github.com/Sumeet-2023/duckie_v2.git
cd duckie_v2
```

### **Step 6: Configure API Server IP**

#### **Step 6a: Open the Configuration File**
```bash
nano src/object_follower/scripts/enhanced_object_detector.py
```

#### **Step 6b: Update Server IP Address**
Find this line (around line 85):
```python
self.api_url = rospy.get_param('~api_url', 'http://172.20.10.3:8000/detect')
```

Change `172.20.10.3` to **your server machine's IP address**:
```python
self.api_url = rospy.get_param('~api_url', 'http://YOUR_SERVER_IP:8000/detect')
```

#### **Step 6c: Save the File**
- Press `Ctrl + O` → Press `Enter` → Press `Ctrl + X`

### **Step 7: Launch the Object Following System**
```bash
./run_object_detection.sh
```

### **Step 8: Test with Tennis Ball** 🎾
- **Place a tennis ball** in front of the DuckieBot camera
- The robot should detect and start following the ball!

---

## 🎮 **Running the System**

### **Expected System Startup:**
```
[INFO] Enhanced Object Detector initialized - Using API: http://YOUR_IP:8000/detect
[INFO] Enhanced Motor Controller - DuckieBot deployment ready
[INFO] Performance Monitor started - Tracking system metrics
```

### **Successful Detection Output:**
```
[INFO] 🎯 TARGET DETECTED!
[INFO] 🚗 Following: pos=(0.23,0.15) dist=0.85m → vel=(0.15,-0.45)
[INFO] 🎯 Target: True | 🟢 Stability: 78.5%
```

### **Performance Monitoring:**
The system continuously reports:
- **Detection Rate** (Hz)
- **Stability Score** (0-100%)
- **Control Performance** metrics
- **Position Tracking** accuracy

---

## ⚙️ **Customization Options**

### **1. Change Target Object**
Edit `enhanced_object_detector.py` to detect different objects by modifying the API parameters or switching to color-based detection.

### **2. Adjust Following Distance**
In the launch files, modify:
```xml
<param name="target_distance" value="0.15"/>  <!-- Distance in meters -->
```

### **3. Control Sensitivity**
Adjust PID parameters for different following behaviors:
```xml
<!-- More aggressive following -->
<param name="kp_lateral" value="1.2"/>
<param name="ki_lateral" value="0.08"/>
<param name="kd_lateral" value="0.4"/>
```

### **4. Speed Settings**
```xml
<param name="max_speed" value="0.20"/>  <!-- Max speed in m/s -->
```

### **5. Safety Parameters**
```xml
<param name="safe_distance_pixels" value="40"/>  <!-- Obstacle avoidance -->
```

---

## 🔧 **Troubleshooting**

### **❌ Common Issues & Solutions**

#### **1. "API call failed" or "API call timed out"**
**Problem:** Cannot connect to YOLOv8 server
**Solutions:**
- ✅ Verify server IP address is correct
- ✅ Ensure YOLOv8 API server is running on port 8000
- ✅ Check network connectivity: `ping YOUR_SERVER_IP`
- ✅ Verify firewall settings on server machine

#### **2. "No camera feed" or "Waiting for data"**
**Problem:** Camera not working
**Solutions:**
- ✅ Check camera hardware connection
- ✅ Restart DuckieBot interface: `docker restart [CONTAINER_ID]`
- ✅ Verify camera topics: `rostopic list | grep camera`

#### **3. "Robot not moving" despite detection**
**Problem:** Motor control issues
**Solutions:**
- ✅ Check motor driver status: `rostopic echo /blueduckie/wheels_driver_node/wheels_cmd`
- ✅ Verify joystick override is disabled
- ✅ Check obstacle detection: `rostopic echo /object_follower/obstacle_detected`

#### **4. "Poor following performance"**
**Problem:** Unstable tracking
**Solutions:**
- ✅ Improve lighting conditions
- ✅ Use a bright tennis ball
- ✅ Adjust PID parameters in launch file
- ✅ Reduce max speed for smoother control

### **📊 Debug Commands**

Check system status:
```bash
# List all active topics
rostopic list

# Monitor detection status
rostopic echo /object_follower/target_found

# Check performance metrics
rostopic echo /object_follower/performance

# View camera feed
rostopic hz /blueduckie/camera_node/image/compressed
```

---

## 🚀 **Advanced Features**

### **1. Multiple Object Types**
The system can be extended to detect:
- Different colored objects (red, blue, green balls)
- Custom objects using YOLOv8 training
- Multiple objects simultaneously

### **2. Behavioral Modes**
Available launch configurations:
- `duckiebot_following_mode.launch` - Aggressive close following
- `duckiebot_deployment.launch` - Conservative safe following

### **3. Performance Analytics**
Real-time metrics include:
- **Detection Rate** (frames per second)
- **Stability Score** (tracking consistency)
- **Control Smoothness** (movement quality)
- **Response Time** (reaction speed)

### **4. API Integration**
The system supports:
- **YOLOv8 API** for advanced object detection
- **Fallback color detection** when API unavailable
- **Asynchronous processing** for smooth performance

---

## 📝 **Quick Start Checklist**

- [ ] ✅ SSH into DuckieBot
- [ ] ✅ Find and enter dt-duckiebot-interface container
- [ ] ✅ Clone the repository
- [ ] ✅ Update server IP in enhanced_object_detector.py
- [ ] ✅ Run the system with `./run_object_detection.sh`
- [ ] ✅ Place tennis ball in camera view
- [ ] ✅ Watch your DuckieBot follow the ball! 🎾

---

## 🎯 **Tips for Best Results**

1. **🌞 Good Lighting:** Ensure adequate lighting for better detection
2. **🎾 Bright Objects:** Use bright, contrasting objects (tennis balls work great!)
3. **📐 Proper Distance:** Start with object 0.5-1.0 meters from robot
4. **🧹 Clear Path:** Ensure clear path for robot movement
5. **⚡ Stable Network:** Maintain good WiFi connection for API calls

---

## 🆘 **Need Help?**

If you encounter issues:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review system logs: `rostopic echo /rosout`
3. Verify all prerequisites are met
4. Ensure server IP configuration is correct

**Happy Robot Following! 🤖🎾**

---

*Built with ❤️ using ROS, OpenCV, YOLOv8, and DuckieBot*