cmake_minimum_required(VERSION 2.8.9 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME lodepng)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# lodepng
#------------------------------------------------------------------------------
# Include CMake external project module.
include(ExternalProject)

set(REPO_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/glm")
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${REPO_SOURCE_DIR}
  SVN_REPOSITORY "http://lpi.googlecode.com/svn/trunk"
  SVN_REVISION "HEAD"
  )
CPM_ExportAdditionalIncludeDir("${REPO_SOURCE_DIR}")

# Specify include dir
ExternalProject_Get_Property(${EP_PROJECT_NAME} source_dir)
include_directories(${source_dir}/include)
CPM_ExportAdditionalIncludeDir("${source_dir}/include")

# Copy source code into an include directory that is unlikely to conflict
# with others' include directories.
file(COPY ${source_dir}/trunk/
     DESTINATION ${source_dir}/lodepng
     FILE_PERMISSIONS OWNER_READ OWNER_WRITE GROUP_READ WORLD_READ
     DIRECTORY_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
                           GROUP_READ
                           WORLD_READ WORLD_EXECUTE)

# Now construct a library that others can include in their project.


