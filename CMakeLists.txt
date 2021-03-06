
#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.6 FATAL_ERROR)
project(neutron_propagation_simulation)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-variable -O0 -g -rdynamic -Wno-unused-variable")

#GTK setup stuff
FIND_PACKAGE(PkgConfig REQUIRED)
PKG_CHECK_MODULES(GTK REQUIRED gtk+-3.0)

#----------------------------------------------------------------------------
# Find Geant4 package, activating all available UI and Vis drivers by default
# You can set WITH_GEANT4_UIVIS to OFF via the command line or ccmake/cmake-gui
# to build a batch mode only executable
#
option(WITH_GEANT4_UIVIS "Build example with Geant4 UI and Vis drivers" ON)
if(WITH_GEANT4_UIVIS)
  find_package(Geant4 REQUIRED ui_all vis_all)
else()
  find_package(Geant4 REQUIRED)
endif()

#----------------------------------------------------------------------------
# Setup Geant4 include directories and compile definitions
# Setup include directory for this project
#
include(${Geant4_USE_FILE})
include_directories(${PROJECT_SOURCE_DIR}/include ${GTK_INCLUDE_DIRS})
LINK_DIRECTORIES(${GTK3_LIBRARY_DIRS})

ADD_DEFINITIONS(${GTK3_CFLAGS_OTHER})

#----------------------------------------------------------------------------
# Locate sources and headers for this project
# NB: headers are included so they will show up in IDEs
#
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)

#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#
add_executable(neutron_propagation neutron_propagation.cc ${sources} ${headers})
target_link_libraries(neutron_propagation ${Geant4_LIBRARIES} ${GTK_LIBRARIES})

#----------------------------------------------------------------------------
# Copy all scripts to the build directory, i.e. the directory in which we
# build B1. This is so that we can run the executable directly because it
# relies on these scripts being in the current working directory.
#
set(NEUTRON_SCRIPTS
    run1_thermal_neutrons.mac
    )

foreach(_script ${NEUTRON_SCRIPTS})
   configure_file(
     ${PROJECT_SOURCE_DIR}/${_script}
     ${PROJECT_BINARY_DIR}/${_script}
     COPYONLY
     )
endforeach()

ADD_DEFINITIONS("-Wno-unused-variable -Wno-unused-parameter")

#----------------------------------------------------------------------------
# For internal Geant4 use - but has no effect if you build this
# example standalone
#
add_custom_target(NEUTRON_PROPAGATION DEPENDS neutron_propagation)

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX/
#
install(TARGETS neutron_propagation DESTINATION bin)


