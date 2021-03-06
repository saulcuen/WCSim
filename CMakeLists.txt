## Based on CMakeLists.txt from AnaEx02 in examples/extended of Geant4.9.6 ##
#----------------------------------------------------------------------------
# Setup the project
cmake_minimum_required(VERSION 2.8.10 FATAL_ERROR)
project(WCSim)

if (CMAKE_COMPILER_IS_GNUCC AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.4)
    message(STATUS "GCC version >= 4.4 required!")
    return()
endif()

#----------------------------------------------------------------------------
# Find ROOT (required package) in CONFIG mode. Looking for ROOTConfig.cmake.
# Crucial for loading the proper definitions!
#
find_package(ROOT CONFIG REQUIRED)
if(NOT ROOT_FOUND)
  message(STATUS "ROOT package not found.") 
  return()
endif()

## Load macros: need to compile ROOT through cmake first and execute bin/thisroot.sh
include(${ROOT_USE_FILE})

#----------------------------------------------------------------------------
# Locate sources and headers for this project
# ## NOT NEEDED FOR DICT
include_directories(${PROJECT_SOURCE_DIR}/include 
                    ${PROJECT_SOURCE_DIR}/../shared/include 
                    ${Geant4_INCLUDE_DIR}
                    ${ROOT_INCLUDE_DIR})

#----------------------------------------------------------------------------
# Add libraries: need to compile the Dict before linking WCSim !!
# in standard makefile, need to make rootcint anyway before standard make
#

## WCSimRootDict.cc regeneration by rootcint
## Use ROOT 5.34.32 as some issues with PARSE_ARGUMENTS were found in older ROOT versions (ROOT 5.34.11)
ROOT_GENERATE_DICTIONARY(${CMAKE_CURRENT_SOURCE_DIR}/src/WCSimRootDict ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimRootEvent.hh ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimRootGeom.hh ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimPmtInfo.hh ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimEnumerations.hh  ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimRootOptions.hh ${CMAKE_CURRENT_SOURCE_DIR}/include/TJNuBeamFlux.hh ${CMAKE_CURRENT_SOURCE_DIR}/include/TNRooTrackerVtx.hh LINKDEF ${CMAKE_CURRENT_SOURCE_DIR}/include/WCSimRootLinkDef.hh)


## Crucial for reading ROOT classes: make shared object library
add_library(WCSimRoot SHARED ./src/WCSimRootEvent.cc ./src/WCSimRootGeom.cc ./src/WCSimPmtInfo.cc ./src/WCSimEnumerations.cc ./src/WCSimRootOptions.cc ./src/TJNuBeamFlux.cc ./src/TNRooTrackerVtx.cc ./src/WCSimRootDict.cxx)
target_link_libraries(WCSimRoot  ${ROOT_LIBRARIES})



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
include(${Geant4_USE_FILE})  ## NOT needed for Dict


#----------------------------------------------------------------------------
# Locate sources and headers for this project
# NB: headers are included so they will show up in IDEs
#
file(GLOB sources ${PROJECT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hh)


#----------------------------------------------------------------------------
# Add the executable, and link it to the Geant4 libraries
#

# To use <unordered_map>, ie. use C++11 now from GCC 4.4 onwards support for unordered_maps.
# Had to change G4float to G4double for QE because of double arithmatic inside the array.
if (CMAKE_COMPILER_IS_GNUCC AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 4.7)
    set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -std=c++11")
else()
    set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -std=c++0x")
endif()



add_executable(WCSim WCSim.cc ${sources} ${headers})
target_link_libraries(WCSim ${Geant4_LIBRARIES} ${ROOT_LIBRARIES} WCSimRoot Tree)  #add profiler to use gperftools



#----------------------------------------------------------------------------
# Copy all scripts to the build directory, i.e. the directory in which we
# build WCSim. This is so that we can run the executable directly because it
# relies on these scripts being in the current working directory.
#
set(WCSIM_SCRIPTS
  macros/jobOptions.mac
  WCSim.mac
  macros/daq.mac
  macros/visOGLSX.mac
  macros/visOGLQT.mac
  macros/visRayTracer.mac
  macros/mPMT.mac
  macros/mPMT_nuPrism1.mac
  macros/mPMT_nuPrism2.mac
  macros/tuning_parameters.mac
  macros/NuPRISM.mac
  )

foreach(_script ${WCSIM_SCRIPTS})
  configure_file(
    ${PROJECT_SOURCE_DIR}/${_script}
    ${PROJECT_BINARY_DIR}/${_script}
    COPYONLY
    )
endforeach()

#----------------------------------------------------------------------------
# For internal Geant4 use - but has no effect if you build this
# example standalone
#
#add_custom_target(WCSim DEPENDS WCSim)

#----------------------------------------------------------------------------
# Install the executable to 'bin' directory under CMAKE_INSTALL_PREFIX
#
#install(TARGETS WCSim DESTINATION bin)


