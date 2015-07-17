cmake_minimum_required(VERSION 3.1)

############# PROJECT ####################

message(STATUS "Binary & Documentation project")

if(APPLE)
  ### Use OSX Homebrew gcc & boost
  message(STATUS "CMAKE_CXX_COMPILER should be \"/usr/local/Cellar/gcc/5.1.0/bin/g++-5\" or similar")
  set(EXTRA_ARG "PATH=\"$ENV{PATH}:/usr/texbin\"") # pdflatex
 
elseif(UNIX)
  ### Risky bet because it could be Solaris
  message(STATUS "Taking from granted this is a Linux box with evince installed")
  
elseif(WIN32)
  ### Supposed BOOST_ROOT, BOOST_INCLUDE & BOOST_LIB defined, i.e. C:\MinGW, C:\MinGW\include && C:\MinGW\lib 
  message(STATUS "CMAKE_CXX_COMPILER should be \"C:/MinGW/bin/g++.exe\" or similar")

  ### Windows default make not good enough
  if(NOT ${CMAKE_GENERATOR} MATCHES "MSYS Makefiles") # another similar option "MinGW Makefiles"
    message(WARNING "Expected MinGW generator: cmake -G \"MSYS Makefiles\" ..")
  endif()

else()
  ### Just in case
  message(FATAL_ERROR "O.S. ${CMAKE_SYSTEM} NOT SUPPORTED")

endif(APPLE)

#### C++ Project Name ###
project(GenealogicalTree CXX)

### Modern C++, GNU compiler ###
message(STATUS "CMAKE_CXX_COMPILER = ${CMAKE_CXX_COMPILER}")
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    # require at least gcc 4.9.2 (some C++14 features)
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9.2)
        message(FATAL_ERROR "GCC version must be at least 4.9.2!")
    endif()
else()
    message(FATAL_ERROR "You are using an unsupported compiler! Compilation has only been tested with g++.")
endif()

########## src ###################
add_subdirectory( src )

