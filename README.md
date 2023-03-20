# UWB Gazebo Sensor Plugin

Forked from: [https://github.com/AUVSL/UWB-Gazebo-Plugin](https://github.com/AUVSL/UWB-Gazebo-Plugin)
And heavily modified.

## README

This project contains several plugins to use in Gazebo simulator:

### Requirements

Libraries ```libignition-math4-dev``` and ```libgazebo9-dev``` must be installed before building this package.

### Build

After cloning the repository in a catkin workspace:
```
$ catkin_make
```

### UWB Plugin

![UWB Plugin in RVIZ](https://user-images.githubusercontent.com/38099967/64428790-e66b6780-d0b4-11e9-8f6f-489d8eb949c8.png)

This plugin simulates a UWB tag sensor. The plugin simulates the reception of UWB ranging measurements from a set of anchors placed on the scenario. The plugin also produces different measurements depending on the type of line of sight between the tag and each anchor. Thus, four models are considered:

- LOS (Line of Sight): In this case there are no obstacles between the tag and the anchor. In this case, the ranging estimation is near to the actual value.
- NLOS Soft (Non Line of Sight Soft) : In this case, there is a thin obstacle between the tag and the anchor. The received power is lower than in the LOS case but the ranging estimation is closer to the actual value.
- NLOS Hard: In this case, there are too many obstacles between the tag and the anchor that the direct ray is unable to reach the receiver. But in this case, there is a path from the tag to the anchor after the signal rebounding in one wall. Thus, the estimated ranging is the corresponding to this new path, so is going to be always longer than the actual distance between the devices.
- NLOS: Tag and anchor are too far apart or there are too many obstacles between them, they are unable to comunicate and generate a ranging estimation.

NOTE: the rebounds are only computed using obstacles placed at the same height than the tag. That means that the rebounds on the floor or the ceil are not considered.

To add the plugin to a Gazebo model, the next code must be present in the .sdf o .urdf.

```xml
<plugin name='libuwb_plugin' filename='libuwb_plugin.so'>
  <update_rate>25</update_rate>
  <nlosSoftWallWidth>0.25</nlosSoftWallWidth>
  <tag_z_offset>1.5</tag_z_offset>
  <tag_link>tag_0</tag_link>
  <anchor_prefix>uwb_anchor</anchor_prefix>
  <all_los>false</all_los>
  <tag_id>0</tag_id>
  <topic_name_ranging>uwb/tag_1_ranging</topic_name_ranging>
  <topic_name_anchors>uwb/tag_1_anchors</topic_name_anchors>
  <topic_name_serial_ranging>uwb/tag_1_serial_ranging</topic_name_serial_ranging>
</plugin>
``` 

* update_rate: num. of rates published by second.
* nlosSoftWallWidth: how thin a wall must be to let the signal go through it. 
* tag_z_offset: a offset in meters to add to the current height of the sensor.
* anchor_prefix: all the anchors placed in the scenario must have a unique name that starts with this prefix.
* all_los: if true, all the anchors are considered as in a LOS scenario. Except if there are too far from the tag, in that case they are considered as NLOS.
* tag_id: tag identifier, a number.
* topic_name_ranging: Topic name to publish the ranging messages from the tag
* topic_name_anchors: Topic name to publish Visualization MarkerArray messages from the tag
* topic_name_serial_ranging: Topic name to publish the ranging messages from the tag similar to real uwb tag serial reading structured as:
  ``` python
  # 0: "DIST",
  # 1: <NUM_READINGS>,

  # 2: "AN0",
  # 3: <ANCHOR_0_ID>,
  # 4: <ANCHOR_0_X>,
  # 5: <ANCHOR_0_Y>,
  # 6: <ANCHOR_0_Z>,
  # 7: <ANCHOR_0_DISTANCE>,

  # 8: "AN1",
  # 9: <ANCHOR_1_ID>,
  #10: <ANCHOR_1_X>,
  #11: <ANCHOR_1_Y>,
  #12: <ANCHOR_1_Z>,
  #13: <ANCHOR_1_DISTANCE>,

  #14: "AN2",
  #15: <ANCHOR_2_ID>,
  #16: <ANCHOR_2_X>,
  #17: <ANCHOR_2_Y>,
  #18: <ANCHOR_2_Z>,
  #19: <ANCHOR_2_DISTANCE>,

  #20: "AN3",
  #21: <ANCHOR_3_ID>,
  #22: <ANCHOR_3_X>,
  #23: <ANCHOR_3_Y>,
  #24: <ANCHOR_3_Z>,
  #25: <ANCHOR_3_DISTANCE>,

  #26: "POS",
  #27: <TAG_SELF_POS_X>,
  #28: <TAG_SELF_POS_Y>,
  #29: <TAG_SELF_POS_Z>,
  #30: <??>

  e.g.:
  "DIST,4,AN0,2F2F,3.05,2.68,0.00,2.21,AN1,2C9D,-0.04,2.91,0.00,2.39,AN2,2ED0,3.02,0.00,0.00,2.19,AN3,2BA2,0.00,0.00,0.00,2.56,POS,1.59,1.65,1.27,44"
  0    1 2   3    4    5    6    7    8   9    10    11   12   13   14  15   16   17   18   19   20  21   22    23   24  25   26  27   28   29   30
  or 
  "DIST,3,AN0,2F2F,3.05,2.68,0.00,2.20,AN1,2ED0,3.02,0.00,0.00,2.23,AN2,2BA2,0.00,0.00,0.00,3.13,POS,1.76,1.59,0.51,40"
  0    1 2   3    4    5    6    7    8   9    10   11   12   13   14  15   16   17   18   19   20  21   22   23   24
  ```

To place an anchor in a Gazebo's world, the only requirement is that the model must have a name starting with ```anchor_prefix```. A simple model could be:

```xml
<model name="uwb_anchor0">
  <pose>0 0 1.5 0 0 0</pose>
  <static>1</static>
  <link name="link">
    <visual name="visual">
      <geometry>
        <box>
          <size>0.2 0.2 0.2</size>
        </box>
      </geometry>
    </visual>
  </link>
</model>
```

otherwise you can place an anchor in the world by naming a link with ```anchor_prefix```. This link can be defined in a SDF or a URDF as long as Gazebo can see it. For fixed joints you might need to enabled:

```xml
<link name="uwb_anchor0">
  <inertial>
    <mass value="1" />
    <origin rpy="0 0 0" xyz="0 0 0"/>
    <inertia ixx="1.0" ixy="0.0" ixz="0.0" iyy="1.0" iyz="0.0" izz="1.0" />
  </inertial>

  <collision>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <box size="1 1 1"/>
    </geometry>
  </collision>
  <visual>
    <origin xyz="0 0 0" rpy="0 0 0"/>
    <geometry>
      <box size="1 1 1"/>
    </geometry>
  </visual>
</link>

<joint name="uwb_anchor_joint" type="fixed">
  <origin xyz="0 0 0" rpy="0 0 0"/>
  <parent link="base_link"/>
  <child link="uwb_anchor0"/>
</joint>

<gazebo reference="uwb_anchor_joint">
  <preserveFixedJoint>true</preserveFixedJoint>
</gazebo>

<gazebo reference="uwb_anchor0">
  <material>Gazebo/White</material>
</gazebo>
```

This plugin publish the next topics:

- ```/uwb/ranging``` : where ```tag_id``` is the value configured in the plugins. This topic publish the UWB range estimations using messages of type: uwb_gazebo_plugin/Ranging.msg

- ```/uwb/anchors``` : where ```tag_id``` is the value configured in the plugins. This topic publish the position of the UWB anchors in the scenario. Each anchor have a different color depending of its current LOS mode: green-LOS, yellow-NLOS Soft, blue-NLOS Hard and red-NLOS. The published messages are of type visualization_msgs/MarkerArray.msg ([visualization_msgs/MarkerArray Message](http://docs.ros.org/melodic/api/visualization_msgs/html/msg/MarkerArray.html))



