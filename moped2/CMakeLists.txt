cmake_minimum_required(VERSION 2.8.3)
project(moped2)

find_package(OpenCV REQUIRED)
find_package(GLEW REQUIRED)
find_package(DevIL REQUIRED)
find_package(GLUT REQUIRED)
find_package(OpenGL REQUIRED)

find_package(catkin REQUIRED COMPONENTS
	roscpp
	sensor_msgs
	pr_msgs
	cv_bridge
	image_transport
)


catkin_package(
  INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/libmoped/include
  CATKIN_DEPENDS 
	roscpp
	sensor_msgs
	pr_msgs
	cv_bridge
	image_transport
)





#add_custom_target(BUILD_LIBMOPED DEPENDS libmoped.a)
#add_custom_command(OUTPUT libmoped.a 
#                   DEPENDS ${PROJECT_SOURCE_DIR}/libmoped/src/moped.cpp
#                   DEPENDS ${PROJECT_SOURCE_DIR}/libmoped/src/config.hpp
#                   DEPENDS ${PROJECT_SOURCE_DIR}/libmoped/src/util.hpp
#                   DEPENDS ${PROJECT_SOURCE_DIR}/libmoped/src/STATUS_DISPLAY.hpp
#                   DEPENDS ${PROJECT_SOURCE_DIR}/libmoped/src/GLOBAL_DISPLAY.hpp
#                   COMMAND make
#                   WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/libmoped
#                   COMMENT Building libmoped.a)


#link_directories(${PROJECT_SOURCE_DIR}/libmoped/lib)

include_directories(
	libmoped/libs/libsiftfast-1.1-src/
	libmoped/libs/ann_1.1.1/
	libmoped/libs/ann_1.1.1/ANN
	libmoped/libs/levmar-2.4
	libmoped/libs/liblbfgs
	libmoped/libs/SiftGPU

	libmoped/include
	libmoped/src
)

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${GLEW_INCLUDE_DIRS}
  ${IL_INCLUDE_DIR}
  ${GLUT_INCLUDE_DIR}
  ${GL_INCLUDE_DIRS}
)

add_library(libsiftfast STATIC libmoped/libs/libsiftfast-1.1-src/libsiftfast.cpp)
set_target_properties(libsiftfast PROPERTIES COMPILE_FLAGS "-w -O3 -march=native -mfpmath=sse -fopenmp -I. -funroll-loops -ffast-math -fno-trapping-math -fsingle-precision-constant")

add_library(libann STATIC
libmoped/libs/ann_1.1.1/bd_search.cpp
libmoped/libs/ann_1.1.1/kd_pr_search.cpp
libmoped/libs/ann_1.1.1/kd_fix_rad_search.cpp
libmoped/libs/ann_1.1.1/kd_tree.cpp
libmoped/libs/ann_1.1.1/kd_dump.cpp
libmoped/libs/ann_1.1.1/bd_pr_search.cpp
libmoped/libs/ann_1.1.1/kd_search.cpp
libmoped/libs/ann_1.1.1/bd_tree.cpp
libmoped/libs/ann_1.1.1/ANN/ANN.cpp
libmoped/libs/ann_1.1.1/perf.cpp
libmoped/libs/ann_1.1.1/kd_split.cpp
libmoped/libs/ann_1.1.1/kd_util.cpp
libmoped/libs/ann_1.1.1/bd_fix_rad_search.cpp
libmoped/libs/ann_1.1.1/brute.cpp
libmoped/libs/ann_1.1.1/ANN/ANN.cpp
)
set_target_properties(libann PROPERTIES COMPILE_FLAGS "-w -O3 -march=native -mfpmath=sse -fopenmp -I. -Iutil -Inn -Ialgorithms -funroll-loops -ffast-math -fno-trapping-math -fsingle-precision-constant")

add_library(liblm STATIC
libmoped/libs/levmar-2.4/lmbc.c
libmoped/libs/levmar-2.4/lmblec.c
libmoped/libs/levmar-2.4/misc.c
libmoped/libs/levmar-2.4/Axb.c
libmoped/libs/levmar-2.4/lm.c
libmoped/libs/levmar-2.4/lmlec.c
)
set_target_properties(liblm PROPERTIES COMPILE_FLAGS "-w -O3 -march=native -mfpmath=sse -fopenmp -funroll-loops -ffast-math -fno-trapping-math -fsingle-precision-constant")

add_library(liblbfgs STATIC libmoped/libs/liblbfgs/lbfgs.c)
set_target_properties(liblbfgs PROPERTIES COMPILE_FLAGS "")

add_library(SiftGPU STATIC
libmoped/libs/SiftGPU/GLTexImage.cpp
libmoped/libs/SiftGPU/SiftMatch.cpp
libmoped/libs/SiftGPU/SiftPyramid.cpp
libmoped/libs/SiftGPU/ProgramGPU.cpp
libmoped/libs/SiftGPU/ShaderMan.cpp
libmoped/libs/SiftGPU/GlobalUtil.cpp
libmoped/libs/SiftGPU/FrameBufferObject.cpp
libmoped/libs/SiftGPU/ProgramGLSL.cpp
libmoped/libs/SiftGPU/SiftGPU.cpp
libmoped/libs/SiftGPU/PyramidGL.cpp
)
set_target_properties(SiftGPU PROPERTIES COMPILE_FLAGS "-w -O3 -march=native -mfpmath=sse -fopenmp -I. -Iutil -Inn -Ialgorithms -funroll-loops -ffast-math -fno-trapping-math -fsingle-precision-constant -DSIFTGPU_NO_CG")
target_link_libraries(SiftGPU   ${GLEW_LIBRARIES} ${IL_LIBRARIES} ${GLUT_LIBRARIES})

add_library(libmoped STATIC libmoped/src/moped.cpp)
set_target_properties(libmoped PROPERTIES COMPILE_FLAGS "-O3 -Wall -march=native -mfpmath=sse -Isrc -Iinclude -funroll-loops -fopenmp -ffast-math -fno-trapping-math")
target_link_libraries(libmoped
  libsiftfast
  libann
  liblm
  liblbfgs
  SiftGPU
  gomp
  ${OpenCV_LIBS}
  ${catkin_LIBRARIES}
  ${GLEW_LIBRARIES}
  ${IL_LIBRARIES}
  ${GLUT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  )

add_executable(moped2 moped.cpp)
set_target_properties(moped2 PROPERTIES COMPILE_FLAGS "-O3 -Wall -march=native -mfpmath=sse -fopenmp")
set_target_properties(moped2 PROPERTIES LINK_FLAGS "-Wl,--start-group")
target_link_libraries(moped2 
  gomp
  libmoped
  ${OpenCV_LIBS}
  ${catkin_LIBRARIES}
  ${GLEW_LIBRARIES}
  ${IL_LIBRARIES}
  ${GLUT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  )


add_executable(moped2_test moped_test.cpp)

set_target_properties(moped2_test PROPERTIES COMPILE_FLAGS "-Wl,--start-group")
set_target_properties(moped2_test PROPERTIES LINK_FLAGS "-O3 -Wall -march=native -mfpmath=sse -fopenmp")
target_link_libraries(moped2_test gomp libmoped  ${catkin_LIBRARIES}  ${GLEW_LIBRARIES} ${IL_LIBRARIES} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES}) 

