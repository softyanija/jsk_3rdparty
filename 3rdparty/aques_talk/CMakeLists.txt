cmake_minimum_required(VERSION 2.8.3)
project(aques_talk)

find_package(catkin REQUIRED)

catkin_package()

# Considering the use of docker in addition to the usual use,
# we use dpkg-architecture command instead of ${CMAKE_SYSTEM_PROCESSOR} to get the architecture.
# https://stackoverflow.com/a/58222507
# http://manpages.ubuntu.com/manpages/bionic/man1/dpkg-architecture.1.html
execute_process(COMMAND dpkg-architecture -qDEB_HOST_GNU_CPU OUTPUT_VARIABLE ARCHITECTURE)
if("${ARCHITECTURE}" MATCHES x86_64* )
set(AQTK2_LNX_LIB_DIR "lib64")
else ()
set(AQTK2_LNX_LIB_DIR "lib")
endif ()

#
include(ExternalProject)
externalproject_add(aqtk2-lnx-eva
  PREFIX download
  URL https://www.a-quest.com/archive/package/aqtk2-lnx-eva_230.zip
  TIMEOUT 120
  CONFIGURE_COMMAND bash -c "echo configure"
  BUILD_COMMAND bash -c "(cd ${AQTK2_LNX_LIB_DIR} && ln -sf libAquesTalk2Eva.so.2.3 libAquesTalk2.so && ln -sf libAquesTalk2Eva.so.2.3 libAquesTalk2.so.2)"
  INSTALL_COMMAND bash -c "cp -P ${AQTK2_LNX_LIB_DIR}/libAquesTalk2* ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION} && cp phont/* ${CMAKE_CURRENT_SOURCE_DIR}/phont/"
  BUILD_IN_SOURCE 1
)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/download/src/aqtk2-lnx-eva/${AQTK2_LNX_LIB_DIR})
link_directories(${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_LIB_DESTINATION})
# compile SampleTalk
add_executable(SampleTalk src/SampleTalk.c)
target_link_libraries(SampleTalk AquesTalk2)

# Make compilation depend on our target:
add_dependencies(SampleTalk aqtk2-lnx-eva)

# install
install(FILES text2wave
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(DIRECTORY launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  USE_SOURCE_PERMISSIONS)



