cmake_minimum_required(VERSION 3.13.5)
project(voxgraph)

add_compile_options(-Wpedantic -Wextra)
# add_compile_options(-Werror)
# add_compile_options(-Wall)

# set to c++11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# set to debug mode if not specified
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

# print compiler being used
if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  MESSAGE("Clang")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  MESSAGE("GNU")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")
  MESSAGE("Intel")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  MESSAGE("MSVC")
endif()


find_package(catkin REQUIRED COMPONENTS roscpp)
find_package(Ceres REQUIRED)
find_package(glog REQUIRED)
find_package(gflags REQUIRED)
find_package(Protobuf REQUIRED)
find_package(cblox REQUIRED)
find_package(cblox_ros REQUIRED)
find_package(voxblox REQUIRED)
find_package(voxblox_ros REQUIRED)

#############
# LIBRARIES #
#############
# NOTE: The library is static to avoid segfaults that
#       otherwise occur in voxblox multi threading
set(VOXG_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/voxgraph_mapper.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/pose_graph_interface/pose_graph_interface.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/pose_graph_interface/node_templates.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/pose_graph_interface/measurement_templates.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/measurement_processors/pointcloud_integrator.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/submap_collection/bounding_box.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/submap_collection/submap_timeline.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/submap_collection/voxgraph_submap.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/submap_collection/voxgraph_submap_collection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/map_tracker/map_tracker.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/map_tracker/transformers/tf_transformer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/frontend/map_tracker/transformers/odometry_transformer.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/pose_graph.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/node/node.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/node/node_collection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/node/pose/pose_4d.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/node/pose/pose_6d.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/constraint.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/constraint_collection.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/relative_pose_constraint.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/absolute_pose_constraint.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/registration_constraint.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/backend/constraint/cost_functions/registration_cost_function.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/rosbag_helper.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/tf_helper.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/submap_registration_helper.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/evaluation/map_evaluation.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/data_servers/projected_map_server.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/data_servers/submap_server.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/data_servers/loop_closure_edge_server.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/odometry_simulator/odometry_simulator.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/visualization/cost_function_visuals.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/visualization/pose_graph_visuals.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/visualization/submap_visuals.cpp
  ${CMAKE_CURRENT_SOURCE_DIR}/src/tools/visualization/loop_closure_visuals.cpp
  )
set(VOXG_HDRS
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/ros_params.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/threading_helper.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/evaluation/map_evaluation.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/tf_helper.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/odometry_simulator/normal_distribution.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/odometry_simulator/odometry_simulator.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/io.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/data_servers/submap_server.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/data_servers/projected_map_server.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/data_servers/loop_closure_edge_server.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/submap_registration_helper.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/visualization/loop_closure_visuals.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/visualization/pose_graph_visuals.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/visualization/cost_function_visuals.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/visualization/submap_visuals.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/rosbag_helper.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/common.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/voxgraph_submap.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/bounding_box.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/weighted_sampler.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/submap_timeline.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/weighted_sampler_inl.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/registration_point.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection/voxgraph_submap_collection.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/pose_graph_interface/measurement_templates.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/pose_graph_interface/node_templates.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/pose_graph_interface/pose_graph_interface.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/map_tracker/transformers/tf_transformer.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/map_tracker/transformers/odometry_transformer.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/map_tracker/map_tracker.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/measurement_processors/gps_processor.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/measurement_processors/pointcloud_integrator.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/voxgraph_mapper.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/frame_names.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/local_parameterization/normalize_angle.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/local_parameterization/angle_local_parameterization.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/node.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/reference_frame_node.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/submap_node.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/pose/pose_4d.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/pose/pose_6d.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/pose/pose.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/node_collection.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/constraint.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/relative_pose_constraint.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/constraint_collection.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/absolute_pose_constraint.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/registration_constraint.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/cost_functions/registration_cost_function.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/cost_functions/relative_pose_cost_function.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/cost_functions/relative_pose_cost_function_inl.h
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/pose_graph.h
  )
set(VOXG_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/odometry_simulator
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/data_servers
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/tools/visualization
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/submap_collection
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/pose_graph_interface
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/map_tracker
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/map_tracker/transformers
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/frontend/measurement_processors
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/local_parameterization
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/node/pose
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint
  ${CMAKE_CURRENT_SOURCE_DIR}/include/voxgraph/backend/constraint/cost_functions
  )

add_library(voxg-lib STATIC
  ${VOXG_HDRS}
  ${VOXG_SRCS}
  )

target_include_directories(voxg-lib
  PUBLIC
  ${CERES_INCLUDE_DIRS}
  ${VOXG_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${cblox_INCLUDE_DIRS}
  ${cblox_ros_INCLUDE_DIRS}
  ${voxblox_INCLUDE_DIRS}
  ${voxblox_ros_INCLUDE_DIRS}
  PRIVATE
  )

target_link_libraries(voxg-lib
  PUBLIC
  ${CATKIN_LIBRARIES}
  ${CERES_LIBRARIES}
  ${cblox_LIBRARIES}
  ${cblox_ros_LIBRARIES}
  ${voxblox_LIBRARIES}
  ${voxblox_ros_LIBRARIES}
  PRIVATE
  )

# get_target_property(OUT voxg-lib INCLUDE_DIRECTORIES)
message(STATUS "PRINTING")
# message(STATUS ${voxblox_ros_INCLUDE_DIRS})
# message(STATUS ${voxblox_ros_LIBRARIES})
message(STATUS "PRINTING")

############
# BINARIES #
############
add_executable(odometry_simulator
  src/odometry_simulator_node.cpp)
target_link_libraries(odometry_simulator PUBLIC voxg-lib ${PROTOBUF_LIBRARIES} glog gflags)

add_executable(registration_test_bench
  src/tools/test_benches/registration_test_bench.cpp)
target_link_libraries(registration_test_bench PUBLIC voxg-lib ${PROTOBUF_LIBRARIES} gflags)

add_executable(voxgraph_mapping_node
  src/voxgraph_mapping_node.cpp)
target_link_libraries(voxgraph_mapping_node PUBLIC voxg-lib ${PROTOBUF_LIBRARIES} glog gflags)


# EXPORT #
##########
# export(TARGETS voxgraph_mapping_node odometry_simulator registration_test_bench)

####################
# GIT VERSION INFO #
####################
# Make the git branch name and git commit hash available in the sources as
# a #define, such that it can easily be appended to logs

IF(EXISTS ${voxgraph_SOURCE_DIR}/../.git)
  find_package(Git)
  IF(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} rev-parse --abbrev-ref HEAD
        WORKING_DIRECTORY "${voxgraph_SOURCE_DIR}"
        OUTPUT_VARIABLE "voxgraph_GIT_BRANCH"
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} log -1 --format=%h
        WORKING_DIRECTORY "${voxgraph_SOURCE_DIR}"
        OUTPUT_VARIABLE "voxgraph_GIT_COMMIT_HASH"
        ERROR_QUIET
        OUTPUT_STRIP_TRAILING_WHITESPACE)
      message(STATUS "Defining GIT_BRANCH: ${voxgraph_GIT_BRANCH}")
      message(STATUS "Defining GIT_COMMIT_HASH: ${voxgraph_GIT_COMMIT_HASH}")
  ELSE(GIT_FOUND)
    set(voxgraph_GIT_BRANCH 0)
    set(voxgraph_GIT_COMMIT_HASH 0)
  ENDIF(GIT_FOUND)
ENDIF(EXISTS ${voxgraph_SOURCE_DIR}/../.git)
add_definitions("-DGIT_BRANCH=\"${voxgraph_GIT_BRANCH}\"")
add_definitions("-DGIT_COMMIT_HASH=\"${voxgraph_GIT_COMMIT_HASH}\"")
