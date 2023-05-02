cmake_minimum_required(VERSION 3.18)
set(CMAKE_SYSTEM_NAME Generic)

find_program(CL65 NAMES cl65 REQUIRED)
set(CMAKE_C_COMPILER ${CL65})
project(qrdemo LANGUAGES C ASM)

# add_compile_options(-Werror -c -t none -Oirs)
add_compile_options(-c -t none -Oirs $<$<CONFIG:Debug>:-g>)
set(CMAKE_C_COMPILE_OBJECT "<CMAKE_C_COMPILER> <DEFINES> <INCLUDES> <FLAGS> -o <OBJECT> -l <OBJECT>.s -T <SOURCE>")
add_link_options(
  -C "${CMAKE_CURRENT_SOURCE_DIR}/mapper.cfg"
  -Ln "${CMAKE_CURRENT_BINARY_DIR}/labels.txt"
  -Wl --dbgfile,"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.dbg"
)

add_executable(${PROJECT_NAME}.nes
  "${CMAKE_CURRENT_SOURCE_DIR}/main.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/qrcodegen.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/crt0.s"
)