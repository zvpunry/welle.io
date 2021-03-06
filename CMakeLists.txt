cmake_minimum_required(VERSION 3.2)

set(executableName welle-io)

project(Welle.Io VERSION 0.0.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 14)

add_definitions(-Wall)

if(MINGW)
    add_definitions(-municode)
endif()

add_definitions(-DSSE_AVAILABLE)

if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: defaulting to release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")

#enable_testing()

include(GNUInstallDirs)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake/Modules)

find_package(Threads REQUIRED)
find_package(Qt5 COMPONENTS Widgets Quick Multimedia Charts REQUIRED)
find_package(FFTW3f REQUIRED)
find_package(Faad REQUIRED)
if (RTLSDR)
  find_package(LibRTLSDR REQUIRED)
endif()
if (SOAPYSDR)
  find_package(SoapySDR NO_MODULE REQUIRED)
  # Note: SoapySDRConfig.cmake sets C++11 standard so it needs to be reset to C++14
  set(CMAKE_CXX_STANDARD 14)
endif()

include_directories(
    src
    src/ofdm
    src/backend
    src/backend/audio
    src/backend/data
    src/output
    src/various
    src/input
    src/gui
    ${FFTW3F_INCLUDE_DIRS}
    ${FAAD_INCLUDE_DIRS}
    ${LIBRTLSDR_INCLUDE_DIRS}
    ${SoapySDR_INCLUDE_DIRS}
)

set(sources
    src/main.cpp
    src/ofdm/ofdm-processor.cpp
    src/ofdm/ofdm-decoder.cpp
    src/ofdm/phasereference.cpp
    src/ofdm/phasetable.cpp
    src/ofdm/freq-interleaver.cpp
    src/backend/viterbi.cpp
    src/backend/viterbi_spiral.c
    src/backend/fic-handler.cpp
    src/backend/msc-handler.cpp
    src/backend/eep-protection.cpp
    src/backend/uep-protection.cpp
    src/backend/fib-processor.cpp
    src/backend/galois.cpp
    src/backend/reed-solomon.cpp
    src/backend/charsets.cpp
    src/backend/firecode-checker.cpp
    src/backend/protTables.cpp
    src/backend/audio/dab-audio.cpp
    src/backend/audio/mp2processor.cpp
    src/backend/audio/mp4processor.cpp
    src/backend/audio/CFaadDecoder.cpp
    src/backend/data/pad_decoder.cpp
    src/backend/data/mot_manager.cpp
    src/backend/data/pad_decoder_adapter.cpp
    src/backend/tools.cpp
    src/output/CAudio.cpp
    src/various/fft.cpp
    src/various/Xtan2.cpp
    src/input/CInputFactory.cpp
    src/input/CNullDevice.cpp
    src/input/CRAWFile.cpp
    src/input/CRTL_TCP_Client.cpp
    src/DabConstants.cpp
    src/gui/CMOTImageProvider.cpp
    src/gui/CStationList.cpp
    src/gui/CGUI.cpp
    src/CRadioController.cpp
    src/CChannels.cpp
    src/CLogFile.cpp
    src/CSplashScreen.cpp
)

if(LIBRTLSDR_FOUND)
  add_definitions (-DHAVE_RTLSDR)
  set(sources  ${sources} src/input/CRTL_SDR.cpp)
endif()

if(SoapySDR_FOUND)
  add_definitions (-DHAVE_SOAPYSDR)
  set(sources  ${sources} src/input/CSoapySdr.cpp)
endif()

execute_process(
  COMMAND git rev-parse --short HEAD
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GIT_COMMIT_HASH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

if(GIT_COMMIT_HASH)
  add_definitions("-DGITHASH=\"${GIT_COMMIT_HASH}\"")
endif()

add_executable (${executableName} ${sources} ${EXTRA_MOCS} resources.qrc)

target_link_libraries (${executableName}
  ${LIBRTLSDR_LIBRARIES}
  ${FFTW3F_LIBRARIES}
  ${FAAD_LIBRARIES}
  ${SoapySDR_LIBRARIES}
  Threads::Threads
)

qt5_use_modules(${executableName} Widgets Quick Multimedia Charts)

INSTALL (TARGETS ${executableName} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

add_custom_target(uninstall
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
