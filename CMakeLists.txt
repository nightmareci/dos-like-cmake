cmake_minimum_required(VERSION 3.10)

set(CMAKE_VERBOSE_MAKEFILE on)

project(dos-like LANGUAGES C)

set(EXAMPLES
	burn
	edit
	julia
	mandelbrot
	plasma
	raycast
	rotozoom
	sound
	stranded
	tracker
	tunnel
	voxel
)

set(DOS_LIKE_SOURCE ${CMAKE_PROJECT_NAME} CACHE STRING "Path to the root of the original dos-like source tree")

if(WIN32)
	if(${CMAKE_C_COMPILER_ID} STREQUAL MSVC)
		add_library(${CMAKE_PROJECT_NAME} STATIC "${DOS_LIKE_SOURCE}/source/dos.h" "${DOS_LIKE_SOURCE}/source/dos.c")
		foreach(EXAMPLE ${EXAMPLES})
			add_executable(${EXAMPLE} "${DOS_LIKE_SOURCE}/source/dos.h" "${DOS_LIKE_SOURCE}/source/${EXAMPLE}.c")
			target_link_libraries(${EXAMPLE} PRIVATE ${CMAKE_PROJECT_NAME})
		endforeach()
	else()
		message(FATAL_ERROR "On Windows, only compiling with MSVC is supported.")
	endif()
else()
	function(link_app_sdl_libraries TARGET MAIN)
		# Library handling that works for all platforms, because this only uses
		# built-in CMake features.

		find_package(GLEW REQUIRED)
		set(OpenGL_GL_PREFERENCE GLVND)
		find_package(OpenGL REQUIRED)
		find_package(Threads REQUIRED)
		find_library(MATH_LIBRARY m REQUIRED)

		target_link_libraries(${TARGET} PRIVATE
			${CMAKE_DL_LIBS}
			GLEW::glew
			${OPENGL_LIBRARIES}
			Threads::Threads
			${MATH_LIBRARY}
		)

		# SDL 2.0 library handling. pkg-config and normal find_package options
		# are used, which covers most of the relevant systems.

		find_package(PkgConfig)
		if(PKG_CONFIG_FOUND)
			# If pkg-config is available, just use it for SDL2, as it's guaranteed to work.

			function(target_pkg_config_import TARGET DEPENDENCY)
				string(TOUPPER ${DEPENDENCY} DEPENDENCY_VARIABLE)
				pkg_check_modules(${DEPENDENCY_VARIABLE} REQUIRED ${DEPENDENCY})
				target_compile_options(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_CFLAGS}")
				target_compile_options(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_CFLAGS_OTHER}")
				target_include_directories(${TARGET} SYSTEM PRIVATE "${${DEPENDENCY_VARIABLE}_INCLUDE_DIRS}")
				target_link_directories(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_LIBRARY_DIRS}")
				target_link_libraries(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_LIBRARIES}")
				target_link_options(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_LDFLAGS}")
				target_link_options(${TARGET} PRIVATE "${${DEPENDENCY_VARIABLE}_LDFLAGS_OTHER}")
			endfunction()

			set(DEPENDENCIES
				sdl2
			)
			foreach(DEPENDENCY ${DEPENDENCIES})
				target_pkg_config_import(${TARGET} ${DEPENDENCY})
			endforeach()
		else()
			# This *can* work without pkg-config, but only if SDL 2.0 is
			# available with find_package(). The pkg-config version above
			# is for systems with older SDL 2.0 versions, that don't come
			# with CMake support.

			find_package(SDL2 REQUIRED)

			target_link_libraries(${TARGET} PRIVATE
				SDL2::SDL2
			)
			if(MAIN)
				target_link_libraries(${TARGET} SDL2::SDL2main)
			endif()
		endif()
	endfunction()

	add_library(${CMAKE_PROJECT_NAME} STATIC "${DOS_LIKE_SOURCE}/source/dos.h" "${DOS_LIKE_SOURCE}/source/dos.c")
	link_app_sdl_libraries(${CMAKE_PROJECT_NAME} FALSE)

	foreach(EXAMPLE ${EXAMPLES})
		add_executable(${EXAMPLE} "${DOS_LIKE_SOURCE}/source/dos.h" "${DOS_LIKE_SOURCE}/source/${EXAMPLE}.c")
		target_link_libraries(${EXAMPLE} PRIVATE ${CMAKE_PROJECT_NAME})
		link_app_sdl_libraries(${EXAMPLE} TRUE)
	endforeach()
endif()
