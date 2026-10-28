cmake_minimum_required(VERSION 3.22)
project(ArmNongMuangLA2A VERSION 1.1.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(JUCE_BUILD_EXTRAS OFF CACHE BOOL "" FORCE)
set(JUCE_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

include(FetchContent)
FetchContent_Declare(JUCE
  GIT_REPOSITORY https://github.com/juce-framework/JUCE.git
  GIT_TAG 8.0.6
  GIT_SHALLOW TRUE)
FetchContent_MakeAvailable(JUCE)

# Fetch NAM Core source and its Eigen / AudioDSPTools submodules without invoking
# NAM Core's standalone command-line tools as part of this plugin build.
FetchContent_Declare(NeuralAmpModelerCore
  GIT_REPOSITORY https://github.com/sdatkinson/NeuralAmpModelerCore.git
  GIT_TAG main
  GIT_SHALLOW FALSE
  GIT_SUBMODULES "Dependencies/eigen;Dependencies/AudioDSPTools"
  GIT_SUBMODULES_RECURSE TRUE
  SOURCE_SUBDIR _source_only)
FetchContent_MakeAvailable(NeuralAmpModelerCore)
set(NAM_CORE_DIR "${neuralampmodelercore_SOURCE_DIR}")
file(GLOB_RECURSE NAM_CORE_SOURCES CONFIGURE_DEPENDS
  "${NAM_CORE_DIR}/NAM/*.cpp"
  "${NAM_CORE_DIR}/NAM/*.c")
if(NOT NAM_CORE_SOURCES)
  message(FATAL_ERROR "NeuralAmpModelerCore sources were not downloaded.")
endif()
add_library(nam_core STATIC ${NAM_CORE_SOURCES})
target_compile_features(nam_core PUBLIC cxx_std_20)
target_include_directories(nam_core PUBLIC
  "${NAM_CORE_DIR}/NAM"
  "${NAM_CORE_DIR}/Dependencies/eigen"
  "${NAM_CORE_DIR}/Dependencies/nlohmann"
  "${NAM_CORE_DIR}/Dependencies/AudioDSPTools/dsp")
target_compile_definitions(nam_core PUBLIC NOMINMAX WIN32_LEAN_AND_MEAN)

juce_add_binary_data(LA2AAssets SOURCES Assets/LA2A_Interface.png)
juce_add_plugin(ArmNongMuangLA2A
  COMPANY_NAME "Arm Nong Muang - Muepim Studio"
  BUNDLE_ID "com.armnongmuang.la2a"
  PLUGIN_MANUFACTURER_CODE Anmg
  PLUGIN_CODE La2a
  FORMATS VST3 Standalone
  PRODUCT_NAME "LA-2A Arm Nong Muang"
  IS_SYNTH FALSE NEEDS_MIDI_INPUT FALSE NEEDS_MIDI_OUTPUT FALSE IS_MIDI_EFFECT FALSE
  EDITOR_WANTS_KEYBOARD_FOCUS TRUE COPY_PLUGIN_AFTER_BUILD FALSE)
juce_generate_juce_header(ArmNongMuangLA2A)
target_sources(ArmNongMuangLA2A PRIVATE
  Source/PluginProcessor.cpp Source/PluginProcessor.h
  Source/PluginEditor.cpp Source/PluginEditor.h)
target_compile_definitions(ArmNongMuangLA2A PUBLIC
  JUCE_WEB_BROWSER=0 JUCE_USE_CURL=0 JUCE_VST3_CAN_REPLACE_VST2=0)
target_link_libraries(ArmNongMuangLA2A PRIVATE
  LA2AAssets nam_core juce::juce_audio_utils juce::juce_audio_processors juce::juce_dsp
  PUBLIC juce::juce_recommended_config_flags juce::juce_recommended_warning_flags)
