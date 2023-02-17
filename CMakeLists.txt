cmake_minimum_required(VERSION 3.7)


set(PROJECT_NAME mcufont)
project(${PROJECT_NAME})

message(STATUS "Processing " ${PROJECT_NAME})

# Sources
#--------------------------------------
set(SrcLib
	decoder/mcufont.h
	decoder/mcufont.mk
	decoder/mf_bwfont.c
	decoder/mf_bwfont.h
	decoder/mf_config.h
	decoder/mf_encoding.c
	decoder/mf_encoding.h
	decoder/mf_font.c
	decoder/mf_font.h
	decoder/mf_justify.c
	decoder/mf_justify.h
	decoder/mf_kerning.c
	decoder/mf_kerning.h
	decoder/mf_rlefont.c
	decoder/mf_rlefont.h
	decoder/mf_scaledfont.c
	decoder/mf_scaledfont.h
	decoder/mf_wordwrap.c
	decoder/mf_wordwrap.h
    decoder/Roboto-Bold24_aa.c
    decoder/fonts.h
)

# Set features
#--------------------------------------
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

#--------------------------------------
set_property(GLOBAL PROPERTY USE_FOLDERS ON)


# Set GCC/Clang flags
#--------------------------------------
if(NOT MSVC)
    # Avoid default flag values
    #--------------------------------------
    set(CMAKE_C_FLAGS "")
    set(CMAKE_C_FLAGS_DEBUG "" )
    set(CMAKE_C_FLAGS_RELEASE "")

    set(CMAKE_CXX_FLAGS "")
    set(CMAKE_CXX_FLAGS_DEBUG "")
    set(CMAKE_CXX_FLAGS_RELEASE "")

    set(CMAKE_OBJC_FLAGS "")
    set(CMAKE_OBJC_FLAGS_DEBUG "")
    set(CMAKE_OBJC_FLAGS_RELEASE "")

    set(CMAKE_OBJCXX_FLAGS "")
    set(CMAKE_OBJCXX_FLAGS_DEBUG "")
    set(CMAKE_OBJCXX_FLAGS_RELEASE "")

    # Set our flags
    #--------------------------------------
    add_compile_options("$<$<CONFIG:Debug>:-g>")
    add_compile_options("$<IF:$<CONFIG:Debug>,-O0,-O2>")
    add_compile_options(-Wall -Wextra)
    add_compile_options(-Wno-switch -Wno-unused-function -Wno-unused-parameter -Wno-implicit-fallthrough)

    if(NOT APPLE)
        add_compile_options(-Wno-cast-function-type)
    endif()
else()
    # Security check
    add_compile_options(/GS)
    # Function level linking
    add_compile_options(/Gy)
    # Exceptions
    add_compile_options(/EHsc)
    if(MSVC_VERSION GREATER_EQUAL 1900)
        # SDL checks 2015+
        add_compile_options(/sdl)
    endif()
    if(MSVC_VERSION LESS_EQUAL 1920)
        # Enable Minimal Rebuild (required for Edit and Continue) (deprecated)
        add_compile_options(/Gm)
    endif()
    add_compile_options(/fp:fast)
    # Program database for edit and continue
    add_compile_options("$<IF:$<CONFIG:Debug>,/ZI,/Zi>")
    # Optimizations
    add_compile_options("$<IF:$<CONFIG:Debug>,/Od,/O2>")
    # Inline function expansion
    add_compile_options("$<IF:$<CONFIG:Debug>,/Ob0,/Ob2>")
    # Basic runtime checks
    add_compile_options("$<$<CONFIG:Debug>:/RTC1>")
    # Force Visual Studio to actualize __cplusplus version macro
    add_compile_options(/Zc:__cplusplus)

endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_definitions(-D_DEBUG)
    add_definitions(-DDEBUG)
    if(EMSCRIPTEN)
        add_link_options(-g)
    endif()
endif()

# Library
#--------------------------------------
add_library(mcufont STATIC
    ${SrcLib}
)
add_library(mcufont::mcufont ALIAS mcufont)

# For all projects
#--------------------------------------
target_include_directories(mcufont PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/decoder>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)
target_include_directories(mcufont PRIVATE decoder)

link_libraries(mcufont)

# Organize IDE Folders
#--------------------------------------
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set_property(TARGET mcufont PROPERTY FOLDER "Libs")


install(TARGETS mcufont EXPORT mcufont)
file(GLOB_RECURSE HEADERS "${CMAKE_CURRENT_LIST_DIR}/include/*.h")
install(FILES ${HEADERS} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
install(EXPORT mcufont FILE mcufont-config.cmake DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/mcufont" NAMESPACE mcufont::)

message(STATUS "Done " ${PROJECT_NAME})
