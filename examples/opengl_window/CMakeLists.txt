
INCLUDE_DIRECTORIES(
    ../../third_party/glad
    ../../third_party
)

FILE(GLOB OpenGLWindow_HDRS "*.h" )

FILE(GLOB OpenGLWindowMac_CPP "tiny_mac*.cpp")
FILE(GLOB OpenGLWindowMacObjC_CPP "tiny_mac*.m")
FILE(GLOB OpenGLWindowWin32_CPP "tiny_win32*.cpp")
FILE(GLOB OpenGLWindowLinux_CPP "tiny_x11*.cpp")

FILE(GLOB OpenGLWindowCommon_CPP "*.cpp" )

LIST(REMOVE_ITEM OpenGLWindowCommon_CPP ${OpenGLWindowMac_CPP} )
LIST(REMOVE_ITEM OpenGLWindowCommon_CPP ${OpenGLWindowWin32_CPP} )
LIST(REMOVE_ITEM OpenGLWindowCommon_CPP ${OpenGLWindowLinux_CPP} )
LIST(REMOVE_ITEM OpenGLWindowCommon_CPP tiny_x11_opengl_window.cpp )
LIST(REMOVE_ITEM OpenGLWindowCommon_CPP tiny_mac_opengl_window.cpp )
#MESSAGE (${OpenGLWindowCommon_CPP})

IF (WIN32)
    SET(OpenGLWindow_SRCS ../../third_party/glad/gl.c ${OpenGLWindowWin32_CPP} ${OpenGLWindowCommon_CPP})
	  ADD_DEFINITIONS(-DGLEW_STATIC)
ELSE()
  IF(APPLE)
  MESSAGE("APPLE!")
  SET(OpenGLWindow_SRCS ${OpenGLWindowMac_CPP} ../../third_party/glad/gl.c ${OpenGLWindowMacObjC_CPP}  ${OpenGLWindowCommon_CPP} )
ELSE()
  MESSAGE("LINUX!")
  SET(OpenGLWindow_SRCS ${OpenGLWindowLinux_CPP} ${OpenGLWindowCommon_CPP} ../../third_party/glad/gl.c)
  ENDIF()
ENDIF(WIN32)

#no Linux detection?
IF(NOT WIN32 AND NOT APPLE)
	INCLUDE_DIRECTORIES(
		../../third_party/glad
		../../third_party/optionalX11
        )
	ADD_DEFINITIONS(-DGLEW_STATIC)
	ADD_DEFINITIONS("-DGLEW_INIT_OPENGL11_FUNCTIONS=1")
  ADD_DEFINITIONS("-DGLEW_DYNAMIC_LOAD_ALL_GLX_FUNCTIONS=1")
	ADD_DEFINITIONS("-DDYNAMIC_LOAD_X11_FUNCTIONS=1")

        SET(OpenGLWindow_SRCS ${OpenGLWindowLinux_CPP} ${OpenGLWindowCommon_CPP}  ../../third_party/glad/gl.c  ../../third_party/glad/glx.c )
ENDIF()

MESSAGE (${OpenGLWindowCommon_CPP})

IF(BUILD_EGL)
    SET(OpenGLWindow_SRCS ${OpenGLWindow_SRCS}  ../../third_party/glad/egl.c)
ENDIF(BUILD_EGL)

SET(OpenGLWindow_SRCS ${OpenGLWindow_SRCS}  ../../third_party/stb_image/stb_image_write.cpp)

MESSAGE("OpenGLWindow_SRCS=${OpenGLWindow_SRCS}")

ADD_LIBRARY(opengl_window ${OpenGLWindow_SRCS} ${OpenGLWindow_HDRS})
if (UNIX AND NOT APPLE)
  target_link_libraries(opengl_window ${DL})
elseif (APPLE)
  find_library(COCOA NAMES Cocoa)
  MESSAGE(${COCOA})
endif ()

if (BUILD_SHARED_LIBS)
  #target_link_libraries(opengl_window )
  if (WIN32 OR APPLE)
   target_link_libraries(opengl_window ${OPENGL_gl_LIBRARY})
  else()
	set (CMAKE_THREAD_PREFER_PTHREAD TRUE)
	FIND_PACKAGE(Threads)
	target_link_libraries(opengl_window ${CMAKE_THREAD_LIBS_INIT})
	target_compile_definitions(opengl_window PUBLIC DYNAMIC_LOAD_X11_FUNCTIONS PRIVATE DYNAMIC_LOAD_X11_FUNCTIONS)

  endif()
endif()

IF(APPLE)
  FIND_PACKAGE(OpenGL)
  target_link_libraries(opengl_window ${COCOA} ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY})
ENDIF()

#target_link_libraries(opengl_window ${OPENGL_gl_LIBRARY})

INSTALL(TARGETS opengl_window
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib${LIB_SUFFIX}
        ARCHIVE DESTINATION lib${LIB_SUFFIX})
