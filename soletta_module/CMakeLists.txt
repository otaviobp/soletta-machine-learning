cmake_minimum_required(VERSION 2.8)

project(sml)

add_definitions(-DSOL_FLOW_NODE_TYPE_MODULE_EXTERNAL)

set(SML_SOLETTA_GEN_H ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/machine_learning_gen.h)
set(SML_SOLETTA_GEN_C ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/machine_learning_gen.c)
set(SML_SOLETTA_JSON ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/machine_learning.json)

execute_process(COMMAND ${PKG_CONFIG_EXECUTABLE} --variable=modulesdir soletta
  OUTPUT_STRIP_TRAILING_WHITESPACE
  OUTPUT_VARIABLE SML_SOLETTA_LIB_PREFIX
  )

execute_process(COMMAND ${PKG_CONFIG_EXECUTABLE} --variable=nodeschemapath soletta
  OUTPUT_STRIP_TRAILING_WHITESPACE
  OUTPUT_VARIABLE SCHEMA_PATH
  )

execute_process(COMMAND ${PKG_CONFIG_EXECUTABLE} --variable=pkgdatadir soletta
  OUTPUT_STRIP_TRAILING_WHITESPACE
  OUTPUT_VARIABLE SOLETTA_DATA_DIR
  )

add_custom_target(GEN_FILES
#FIXME use add_custom_command otherwise it always will build this gen files
#add_custom_command(
# OUTPUT ${SML_SOLETTA_GEN_H} ${SML_SOLETTA_GEN_C}
  COMMAND ${CMAKE_FIND_ROOT_PATH}/${SOLETTA_PREFIX}/bin/sol-flow-node-type-validate.py ${CMAKE_FIND_ROOT_PATH}/${SCHEMA_PATH} ${SML_SOLETTA_JSON}
  COMMAND ${CMAKE_FIND_ROOT_PATH}/${SOLETTA_PREFIX}/bin/sol-flow-node-type-gen.py --prefix=sol-flow-node-type --genspec-schema=${CMAKE_FIND_ROOT_PATH}/${SCHEMA_PATH} ${SML_SOLETTA_JSON} ${SML_SOLETTA_GEN_H} ${SML_SOLETTA_GEN_C} ${CMAKE_BINARY_DIR}/soletta_module/machine_learning.json
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/machine_learning.json
  )

set(BUILT_SOURCES
  ${SML_SOLETTA_GEN_H}
  ${SML_SOLETTA_GEN_C}
  )
SET_SOURCE_FILES_PROPERTIES(${BUILT_SOURCES} PROPERTIES
  GENERATED true
  )

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/../sml/common/include/
  ${SOLETTA_INCLUDE_DIRS}
  ${SML_INCLUDE_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/
  )

link_directories(
  ${SOLETTA_LIBRARY_DIRS}
  ${FUZZY_LITE_LIBRARY_DIRS}
  ${FANN_LIBRARY_DIRS}
  )

add_library(machine-learning SHARED
  ${CMAKE_CURRENT_SOURCE_DIR}/machine_learning/machine_learning.c
  )
set_target_properties(machine-learning
  PROPERTIES PREFIX ""
  )


target_link_libraries(machine-learning
  ${SOLETTA_LIBRARIES}
  sml_libs
  m)

add_dependencies(machine-learning
  GEN_FILES
  )

install(TARGETS machine-learning
  LIBRARY DESTINATION ${SML_SOLETTA_LIB_PREFIX}/flow/
  )

install(FILES ${CMAKE_BINARY_DIR}/soletta_module/machine_learning.json
    DESTINATION ${SOLETTA_DATA_DIR}/flow/descriptions
)
