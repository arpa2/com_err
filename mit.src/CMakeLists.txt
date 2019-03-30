add_executable (compile_et compile_et.c y.tab.c)

add_custom_command (OUTPUT y.tab.c
		COMMAND bison -y ${CMAKE_CURRENT_SOURCE_DIR}/error_table.y
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/error_table.y et_lex.lex.c)

add_custom_command (OUTPUT et_lex.lex.c
		COMMAND flex -o et_lex.lex.c ${CMAKE_CURRENT_SOURCE_DIR}/et_lex.lex.l
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/et_lex.lex.l)

# add_custom_command (TARGET compile_et
# 		PRE_BUILD
# 		COMMAND yacc ${CMAKE_CURRENT_SOURCE_DIR}/parse.y
# 		COMMAND lex  ${CMAKE_CURRENT_SOURCE_DIR}/lex.l)

# Compile the files only once
add_library(_libcomerr OBJECT com_err.c error_message.c et_name.c init_et.c)

add_library(com_err_shared SHARED $<TARGET_OBJECTS:_libcomerr>)
set_target_properties(com_err_shared PROPERTIES
    OUTPUT_NAME com_err
)
add_library(com_err_static STATIC $<TARGET_OBJECTS:_libcomerr>)
set_target_properties(com_err_static PROPERTIES
    OUTPUT_NAME com_err
)

include_directories (${CMAKE_CURRENT_SOURCE_DIR})

install(
    TARGETS compile_et
    DESTINATION ${CMAKE_INSTALL_SBINDIR}
)

install(
    TARGETS com_err_shared
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

install(
    TARGETS com_err_static
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

