cmake_minimum_required(VERSION 2.8.3)
project(imagesender)

find_package(catkin REQUIRED COMPONENTS
)
catkin_package(
  CATKIN_DEPENDS 
	rospy
	OpenCV
	cv_bridge
	sensor_msgs
	std_msgs
)
