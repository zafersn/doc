<p>Hello everbody.</p>
<p>Using ROS libraries to broadcast ROS after reading sensor data in Arduino is a bit annoying in complex systems. Data losses, involuntary behaviors, etc. And especially when I used RVIZ, I encountered serious problems.  I have had such problems.</p>
<p>As a solution, I decided to try a method like this:I decided it was unnecessary to publish the sensor data directly on the arduino "ROS". Instead, I decided to send the data to the ROS by serial connection and then to make a new publishing from the ROS after processing the incoming data on the ROS. I don't want to extend of this topic too much. When you examine the codes below, you will see how I want to do it.</p>
<p>In this example, I read the encoder data with arduino and sent it to the computer(ROS) via serial communication via USB.</p>
<h2>VIDEO OF THE PROJECT:</h2>
<p>https://youtu.be/8hrUqDxv-XI</p>
<p> </p>
<p> </p>
<h2>WIRING DIAGRAM:</h2>
<p><a href="https://github.com/zafersn/doc/blob/master/img/how-does-arduino-communicate-directly-with-ros.png"><img class="aligncenter size-large wp-image-1514" src="http://stackcuriosity.com/wp-content/uploads/2017/12/fritzing-1024x704.png" alt="" width="1024" height="704" /></a></p>
<h2>ARDUINO CODE:</h2>
```
<pre class="lang:c decode:true" title="Arduino Code">#include "Arduino.h"

int encoder_a=2;   // encoder 2 and 3 will be connected to pine
int encoder_b=3;
long pozisyon = 0;   //position of the encoder
int encoder_resolution=4200;  //encoder resolution

//L298 MOTOR DRIVE PIN CONNECTION DIAGRAM:
int enA = 10;  //PWM
int in1 = 9;   //DIRECTIOM
int in2 = 8;   //DIRECTIOM

void setup() {
  Serial.begin(57600);
  attachInterrupt(1, encoder_kesme_a, RISING);
  attachInterrupt(0, encoder_kesme_b, RISING);
  pinMode(enA, OUTPUT);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);

  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
  delay(500);

}

void loop() {
   
  Serial.print("!");
  Serial.println(pozisyon);
  Serial.println("#");
  delay(10);
  analogWrite(enA, 80);
}
//Interrupt

void encoder_kesme_a()
{

  if (digitalRead(encoder_a) == digitalRead(encoder_b))
  {
  if(pozisyon == 0) pozisyon = 4200;
      else pozisyon--;
  }
  else
  { 
      if(pozisyon == 4200) pozisyon = 0;
  else pozisyon++;
  }
}

void encoder_kesme_b()
{
  if (digitalRead(encoder_b) == digitalRead(encoder_a))
  {
      if(pozisyon == 4200) pozisyon = 0;
  else pozisyon++;
  }
  else
  {
      if(pozisyon == 0) pozisyon = 4200;
      else pozisyon--;
  }
}
</pre>
```
<h3 id="_interrupt_numbers" class="float">Interrupt Numbers</h3>
<div class="paragraph">
<p>Normally you should use digitalPinToInterrupt(pin), rather than place an interrupt number directly into your sketch. The specific pins with interrupts, and their mapping to interrupt number varies on each type of board. Direct use of interrupt numbers may seem simple, but it can cause compatibility trouble when your sketch is run on a different board.</p>
</div>
<div class="paragraph">
<p>However, older sketches often have direct interrupt numbers. Often number 0 (for digital pin 2) or number 1 (for digital pin 3) were used. The table below shows the available interrupt pins on various boards.</p>
</div>
<div class="paragraph">
<p>Note that in the table below, the interrupt numbers refer to the number to be passed to attachInterrupt(). For historical reasons, this numbering does not always correspond directly to the interrupt numbering on the atmega chip (e.g. int.0 corresponds to INT4 on the Atmega2560 chip).</p>
</div>
<table class="tableblock frame-all grid-all spread" style="height: 161px;" width="674"><colgroup> <col /> <col /> <col /> <col /> <col /> <col /> <col /></colgroup>
<thead>
<tr>
<th class="tableblock halign-left valign-top">BOARD</th>
<th class="tableblock halign-left valign-top">INT.0</th>
<th class="tableblock halign-left valign-top">INT.1</th>
<th class="tableblock halign-left valign-top">INT.2</th>
<th class="tableblock halign-left valign-top">INT.3</th>
<th class="tableblock halign-left valign-top">INT.4</th>
<th class="tableblock halign-left valign-top">INT.5</th>
</tr>
</thead>
<tbody>
<tr>
<td class="tableblock halign-left valign-top">
<p class="tableblock">Uno, Ethernet</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">2</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">3</p>
</td>
<td class="tableblock halign-left valign-top"> </td>
<td class="tableblock halign-left valign-top"> </td>
<td class="tableblock halign-left valign-top"> </td>
<td class="tableblock halign-left valign-top"> </td>
</tr>
<tr>
<td class="tableblock halign-left valign-top">
<p class="tableblock">Mega2560</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">2</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">3</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">21</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">20</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">19</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">18</p>
</td>
</tr>
<tr>
<td class="tableblock halign-left valign-top">
<p class="tableblock">32u4 based (e.g Leonardo, Micro)</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">3</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">2</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">0</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">1</p>
</td>
<td class="tableblock halign-left valign-top">
<p class="tableblock">7</p>
</td>
<td class="tableblock halign-left valign-top"> </td>
</tr>
</tbody>
</table>
<div class="paragraph">
<p>For Due, Zero, MKR1000 and 101 boards the <strong>interrupt number = pin number</strong>.</p>
</div>
<p>See for <a href="https://www.arduino.cc/reference/en/language/functions/external-interrupts/attachinterrupt/">detail</a></p>
<h2>ROS ENCODER CODE:</h2>
<p> </p>
<p>I use "sensor_msgs :: JointState" to broadcast the encoder message on the ROS.  Here, the data received via usb is converted to the message type "sensor_MSGS :: JointState". See for <a href="http://docs.ros.org/api/sensor_msgs/html/msg/JointState.html">detail</a> .</p>
<p> </p>
```
<pre class="lang:c++ decode:true" title="ROS">#include "ros/ros.h"
#include &lt;sensor_msgs/JointState.h&gt;
#include &lt;std_msgs/Header.h&gt;
#include "serial/serial.h"
#include &lt;string&gt;
#include &lt;iostream&gt;
#include &lt;cstdio&gt;

using std::string;

int pozisyon = 0;
int ang=0;
double encoder_resolution=4200;


char a[] = {"base_tilt_joint"}; // 

double pos[]={0.0}; 
double vel[]={0.0};
double eff[]={0.0};
serial::Serial my_serial("/dev/ttyACM1", 57600, serial::Timeout::simpleTimeout(10));
sensor_msgs::JointState robot_state;

double degreTOrad(double degre);
//string trim(const string&amp; str);

int main(int argc, char **argv)
{

  ros::init(argc, argv, "joint_state_publisher");

  ros::NodeHandle n;
  
  std_msgs::Header header;


  ros::Publisher chatter_pub = n.advertise&lt;sensor_msgs::JointState&gt;("joint_states", 10);

  ros::Rate loop_rate(100);



  robot_state.header;
  robot_state.header.stamp=ros::Time::now();
  robot_state.name.resize(1);
  robot_state.velocity.resize(1);
  robot_state.position.resize(1); /// here used for arduino time
  robot_state.effort.resize(1); /// here used for arduino time

    robot_state.name[0]=(a);
    pos[0]=degreTOrad(pozisyon);
	


    robot_state.position[0] = pos[0];
    robot_state.velocity[0] = vel[0];
    robot_state.effort[0] = eff[0];
    if(my_serial.isOpen())  // if connection is successful
      std::cout &lt;&lt; " Yes." &lt;&lt;std:: endl;
    else
      std::cout &lt;&lt; " No." &lt;&lt;std:: endl;
  while (ros::ok())
  {

    string s= my_serial.read(32);
 if(s[0]=='!'&amp;&amp;s.find("#")!=-1){
  string sss=s.substr(s.find("!")+1,s.find("#")-1);
  int a= std::stoi((sss.c_str()));
  pozisyon=a;
}

   ang=((pozisyon/encoder_resolution)*360.0);  // calculate angel according to pulse count
   ang=int(ang)%360;
  // ROS_INFO("anglee: %d",(ang));
   
   
    pos[0]=degreTOrad(ang);
    robot_state.header.stamp=ros::Time::now();
    robot_state.position[0] = pos[0];
    robot_state.velocity[0] = 0;
    robot_state.effort[0] = 0;
   
    chatter_pub.publish(robot_state);

    ros::spinOnce();

    loop_rate.sleep();

  }


  return 0;
}

double degreTOrad(double degre){   //degre to radyan for rviz
  
  
  return (degre/57.2958);
  
  }</pre>
  ```
<p> </p>
<p> </p>
<h2>ROS CMakeList.txt Add SERIAL LIBRARY FOR USB COMMUNICATION:</h2>
<p> </p>
<p>You must install serial communication libraries before you run the application.</p>
<p><a href="http://wiki.ros.org/serial">click here</a> to follow the directions.</p>
<p> </p>
```
<pre class="lang:xhtml decode:true " title="CMakeList.txt">cmake_minimum_required(VERSION 2.8.3)
project(lidar_package)

## Compile as C++11, supported in ROS Kinetic and newer
# add_compile_options(-std=c++11)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  sensor_msgs
  std_msgs
  serial

)

generate_messages(DEPENDENCIES   sensor_msgs)
add_compile_options(-std=c++11)
catkin_package(CATKIN_DEPENDS roscpp  DEPENDS libserial)
#catkin_package()

include_directories(include ${catkin_INCLUDE_DIRS})
add_executable(ENCODER_node src/encoder.cpp)
target_link_libraries(ENCODER_node  ${catkin_LIBRARIES})
#add_executable(talker src/encoder.cpp)
#target_link_libraries(talker ${catkin_LIBRARIES})
add_dependencies(ENCODER_node encoder_generate_messages_cpp)





add_executable(listener_encoder src/auto_subcrip.cpp)
#target_link_libraries(listener_encoder ${catkin_LIBRARIES} /usr/lib/libserial.so)
 target_link_libraries(listener_encoder ${catkin_LIBRARIES} )

add_dependencies(listener_encoder encoder_generate_messages_cpp)


add_executable(publish_encoder src/encoderAuto.cpp)
target_link_libraries(publish_encoder ${catkin_LIBRARIES} )
add_dependencies(publish_encoder encoder_generate_messages_cpp)




## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)
#  laser_geometry

## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

################################################
## Declare ROS messages, services and actions ##
################################################

## To declare and build messages, services or actions from within this
## package, follow these steps:
## * Let MSG_DEP_SET be the set of packages whose message types you use in
##   your messages/services/actions (e.g. std_msgs, actionlib_msgs, ...).
## * In the file package.xml:
##   * add a build_depend tag for "message_generation"
##   * add a build_depend and a run_depend tag for each package in MSG_DEP_SET
##   * If MSG_DEP_SET isn't empty the following dependency has been pulled in
##     but can be declared for certainty nonetheless:
##     * add a run_depend tag for "message_runtime"
## * In this file (CMakeLists.txt):
##   * add "message_generation" and every package in MSG_DEP_SET to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * add "message_runtime" and every package in MSG_DEP_SET to
##     catkin_package(CATKIN_DEPENDS ...)
##   * uncomment the add_*_files sections below as needed
##     and list every .msg/.srv/.action file to be processed
##   * uncomment the generate_messages entry below
##   * add every package in MSG_DEP_SET to generate_messages(DEPENDENCIES ...)

## Generate messages in the 'msg' folder
# add_message_files(
#   FILES
#   Message1.msg
#   Message2.msg
# )

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate actions in the 'action' folder
# add_action_files(
#   FILES
#   Action1.action
#   Action2.action
# )

## Generate added messages and services with any dependencies listed here
# generate_messages(
#   DEPENDENCIES
#   std_msgs
# )

################################################
## Declare ROS dynamic reconfigure parameters ##
################################################

## To declare and build dynamic reconfigure parameters within this
## package, follow these steps:
## * In the file package.xml:
##   * add a build_depend and a run_depend tag for "dynamic_reconfigure"
## * In this file (CMakeLists.txt):
##   * add "dynamic_reconfigure" to
##     find_package(catkin REQUIRED COMPONENTS ...)
##   * uncomment the "generate_dynamic_reconfigure_options" section below
##     and list every .cfg file to be processed

## Generate dynamic reconfigure parameters in the 'cfg' folder
# generate_dynamic_reconfigure_options(
#   cfg/DynReconf1.cfg
#   cfg/DynReconf2.cfg
# )

###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## INCLUDE_DIRS: uncomment this if you package contains header files
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
#  INCLUDE_DIRS include
#  LIBRARIES lidar_package
#  CATKIN_DEPENDS roscpp rospy std_msgs
#  DEPENDS system_lib
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
 # include
  ${catkin_INCLUDE_DIRS}
)

## Declare a C++ library
# add_library(${PROJECT_NAME}
#   src/${PROJECT_NAME}/lidar_package.cpp
# )

## Add cmake target dependencies of the library
## as an example, code may need to be generated before libraries
## either from message generation or dynamic reconfigure
# add_dependencies(${PROJECT_NAME} ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Declare a C++ executable
## With catkin_make all packages are built within a single CMake context
## The recommended prefix ensures that target names across packages don't collide
# add_executable(${PROJECT_NAME}_node src/lidar_package_node.cpp)

## Rename C++ executable without prefix
## The above recommended prefix causes long target names, the following renames the
## target back to the shorter version for ease of user use
## e.g. "rosrun someones_pkg node" instead of "rosrun someones_pkg someones_pkg_node"
# set_target_properties(${PROJECT_NAME}_node PROPERTIES OUTPUT_NAME node PREFIX "")

## Add cmake target dependencies of the executable
## same as for the library above
# add_dependencies(${PROJECT_NAME}_node ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

## Specify libraries to link a library or executable target against
# target_link_libraries(${PROJECT_NAME}_node
#   ${catkin_LIBRARIES}
# )

#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_lidar_package.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)</pre>
```
<h1>Please contact us for any incorrect information and suggestions.</h1>
<h1>Do not forget !!! As information is shared, it multiplies.</h1>
<h2>Thanks a lot to Ahmet KAĞIZMAN for his help.</h2>
<p> </p>
