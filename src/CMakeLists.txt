if (IMGUI_WS_ALL_WARNINGS)
    if (CMAKE_COMPILER_IS_GNUCC OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")
    else()
        # todo : windows
    endif()
endif()

file(READ "imgui-ws.js" src_imgui_ws_js_src)

# split large JS into multiple raw string literal pieces to satisfy MSVC string-size limit
set(chunk_size 4096)
string(LENGTH "${src_imgui_ws_js_src}" src_js_len)
set(src_imgui_ws_js "")
set(pos 0)
while(pos LESS src_js_len)
    math(EXPR remaining "${src_js_len} - ${pos}")
    if(remaining GREATER chunk_size)
        set(cur_len ${chunk_size})
    else()
        set(cur_len ${remaining})
    endif()
    string(SUBSTRING "${src_imgui_ws_js_src}" ${pos} ${cur_len} chunk)
    if(src_imgui_ws_js STREQUAL "")
        set(src_imgui_ws_js "R\"IMGUIWSJS(${chunk})IMGUIWSJS\"")
    else()
        set(src_imgui_ws_js "${src_imgui_ws_js} R\"IMGUIWSJS(${chunk})IMGUIWSJS\"")
    endif()
    math(EXPR pos "${pos} + ${cur_len}")
endwhile()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/common.h.in ${CMAKE_CURRENT_SOURCE_DIR}/common.h @ONLY)

add_library(imgui-ws STATIC
    imgui-ws.cpp
    compressor-xor-rle-per-draw-list.cpp
    compressor-xor-rle-per-draw-list-with-vtx-offset.cpp
    )

target_include_directories(imgui-ws PUBLIC
    ../include
    )

target_link_libraries(imgui-ws PRIVATE
    imgui
    incppect
    ${CMAKE_DL_LIBS}
    ${CMAKE_THREAD_LIBS_INIT}
    )
