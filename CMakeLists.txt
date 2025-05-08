cmake_minimum_required(VERSION 3.15)
project(ParMETIS
  VERSION 4.0.4
  DESCRIPTION "Parallel Graph Partitioning Library"
  LANGUAGES C
)


set(ParMETIS_PRERELEASE "-rc.4")
set(ParMETIS_VERSION_STRING "${PROJECT_VERSION}${ParMETIS_PRERELEASE}")
set(ParMETIS_VERSION_MAJOR 5)
set(ParMETIS_VERSION_MINOR 3)
set(ParMETIS_VERSION_PATCH 1)
set(ParMETIS_VERSION "${ParMETIS_VERSION_MAJOR}.${ParMETIS_VERSION_MINOR}.${ParMETIS_VERSION_PATCH}${ParMETIS_PRERELEASE}")
set(ParMETIS_VERSION_NUMBER
  ((ParMETIS_VERSION_MAJOR << 16) | (ParMETIS_VERSION_MINOR << 9) | ParMETIS_VERSION_PATCH)
)

# -----------------------------------------------------------------------------
# 1) Options
# -----------------------------------------------------------------------------

option(IDX64      "enable 64‑bit indices (IDXTYPEWIDTH=64)"   OFF)
option(REAL64     "enable 64‑bit reals   (REALTYPEWIDTH=64)" OFF)
option(SHARED     "build shared library"                     OFF)
option(GDB        "enable gdb symbols"                      OFF)
option(ASSERT     "enable ASSERT"                           OFF)
option(ASSERT2    "enable ASSERT2"                          OFF)
option(DEBUG      "enable DEBUG"                            OFF)
option(GPROF      "enable gprof profiling"                  OFF)
option(VALGRIND   "enable valgrind instrumentation"         OFF)
option(OPENMP     "enable OpenMP support"                   OFF)

option(BUILD_SHARED_LIBS
  "Build ParMETIS as a shared library (default OFF → static)"
  ${SHARED}
)
# -----------------------------------------------------------------------------
# 2) Map options to compile-time widths
# -----------------------------------------------------------------------------
if(IDX64)
  set(IDXTYPEWIDTH 64)
else()
  set(IDXTYPEWIDTH 32)
endif()

if(REAL64)
  set(REALTYPEWIDTH 64)
else()
  set(REALTYPEWIDTH 32)
endif()



# -----------------------------------------------------------------------------
# 2) Dependencies
# -----------------------------------------------------------------------------
find_package(MPI      REQUIRED)   # MPI::MPI_C
find_package(GKlib    REQUIRED)   # GKlib::GKlib
find_package(Metis    REQUIRED)   # METIS::metis
if(OPENMP)
  find_package(OpenMP REQUIRED)
endif()

option(PARMETIS_ENABLE_TESTS  "Enable PARMETIS testing (default OFF)"  ON)
if (PARMETIS_ENABLE_TESTS)
  enable_testing()
endif()
# -----------------------------------------------------------------------------
# 3) Subdirectories
# -----------------------------------------------------------------------------
add_subdirectory(include)       # installs parmetis.h
add_subdirectory(libparmetis)   # builds ParMETIS lib
add_subdirectory(programs)      # builds pm_* executables

# -----------------------------------------------------------------------------
# 4) Installation & Export
# -----------------------------------------------------------------------------
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(
  EXPORT    ParMETISTargets
  NAMESPACE ParMETIS::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ParMETIS
)

# 4b) Generate and install Config files for find_package(ParMETIS)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/ParMETISConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
  "${CMAKE_CURRENT_LIST_DIR}/cmake/ParMETISConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/ParMETISConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ParMETIS
)

install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/ParMETISConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/ParMETISConfigVersion.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/ParMETIS
)

# -----------------------------------------------------------------------------
# 5) CPack (optional)
# -----------------------------------------------------------------------------
include(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_NAME    "ParMETIS")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "ParMETIS Graph Partitioning Library")
set(CPACK_PACKAGE_VERSION_MAJOR "${ParMETIS_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${ParMETIS_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${ParMETIS_VERSION_PATCH}")
set(CPACK_PACKAGE_VERSION        "${ParMETIS_VERSION}")
set(CPACK_PACKAGE_NAME "${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}${ParMETIS_PRERELEASE}")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION}${ParMETIS_PRERELEASE}")
SET(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}${ParMETIS_PRERELEASE}-${CMAKE_SYSTEM_NAME}")

# Optionally point to your LICENSE and README
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README  "${CMAKE_SOURCE_DIR}/README.md")
set(CPACK_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_IGNORE_FILES
    "/build/;/.spack-env/;spack*;metis-*;.git/;~$;${CPACK_SOURCE_IGNORE_FILES}"
)

include(CPack)