cmake_minimum_required(VERSION 3.22)
project(QGLViewer VERSION 2.8.0)

# Same checks as Sofa.GUI.Qt
# i.e find Qt6, then if not, Qt5, then if not error
find_package(Qt6 COMPONENTS Core CoreTools QUIET)
if (NOT Qt6Core_FOUND)
    find_package(Qt5 COMPONENTS Core QUIET)
endif()

if (Qt5Core_FOUND)
    sofa_find_package(Qt5 COMPONENTS Core Gui Xml OpenGL Widgets REQUIRED)
    set(QGLViewer_QT_TARGETS Qt5::Core Qt5::Gui Qt5::Xml Qt5::OpenGL Qt5::Widgets)
elseif (Qt6Core_FOUND)
    sofa_find_package(Qt6 COMPONENTS Gui GuiTools Widgets WidgetsTools OpenGLWidgets Xml REQUIRED)
    set(QGLViewer_QT_TARGETS ${QGLViewer_QT_TARGETS} Qt::Core Qt::Gui Qt::Widgets Qt::OpenGLWidgets Qt::Xml)
endif()

set(MOC_HEADER_FILES
    QGLViewer/camera.h
    QGLViewer/frame.h
    QGLViewer/keyFrameInterpolator.h
    QGLViewer/manipulatedCameraFrame.h
    QGLViewer/manipulatedFrame.h
    QGLViewer/qglviewer.h
    )
# These header files don't need to be processed by the moc.
set(HEADER_FILES
    QGLViewer/VRender/AxisAlignedBox.h
    QGLViewer/VRender/Exporter.h
    QGLViewer/VRender/NVector3.h
    QGLViewer/VRender/Optimizer.h
    QGLViewer/VRender/ParserGL.h
    QGLViewer/VRender/Primitive.h
    QGLViewer/VRender/PrimitivePositioning.h
    QGLViewer/VRender/SortMethod.h
    QGLViewer/VRender/Types.h
    QGLViewer/VRender/VRender.h
    QGLViewer/VRender/Vector2.h
    QGLViewer/VRender/Vector3.h
    QGLViewer/VRender/gpc.h
    QGLViewer/config.h
    QGLViewer/constraint.h
    QGLViewer/domUtils.h
    QGLViewer/mouseGrabber.h
    QGLViewer/quaternion.h
    QGLViewer/vec.h
    QGLViewer/camera.h
    QGLViewer/frame.h
    QGLViewer/keyFrameInterpolator.h
    QGLViewer/manipulatedCameraFrame.h
    QGLViewer/manipulatedFrame.h
    QGLViewer/qglviewer.h
    )
set(SOURCE_FILES
    QGLViewer/VRender/BSPSortMethod.cpp
    QGLViewer/VRender/BackFaceCullingOptimizer.cpp
    QGLViewer/VRender/EPSExporter.cpp
    QGLViewer/VRender/Exporter.cpp
    QGLViewer/VRender/FIGExporter.cpp
    QGLViewer/VRender/NVector3.cpp
    QGLViewer/VRender/ParserGL.cpp
    QGLViewer/VRender/Primitive.cpp
    QGLViewer/VRender/PrimitivePositioning.cpp
    QGLViewer/VRender/TopologicalSortMethod.cpp
    QGLViewer/VRender/VRender.cpp
    QGLViewer/VRender/Vector2.cpp
    QGLViewer/VRender/Vector3.cpp
    QGLViewer/VRender/VisibilityOptimizer.cpp
    QGLViewer/VRender/gpc.cpp
    QGLViewer/camera.cpp
    QGLViewer/constraint.cpp
    QGLViewer/frame.cpp
    QGLViewer/keyFrameInterpolator.cpp
    QGLViewer/manipulatedCameraFrame.cpp
    QGLViewer/manipulatedFrame.cpp
    QGLViewer/mouseGrabber.cpp
    QGLViewer/qglviewer.cpp
    QGLViewer/quaternion.cpp
    QGLViewer/saveSnapshot.cpp
    QGLViewer/vec.cpp
    )
set(UI_FILES
    QGLViewer/ImageInterface.ui
    QGLViewer/VRenderInterface.ui
    )

if (Qt5Core_FOUND)
    qt5_wrap_cpp(MOC_FILES ${MOC_HEADER_FILES})
    qt5_wrap_ui(FORM_FILES ${UI_FILES})
elseif (Qt6Core_FOUND)
    qt6_wrap_cpp(MOC_FILES ${MOC_HEADER_FILES})
    qt6_wrap_ui(FORM_FILES ${UI_FILES})
endif()

add_library(${PROJECT_NAME} SHARED ${HEADER_FILES} ${MOC_FILES} ${FORM_FILES} ${SOURCE_FILES})

set_target_properties(${PROJECT_NAME} PROPERTIES FOLDER "SofaExtlibs")
target_link_libraries(${PROJECT_NAME} PUBLIC ${QGLViewer_QT_TARGETS})
if(TARGET OpenGL::GL AND TARGET OpenGL::GLU) # Imported targets defined since CMake 3.8
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL::GL OpenGL::GLU)
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC ${OPENGL_LIBRARIES})
    target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC ${OPENGL_INCLUDE_DIR})
endif()

target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>")
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC "$<INSTALL_INTERFACE:include/extlibs/${PROJECT_NAME}>")

if(WIN32)
    target_compile_options(${PROJECT_NAME} PRIVATE "-DCREATE_QGLVIEWER_DLL")
    target_compile_options(${PROJECT_NAME} PRIVATE "/wd4996") # remove warnings about deprecation (CRT,etc)
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES DEBUG_POSTFIX "_d")

sofa_create_package_with_targets(
    PACKAGE_NAME ${PROJECT_NAME}
    PACKAGE_VERSION ${PROJECT_VERSION}
    TARGETS ${PROJECT_NAME}
    INCLUDE_INSTALL_DIR "extlibs"
    )

# Qt deployment
if(CMAKE_SYSTEM_NAME STREQUAL Windows)
    include(windeployqt)
    windeployqt(${PROJECT_NAME} ${CMAKE_BINARY_DIR}/bin bin)
endif()
