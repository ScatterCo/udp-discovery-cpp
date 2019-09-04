cmake_minimum_required(VERSION 3.4)

if(APPLE)
	set(platform "osx")
elseif(UNIX AND NOT APPLE)
	set(platform "linux")
else()
	set(platform "win32")
endif()

set(LIB_SOURCES
	endpoint.cpp
	ip_port.cpp
	protocol.cpp)
set(LIB_HEADERS
	endpoint.hpp
	ip_port.hpp
	protocol.hpp)

add_library(udp-discovery STATIC ${LIB_SOURCES} ${LIB_HEADERS})

target_include_directories(udp-discovery INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                                                   $<INSTALL_INTERFACE:include>)

if(APPLE)
elseif(UNIX)
	set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pthread")
endif()

if(APPLE)
elseif(UNIX)
	set(DISCOVERY_EXAMPLE_LIBS ${DISCOVERY_EXAMPLE_LIBS}
		rt)
	set(DISCOVERY_TOOL_LIBS ${DISCOVERY_TOOL_LIBS}
		rt)
endif()

option(BUILD_EXAMPLE "build the example project" ON)
option(BUILD_TOOL "build the disocvery tool" ON)

if(BUILD_EXAMPLE)
	set(DISCOVERY_EXAMPLE_SOURCES discovery_example.cpp)
	set(DISCOVERY_EXAMPLE_LIBS udp-discovery)
	if(WIN32)
		set(DISCOVERY_EXAMPLE_LIBS ${DISCOVERY_EXAMPLE_LIBS} Ws2_32)
	endif(WIN32)
	add_executable(udp-discovery-example ${DISCOVERY_EXAMPLE_SOURCES})
	target_link_libraries(udp-discovery-example ${DISCOVERY_EXAMPLE_LIBS})
endif()

if(BUILD_TOOL)
	set(DISCOVERY_TOOL_SOURCES discovery_tool.cpp)
	set(DISCOVERY_TOOL_LIBS udp-discovery)
	add_executable(udp-discovery-tool ${DISCOVERY_TOOL_SOURCES})
	target_link_libraries(udp-discovery-tool ${DISCOVERY_TOOL_LIBS})
endif()


install(
    TARGETS udp-discovery
    EXPORT udp-discovery-targets
    CONFIGURATIONS Debug
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/${platform}/Debug"
)

install(
    TARGETS udp-discovery
    EXPORT udp-discovery-targets
    CONFIGURATIONS Release
    DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/${platform}/Release"
)

install(
	FILES ${LIB_HEADERS}
	DESTINATION "${CMAKE_INSTALL_PREFIX}/include"
)

include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/udpDiscoveryCppConfig.cmake.in
                              ${CMAKE_CURRENT_BINARY_DIR}/udpDiscoveryCppConfig.cmake
                              INSTALL_DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/udpDiscoveryCppConfig.cmake
	DESTINATION "${CMAKE_INSTALL_PREFIX}/lib/cmake"
)

install(
  EXPORT udp-discovery-targets DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/cmake
)
