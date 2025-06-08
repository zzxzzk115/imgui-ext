set(TARGET_NAME example_default_sdl2_opengl3)

# add executable target
add_executable(${TARGET_NAME} main.cpp)

# link libraries
target_link_libraries(${TARGET_NAME} PRIVATE ImGuiExt)

target_set_common_properties(${TARGET_NAME})

# copy SDL2 libraries after build
add_custom_command(
        TARGET ${TARGET_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        $<TARGET_FILE_DIR:SDL2::SDL2>
        $<TARGET_FILE_DIR:${TARGET_NAME}>
)