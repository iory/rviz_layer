# these plugins should be uploaded to upstream repository
# https://github.com/ros-visualization/rviz/pull/634

# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(rviz_layer)

add_definitions(-std=c++11 -march=native)

# Load catkin and all dependencies required for this package
# TODO: remove all from COMPONENTS that are not catkin packages.
find_package(catkin REQUIRED COMPONENTS
  rviz
  jsk_recognition_msgs
  jsk_topic_tools
  jsk_rviz_plugins
  cv_bridge)

# Use ccache if installed to make it fast to generate object files
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE ccache)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK ccache)
endif(CCACHE_FOUND)

catkin_package(
    DEPENDS rviz
    CATKIN_DEPENDS rviz
    jsk_recognition_utils cv_bridge jsk_topic_tools jsk_rviz_plugins
    INCLUDE_DIRS
    LIBRARIES
)

message(" catkin_LIBRARIES : " ${catkin_LIBRARIES})
include_directories(include ${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS})

if(rviz_QT_VERSION VERSION_LESS "5")
  find_package(Qt4 COMPONENTS QtCore QtGui REQUIRED)
  include(${QT_USE_FILE})
else()
  find_package(Qt5Widgets REQUIRED)
endif()
add_definitions(-DQT_NO_KEYWORDS -g)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(HEADER_FILES
  include/rviz_layer/pictogram_array_display.h
)

if(rviz_QT_VERSION VERSION_LESS "5")
  qt4_wrap_cpp(MOC_FILES ${HEADER_FILES})
else()
  qt5_wrap_cpp(MOC_FILES ${HEADER_FILES})
endif()

set(SOURCE_FILES
  src/pictogram_array_display.cpp
  ${MOC_FILES}
  )

link_directories(${catkin_LIBRARY_DIRS})

add_library(rviz_layer ${SOURCE_FILES} ${UIC_FILES})

if(rviz_QT_VERSION VERSION_LESS "5")
  target_link_libraries(rviz_layer ${QT_LIBRARIES} ${catkin_LIBRARIES} ${rviz_DEFAULT_PLUGIN_LIBRARIES})
else()
  target_link_libraries(rviz_layer Qt5::Widgets ${catkin_LIBRARIES} ${rviz_DEFAULT_PLUGIN_LIBRARIES})
endif()

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  set_target_properties(rviz_layer PROPERTIES LINK_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -z defs")
endif()

install(FILES plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
install(TARGETS rviz_layer
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})

install(DIRECTORY cfg icons launch samples scripts
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  USE_SOURCE_PERMISSIONS)
install(DIRECTORY ./src/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")
