set( CEGUI_TARGET_NAME ${CEGUI_BASE_LIBNAME} )

# we do not use cegui_gather_files() here since our needs are more complex.
file (GLOB CORE_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" *.cpp)
file (GLOB WIDGET_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" elements/*.cpp)
file (GLOB FALAGARD_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" falagard/*.cpp)
set (CORE_SOURCE_FILES ${CORE_SOURCE_FILES} ${WIDGET_SOURCE_FILES} ${FALAGARD_SOURCE_FILES})

file (GLOB CORE_HEADER_FILES ../include/*.h)
file (GLOB WIDGET_HEADER_FILES ../include/elements/*.h)
file (GLOB FALAGARD_HEADER_FILES ../include/falagard/*.h)

list (APPEND CORE_HEADER_FILES
    ${CMAKE_CURRENT_BINARY_DIR}/../include/CEGUIConfig.h
    ${CMAKE_CURRENT_BINARY_DIR}/../include/CEGUIVersion.h
)

# never compile this directly, it's included in another file
list (REMOVE_ITEM CORE_SOURCE_FILES minibidi.cpp)

if (NOT CEGUI_HAS_DEFAULT_LOGGER)
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIDefaultLogger.cpp)
endif()

if (NOT CEGUI_HAS_FREETYPE)
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIFreeTypeFont.cpp)
endif()

if (NOT CEGUI_HAS_PCRE_REGEX)
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIPCRERegexMatcher.cpp)
endif()

if (NOT CEGUI_USE_FRIBIDI)
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIFribidiVisualMapping.cpp)
endif()

if (NOT CEGUI_USE_MINIBIDI)
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIMinibidiVisualMapping.cpp)
endif ()

if (CEGUI_HAS_MINIZIP_RESOURCE_PROVIDER)
    list (APPEND CORE_SOURCE_FILES minizip/unzip.cpp)
    list (APPEND CORE_SOURCE_FILES minizip/ioapi.cpp)

    if (WIN32)
        list (APPEND CORE_SOURCE_FILES minizip/iowin32.cpp)
    endif()
else()
    list (REMOVE_ITEM CORE_SOURCE_FILES CEGUIMinizipResourceProvider.cpp)
endif()

if (NOT CEGUI_HAS_EXPRESSION_DIM)
    list(REMOVE_ITEM CORE_SOURCE_FILES
        falagard/CEGUIFalExpressionDim.cpp
        falagard/CEGUIFalExpressionDimParser.cpp
        falagard/CEGUIFalExpressionDimScanner.cpp
    )
endif()

# we do not use the common header install function since we need to install to
# multiple sub-dirs.
cegui_add_library(TRUE FALSE)

if (CEGUI_HAS_FREETYPE)
    cegui_add_dependency(FREETYPE)
endif ()

if (CEGUI_HAS_PCRE_REGEX)
    cegui_add_dependency(PCRE)
    if (WIN32)
        add_definitions(-DPCRE_STATIC)
    endif()
endif ()

if (CEGUI_USE_FRIBIDI)
    cegui_add_dependency(FRIBIDI)
endif ()

if (CEGUI_HAS_MINIZIP_RESOURCE_PROVIDER)
    cegui_add_dependency(ZLIB)
endif ()

if (WIN32)
    target_link_libraries(${CEGUI_TARGET_NAME} winmm)
elseif (UNIX AND NOT APPLE)
    target_link_libraries(${CEGUI_TARGET_NAME} ${CMAKE_DL_LIBS})
endif()

if (UNIX AND NOT APPLE AND NOT WIN32)
    install(FILES ${CORE_HEADER_FILES} DESTINATION include/${CMAKE_PROJECT_NAME})
    install(FILES ${WIDGET_HEADER_FILES} DESTINATION include/${CMAKE_PROJECT_NAME}/elements)
    install(FILES ${FALAGARD_HEADER_FILES} DESTINATION include/${CMAKE_PROJECT_NAME}/falagard)
endif()
