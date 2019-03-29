
add_executable (compile_et compile_et.c y.tab.c lex.yy.c)

add_custom_command (OUTPUT y.tab.c
		COMMAND yacc ${CMAKE_CURRENT_SOURCE_DIR}/parse.y
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/parse.y)

add_custom_command (OUTPUT lex.yy.c
		COMMAND lex  ${CMAKE_CURRENT_SOURCE_DIR}/lex.l
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/lex.l)

# add_custom_command (TARGET compile_et
# 		PRE_BUILD
# 		COMMAND yacc ${CMAKE_CURRENT_SOURCE_DIR}/parse.y
# 		COMMAND lex  ${CMAKE_CURRENT_SOURCE_DIR}/lex.l)

add_library (com_err com_err.c error.c)

include_directories (${CMAKE_CURRENT_SOURCE_DIR})


