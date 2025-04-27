# libparmetis/CMakeLists.txt

# grab all the .c files in this directory
file(GLOB PARMETIS_SOURCES CONFIGURE_DEPENDS *.c)

# build either SHARED or STATIC
if(BUILD_SHARED_LIBS)
  add_library(parmetis SHARED ${PARMETIS_SOURCES})
else()
  add_library(parmetis STATIC ${PARMETIS_SOURCES})
endif()

# headers are in ../include
target_include_directories(parmetis
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include/ParMETIS>
)

# pass along your width‐macros
target_compile_definitions(parmetis
  PUBLIC
    IDXTYPEWIDTH=${IDXTYPEWIDTH}
    REALTYPEWIDTH=${REALTYPEWIDTH}
)

# OpenMP if requested
if(OPENMP)
  find_package(OpenMP REQUIRED)
  target_compile_definitions(parmetis PUBLIC OPENMP)
  target_link_libraries(parmetis PUBLIC OpenMP::OpenMP_C)
endif()

# link to MPI, METIS and GKlib
target_link_libraries(parmetis
  PUBLIC
    MPI::MPI_C
    Metis::metis
    GKlib::GKlib
)

install(TARGETS parmetis
  EXPORT    ParMETISTargets
  LIBRARY   DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME   DESTINATION ${CMAKE_INSTALL_BINDIR}
  ARCHIVE   DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
