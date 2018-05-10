cmake_minimum_required(VERSION 2.8.3)
project(fastslam)

# SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC -g -rdynamic -msse4")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

##############################################################################
# Qt Environment
##############################################################################

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
#SET(CMAKE_AUTOUIC ON)

# Find the QtWidgets library
find_package(Qt5 REQUIRED
    Core
    Gui
    Widgets
    PrintSupport
)

# search for everything we need to build the package
find_package(Eigen3 REQUIRED)

##############################################################################
# Sections
##############################################################################

# Add the include directories for the Qt 5 Widgets module to
# the compile lines.
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(
  ${EIGEN3_INCLUDE_DIRS}

  ${Qt5Core_INCLUDE_DIRS}
  ${Qt5Gui_INCLUDE_DIRS}
  ${Qt5Widgets_INCLUDE_DIRS}
  ${Qt5PrintSupport_INCLUDE_DIRS}
)

# Use the compile definitions defined in the Qt 5 Widgets module
add_definitions(${Qt5Widgets_DEFINITIONS})

# Add compiler flags for building executables (-fPIE)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${Qt5Widgets_EXECUTABLE_COMPILE_FLAGS}")

file(GLOB QT_FORMS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ui/*.ui)
file(GLOB QT_RESOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} resources/*.qrc)
file(GLOB_RECURSE QT_MOC RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} FOLLOW_SYMLINKS include/*.h)

QT5_ADD_RESOURCES(QT_RESOURCES_CPP ${QT_RESOURCES})
QT5_WRAP_UI(QT_FORMS_HPP ${QT_FORMS})
QT5_WRAP_CPP(QT_MOC_HPP ${QT_MOC})
# qt5_generate_moc(main.cpp main.moc)

##############################################################################
# Sources
##############################################################################
file(GLOB_RECURSE QT_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} FOLLOW_SYMLINKS src/*.cpp)
# Remove main.cpp and test.cpp from the QT_SOURCES list
list(REMOVE_ITEM QT_SOURCES "src/test.cpp" "src/main.cpp")
message(STATUS "QT_SOURCES: " ${QT_SOURCES})

# Tell CMake to create the executable
add_executable(${PROJECT_NAME} src/main.cpp ${QT_SOURCES} ${QT_RESOURCES_CPP} ${QT_FORMS_HPP} ${QT_MOC_HPP})
# add_executable(${PROJECT_NAME}_test src/test.cpp ${QT_SOURCES} ${QT_RESOURCES_CPP} ${QT_FORMS_HPP} ${QT_MOC_HPP})

#Link the executable to the Qt 5 widgets library.
target_link_libraries(${PROJECT_NAME}
    Qt5::Core
    Qt5::Widgets
    Qt5::PrintSupport
)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_NAME} ../
    COMMAND ${CMAKE_COMMAND} -E copy compile_commands.json ../
)
