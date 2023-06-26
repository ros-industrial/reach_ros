cmake_minimum_required(VERSION 3.10.0)

# Extract package name and version from package.xml
find_package(ros_industrial_cmake_boilerplate REQUIRED)
extract_package_metadata(pkg)
project(${pkg_extracted_name} VERSION ${pkg_extracted_version} LANGUAGES CXX)

find_package(reach REQUIRED)

find_package(
  catkin REQUIRED
  COMPONENTS tf2_eigen
             interactive_markers
             moveit_core
             moveit_msgs
             moveit_ros_planning_interface
             sensor_msgs
             visualization_msgs)

catkin_package(
  INCLUDE_DIRS
  include
  LIBRARIES
  ${PROJECT_NAME}
  CATKIN_DEPENDS
  tf2_eigen
  interactive_markers
  moveit_core
  moveit_msgs
  moveit_ros_planning_interface
  sensor_msgs
  visualization_msgs)

# ######################################################################################################################
# BUILD ##
# ######################################################################################################################

include_directories(${catkin_INCLUDE_DIRS} include)

# Plugin Implementations
add_library(
  ${PROJECT_NAME}
  src/utils.cpp
  # Evaluator
  src/evaluation/manipulability_moveit.cpp
  src/evaluation/joint_penalty_moveit.cpp
  src/evaluation/distance_penalty_moveit.cpp
  # IK Solver
  src/ik/moveit_ik_solver.cpp
  # Display
  src/display/ros_display.cpp
  # Target pose generator
  src/target_pose_generator/transformed_point_cloud_target_pose_generator.cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} reach::reach)

# Plugin Library
add_library(${PROJECT_NAME}_plugins SHARED src/plugins.cpp)
target_link_libraries(${PROJECT_NAME}_plugins ${PROJECT_NAME})
target_cxx_version(${PROJECT_NAME}_plugins PUBLIC VERSION 14)

# Reach study node
add_executable(${PROJECT_NAME}_node src/reach_study_node.cpp)
target_link_libraries(
  ${PROJECT_NAME}_node
  ${catkin_LIBRARIES}
  yaml-cpp
  reach::reach)
target_cxx_version(${PROJECT_NAME}_node PUBLIC VERSION 14)

# Demo
add_subdirectory(demo)

# ######################################################################################################################
# INSTALL ##
# ######################################################################################################################

install(
  TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_plugins ${PROJECT_NAME}_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/ DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})

install(DIRECTORY launch demo DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
