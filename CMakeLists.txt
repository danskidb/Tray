cmake_minimum_required(VERSION 3.13 FATAL_ERROR) # target_link_directories

project(tray CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Generate 'compile_commands.json' for clang_complete
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

file(GLOB SRCS
		${CMAKE_CURRENT_LIST_DIR}/*.hpp
        ${CMAKE_CURRENT_LIST_DIR}/*.h
        ${CMAKE_CURRENT_LIST_DIR}/*.ico
        ${CMAKE_CURRENT_LIST_DIR}/*.png)

if(WIN32)
	list(APPEND SRCS ${CMAKE_CURRENT_SOURCE_DIR}/tray_windows.cpp)
else()
	if(UNIX)
		if(APPLE)
			find_library(COCOA Cocoa REQUIRED)
			list(APPEND SRCS ${CMAKE_CURRENT_SOURCE_DIR}/tray_darwin.mm)
		else()
			FIND_PACKAGE(PkgConfig)
			PKG_CHECK_MODULES(GTK REQUIRED gtk+-3.0)
			PKG_CHECK_MODULES(APPINDICATOR REQUIRED appindicator3-0.1)
			list(APPEND SRCS ${CMAKE_CURRENT_SOURCE_DIR}/tray_linux.cpp)
		endif()
	endif()
endif()

add_library(tray STATIC ${SRCS})
set_property(TARGET tray PROPERTY C_STANDARD 99)
if(WIN32)
	target_compile_definitions(tray PRIVATE TRAY_WINAPI=1 WIN32_LEAN_AND_MEAN NOMINMAX)
	#target_compile_options(tray PRIVATE "/MT$<$<CONFIG:Debug>:d>")
else()
	if(UNIX)
		if(APPLE)
			target_compile_definitions(tray PRIVATE TRAY_APPKIT=1)
			target_link_libraries(tray PRIVATE ${COCOA})
		else()
			target_compile_options(tray PRIVATE ${GTK_CFLAGS})
			target_link_directories(tray PRIVATE ${GTK_LIBRARY_DIRS})
			target_link_libraries(tray PRIVATE ${GTK_LIBRARIES})

			target_compile_options(tray PRIVATE ${APPINDICATOR_CFLAGS})
			target_link_directories(tray PRIVATE ${APPINDICATOR_LIBRARY_DIRS})
			target_compile_definitions(tray PRIVATE TRAY_APPINDICATOR=1)
			target_link_libraries(tray PRIVATE ${APPINDICATOR_LIBRARIES})
		endif()
	endif()
endif()

add_library(tray::tray ALIAS tray)

add_executable(tray_example ${CMAKE_CURRENT_SOURCE_DIR}/example.cpp)
target_link_libraries(tray_example tray::tray)

configure_file(${CMAKE_CURRENT_LIST_DIR}/icon.ico ${CMAKE_BINARY_DIR}/icon.ico COPYONLY)
configure_file(${CMAKE_CURRENT_LIST_DIR}/icon.png ${CMAKE_BINARY_DIR}/icon.png COPYONLY)

INSTALL(TARGETS tray tray DESTINATION lib)

IF(NOT WIN32)
	INSTALL(FILES tray.hpp DESTINATION include)
ENDIF()

