include_regular_expression("^.*$")

add_definitions(-DUSE_JPIP)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${OPENJPEG_BINARY_DIR}/src/lib/openjp2 # opj_config.h
  ${OPENJPEG_SOURCE_DIR}/src/lib/openjp2
  ${FCGI_INCLUDE_DIRS}
  ${CURL_INCLUDE_DIRS}
)

# Defines the source code for the library
set(OPENJPIP_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/boxheader_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/codestream_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/imgreg_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/marker_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/msgqueue_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/box_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/faixbox_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/index_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/metadata_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/placeholder_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/byte_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/ihdrbox_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/manfbox_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/mhixbox_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/target_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/cachemodel_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/j2kheader_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/jp2k_encoder.c
  ${CMAKE_CURRENT_SOURCE_DIR}/openjpip.c
  ${CMAKE_CURRENT_SOURCE_DIR}/query_parser.c
  ${CMAKE_CURRENT_SOURCE_DIR}/channel_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/session_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/jpip_parser.c
  ${CMAKE_CURRENT_SOURCE_DIR}/sock_manager.c
  ${OPENJPEG_SOURCE_DIR}/src/lib/openjp2/opj_malloc.c
  )

set(SERVER_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/auxtrans_manager.c
  )

set(LOCAL_SRCS
  ${CMAKE_CURRENT_SOURCE_DIR}/jp2k_decoder.c
  ${CMAKE_CURRENT_SOURCE_DIR}/imgsock_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/jpipstream_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/cache_manager.c
  ${CMAKE_CURRENT_SOURCE_DIR}/dec_clientmsg_handler.c
  )

# Build the library
add_library(openjpip ${OPENJPIP_SRCS} ${LOCAL_SRCS})
if(WIN32)
  if(BUILD_SHARED_LIBS)
    target_compile_definitions(openjpip PRIVATE OPJ_EXPORTS)
  else()
    target_compile_definitions(openjpip PUBLIC OPJ_STATIC)
  endif()
endif()
set_target_properties(openjpip
  PROPERTIES ${OPENJPEG_LIBRARY_PROPERTIES})
configure_opj_framework(openjpip)
target_compile_options(openjpip PRIVATE ${OPENJPEG_LIBRARY_COMPILE_OPTIONS})
target_link_libraries(openjpip ${OPENJPEG_LIBRARY_NAME})
if(WIN32)
  # add Winsock on windows+mingw
  target_link_libraries(openjpip ws2_32)
endif()

# Install library
install(TARGETS openjpip
  EXPORT OpenJPEGTargets
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Applications
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Libraries
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Libraries
  FRAMEWORK DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime OPTIONAL
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  RESOURCE DESTINATION ${OPJ_RESOURCEDIR}
)

if(BUILD_JPIP_SERVER)
  add_library(openjpip_server STATIC ${OPENJPIP_SRCS} ${SERVER_SRCS})
  target_link_libraries(openjpip_server ${FCGI_LIBRARIES} ${CURL_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
  set_target_properties(openjpip_server
    PROPERTIES COMPILE_FLAGS "-DSERVER")
  configure_opj_framework(openjpip_server)
  install(TARGETS openjpip_server
    EXPORT OpenJPEGTargets
    DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Libraries
    FRAMEWORK DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime OPTIONAL
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    RESOURCE DESTINATION ${OPJ_RESOURCEDIR}
  )
endif()
