cmake_minimum_required(VERSION 3.0)

PROJECT(Avidemux_CLI)

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/../../cmake" "${CMAKE_MODULE_PATH}")
include(avidemuxVersion)

MESSAGE("")
MESSAGE("#########################################")
MESSAGE("Configure for avidemux Cli Started")
MESSAGE("#########################################")
MESSAGE("")

MESSAGE(STATUS "Checking for avidemux development files ..")

IF(NOT FAKEROOT)
	SET(AVIDEMUX_FAKEROOT "")
else(NOT FAKEROOT)
	SET(AVIDEMUX_FAKEROOT "${FAKEROOT}")
endif(NOT FAKEROOT)

SET(ADM_HEADER_DIR ${AVIDEMUX_FAKEROOT}${CMAKE_INSTALL_PREFIX}/include/avidemux/${AVIDEMUX_API_VERSION})
SET(ADM_CMAKE_DIR  ${ADM_HEADER_DIR}/cmake)
# Common definitions...
SET(CMAKE_MODULE_PATH "${ADM_CMAKE_DIR}" "${CMAKE_MODULE_PATH}")
MESSAGE(STATUS  "Checking for avidemux include folder (i.e. CMAKE_INSTALL_PREFIX/include/avidemux/${AVIDEMUX_API_VERSION}")
if(NOT EXISTS "${ADM_HEADER_DIR}")
        MESSAGE(STATUS  "Make sure you installed all the files.\n i cannot find avidemux include folder.cmake .\nSet CMAKE_INSTALL_PREFIX to the install folder, current value is ${CMAKE_INSTALL_PREFIX}")
        MESSAGE(FATAL_ERROR  "Aborting")
endif(NOT EXISTS "${ADM_HEADER_DIR}")

MESSAGE(STATUS "Found avidemux include folder. good.")
MESSAGE(STATUS "Checking for cmake subfolder")

if(NOT EXISTS "${ADM_CMAKE_DIR}/commonCmakeApplication.cmake")
        MESSAGE(STATUS  "Make sure you installed all the files.\n I cannot find content of the cmake subfolder .\n")
        MESSAGE(STATUS  "Set CMAKE_INSTALL_PREFIX to the install folder, current value is ${CMAKE_INSTALL_PREFIX}")
        MESSAGE(STATUS  "I was looking for commonCmakeApplication.cmake in  ${ADM_CMAKE_DIR}")
        MESSAGE(FATAL_ERROR  "Aborting")
endif(NOT EXISTS "${ADM_CMAKE_DIR}/commonCmakeApplication.cmake")
MESSAGE(STATUS "Found cmake subfolder.good.")


include(commonCmakeApplication)
INCLUDE(admWindRes)

##########################################
# Config
##########################################
ADD_DEFINITIONS(-DADM_UI_TYPE_BUILD=ADM_UI_CLI)
SET(CONFIG_HEADER_TYPE ADM_BUILD_CLI)
SET(UI_SUFFIX cli)

CONFIGURE_FILE("${ADM_CMAKE_DIR}/config.h.cmake" "${CMAKE_BINARY_DIR}/config/cli/config.h")
MESSAGE(STATUS "CLI config.h generated")

INCLUDE_DIRECTORIES(BEFORE "${CMAKE_BINARY_DIR}/config/cli/")

########################################
# Add subdirectories 
########################################
ADD_SUBDIRECTORY(../common ./common)
ADD_SUBDIRECTORY(ADM_UIs ./ADM_UIsCli)
ADD_SUBDIRECTORY(ADM_userInterfaces ./ADM_userInterfacesCli)

SDLify(../common/main.cpp)
if (ADM_SUBVERSION)
        ADD_SOURCE_CFLAGS(../common/main.cpp "-DADM_SUBVERSION=\"${ADM_SUBVERSION}\"")
endif (ADM_SUBVERSION)
ADD_SOURCE_CFLAGS(../common/main.cpp "-DADM_VERSION=\"${AVIDEMUX_VERSION}\"")

###########################################
# Version Info
###########################################
if (WIN32)
        WINDRESIFY(mainCLI  "${CMAKE_CURRENT_SOURCE_DIR}/../common/xpm/adm.ico" src "avidemux_cli" "Avidemux Video Editor, CLI Version")
        SET(ADM_EXE_SRCS ${ADM_EXE_SRCS} ${src})
endif (WIN32)

###########################################
# Executable
###########################################
include_directories("${PTHREAD_INCLUDE_DIR}")
ADD_EXECUTABLE(avidemux3_cli ${ADM_EXE_SRCS})

###########################################
# Construct common libraries
###########################################
FOREACH (_libName ${commonLibs1})
        TARGET_LINK_LIBRARIES(avidemux3_cli ${_libName})
ENDFOREACH (_libName ${commonLibs1})

TARGET_LINK_LIBRARIES(avidemux3_cli ADM_filtersCli6)

FOREACH (_libName ${commonLibs2})
        TARGET_LINK_LIBRARIES(avidemux3_cli ${_libName})
ENDFOREACH (_libName ${commonLibs2})

FOREACH (_libName ${coreLibs})
        TARGET_LINK_LIBRARIES(avidemux3_cli ${_libName})
ENDFOREACH (_libName ${coreLibs})

#############################################
# Add gtk specific libs
#############################################
TARGET_LINK_LIBRARIES(avidemux3_cli
	ADM_UI_Cli6
	ADM_dialogCli6
	ADM_toolkit6
	ADM_coreAudio6
	ADM_coreUtils6
	ADM_gui2Cli6
	ADM_toolkitCli6
	ADM_shellCli
)

###########################################
# External libs
###########################################
# gettext
IF (GETTEXT_FOUND)
	TARGET_LINK_LIBRARIES(avidemux3_cli ${GETTEXT_LIBRARY_DIR})
ENDIF (GETTEXT_FOUND)


# SDL
IF (USE_SDL)
        TARGET_LINK_LIBRARIES(avidemux3_cli ${SDL2_LIBRARY} ${SDL2_MAIN})
ENDIF (USE_SDL)

###########################################
# OS Specific
###########################################
if (WIN32 OR APPLE)
	set_property(TARGET avidemux3_cli PROPERTY OUTPUT_NAME avidemux_cli)
endif (WIN32 OR APPLE)

IF (MINGW)
	target_link_libraries(avidemux3_cli winmm -mwindows -Wl,-subsystem,console -Wl,--export-all-symbols)
ENDIF (MINGW)

# Needed by RubberBand
if (APPLE)
	target_link_libraries(avidemux3_cli "-framework Accelerate")
endif (APPLE)

###########################################
# Install
###########################################
ADM_LINK_THREAD(avidemux3_cli)
ADM_INSTALL_BIN(avidemux3_cli)

DISPLAY_SUMMARY_LIST()

include(admPackager)
admPackager(cliPackage)
INSTALL(FILES ${CMAKE_BINARY_DIR}/config/cli/config.h DESTINATION "${AVIDEMUX_INCLUDE_DIR}/avidemux/${AVIDEMUX_API_VERSION}/cli" COMPONENT dev) 
INSTALL(FILES ${CMAKE_SOURCE_DIR}/ADM_UIs/include/ADM_UI_Cli6_export.h DESTINATION "${AVIDEMUX_INCLUDE_DIR}/avidemux/${AVIDEMUX_API_VERSION}/cli/ADM_UIs" COMPONENT dev) 
MESSAGE("")

