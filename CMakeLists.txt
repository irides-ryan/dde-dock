cmake_minimum_required(VERSION 3.7)

set(VERSION 4.0)

project(dde-dock)

#set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_CXX_FLAGS "-g -Wall")

include(GNUInstallDirs)

if (DEFINED DOCK_TRAY_USE_NATIVE_POPUP)
    add_definitions(-DDOCK_TRAY_USE_NATIVE_POPUP)
endif ()

if (NOT (${CMAKE_BUILD_TYPE} MATCHES "Debug"))
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Ofast")

    # generate qm
    execute_process(COMMAND bash "translate_generation.sh"
                    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endif ()

# Test architecture
execute_process(COMMAND dpkg-architecture -qDEB_BUILD_ARCH OUTPUT_VARIABLE ARCHITECTURE RESULT_VARIABLE EXIT_CODE)
if (${EXIT_CODE} EQUAL 0)
    string(STRIP ${ARCHITECTURE} ARCHITECTURE)

    if (${ARCHITECTURE} STREQUAL "sw_64")
        # add compiler flags -mieee for mathmatic
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mieee")
    endif()
endif()

file(GLOB INTERFACES "interfaces/*.h")

add_subdirectory("frame")
add_subdirectory("plugins")

# Install settings
if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX /usr)
endif ()

## qm files
file(GLOB QM_FILES "translations/*.qm")
install(FILES ${QM_FILES}
        DESTINATION share/dde-dock/translations)

## dev files
install(FILES ${INTERFACES}
        DESTINATION include/dde-dock)

install(FILES "dde-dock.pc"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

install(FILES "cmake/DdeDock/DdeDockConfig.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DdeDock)

## services files
install(FILES frame/com.deepin.dde.Dock.service
        DESTINATION /usr/share/dbus-1/services)
