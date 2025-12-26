cmake_minimum_required(VERSION 3.8)
project(message_logger)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -fPIC)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)

# Use std::cout by default for ROS2
option(MELO_USE_COUT "Use std::cout" ON)

if(DEFINED MELO_FUNCTION_PRINTS)
  add_definitions(-DMELO_FUNCTION_PRINTS)
endif()

# Configure header file
configure_file(
  include/message_logger/log/log_messages_backend_config.hpp.in
  ${CMAKE_CURRENT_BINARY_DIR}/include/message_logger/log/log_messages_backend_config.hpp
  @ONLY
)

# Library
add_library(${PROJECT_NAME} SHARED
  src/time/Time.cpp
  src/time/TimeStd.cpp
)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  $<INSTALL_INTERFACE:include>
)

if(NOT MELO_USE_COUT)
  ament_target_dependencies(${PROJECT_NAME} rclcpp)
endif()

# Install
install(TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY include/
  DESTINATION include
  PATTERN "*.in" EXCLUDE
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/message_logger/log/log_messages_backend_config.hpp
  DESTINATION include/message_logger/log
)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)
ament_export_dependencies(rclcpp)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
