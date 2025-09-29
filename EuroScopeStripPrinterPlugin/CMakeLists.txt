cmake_minimum_required(VERSION 3.20)
project(EuroScopeStripPrinterPlugin LANGUAGES CXX)

# --- Find the EuroScope SDK headers ---
# Pass this on the command line or set the env var.
#   cmake -S . -B build -G "Visual Studio 17 2022" -A Win32 -DEUROSCOPE_SDK_DIR="C:/.../EuroscopeSDK"
if(NOT DEFINED EUROSCOPE_SDK_DIR)
  if(DEFINED ENV{EUROSCOPE_SDK_DIR})
    set(EUROSCOPE_SDK_DIR "$ENV{EUROSCOPE_SDK_DIR}")
  else()
    message(FATAL_ERROR "Set EUROSCOPE_SDK_DIR to the folder that contains EuroScopePlugIn.h")
  endif()
endif()
message(STATUS "Using EUROSCOPE_SDK_DIR=${EUROSCOPE_SDK_DIR}")

# --- Target ---
add_library(EuroScopeStripPrinterPlugin SHARED
  ${CMAKE_CURRENT_SOURCE_DIR}/EuroScopeStripPrinterPlugin.cpp
)

target_include_directories(EuroScopeStripPrinterPlugin PRIVATE
  "${EUROSCOPE_SDK_DIR}"
)

target_compile_features(EuroScopeStripPrinterPlugin PRIVATE cxx_std_17)
target_compile_definitions(EuroScopeStripPrinterPlugin PRIVATE
  WIN32_LEAN_AND_MEAN
  NOMINMAX
  _WIN32_WINNT=0x0601
)

# ============================================
# A) USE THE .def FILE FOR EXACT EXPORT NAMES
#    (Put your .def beside the .cpp)
# ============================================
set_target_properties(EuroScopeStripPrinterPlugin PROPERTIES
  VS_MODULE_DEFINITION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/EuroScopeStripPrinterPlugin.def"   # <-- EXACTLY HERE
  OUTPUT_NAME "EuroScopeStripPrinterPlugin"
  RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIG>"
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIG>"
  LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIG>"
)

# ============================================
# B) LINK THE EUROSCOPE IMPORT LIB
#    (EuroScopePlugIn.lib or EuroScopePlugInDll.lib)
#    Adjust PATHS if your .lib is in a subfolder like 'lib'
# ============================================
find_library(EUROSCOPE_LIB
  NAMES EuroScopePlugIn EuroScopePlugInDll
  PATHS
    "${EUROSCOPE_SDK_DIR}"
    "${EUROSCOPE_SDK_DIR}/lib"
    "${EUROSCOPE_SDK_DIR}/Lib"
  NO_DEFAULT_PATH
)

if(NOT EUROSCOPE_LIB)
  message(FATAL_ERROR "Could not find EuroScope import library (.lib) under ${EUROSCOPE_SDK_DIR}.")
endif()

target_link_libraries(EuroScopeStripPrinterPlugin PRIVATE
  "${EUROSCOPE_LIB}"     # <-- EXACTLY HERE
  # Windows libs you might touch; kernel32 is implicit
  user32
  advapi32
  shell32
)

# (Nothing else required.)
