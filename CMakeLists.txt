cmake_minimum_required(VERSION 3.21)

# "Apparel Preview" - preview armor on the player in Container/Barter menus
# without equipping. One CommonLibSSE-NG DLL for GOG / SE 1.5.97 + 1.6.1170 + 1.7.99+
# (AE offsets verified against the 1.7.104 binary).
project(ApparelPreview VERSION 0.4.3 LANGUAGES CXX)

# Use our supplied pinned CommonLib submodule as dependency instead
# of the pinned (but outdated) CommonLib vcpkg package.
# Pinned Version used for this release is 6.8.0
set(COMMONLIB_SSE_FOLDER ${CMAKE_CURRENT_SOURCE_DIR}/extern/src/CommonLibSSE-NG CACHE PATH "CommonLibSSE-NG source")
add_subdirectory(${COMMONLIB_SSE_FOLDER} ${CMAKE_BINARY_DIR}/CommonLibSSE)
# ToDo: Why do we need to include this in the main CMakeLists.txt
# These 3 lines should be externalized or simplified.
include(${COMMONLIB_DIR}/cmake/CommonLibSSE.cmake)

# jsoncpp's exported config predates CMake 3.5's policy floor.
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)
find_package(jsoncpp CONFIG REQUIRED)  # DIII_API.h passes Json::Value across the API
unset(CMAKE_POLICY_VERSION_MINIMUM)
find_package(imgui CONFIG REQUIRED)  # FLICK/FUCK API surface (FUCK_API.h needs imgui >= 1.92 types)

add_commonlibsse_plugin(${PROJECT_NAME}
    SOURCES
        src/PCH.h
        src/plugin.cpp
        src/VersionCheck.h
        src/VersionCheck.cpp
        src/PreviewSet.h
        src/Settings.h
        src/Settings.cpp
        src/REAugments.h
        src/REAugments.cpp
        src/PreviewSession.h
        src/PreviewSession.cpp
        src/MenuWatcher.h
        src/MenuWatcher.cpp
        src/ListMarker.h
        src/ListMarker.cpp
        src/HoverTracker.h
        src/HoverTracker.cpp
        src/BipedHooks.h
        src/BipedHooks.cpp
        src/InputListener.h
        src/InputListener.cpp
        src/NavButton.h
        src/NavButton.cpp
        src/DIIIIntegration.h
        src/DIIIIntegration.cpp
        src/SettingsUI.h
        src/SettingsUI.cpp
)

target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_23)
target_precompile_headers(${PROJECT_NAME} PRIVATE src/PCH.h)
target_include_directories(${PROJECT_NAME} PRIVATE extern)
target_link_libraries(${PROJECT_NAME} PRIVATE JsonCpp::JsonCpp imgui::imgui)
# FLICK / FUCK is consumed at runtime via its C-ABI (extern/FUCK_API.h +
# FUCK.dll's RequestFUCK export); we only need imgui's headers/types for the
# API surface, FUCK.dll does the rendering.
target_compile_definitions(${PROJECT_NAME} PRIVATE IMGUI_DEFINE_MATH_OPERATORS)

# Pure-logic unit tests (no RE/engine deps).
add_executable(PreviewSetTests tests/test_preview_set.cpp)
target_compile_features(PreviewSetTests PRIVATE cxx_std_23)
target_include_directories(PreviewSetTests PRIVATE src)

if(NOT DEFINED OUTPUT_FOLDER)
    set(OUTPUT_FOLDER "C:/Games/Nolvus/Instances/Nolvus Awakening/MODS/mods/Apparel Preview")
endif()
set(DLL_FOLDER "${OUTPUT_FOLDER}/SKSE/Plugins")
message(STATUS "ApparelPreview deploy folder: ${DLL_FOLDER}")
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND "${CMAKE_COMMAND}" -E make_directory "${DLL_FOLDER}"
    COMMAND "${CMAKE_COMMAND}" -E copy_if_different "$<TARGET_FILE:${PROJECT_NAME}>" "${DLL_FOLDER}/$<TARGET_FILE_NAME:${PROJECT_NAME}>"
    COMMAND "${CMAKE_COMMAND}" -E copy_directory "${CMAKE_CURRENT_SOURCE_DIR}/dist" "${OUTPUT_FOLDER}"
    VERBATIM)
