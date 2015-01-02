#=============================================================================
# CMake configuration file for ChronoVehicle_SDL_Demo
#
#=============================================================================

cmake_minimum_required(VERSION 2.8)

project(ChronoVehicle_SDL_Demo)

# Location of additional CMAKE modules
SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)


# ------------------------------------------------------------------------------
# Specify output locations for libraries and executables
# ------------------------------------------------------------------------------
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

IF (MSVC OR XCODE_VERSION)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
ENDIF()

FIND_PACKAGE(ChronoEngine COMPONENTS unit_IRRLICHT)

IF(CHRONOENGINE_LIBRARY_DEBUG)
  MESSAGE(STATUS "ChronoEngine library (debug) found")
ENDIF()
IF(CHRONOENGINE_LIBRARY_RELEASE)
  MESSAGE(STATUS "ChronoEngine library (release) found")
ENDIF()

# Stop now if the path to the Chrono SDK is invalid or if we could not find
# at least one of the debug or release Chrono libraries.
IF(NOT EXISTS "${CH_CHRONO_SDKDIR}/physics/ChSystem.h")
  MESSAGE(FATAL_ERROR "Cannot find the Chrono SDK. Make sure CH_CHRONO_SDKDIR is set correctly.")
  RETURN()
ENDIF()
IF(NOT CHRONOENGINE_LIBRARY_DEBUG AND NOT CHRONOENGINE_LIBRARY_RELEASE)
  MESSAGE(FATAL_ERROR "Cannot find the ChronoEngine library. Check CH_LIBDIR_DEBUG and CH_LIBDIR_RELEASE.")
  RETURN()
ENDIF()

# Add paths to Chrono headers
INCLUDE_DIRECTORIES(${CHRONOENGINE_INCLUDES})

#--------------------------------------------------------------
# Find the ChronoVehicle library
#SET(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})
FIND_PACKAGE(ChronoVehicle)
# Stop now if we could not find the ChronoVehicle SDK
IF(NOT EXISTS "${CH_VEHICLE_SDKDIR}/subsys/ChVehicle.h")
MESSAGE(FATAL_ERROR "Cannot find the ChronoVehicle SDK. Make sure CH_VEHICLE_SDKDIR is set correctly.")
RETURN()
ENDIF()
# Check debug/release ChronoVehicle libraries. Stop now if neither could be found.
IF(CHRONOVEHICLE_LIBRARY_DEBUG)
MESSAGE(STATUS "ChronoVehicle library (debug) found")
ENDIF()
IF(CHRONOVEHICLE_LIBRARY_RELEASE)
MESSAGE(STATUS "ChronoVehicle library (release) found")
ENDIF()
IF(NOT CHRONOVEHICLE_LIBRARY_DEBUG AND NOT CHRONOVEHICLE_LIBRARY_RELEASE)
MESSAGE(FATAL_ERROR "Cannot find the ChronoVehicle library. Check CH_VEHICLE_LIBDIR_DEBUG and CH_VEHICLE_LIBDIR_RELEASE.")
RETURN()
ENDIF()
MESSAGE(STATUS "ChronoVehicle includes: ${CHRONOVEHICLE_INCLUDES}")
MESSAGE(STATUS "ChronoVehicle libraries: ${CHRONOVEHICLE_LIBRARIES}")
#--------------------------------------------------------------
# Generate the configuration file
SET(CHRONOVEHICLE_DATA_DIR "${CH_VEHICLE_SDKDIR}/data/")
MESSAGE(STATUS "PRJ bin dir: ${PROJECT_BINARY_DIR}")
MESSAGE(STATUS "CMK bin dir: ${CMAKE_BINARY_DIR}")
#--------------------------------------------------------------

#SET(CHRONOVEHICLE_IRRLICHT_LIB_DIR_DEBUG "" CACHE FILEPATH "Where is your ChronoVehicle_Irrlicht Debug lib?")
SET(CHRONOVEHICLE_IRRLICHT_LIB "" CACHE FILEPATH "Where is your ChronoVehicle_Irrlicht lib?")
SET(CHRONOVEHICLE_UTILS_LIB "" CACHE FILEPATH "Where is your ChronoVehicle_Utils lib?")

INCLUDE_DIRECTORIES(${CHRONOVEHICLE_INCLUDES})

# ------------------------------------------------------------------------------
# Find SDL2 library
# ------------------------------------------------------------------------------

IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
	SET(CH_SDL2_SDKDIR "" CACHE PATH "Where is your SDL2 SDK installed?")
	SET(CH_SDL2LIB "" CACHE FILEPATH "Where is your SDL2 library?")
	SET(CH_SDL2mainLIB "" CACHE FILEPATH "Where is your SDL2main library?")
ELSEIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
	FIND_PATH(CH_SDL2_SDKDIR NAMES sdl.h PATHS "/usr/include/sdl2" "/usr/local/include/sdl2")
	FIND_LIBRARY(CH_SDL2LIB NAMES SDL2 PATHS "/usr/local/lib" ${CH_SDL2_SDKDIR}/lib/Linux)
	FIND_LIBRARY(CH_SDL2mainLIB NAMES SDL2main PATHS "/usr/local/lib" ${CH_SDL2_SDKDIR}/lib/Linux)
ENDIF()
IF(EXISTS "${CH_SDL2_SDKDIR}/include")
	SET(CH_SDL2INC "${CH_SDL2_SDKDIR}/include")
ELSE()
	SET(CH_SDL2INC "${CH_SDL2_SDKDIR}")
ENDIF()
INCLUDE_DIRECTORIES(${CH_SDL2INC})
MARK_AS_ADVANCED(CLEAR CH_SDL2_SDKDIR)
MARK_AS_ADVANCED(CLEAR CH_SDL2LIB)
MARK_AS_ADVANCED(CLEAR CH_SDL2mainLIB)


# ------------------------------------------------------------------------------
# Automatically copy DLLs 
# ------------------------------------------------------------------------------
IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    # HACK?
    IF(CH_LIBDIR_DEBUG)
        SET(CH_BINDIR "${CH_LIBDIR_DEBUG}/../../bin")
    ENDIF()
    IF(CH_LIBDIR_RELEASE)
        SET(CH_BINDIR "${CH_LIBDIR_RELEASE}/../../bin")
    ENDIF()

    # Create custom target for copying DLLs; add it to the default build target
    ADD_CUSTOM_TARGET(COPY_DLLS ALL)

    # Create custom commands, invoked post-build to copy DLLs to the appropriate
    # directory (depending on the configuration selected at build time in VS)
    ADD_CUSTOM_COMMAND(
        TARGET COPY_DLLS POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CH_BINDIR}/$<CONFIGURATION>/ChronoEngine.dll"
            "${CMAKE_BINARY_DIR}/bin/$<CONFIGURATION>"
    )

    IF(ENABLE_IRRLICHT)
        IF("${CH_COMPILER}" STREQUAL "COMPILER_MSVC")
            SET(CH_IRRLICHT_DLL "${CH_IRRLICHT_SDKDIR}/bin/Win32-VisualStudio/Irrlicht.dll")
        ELSEIF("${CH_COMPILER}" STREQUAL "COMPILER_MSVC_X64")
            SET(CH_IRRLICHT_DLL "${CH_IRRLICHT_SDKDIR}/bin/Win64-VisualStudio/Irrlicht.dll")
        ENDIF()

        ADD_CUSTOM_COMMAND(
            TARGET COPY_DLLS POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${CH_BINDIR}/$<CONFIGURATION>/ChronoEngine_IRRLICHT.dll"
                "${CMAKE_BINARY_DIR}/bin/$<CONFIGURATION>"
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${CH_IRRLICHT_DLL}"
                "${CMAKE_BINARY_DIR}/bin/$<CONFIGURATION>"
        )
    ENDIF()

    IF(ENABLE_IRRKLANG)
        IF("${CH_COMPILER}" STREQUAL "COMPILER_MSVC")
            SET(CH_IRRKLANG_DLL "${CH_IRRKLANG_SDKDIR}/bin/Win32-VisualStudio/irrKlang.dll")
        ELSEIF("${CH_COMPILER}" STREQUAL "COMPILER_MSVC_X64")
            SET(CH_IRRKLANG_DLL "${CH_IRRKLANG_SDKDIR}/bin/Winx64-VisualStudio/irrKlang.dll")
        ENDIF()

        ADD_CUSTOM_COMMAND(
            TARGET COPY_DLLS POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                "${CH_IRRKLANG_DLL}"
                "${CMAKE_BINARY_DIR}/bin/$<CONFIGURATION>"
        )
    ENDIF()

ENDIF()


# ------------------------------------------------------------------------------
# Add paths to the top of the source directory and the binary directory
# ------------------------------------------------------------------------------

INCLUDE_DIRECTORIES(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}
)



CONFIGURE_FILE(
  ${CMAKE_MODULE_PATH}/config.h.in
  ${PROJECT_BINARY_DIR}/config.h
  )

INCLUDE_DIRECTORIES(${PROJECT_BINARY_DIR})

ADD_SUBDIRECTORY(src)
