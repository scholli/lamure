############################################################
# CMake Build Script for the preprocessing library

cmake_minimum_required(VERSION 2.8.7)
project(${CMAKE_PROJECT_NAME}_preprocessing)

set(PROJECT_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)
set(PROJECT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

# main sources
file(GLOB MAIN_SRC ${PROJECT_SOURCE_DIR}/*.cpp)
file(GLOB MAIN_INC ${PROJECT_INCLUDE_DIR}/lamure/pre/*.h)

# IO
file(GLOB IO_SRC ${PROJECT_SOURCE_DIR}/io/*.cpp)
file(GLOB IO_INC ${PROJECT_INCLUDE_DIR}/lamure/pre/io/*.h)

file(GLOB PLY_SRC ${PROJECT_SOURCE_DIR}/io/ply/*.cpp)
file(GLOB PLY_INC ${PROJECT_INCLUDE_DIR}/lamure/pre/io/ply/*.h)

SOURCE_GROUP("preprocessing" FILES ${MAIN_SRC})
SOURCE_GROUP("preprocessing" FILES ${MAIN_INC})

SOURCE_GROUP("io" FILES ${IO_SRC})
SOURCE_GROUP("io" FILES ${IO_INC})

SOURCE_GROUP("io\\ply" FILES ${PLY_SRC})
SOURCE_GROUP("io\\ply" FILES ${PLY_INC})

set(PROJECT_SOURCES ${MAIN_SRC} ${IO_SRC} ${PLY_SRC})
set(PROJECT_INCLUDES ${MAIN_INC} ${IO_INC} ${PLY_INC})

include_directories(${PROJECT_INCLUDE_DIR} 
                    ${COMMON_INCLUDE_DIR}
                    ${LAMURE_CONFIG_DIR}
                    ${CGAL_INCLUDE_DIR}
                    ${GMP_INCLUDE_DIR} 
                    ${MPFR_INCLUDE_DIR}
                    )

include_directories(SYSTEM ${SCHISM_INCLUDE_DIRS}
                           ${Boost_INCLUDE_DIR})

link_directories(${SCHISM_LIBRARY_DIRS})

add_library(${PROJECT_NAME} SHARED ${PROJECT_INCLUDES} ${PROJECT_SOURCES})

add_dependencies(${PROJECT_NAME} lamure_common)

IF (MSVC)
    SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-D LAMURE_PREPROCESSING_LIBRARY -DBOOST_ALL_NO_LIB")
ENDIF(MSVC)


set(PREPROC_INCLUDE_DIR ${PROJECT_INCLUDE_DIR} PARENT_SCOPE)
set(PREPROC_LIBRARY ${PROJECT_NAME} PARENT_SCOPE)
set(PREPROC_LIBRARY ${PROJECT_NAME})

############################################################
# Libraries



target_link_libraries(${PROJECT_NAME}
    ${COMMON_LIBRARY}
    ${PROJECT_LIBS}
    optimized ${Boost_THREAD_LIBRARY_RELEASE} debug ${Boost_THREAD_LIBRARY_DEBUG}
    )

if (${LAMURE_USE_CGAL_FOR_NNI})
  target_link_libraries(${PROJECT_NAME}
      ${GMP_LIBRARY}
      ${MPFR_LIBRARY}
      optimized ${CGAL_LIBRARY} debug ${CGAL_LIBRARY_DEBUG}
      optimized ${CGAL_CORE_LIBRARY} debug ${CGAL_CORE_LIBRARY_DEBUG}
      )
endif ()

###############################################################################
# install 
###############################################################################

IF (MSVC)
  install (TARGETS ${PROJECT_NAME}
    CONFIGURATIONS Release
    RUNTIME DESTINATION bin/Release
    LIBRARY DESTINATION lib/Release
    ARCHIVE DESTINATION lib/Release
  )

  install (TARGETS ${PROJECT_NAME}
    CONFIGURATIONS Debug
    RUNTIME DESTINATION bin/Debug
    LIBRARY DESTINATION lib/Debug
    ARCHIVE DESTINATION lib/Debug
  )
ELSEIF (UNIX)
  install (TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION lib
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
  )
ENDIF (MSVC)

# header files 
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/lamure/pre DESTINATION include/lamure FILES_MATCHING PATTERN "*.h")
