cmake_minimum_required(VERSION 3.1)

# Useful commands 
set(INVOKE_CMAKE_SCRIPT ${CMAKE_COMMAND} -P) 
set(COPY_DIFF ${CMAKE_COMMAND} -E copy_if_different) 
set(COPY_DIR ${CMAKE_COMMAND} -E copy_directory) 
set(REMOVE_DIR ${CMAKE_COMMAND} -E remove_directory)
set(REMOVE ${CMAKE_COMMAND} -E remove -f)
set(ENV_CMD ${CMAKE_COMMAND} -E env ${EXTRA_ARG})

if("${CMAKE_CURRENT_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")

 # It's expected a LOCAL_SOURCE_DIR=<valid path> commandline parameter
 set(TEMPLATE_DIR "${LOCAL_SOURCE_DIR}/template")
 set(SRC_DIR "${LOCAL_SOURCE_DIR}/src")
 set(INFO_YAML "${LOCAL_SOURCE_DIR}/../info.yaml")
else()

 set(TEMPLATE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
 set(SRC_DIR "${CMAKE_SOURCE_DIR}/src")
 set(INFO_YAML "${CMAKE_SOURCE_DIR}/../info.yaml")
endif()

## Working directories ##
set(GIT_DIR "${CMAKE_SOURCE_DIR}")

if(VERSION_HARDCODED)
   set(VERSION_IN "${TEMPLATE_DIR}/version_hardcoded.h.in")
else()
   set(VERSION_IN "${TEMPLATE_DIR}/version.h.in")
endif()
set(VERSION_OUT "${SRC_DIR}/version.h") 

## System Info & escape string 
if(WIN32)
 set(GIT_SYSTEM "$ENV{USERNAME}@$ENV{COMPUTERNAME}")
 
else()
 site_name(CMAKE_HOSTNAME)
 set(GIT_SYSTEM "$ENV{USER}@${CMAKE_HOSTNAME}")

endif(WIN32)

if(VERSION_HARDCODED)

  ### FAKE info ###
  set(CMAKE_VERSION_FIRST "0")
  set(CMAKE_VERSION_MIDDLE "0")
  set(CMAKE_VERSION_LAST "0")
  set(CMAKE_GIT_DETAILS "0123456 (HEAD, origin/develop, origin/HEAD, develop) FAKE: hardcoded values without using GIT")
  set(CMAKE_GIT_COMMIT_HASH "0123456")

  message(STATUS "Using harcoded values, i.e. 0123456, at generated version.h")

 else(VERSION_HARDCODED)

 ### Read values from global info.yaml ####
 file(READ "${INFO_YAML}" INFO)
 string(REGEX REPLACE ";" "\\\\;" INFO "${INFO}")
 string(REGEX REPLACE "\n" ";" INFO "${INFO}")
 foreach(LINE "${INFO}")

   string(REGEX MATCH "[ ]*VERSION_CORE: [0-9]*\.[0-9]*\.[0-9]*" VERSION_LINE "${LINE}")
   if(DEFINED VERSION_LINE)
    string(STRIP "${VERSION_LINE}" VERSION_LINE)
    message(STATUS "${VERSION_LINE}")
    string(REGEX REPLACE "VERSION_CORE: \([0-9]*\).*" "\\1" FIRST_DIGIT "${VERSION_LINE}")
    string(REGEX REPLACE "VERSION_CORE: [0-9]*\.\([0-9]*\).*" "\\1" MIDDLE_DIGIT "${VERSION_LINE}")
    string(REGEX REPLACE "VERSION_CORE: [0-9]*\.[0-9]*\.\([0-9]*\)" "\\1" LAST_DIGIT "${VERSION_LINE}")
   else()
    message(FATAL_ERROR "VERSION not found at ${INFO_YAML}")
   endif()

 endforeach(LINE "${INFO}")

 ### Human Info ###
 set(CMAKE_VERSION_FIRST "${FIRST_DIGIT}")
 set(CMAKE_VERSION_MIDDLE "${MIDDLE_DIGIT}")
 set(CMAKE_VERSION_LAST "${LAST_DIGIT}")

 #### Git Info ####
 execute_process(
  COMMAND git log -1 --format=%h 
  WORKING_DIRECTORY ${GIT_DIR} 
  OUTPUT_VARIABLE CMAKE_GIT_COMMIT_HASH
  OUTPUT_STRIP_TRAILING_WHITESPACE
 )
 execute_process(
  COMMAND git log --oneline --decorate -1
  WORKING_DIRECTORY ${GIT_DIR} 
  OUTPUT_VARIABLE CMAKE_GIT_DETAILS
  OUTPUT_STRIP_TRAILING_WHITESPACE
 )

 message(STATUS "Using GIT values at generated version.h")
endif(VERSION_HARDCODED)

# Extra variables to be used in those templates

## WARNING: LaTeX special characteres, as 'underscore', must be avoided
string(REGEX REPLACE "_" "-" CMAKE_GIT_DETAILS ${CMAKE_GIT_DETAILS})
string(REGEX REPLACE "&" " and " CMAKE_GIT_DETAILS ${CMAKE_GIT_DETAILS})
string(REGEX REPLACE "_" "-" GIT_SYSTEM ${GIT_SYSTEM})
string(REGEX REPLACE "&" " and " GIT_SYSTEM ${GIT_SYSTEM})

configure_file(${VERSION_IN} ${VERSION_OUT})

message(STATUS "GIT_DETAILS=\"${CMAKE_GIT_DETAILS}\"")
message(STATUS "GIT_SYSTEM=\"${GIT_SYSTEM}\"")

