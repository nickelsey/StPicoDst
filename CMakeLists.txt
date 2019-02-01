cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)
cmake_policy(SET CMP0042 NEW)
cmake_policy(SET CMP0048 NEW)

# project title
project(StPicoDst VERSION 0.1.0)

set(CMAKE_BUILD_TYPE RELEASE)

# options
option(BUILD_SHARED_LIBS "Build shared libs" ON)
option(PICO_SL18B "Build the SL18b version of StPicoDst" ON)
option(PICO_SL18H "Build the SL18h version of StPicoDst" OFF)
option(PICO_INSTALL_HEADERS "Install StPicoEvent headers" ON)
if(PICO_SL18H)
	set(PICO_SL18B OFF)
endif(PICO_SL18H)

# compiler & system checks
include(cmake/SysCheck.cmake)

# add module path to cmake
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake/Modules)

# helper functions
include(cmake/Utils.cmake)

# dependencies
include(cmake/Dependencies.cmake)

# build options
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O2 -fPIC")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-narrowing")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11")

# add -D_VANILLA_ROOT_ to CMAKE_CXX_FLAGS. Necessary to turn off all StRoot 
# dependencies
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_VANILLA_ROOT_")


# start the build - define source file and header variables
set(PICO_SRCS)
set(PICO_HDRS)
set(PICO_LINKDEF)

# finally, add the source tree - dependent on library version
if (PICO_SL18H)
	add_subdirectory(SL18H/StPicoEvent)
	set(PICO_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/SL18H)
	include_directories(${PICO_INCLUDE})	
elseif (PICO_SL18B)
	add_subdirectory(SL18B/StPicoEvent)
	set(PICO_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/SL18B)
	include_directories(${PICO_INCLUDE})	
else(PICO_SL18H)
	message(FATAL "No library choice specified for StPicoDst - either BUILD_SL18H or SL18B")
endif(PICO_SL18H)

# use ROOT to generate the dictionary file
set(PICO_DICT_NAME StPicoDst_dict)
set(PICO_DICT_TARGET StPicoDst_dict.cxx)
ROOT_GENERATE_DICTIONARY(${PICO_DICT_NAME} ${PICO_HDRS} LINKDEF ${PICO_LINKDEF}
                         OPTIONS "-D_VANILLA_ROOT_")

# build target library
add_library(StPicoDst ${PICO_SRCS} ${PICO_DICT_TARGET})
target_link_libraries(StPicoDst PRIVATE ${PICO_DEPENDENCY_LIBS})
target_include_directories(StPicoDst INTERFACE $<INSTALL_INTERFACE:include>)
target_compile_options(StPicoDst INTERFACE "--std=c++11")
install(TARGETS StPicoDst EXPORT StPicoDst DESTINATION lib)
install(EXPORT StPicoDst NAMESPACE pico_ DESTINATION lib/StPicoDst)
link_directories(${CMAKE_INSTALL_PREFIX}/lib)

set (PICO_LIBRARIES StPicoDst)

# add an uninstall option, since cmake does not provide one by default
if(NOT TARGET uninstall)
  configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in
                 ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
                 IMMEDIATE
                 @ONLY)

  add_custom_target(uninstall
                    COMMAND ${CMAKE_COMMAND} -P
                            ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif()

# export the analysis variables if not root directory
if(NOT CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
	set(PICO_BUILD_SL18H ${PICO_SL18H} PARENT_SCOPE)
	set(PICO_BUILD_SL18B ${PICO_SL18B} PARENT_SCOPE)
	set(PICO_INCLUDE_DIRS ${PICO_INCLUDE} ${ROOT_INCLUDE_DIRS} PARENT_SCOPE)
	set(PICO_LIBS ${PICO_LIBRARIES} PARENT_SCOPE)
  set(PICO_DEPENDENCY_LIBS ${PICO_DEPENDENCY_LIBS} PARENT_SCOPE)
endif(NOT CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)