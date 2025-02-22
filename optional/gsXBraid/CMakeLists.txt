### CMakeLists.txt --- XBraid library
## Author: Angelos Mantzaflaris
#
# The library is header-only, the extra code is downloaded in place of
# the source folder of G+Smo
#
######################################################################

## Spectra extension
project(gsXBraidExtension)

# Collect file names
aux_header_directory(${CMAKE_CURRENT_SOURCE_DIR} ${PROJECT_NAME}_HEADERS)
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR} ${PROJECT_NAME}_SOURCES)
aux_tmpl_header_directory(${CMAKE_CURRENT_SOURCE_DIR} ${PROJECT_NAME}_HPPFILES)

# Apply same configuration as G+Smo
include(gsConfig)

if(CMAKE_C_COMPILER_ID MATCHES "MSVC")
  add_definitions(-D_CRT_NONSTDC_NO_WARNINGS)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

include(gsFetch)
gismo_fetch_directory(XBraid
  GIT_REPOSITORY https://github.com/XBraid/xbraid.git
  DESTINATION external
)


if( (NOT GISMO_BUILD_LIB) )
aux_instance_directory (${CMAKE_CURRENT_SOURCE_DIR} ${PROJECT_NAME}_INS)
if(${PROJECT_NAME}_INS)
  LIST( REMOVE_ITEM ${PROJECT_NAME}_CPP ${${PROJECT_NAME}_INS})
endif()
endif()

# Set XBraid library header files
set(${PROJECT_NAME}_HEADERS "${${PROJECT_NAME}_HEADERS}"
${gismo_externals}/XBraid/braid/_braid.h
${gismo_externals}/XBraid/braid/_braid.h
${gismo_externals}/XBraid/braid/base.h
${gismo_externals}/XBraid/braid/adjoint.h
${gismo_externals}/XBraid/braid/delta.h
${gismo_externals}/XBraid/braid/status.h
${gismo_externals}/XBraid/braid/tape.h
${gismo_externals}/XBraid/braid/util.h
${gismo_externals}/XBraid/braid/braid.h
${gismo_externals}/XBraid/braid/braid_status.h
${gismo_externals}/XBraid/braid/braid_test.h)

if(NOT GISMO_WITH_MPI )
set(${PROJECT_NAME}_HEADERS "${${PROJECT_NAME}_HEADERS}"
  ${gismo_externals}/XBraid/braid/mpistubs.h)
set(${PROJECT_NAME}_SOURCES "${${PROJECT_NAME}_SOURCES}"
  ${gismo_externals}/XBraid/braid/mpistubs.c)
endif()

# Set XBraid library sources files
set(${PROJECT_NAME}_SOURCES "${${PROJECT_NAME}_SOURCES}"
${gismo_externals}/XBraid/braid/access.c
${gismo_externals}/XBraid/braid/adjoint.c
${gismo_externals}/XBraid/braid/base.c
${gismo_externals}/XBraid/braid/braid.c
${gismo_externals}/XBraid/braid/braid_status.c
${gismo_externals}/XBraid/braid/braid_test.c
${gismo_externals}/XBraid/braid/communication.c
${gismo_externals}/XBraid/braid/delta.c
${gismo_externals}/XBraid/braid/distribution.c
${gismo_externals}/XBraid/braid/drive.c
${gismo_externals}/XBraid/braid/grid.c
${gismo_externals}/XBraid/braid/hierarchy.c
${gismo_externals}/XBraid/braid/interp.c
${gismo_externals}/XBraid/braid/norm.c
${gismo_externals}/XBraid/braid/refine.c
${gismo_externals}/XBraid/braid/relax.c
${gismo_externals}/XBraid/braid/residual.c
${gismo_externals}/XBraid/braid/restrict.c
${gismo_externals}/XBraid/braid/space.c
${gismo_externals}/XBraid/braid/step.c
${gismo_externals}/XBraid/braid/tape.c
${gismo_externals}/XBraid/braid/util.c
${gismo_externals}/XBraid/braid/uvector.c)

# Set XBraid library include files
set(XBRAID_INCLUDE_DIR ${gismo_externals}/XBraid/braid CACHE INTERNAL "")
include_directories(${XBRAID_INCLUDE_DIR})

# Compile gsXBraid extension as part of the G+Smo library
add_library(${PROJECT_NAME} OBJECT
  ${${PROJECT_NAME}_HEADERS}
  ${${PROJECT_NAME}_HPPFILES}
  ${${PROJECT_NAME}_SOURCES}
 )

# Set standard properties for all G+Smo extensions
set_target_properties(${PROJECT_NAME} PROPERTIES
  COMPILE_DEFINITIONS gismo_EXPORTS
  POSITION_INDEPENDENT_CODE ON
  LINKER_LANGUAGE CXX
  #START Export all symbols from this extension
  CXX_VISIBILITY_PRESET default
  C_VISIBILITY_PRESET default
  VISIBILITY_INLINES_HIDDEN 0
  #END Export all symbols from this extension
  FOLDER "G+Smo extensions" 
  )

if( GISMO_WITH_MPI )
  target_include_directories(${PROJECT_NAME} PRIVATE ${MPI_INCLUDE_PATH})
else()
  add_definitions("-Dbraid_SEQUENTIAL")
endif()

# Add gsXBraid extension to the list of G+Smo extensions
set(gismo_EXTENSIONS ${gismo_EXTENSIONS} $<TARGET_OBJECTS:${PROJECT_NAME}>
  CACHE INTERNAL "gismo extensions to be included")

# Install gsXBraid header files
install(DIRECTORY ${PROJECT_SOURCE_DIR}
  DESTINATION include/gismo/gsXBraid
  FILES_MATCHING PATTERN "*.h")

# Add filedata folder
add_definitions(-DXBRAID_DATA_DIR="${CMAKE_CURRENT_SOURCE_DIR}/filedata/") 

# Add example files
include_directories(${CODIPACK_INCLUDE_DIR})
aux_cpp_directory(${CMAKE_CURRENT_SOURCE_DIR}/examples FILES)
foreach(file ${FILES})
  add_gismo_executable(${file})
  get_filename_component(tarname ${file} NAME_WE) # name without extension
  set_property(TEST ${tarname} PROPERTY LABELS "${PROJECT_NAME}")
  set_target_properties(${tarname} PROPERTIES FOLDER "${PROJECT_NAME}")
  if( GISMO_WITH_MPI )
    target_include_directories(${tarname} PRIVATE ${MPI_INCLUDE_PATH})
  endif()
  # Install the example executables (optionally)
  install(TARGETS ${tarname} DESTINATION "${BIN_INSTALL_DIR}" COMPONENT exe OPTIONAL)
endforeach(file ${FILES})

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin/)
