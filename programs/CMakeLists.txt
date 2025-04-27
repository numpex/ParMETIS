# programs/CMakeLists.txt

# A helper macro: name + list of .c sources
macro(pm_program prog)
  add_executable(${prog} ${ARGN})
  target_include_directories(${prog} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
  target_link_libraries(${prog}
    PRIVATE
      parmetis
      MPI::MPI_C
      Metis::metis
      GKlib::GKlib
  )
  if(OPENMP)
    find_package(OpenMP REQUIRED)
    target_link_libraries(${prog} PRIVATE OpenMP::OpenMP_C)
  endif()
  install(TARGETS ${prog}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  )
endmacro()

# now invoke it for each program:
pm_program(pm_ptest    ptest.c    io.c    adaptgraph.c)
add_test(NAME pm_ptest COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} 2 ${PROJECT_BINARY_DIR}/bin/pm_ptest ${PROJECT_SOURCE_DIR}/graphs/rotor.graph)
pm_program(pm_mtest    mtest.c    io.c)
add_test(NAME pm_mtest COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} 2 ${PROJECT_BINARY_DIR}/bin/pm_mtest ${PROJECT_SOURCE_DIR}/graphs/bricks.hex3d)
pm_program(pm_parmetis parmetis.c io.c    adaptgraph.c)
pm_program(pm_pometis  pometis.c  io.c)
pm_program(pm_dglpart  dglpart.c)