cmake_minimum_required(VERSION 3.18)
set(CMAKE_SYSTEM_NAME Generic)

find_program(CL65 NAMES cl65 REQUIRED)
set(CMAKE_C_COMPILER ${CL65})
project(qrdemo LANGUAGES C ASM)

add_compile_options(-Werror -c -t none -Oirs -Ln ${CMAKE_CURRENT_BINARY_DIR}/labels.txt $<$<CONFIG:Debug>:-g>)
set(CMAKE_C_COMPILE_OBJECT "<CMAKE_C_COMPILER> <DEFINES> <INCLUDES> <FLAGS> -o <OBJECT> -l <OBJECT>.s -T <SOURCE>")
add_link_options(
  -C "${CMAKE_CURRENT_SOURCE_DIR}/mapper.cfg"
  -Ln "${CMAKE_CURRENT_BINARY_DIR}/labels.txt"
  -Wl --dbgfile,"${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.dbg"
)

find_package(Python REQUIRED)
set(CHRGEN "${CMAKE_CURRENT_SOURCE_DIR}/chrgen.py")
set(ASCII_CHR "${CMAKE_CURRENT_SOURCE_DIR}/ascii.chr")
set(CHR_H "${CMAKE_CURRENT_BINARY_DIR}/chr.h")
set(CHR_S "${CMAKE_CURRENT_BINARY_DIR}/chr.s")
add_custom_command(
  OUTPUT ${CHR_H} ${CHR_S}
  COMMAND ${Python_EXECUTABLE} ${CHRGEN} ${ASCII_CHR} ${CHR_H} ${CHR_S}
  DEPENDS ${CHRGEN} ${ASCII_CHR}
)

add_executable(${PROJECT_NAME}.nes
  "${CMAKE_CURRENT_SOURCE_DIR}/keyboard.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/qrcodegen.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/screen_editor.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/screen_qr.c"
  "${CMAKE_CURRENT_SOURCE_DIR}/rsmt.s"
  "${CMAKE_CURRENT_SOURCE_DIR}/crt0.s"
  "${CMAKE_CURRENT_SOURCE_DIR}/lz4vram.s"
  ${CHR_S}
)
set_target_properties(${TARGET_NAME} PROPERTIES
  LINK_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/mapper.cfg"
)
