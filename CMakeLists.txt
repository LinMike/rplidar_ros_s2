cmake_minimum_required(VERSION 2.8.3)
project(rplidar_ros_s2)

set(RPLIDAR_SDK_PATH "./sdk/")
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")

FILE(GLOB RPLIDAR_SDK_SRC 
  "${RPLIDAR_SDK_PATH}/src/arch/linux/*.cpp"
  "${RPLIDAR_SDK_PATH}/src/hal/*.cpp"
  "${RPLIDAR_SDK_PATH}/src/*.cpp"
)

find_package(catkin REQUIRED COMPONENTS
  roscpp
  rosconsole
  sensor_msgs
)

include_directories(
  ${RPLIDAR_SDK_PATH}/include
  ${RPLIDAR_SDK_PATH}/src
  ${catkin_INCLUDE_DIRS}
)

catkin_package()

add_executable(rplidarNodeClient src/client.cpp)
target_link_libraries(rplidarNodeClient ${catkin_LIBRARIES})

# catkin_make --pkg rplidar_ros_s2 -DBUILD_NODELET=ON
option(BUILD_NODELET "BUILD RPLIDAR NODELET INSTEAD OF RPLIDAR NODE" ON)
if(BUILD_NODELET)
  message(STATUS "BUILDING RPLIDAR NODE WITH NODELET ${BUILD_NODELET}")
  add_definitions(-DBUILD_NODELET=1) # #if BUILD_NODELET in node.cpp
  add_library(rplidarNodelet src/node.cpp ${RPLIDAR_SDK_SRC})
  target_link_libraries(rplidarNodelet ${catkin_LIBRARIES})

  install(TARGETS rplidarNodelet rplidarNodeClient
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )
  # 将插件的xml文件安装到devel/share目录下，类加载器找到xml才能加载类插件
  # file(COPY "rplidar_nodelet.xml" DESTINATION ${CATKIN_DEVEL_PREFIX})
  install(FILES rplidar_nodelet.xml
    DESTINATION ${CATKIN_DEVEL_PREFIX}
  )
  
else()
  message(STATUS "BUILDING RPLIDAR NODE WITHOUT NODELET ${BUILD_NODELET}")
  remove_definitions(-DBUILD_NODELET)
  add_executable(rplidarNode src/node.cpp ${RPLIDAR_SDK_SRC})
  target_link_libraries(rplidarNode ${catkin_LIBRARIES})

  install(TARGETS rplidarNode rplidarNodeClient
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )
endif(BUILD_NODELET)



install(DIRECTORY launch rviz sdk
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  USE_SOURCE_PERMISSIONS
)
