# Copyright 2022 DeepMind Technologies Limited
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

add_library(threadpool STATIC)
target_sources(
  threadpool
  PUBLIC threadpool.h
  PRIVATE threadpool.cc
)
target_link_libraries(
  threadpool
  absl::base
)
target_include_directories(threadpool PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_library(
  libmjpc STATIC
  states/state.cc
  states/state.h
  agent.cc
  agent.h
  trajectory.cc
  trajectory.h
  utilities.cc
  utilities.h
  tasks/tasks.cc
  tasks/tasks.h
  tasks/acrobot/acrobot.cc
  tasks/acrobot/acrobot.h
  tasks/cartpole/cartpole.cc
  tasks/cartpole/cartpole.h
  tasks/fingers/fingers.cc
  tasks/fingers/fingers.h
  tasks/hand/hand.cc
  tasks/hand/hand.h
  tasks/humanoid/stand/stand.cc
  tasks/humanoid/stand/stand.h
  tasks/humanoid/tracking/tracking.cc
  tasks/humanoid/tracking/tracking.h
  tasks/humanoid/walk/walk.cc
  tasks/humanoid/walk/walk.h
  tasks/manipulation/common.cc
  tasks/manipulation/common.h
  tasks/manipulation/manipulation.cc
  tasks/manipulation/manipulation.h
  tasks/panda/panda.cc
  tasks/panda/panda.h
  tasks/particle/particle.cc
  tasks/particle/particle.h
  tasks/quadrotor/quadrotor.cc
  tasks/quadrotor/quadrotor.h
  tasks/quadruped/quadruped.cc
  tasks/quadruped/quadruped.h
  tasks/swimmer/swimmer.cc
  tasks/swimmer/swimmer.h
  tasks/walker/walker.cc
  tasks/walker/walker.h
  tasks/smpl/stand/stand.cc
  tasks/smpl/stand/stand.h
  tasks/smpl/walk/walk.cc
  tasks/smpl/walk/walk.h
  tasks/smpl/motion/motion.cc
  tasks/smpl/motion/motion.h
  tasks/myosuite/walk/walk.cc
  tasks/myosuite/walk/walk.h
  tasks/common_rig/walk/walk.h
  tasks/common_rig/walk/walk.cc
  tasks/common_rig/motion/motion.h
  tasks/common_rig/motion/motion.cc
  planners/planner.cc
  planners/planner.h
  planners/policy.h
  planners/include.cc
  planners/include.h
  planners/cost_derivatives.cc
  planners/cost_derivatives.h
  planners/model_derivatives.cc
  planners/model_derivatives.h
  planners/robust/robust_planner.cc
  planners/robust/robust_planner.h
  planners/sampling/planner.cc
  planners/sampling/planner.h
  planners/sampling/policy.cc
  planners/sampling/policy.h
  planners/gradient/gradient.cc
  planners/gradient/gradient.h
  planners/gradient/planner.cc
  planners/gradient/planner.h
  planners/gradient/policy.cc
  planners/gradient/policy.h
  planners/gradient/settings.h
  planners/gradient/spline_mapping.cc
  planners/gradient/spline_mapping.h
  planners/ilqg/backward_pass.cc
  planners/ilqg/backward_pass.h
  planners/ilqg/boxqp.h
  planners/ilqg/planner.cc
  planners/ilqg/planner.h
  planners/ilqg/policy.cc
  planners/ilqg/policy.h
  planners/ilqg/settings.h
  planners/ilqs/planner.cc
  planners/ilqs/planner.h
  estimators/batch.cc
  estimators/batch.h
  estimators/estimator.h
  estimators/include.cc
  estimators/include.h
  estimators/kalman.cc
  estimators/kalman.h
  estimators/unscented.cc
  estimators/unscented.h
  direct/direct.cc
  direct/direct.h
  direct/trajectory.h
  direct/model_parameters.cc
  direct/model_parameters.h
  app.cc
  app.h
  norm.cc
  norm.h
  simulate.cc
  simulate.h
  task.cc
  task.h
  $<TARGET_OBJECTS:mujoco::platform_ui_adapter>
)
set_target_properties(libmjpc PROPERTIES OUTPUT_NAME mjpc)
target_compile_options(libmjpc PUBLIC ${MJPC_COMPILE_OPTIONS})
target_compile_definitions(libmjpc PRIVATE MJSIMULATE_STATIC)

# JSON
set(JSON_BuildTests OFF CACHE INTERNAL "")
add_subdirectory(nlohmann_json)

target_link_libraries(
  libmjpc
  nlohmann_json
  absl::any_invocable
  absl::flat_hash_map
  absl::random_random
  glfw
  lodepng
  mujoco::mujoco
  mujoco::platform_ui_adapter
  threadpool
  Threads::Threads
)
target_include_directories(libmjpc
  PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/..
  ${CMAKE_CURRENT_BINARY_DIR}/..
)

add_executable(
  mjpc
  main.cc
)
target_link_libraries(
  mjpc
  absl::flags
  absl::flags_parse
  absl::random_random
  absl::strings
  libmjpc
  mujoco::mujoco
  threadpool
  Threads::Threads
)
target_include_directories(mjpc PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/..)
target_compile_options(mjpc PUBLIC ${MJPC_COMPILE_OPTIONS})
target_link_options(mjpc PRIVATE ${MJPC_LINK_OPTIONS})
target_compile_definitions(mjpc PRIVATE MJSIMULATE_STATIC)
if(APPLE)
  target_sources(mjpc PRIVATE macos_gui.mm)
  target_link_libraries(mjpc "-framework Cocoa")
endif()

add_subdirectory(tasks)

if(BUILD_TESTING AND MJPC_BUILD_TESTS)
  enable_testing()
  add_subdirectory(test)
endif()

if(MJPC_BUILD_GRPC_SERVICE)
  add_subdirectory(grpc)
endif()
