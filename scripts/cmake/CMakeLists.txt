# openFrameworks CMake files
# --------------------------
# This file contains a set of macros which make it  
# easy to add new addons, include paths, linkerfiles etc.. to your 
# openFrameworks application.
#
# What means in practice, is that you never need to configure (but still can)
# your IDE (like XCode/VS). You use a tiny (~10 lines) CMakeFile for you 
# emptyExamply application and use this to either generate a project file
# for your favorite IDE or you run it from the command line.
#
# Adding new addons will be piece of cake, you don't need to setup
# any include paths, linker settings etc. You just call of_addon_add(openCV)
# and everything works, cross platform!
#
# 
# Macros used for OF
# ------------------
# 
#      of_add_install_file(source dest):  
#      --------------------------------
#      When installing we copy `source` to `dest`, this is for example
#      used to copy the necessary DLL files, or dylibs on Mac. This can
#      be any file on your hard disk.
#
#
#      of_add_install_dir(source dest):
#      --------------------------------
#      Copies a complete directory from `source` to `dest` when installing.
#      This can be any directory on your hard disk.
#
#
#      of_add_extern_install_file(source dest):
#      --------------------------
#      The `extern` directory is the `of\libs\` directory which contains 
#      the third party libs that OF uses. Like `tess2`, `FreeImage` etc..
#
#
#      of_install_app():
#      -----------------
#      This is an wrapper which does a lot of actuall work. It will make 
#      sure all necessary files are copied and an executable will be created
#      and linked against all necessary libraries. This is the 'core' of 
#      these cmake files.
#
#
#      of_add_extern_lib(lib):
#      ----------------------
#      Add a library to link with from the `of\libs\` dir.
#
#
#      of_add_extern_debug_lib(lib):
#      -----------------------------
#      Add a debug library from the `of\libs\` dir.
#
#
#      of_add_lib(lib):
#      ----------------
#      Add a general linker lib. 
#
#
#      of_add_extern_dll(dll):
#      ----------------
#      Add a dll from the OF-dll dir, which will be copied to the install dir.
#
#
#      of_add_extern_include_dir(dir):
#      -------------------------------
#      Adds a include dir from the `of\libs\` dir.
#
#      
#      of_app_add_source_file(file):
#      -----------------------------
#      Add a source file which must be stored under `[appdir]\src\`
#      For example: of_app_add_source_file(testApp.cpp)
#
#
#      of_add_of_include_dir(dir):
#      ----------------------
#      Add a include directory for one of the subdirs in `of\libs\openFrameworks\`
#
#
#      of_app_initialize(app_dir app_name):
#      -------------------------------------
#      Must be called first thing in a cmakelists.txt file for an application.
#      The first parameter is the directory to the 'base' dir of the application,
#      So the one containing `bin\`, `src\` etc.. The `app_name` is the name of 
#      the application; by default `testApp`
#
#
#
#

cmake_minimum_required(VERSION 2.8)


# ------------------------------------------------------------------------------
# O P E N F R A M E W O R K   M A C R O S
# ------------------------------------------------------------------------------
macro(of_add_install_file source dest)
  list(APPEND of_install_source_files ${source})
  list(APPEND of_install_dest_files ${dest})
endmacro(of_add_install_file)

macro(of_add_install_dir source dest)
  list(APPEND of_install_source_dirs ${source})
  list(APPEND of_install_dest_dirs ${dest})
endmacro(of_add_install_dir)

macro(of_add_extern_install_file source dest)
  of_add_install_file(${of_extern_dir}/${source} ${dest})
endmacro(of_add_extern_install_file)

macro(of_add_extern_install_dir source dest)
  of_add_install_dir(${of_extern_dir}/${source} ${dest})
endmacro(of_add_extern_install_dir)

macro(of_install_app)
  if(WIN32)
    add_definitions(-D_WINSOCKAPI_)
    install(FILES ${of_dlls} DESTINATION ${of_app_install_dir})

    # make sure the exe is stored in /bin/
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${of_app_install_dir})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${of_app_install_dir})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${of_app_install_dir})

  endif()

  if(APPLE)
    add_executable(${of_app_name} MACOSX_BUNDLE ${of_app_sources})
  elseif(WIN32)
    add_executable(${of_app_name} ${of_app_sources})
    set_target_properties(${of_app_name} PROPERTIES OUTPUT_NAME_DEBUG ${of_app_name}_debug)
    set_target_properties(${of_app_name} PROPERTIES OUTPUT_NAME_RELEASE ${of_app_name}_release)
  elseif(UNIX AND NOT APPLE)
    add_executable(${of_app_name} MACOSX_BUNDLE ${of_app_sources})
  endif()

  target_link_libraries(${of_app_name} general ${of_libs})

  if(${of_debug_libs})
    target_link_libraries(${of_app_name} debug ${of_debug_libs})
  endif()

  install(TARGETS ${of_app_name} DESTINATION ${of_app_install_dir})

  list(LENGTH of_install_source_files source_len)
  math(EXPR list_max_index ${source_len}-1)
  foreach(i RANGE ${list_max_index})
    list(GET of_install_source_files ${i} source_file)
    list(GET of_install_dest_files ${i} dest_file)
    if(APPLE)
      install(FILES ${source_file} DESTINATION ${of_app_install_dir}/${of_app_name}.app/${dest_file})
    endif()
  endforeach(i)

  list(LENGTH of_install_source_dirs source_len)
  math(EXPR list_max_index ${source_len}-1)
  foreach(i RANGE ${list_max_index})
    list(GET of_install_source_dirs ${i} source_dir)
    list(GET of_install_dest_dirs ${i} dest_dir)
    if(APPLE)
      install(DIRECTORY ${source_dir} DESTINATION ${of_app_install_dir}/${of_app_name}.app/${dest_dir})
    endif()
  endforeach(i)

  if(APPLE)
    add_custom_command(TARGET ${of_app_name}
      POST_BUILD COMMAND 
      ${CMAKE_INSTALL_NAME_TOOL} -change "./libfmodex.dylib" "@executable_path/libfmodex.dylib" $<TARGET_FILE:${of_app_name}>)
  endif()

endmacro(of_install_app)

macro(of_add_extern_lib lib)
  list(APPEND of_libs ${of_extern_dir}/${lib})
endmacro(of_add_extern_lib)

macro(of_add_extern_debug_lib lib)
  list(APPEND of_debug_libs ${of_extern_dir}/${lib})
endmacro(of_add_extern_debug_lib)

macro(of_add_extern_link_dir dir)
  link_directories(${of_extern_dir}/${dir})
endmacro(of_add_extern_link_dir)

macro(of_add_lib lib)
  list(APPEND of_libs ${lib})
endmacro(of_add_lib)

macro(of_add_extern_include_dir dir)
  include_directories(${of_extern_dir}/${dir})
endmacro(of_add_extern_include_dir)

macro(of_app_add_source_file src_file)
  source_group("src\\application" FILES ${src_file})

  if(of_app_sources) 
    list(APPEND of_app_sources ${of_app_src_dir}/${src_file})
  else()
    set(of_app_sources ${of_app_src_dir}/${src_file})
  endif()
endmacro(of_app_add_source_file)

macro(of_app_add_include_dir dir) 
  include_directories(${of_app_src_dir}/${dir})
endmacro()

macro(of_add_source_files files)
    list(APPEND of_app_sources ${${files}})
endmacro()

macro(of_app_add_source_files)
  file(GLOB_RECURSE app_source_files ${of_app_src_dir}/*.cpp)
  if(of_app_sources)
    list(APPEND of_app_sources ${app_source_files})
  else()
    set(of_app_sources ${app_source_files})
  endif()
endmacro(of_app_add_source_files)

macro(of_add_include_dir dir) 
    include_directories(${dir})
endmacro(of_add_include_dir)

macro(of_add_of_include_dir dir)
    include_directories(${of_inc_dir}/${dir})
endmacro(of_add_of_include_dir)

macro(of_add_extern_dll dll)
  list(APPEND of_dlls ${of_base_dir}/export/vs2010/${dll})
endmacro(of_add_extern_dll)

macro(of_add_exe_linker_flag flag)
   set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${flag}")
endmacro(of_add_exe_linker_flag)

macro(of_add_exe_linker_flag_release flag)
   set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS_RELEASE} ${flag}")
endmacro(of_add_exe_linker_flag_release)

macro(of_add_exe_linker_flag_debug flag)
   set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} ${flag}")
endmacro(of_add_exe_linker_flag_debug)

macro(of_app_initialize app_name)
  set(of_app_name ${app_name})
  set(of_app_base_dir ${CMAKE_CURRENT_LIST_DIR}/../../)
  set(of_app_src_dir ${of_app_base_dir}/src)
  set(of_app_include_dir ${of_app_base_dir}/src/)
  set(of_app_install_dir ${of_app_base_dir}/bin/)
  set(of_app_install_dir_windows ${of_app_base_dir}/scripts/cmake/build/)
  include_directories(${of_app_include_dir})
endmacro(of_app_initialize)

macro(of_app_add_addon name)
  include(${of_base_dir}/addons/${name}/scripts/cmake/CMakeLists.txt)
endmacro()

macro(of_addon_begin name)
  set(of_addon_name ${name})
  set(of_addon_base_dir ${CMAKE_CURRENT_LIST_DIR}/../../)
endmacro(of_addon_begin)

macro(of_addon_add_source_file src_file)
  list(APPEND of_addon_source_files ${of_addon_base_dir}/${src_file})
endmacro(of_addon_add_source_file)

macro(of_addon_add_include_dir dir)
#  message(FATAL_ERROR ${of_addon_base_dir} "<----------------")
  of_add_include_dir(${of_addon_base_dir}/${dir})
endmacro(of_addon_add_include_dir)

macro(of_addon_add_lib lib)
  of_add_lib(${of_addon_base_dir}/${lib})
endmacro(of_addon_add_lib)

macro(of_addon_end)
  add_library(${of_addon_name} ${of_addon_source_files})
  set(${of_addon_name} "")
  set(${of_addon_source_files} "")
  of_add_lib(${of_addon_name})
endmacro(of_addon_end)

set(of_base_dir ${CMAKE_CURRENT_LIST_DIR}/../../)
set(of_extern_dir ${of_base_dir}/libs)
set(of_src_dir ${of_base_dir}/libs/openFrameworks)
set(of_inc_dir ${of_base_dir}/libs/openFrameworks)

# ------------------------------------------------------------------------------
# O P E N F R A M E W O R K   P R O J E C T   S E T U P 
# ------------------------------------------------------------------------------

# Extern libs
if(APPLE)
  of_add_extern_include_dir(assimp/include/)
  of_add_extern_include_dir(cairo/include/)
  of_add_extern_include_dir(cairo/include/cairo/)
  of_add_extern_include_dir(cairo/include/libpng15/)
  of_add_extern_include_dir(cairo/include/pixman-1)
  of_add_extern_include_dir(fmodex/include/)
  of_add_extern_include_dir(FreeImage/include/)
  of_add_extern_include_dir(freetype/include)
  of_add_extern_include_dir(freetype/include/freetype2/)
  of_add_extern_include_dir(glew/include/)
  of_add_extern_include_dir(glut/include/)
  of_add_extern_include_dir(poco/include/)
  of_add_extern_include_dir(rtAudio/include/)
  of_add_extern_include_dir(tess2/include/)
endif()

if(WIN32)
  of_add_extern_include_dir(glut/include/)
  of_add_extern_include_dir(rtAudio/include/)
  of_add_extern_include_dir(quicktime/include)
  of_add_extern_include_dir(freetype/include)
  of_add_extern_include_dir(freetype/include/freetype2/)
  of_add_extern_include_dir(FreeImage/include/)
  of_add_extern_include_dir(fmodex/include/)
  of_add_extern_include_dir(videoInput/include/)
  of_add_extern_include_dir(glew/include/)
  of_add_extern_include_dir(glu/include/)
  of_add_extern_include_dir(tess2/include/)
  of_add_extern_include_dir(cairo/include/cairo/)
  of_add_extern_include_dir(poco/include/)
endif()

if(UNIX AND NOT APPLE) 
  of_add_extern_include_dir(FreeImage/include/)
  of_add_extern_include_dir(assimp/include/)
  of_add_extern_include_dir(assimp/include/Compiler)
  of_add_extern_include_dir(cairo/include/)
  of_add_extern_include_dir(cairo/include/pixman-1)
  of_add_extern_include_dir(cairo/include/cairo/)
  of_add_extern_include_dir(cairo/include/libpng15/)
  of_add_extern_include_dir(fmodex/include/)
  of_add_extern_include_dir(freetype/include)
  of_add_extern_include_dir(freetype/include/freetype2/)
  of_add_extern_include_dir(freetype/include/freetype2/freetype/config)
  of_add_extern_include_dir(freetype/include/freetype2/freetype/internal)
  of_add_extern_include_dir(freetype/include/freetype2/freetype/internal/services)
  of_add_extern_include_dir(kiss/include/)
  of_add_extern_include_dir(portaudio/include/)
  of_add_extern_include_dir(rtAudio/include/)
  of_add_extern_include_dir(videoInput/include/)
  of_add_extern_include_dir(poco/include/)
  of_add_extern_include_dir(glu/include)
  of_add_extern_include_dir(tess2/include)

  # not sure about these 
  of_add_include_dir(/usr/include/gstreamer-1.0)
  of_add_include_dir(/usr/include/glib-2.0)
  of_add_include_dir(/usr/lib/x86_64-linux-gnu/glib-2.0/include)
  of_add_include_dir(/usr/include/GL)
  of_add_include_dir(/usr/include/gtk-2.0)
  of_add_include_dir(/usr/lib/x86_64-linux-gnu/gtk-2.0/include)
  of_add_include_dir(/usr/include/atk-1.0)
  of_add_include_dir(/usr/include/cairo)
  of_add_include_dir(/usr/include/gdk-pixbuf-2.0)
  of_add_include_dir(/usr/include/pango-1.0)
  of_add_include_dir(/usr/include/gio-unix-2.0)
  of_add_include_dir(/usr/include/glib-2.0)
#  of_add_include_dir(/usr/lib/x86_64-linux-gnu/glib-2.0)
  of_add_include_dir(/usr/include/pixman-1)
  of_add_include_dir(/usr/include/freetype2) # whu include this here + in OF ? 
  of_add_include_dir(/usr/include/libpng12)  # why not 15
  of_add_of_include_dir(.settings)
endif()

set(of_sources_3d
  ${of_src_dir}/3d/of3dUtils.cpp
  ${of_src_dir}/3d/ofCamera.cpp
  ${of_src_dir}/3d/ofEasyCam.cpp
  ${of_src_dir}/3d/ofMesh.cpp
  ${of_src_dir}/3d/ofNode.cpp
)

set(of_sources_app
  ${of_src_dir}/app/ofAppGlutWindow.cpp
  ${of_src_dir}/app/ofAppRunner.cpp
)

set(of_sources_communication
  ${of_src_dir}/communication/ofArduino.cpp
  ${of_src_dir}/communication/ofSerial.cpp
)

set(of_sources_events
  ${of_src_dir}/events/ofEvents.cpp
)

set(of_sources_gl
  ${of_src_dir}/gl/ofFbo.cpp
  ${of_src_dir}/gl/ofGLRenderer.cpp
  ${of_src_dir}/gl/ofLight.cpp
  ${of_src_dir}/gl/ofMaterial.cpp
  ${of_src_dir}/gl/ofShader.cpp
  ${of_src_dir}/gl/ofTexture.cpp
  ${of_src_dir}/gl/ofVbo.cpp
  ${of_src_dir}/gl/ofVboMesh.cpp
)

set(of_sources_graphics
  ${of_src_dir}/graphics/ofBitmapFont.cpp
  ${of_src_dir}/graphics/ofCairoRenderer.cpp
  ${of_src_dir}/graphics/ofGraphics.cpp
  ${of_src_dir}/graphics/ofImage.cpp
  ${of_src_dir}/graphics/ofPath.cpp
  ${of_src_dir}/graphics/ofPixels.cpp
  ${of_src_dir}/graphics/ofPolyline.cpp
  ${of_src_dir}/graphics/ofTessellator.cpp
  ${of_src_dir}/graphics/ofTrueTypeFont.cpp
)

set(of_sources_math
  ${of_src_dir}/math/ofMath.cpp
  ${of_src_dir}/math/ofMatrix3x3.cpp
  ${of_src_dir}/math/ofMatrix4x4.cpp
  ${of_src_dir}/math/ofQuaternion.cpp
  ${of_src_dir}/math/ofVec2f.cpp
  ${of_src_dir}/math/ofVec4f.cpp
)

set(of_sources_sound
  ${of_src_dir}/sound/ofBaseSoundPlayer.cpp
  ${of_src_dir}/sound/ofBaseSoundStream.cpp
  ${of_src_dir}/sound/ofFmodSoundPlayer.cpp
  ${of_src_dir}/sound/ofOpenALSoundPlayer.cpp
  ${of_src_dir}/sound/ofPASoundStream.cpp
  ${of_src_dir}/sound/ofRtAudioSoundStream.cpp
  ${of_src_dir}/sound/ofSoundPlayer.cpp
  ${of_src_dir}/sound/ofSoundStream.cpp
)

set(of_sources_types
  ${of_src_dir}/types/ofBaseTypes.cpp
  ${of_src_dir}/types/ofColor.cpp
  ${of_src_dir}/types/ofPoint.cpp
  ${of_src_dir}/types/ofRectangle.cpp
)

set(of_sources_utils
  ${of_src_dir}/utils/ofFileUtils.cpp
  ${of_src_dir}/utils/ofLog.cpp
  ${of_src_dir}/utils/ofSystemUtils.cpp
  ${of_src_dir}/utils/ofThread.cpp
  ${of_src_dir}/utils/ofURLFileLoader.cpp
  ${of_src_dir}/utils/ofUtils.cpp
)

set(of_sources_video
  ${of_src_dir}/video/ofDirectShowGrabber.cpp

  ${of_src_dir}/video/ofQtUtils.cpp
  ${of_src_dir}/video/ofQuickTimeGrabber.cpp
  ${of_src_dir}/video/ofQuickTimePlayer.cpp
  ${of_src_dir}/video/ofVideoGrabber.cpp
  ${of_src_dir}/video/ofVideoPlayer.cpp
)
if(APPLE)
  list(APPEND of_sources_video
    ${of_src_dir}/video/ofQTKitGrabber.mm  
    ${of_src_dir}/video/ofQTKitMovieRenderer.m
    ${of_src_dir}/video/ofQTKitPlayer.mm
    )

  find_library(fr_core_foundation CoreFoundation)
  find_library(fr_cocoa Cocoa)
  find_library(fr_avfoundation AVFoundation)
  find_library(fr_core_video CoreVideo)
  find_library(fr_core_media CoreMedia)
  find_library(fr_core_media_io CoreMediaIO)
  find_library(fr_qtkit QTKit)
  find_library(fr_qtime QuickTime)
  find_library(fr_accel Accelerate)
  find_library(fr_appkit AppKit)
  find_library(fr_iokit IOKit)
  find_library(fr_agl AGL)
  find_library(fr_app_services ApplicationServices)
  find_library(fr_audio_toolbox AudioToolbox)
  find_library(fr_carbon Carbon)
  find_library(fr_core_audio CoreAudio)

  of_add_lib(${fr_core_foundation})
  of_add_lib(${fr_cocoa})
  of_add_lib(${fr_avfoundation})
  of_add_lib(${fr_core_video})
  of_add_lib(${fr_core_media})
  of_add_lib(${fr_core_media_io})
  of_add_lib(${fr_qtime})
  of_add_lib(${fr_qtkit})
  of_add_lib(${fr_accel})
  of_add_lib(${fr_appkit})
  of_add_lib(${fr_iokit})
  of_add_lib(${fr_agl})
  of_add_lib(${fr_app_services})
  of_add_lib(${fr_audio_toolbox})
  of_add_lib(${fr_carbon})
  of_add_lib(${fr_core_audio})

endif()

if(UNIX AND NOT APPLE)
  list(APPEND of_sources_video 
    ${of_src_dir}/video/ofGstUtils.cpp
    ${of_src_dir}/video/ofGstVideoGrabber.cpp
    ${of_src_dir}/video/ofGstVideoPlayer.cpp
    )
endif()

set(of_sources
  ${of_sources_3d}
  ${of_sources_app}
  ${of_sources_communication}
  ${of_sources_events}
  ${of_sources_gl}
  ${of_sources_graphics}
  ${of_sources_math}
  ${of_sources_sound}
  ${of_sources_types}
  ${of_sources_utils}
  ${of_sources_video}
  )

source_group("src\\of\\3d" FILES ${of_sources_3d})
source_group("src\\of\\app" FILES ${of_sources_app})
source_group("src\\of\\communication" FILES ${of_sources_communication})
source_group("src\\of\\events" FILES ${of_sources_events})
source_group("src\\of\\gl" FILES ${of_sources_gl})
source_group("src\\of\\graphics" FILES ${of_sources_graphics})
source_group("src\\of\\math" FILES ${of_sources_math})
source_group("src\\of\\sound" FILES ${of_source_sound})
source_group("src\\of\\types" FILES ${of_sources_types})
source_group("src\\of\\utils" FILES ${of_sources_utils})
source_group("src\\of\\video" FILES ${of_sources_video})

add_library(openframeworks_types ${of_sources_types})
add_library(openframeworks_3d ${of_sources_3d})
add_library(openframeworks_app ${of_sources_app})
add_library(openframeworks_communication ${of_sources_communication})
add_library(openframeworks_events ${of_sources_events})
add_library(openframeworks_gl ${of_sources_gl})
add_library(openframeworks_graphics ${of_sources_graphics})
add_library(openframeworks_math ${of_sources_math})
add_library(openframeworks_sound ${of_sources_sound})
add_library(openframeworks_utils ${of_sources_utils})
add_library(openframeworks_video ${of_sources_video})

of_add_lib(openframeworks_types)
of_add_lib(openframeworks_3d)
of_add_lib(openframeworks_app)
of_add_lib(openframeworks_communication)
of_add_lib(openframeworks_events)
of_add_lib(openframeworks_gl)
of_add_lib(openframeworks_graphics)
of_add_lib(openframeworks_math)
of_add_lib(openframeworks_sound)
of_add_lib(openframeworks_utils)
of_add_lib(openframeworks_video)

of_add_of_include_dir(.)
of_add_of_include_dir(3d)
of_add_of_include_dir(app)
of_add_of_include_dir(communication)
of_add_of_include_dir(events)
of_add_of_include_dir(gl)
of_add_of_include_dir(graphics)
of_add_of_include_dir(math)
of_add_of_include_dir(sound)
of_add_of_include_dir(types)
of_add_of_include_dir(utils)
of_add_of_include_dir(video)


# OF APPLE + XCODE/CLANG
if(APPLE)
  find_library(fr_carbon Carbon)
  find_library(fr_core_foundation CoreFoundation)
  find_library(fr_opengl OpenGL)
  of_add_lib(${fr_carbon})
  of_add_lib(${fr_core_foundation})
  of_add_lib(${fr_opengl})

  find_path(cocoa_inc_dir Cocoa/Cocoa.h)
  include_directories(${cocoa_inc_dir}/Headers/)

  # @todo - set correct flags here
  set_target_properties(openframeworks_types PROPERTIES COMPILE_FLAGS "-ObjC++")
  set_target_properties(openframeworks_utils PROPERTIES COMPILE_FLAGS "-x objective-c++ -arch i386")

  of_add_extern_install_file(fmodex/lib/osx/libfmodex.dylib Contents/MacOS/)
  of_add_extern_install_dir(glut/lib/osx/GLUT.framework/ Contents/Frameworks/GLUT.framework/)

  set(CMAKE_OSX_ARCHITECTURES "i386")
  set(CMAKE_C_FLAGS "-m32 -arch i386")
  set(CMAKE_CXX_FLAGS "-m32 -arch i386")

  of_add_extern_lib(fmodex/lib/osx/libfmodex.dylib)
  of_add_extern_lib(glut/lib/osx/GLUT.framework/Versions/A/GLUT)
  of_add_extern_lib(FreeImage/lib/osx/freeimage.a)
  of_add_extern_lib(freetype/lib/osx/freetype.a)
  of_add_extern_lib(glew/lib/osx/glew.a)
  of_add_extern_lib(poco/lib/osx/PocoCrypto.a)
  of_add_extern_lib(poco/lib/osx/PocoData.a)
  of_add_extern_lib(poco/lib/osx/PocoDataODBC.a)
  of_add_extern_lib(poco/lib/osx/PocoDataSQLite.a)
  of_add_extern_lib(poco/lib/osx/PocoFoundation.a)
  of_add_extern_lib(poco/lib/osx/PocoNet.a)
  of_add_extern_lib(poco/lib/osx/PocoNetSSL.a)
  of_add_extern_lib(poco/lib/osx/PocoUtil.a)
  of_add_extern_lib(poco/lib/osx/PocoXML.a)
  of_add_extern_lib(poco/lib/osx/PocoZip.a)
  of_add_extern_lib(rtAudio/lib/osx/rtAudio.a)
  of_add_extern_lib(tess2/lib/osx/tess2.a)
  of_add_extern_lib(cairo/lib/osx/cairo-script-interpreter.a)
  of_add_extern_lib(cairo/lib/osx/cairo.a)
  of_add_extern_lib(cairo/lib/osx/pixman-1.a)

endif()

# OF WIN32 + MSVC 2010 
if(WIN32)
  add_definitions(
    -DWIN32 
    -D_CONSOLE 
    -DPOCO_STATIC 
    -DCAIRO_WIN32_STATIC_BUILD 
    -DDISABLE_SOME_FLOATING_POINT
    -DUNICODE
    -D_UNICODE
    )

  of_add_extern_link_dir(poco/lib/vs2010)
  of_add_extern_link_dir(glut/lib/vs2010)

  of_add_extern_debug_lib(rtAudio/lib/vs2010/rtAudioD.lib)
  of_add_extern_lib(rtAudio/lib/vs2010/rtAudio.lib)

  of_add_exe_linker_flag("/NODEFAULTLIB:LIBC /NODEFAULTLIB:LIBCMT")
  of_add_exe_linker_flag_release(${of_extern_dir}/poco/lib/vs2010/PocoFoundationmd.lib)
  of_add_exe_linker_flag_release("/NODEFAULTLIB:rtAudioD.lib")
  of_add_exe_linker_flag_debug(${of_extern_dir}/poco/lib/vs2010/PocxfoFoundationmdd.lib)
  of_add_exe_linker_flag_debug("/NODEFAULTLIB:PocoFoundationmd.lib /NODEFAULTLIB:rtAudio.lib")

  of_add_extern_lib(cairo/lib/vs2010/pixman-1.lib)
  of_add_extern_lib(cairo/lib/vs2010/cairo-static.lib)
  of_add_extern_lib(fmodex/lib/vs2010/fmodex_vc.lib)
  of_add_extern_lib(fmodex/lib/vs2010/fmodexL_vc.lib)
  of_add_extern_lib(FreeImage/lib/vs2010/FreeImage.lib)
  of_add_extern_lib(freetype/lib/vs2010/libfreetype.lib)
  of_add_extern_lib(glew/lib/vs2010/glew32s.lib)
  of_add_extern_lib(glu/lib/vs2010/glu32.lib)
  of_add_extern_lib(quicktime/lib/vs2010/qtmlClient.lib)
  of_add_extern_lib(quicktime/lib/vs2010/QTSClient.lib)
  of_add_extern_lib(quicktime/lib/vs2010/Rave.lib)
  of_add_extern_lib(rtAudio/lib/vs2010/dsound.lib)
  of_add_extern_lib(tess2/lib/vs2010/tess2.lib)
  of_add_extern_lib(videoInput/lib/vs2010/videoInput.lib)

  of_add_lib(OpenGL32.lib)
  of_add_lib(msimg32.lib)

  of_add_extern_dll(Assimp32.dll)
  of_add_extern_dll(fmodex.dll)
  of_add_extern_dll(fmodexL.dll)
  of_add_extern_dll(FreeImage.dll)
  of_add_extern_dll(FreeType-6.dll)
  of_add_extern_dll(glut32.dll)
  of_add_extern_dll(Zlib.dll)

  add_definitions("/wd4244 /wd4756")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /D_ITERATOR_DEBUG_LEVEL=0")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /D_ITERATOR_DEBUG_LEVEL=2")

endif()

if(UNIX AND NOT APPLE)
  # from makeFilecommon
  #  of_add_lib(-jack)
  of_add_lib(-glu)
  of_add_lib(-glew)
 #  of_add_lib(gstreamer-1.0)
 #  of_add_lib(gstreamer-video-1.0)
 #  of_add_lib(streamer-base-1.0)
 #  of_add_lib(gstreamer-app-1.0)
 #  of_add_lib(libudev)
  of_add_lib(cairo)
 #  of_add_lib(zlib)
  of_add_lib(glut)
  of_add_lib(GL)
  of_add_lib(asound)
  of_add_lib(sndfile)
  of_add_lib(vorbis)
  of_add_lib(FLAC)
  of_add_lib(ogg)
  of_add_lib(freeimage)
  of_add_lib(portaudio)
  of_add_lib(freetype)

  set(linux_path linux64)
  of_add_extern_lib(poco/lib/${linux_path}/libPocoNet.a)
  of_add_extern_lib(poco/lib/${linux_path}/libPocoXML.a)
  of_add_extern_lib(poco/lib/${linux_path}/libPocoUtil.a)
  of_add_extern_lib(poco/lib/${linux_path}/libPocoFoundation.a)
 # of_add_extern_lib(poco/lib/{$linux_path}/)


endif()




