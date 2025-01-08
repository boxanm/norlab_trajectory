cmake_minimum_required(VERSION 3.15)
project(norlab_trajectory)

set(CMAKE_BUILD_TYPE "Release")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -march=native)
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

find_package(Eigen3 3.3 REQUIRED)
find_package(steam REQUIRED)
find_package(Boost COMPONENTS system REQUIRED)
find_package(OpenMP REQUIRED)

# Set RPATH to include the local library path
set(CMAKE_INSTALL_RPATH "@loader_path/")
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_SKIP_BUILD_RPATH FALSE)

# Requires CMake 3.15
find_package(Python COMPONENTS Interpreter Development REQUIRED)
if (Python_FOUND)
    message(STATUS "Python found: ${Python_VERSION}")
else()
    message(FATAL_ERROR "Python not found")
endif()

# Use Python to locate pybind11 installed via pip
execute_process(COMMAND ${Python_EXECUTABLE} -c "import pybind11; print(pybind11.get_cmake_dir())"
                OUTPUT_VARIABLE pybind11_DIR
                OUTPUT_STRIP_TRAILING_WHITESPACE)

find_package(pybind11 CONFIG REQUIRED)

include_directories(include/norlab_trajectory)

# Library
add_library(norlab_trajectory src/Trajectory.cpp)
target_link_libraries(norlab_trajectory Eigen3::Eigen ${OpenMP_CXX_LIBRARIES} steam)


# Pybind11 Module
if (pybind11_FOUND)
  pybind11_add_module(pynorlab_trajectory src/bindings.cpp)
  target_link_libraries(pynorlab_trajectory PUBLIC norlab_trajectory)
  find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
  install(TARGETS pynorlab_trajectory
      DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
    # DESTINATION python
  )

  # Installation
  install(DIRECTORY include/
    DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
  )
  # Install the __init__.py file to the same directory as the .so module
    install(FILES norlab_trajectory/__init__.py
        DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
    )

    install(FILES "${lgmath_DIR}/liblgmath.dylib" "${steam_DIR}/libsteam.dylib"
      DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
    )

    install(TARGETS norlab_trajectory
      EXPORT export_norlab_trajectory
      RUNTIME DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
      ARCHIVE DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
      LIBRARY DESTINATION "${Python3_SITEARCH}/${PROJECT_NAME}"
    )

else()
  message(FATAL_ERROR "pybind11 is required!")
endif()
