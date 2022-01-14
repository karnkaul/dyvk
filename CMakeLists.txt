cmake_minimum_required(VERSION 3.14)
set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "")
set(CMAKE_DEBUG_POSTFIX "-d")


# project parameters
set(project_name dyvk)
set(${project_name}_version 1.0.1)

project(${project_name} VERSION "${${project_name}_version}")
set(${project_name}_soversion ${PROJECT_VERSION_MAJOR})

# cmake-utils
include(FetchContent)
set(FETCHCONTENT_QUIET OFF)
FetchContent_Declare(
  cmake-utils
  GIT_REPOSITORY https://github.com/karnkaul/cmake-utils
  GIT_TAG v1.0.1
)
FetchContent_MakeAvailable(cmake-utils)
FetchContent_GetProperties(cmake-utils)

set(${PROJECT_NAME}_soversion ${PROJECT_VERSION_MAJOR})
add_library(${PROJECT_NAME})
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
set_property(TARGET ${PROJECT_NAME} PROPERTY VERSION ${${PROJECT_NAME}_version})
set_property(TARGET ${PROJECT_NAME} PROPERTY SOVERSION ${${PROJECT_NAME}_soversion})
set_property(TARGET ${PROJECT_NAME} PROPERTY INTERFACE_${PROJECT_NAME}_MAJOR_VERSION ${${PROJECT_NAME}_soversion})
set_property(TARGET ${PROJECT_NAME} APPEND PROPERTY COMPATIBLE_INTERFACE_STRING ${PROJECT_NAME}_MAJOR_VERSION)

target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC 
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
if(CMAKE_SYSTEM_NAME STREQUAL Linux)
  target_link_libraries(${PROJECT_NAME} PRIVATE dl)
endif()

target_compile_definitions(${PROJECT_NAME} PUBLIC VK_NO_PROTOTYPES)
target_sources(${PROJECT_NAME} PRIVATE 
  include/vulkan/vulkan.h
  include/vulkan/vulkan.hpp
  src/impl.cpp
)
get_target_property(sources ${PROJECT_NAME} SOURCES)
source_group(TREE "${CMAKE_CURRENT_SOURCE_DIR}" FILES ${sources})

if(DYVK_INSTALL)
  include("${cmake-utils_SOURCE_DIR}/cmake-utils.cmake")
  install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h*"
  )
  install_and_export_target(TARGET ${PROJECT_NAME})
endif()
