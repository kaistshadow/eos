## Add logic for ".SO" file creation by VICTOR
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../cmake")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../Install/cmake")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../shadow/cmake")
include(ShadowTools) # load ShadowTools Module
set( NODE_SO_NAME libnodeos )
set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../Install")

find_package(OpenSSL_1_1_0h REQUIRED)
message(STATUS "OPENSSL_1_1_0H_FOUND=${OPENSSL_1_1_0H_FOUND}")

add_shadow_plugin(${NODE_SO_NAME} main.cpp )
add_dependencies(${NODE_SO_NAME} ${NODE_EXECUTABLE_NAME})

add_executable( ${NODE_EXECUTABLE_NAME} main.cpp )

if( UNIX AND NOT APPLE )
  set(rt_library rt )
endif()

find_package( Gperftools QUIET )
if( GPERFTOOLS_FOUND )
    message( STATUS "Found gperftools; compiling ${NODE_EXECUTABLE_NAME} with TCMalloc")
    list( APPEND PLATFORM_SPECIFIC_LIBS tcmalloc )
endif()

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/../../.git)
  find_package(Git)
  if(GIT_FOUND)
    execute_process(
      COMMAND ${GIT_EXECUTABLE} rev-parse --short=8 HEAD
      WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../.."
      OUTPUT_VARIABLE "nodeos_BUILD_VERSION"
      ERROR_QUIET
      OUTPUT_STRIP_TRAILING_WHITESPACE)
    message(STATUS "Git commit revision: ${nodeos_BUILD_VERSION}")
  else()
    set(nodeos_BUILD_VERSION 0)
  endif()
else()
  set(nodeos_BUILD_VERSION 0)
endif()

configure_file(config.hpp.in config.hpp ESCAPE_QUOTES)

target_include_directories(${NODE_EXECUTABLE_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
## Add logic for ".SO" file creation by VICTOR
target_include_directories(${NODE_SO_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

if(UNIX)
  if(APPLE)
    set(whole_archive_flag "-force_load")
    set(no_whole_archive_flag "")
    set(build_id_flag "")
  else()
    set(whole_archive_flag "--whole-archive")
    set(no_whole_archive_flag "--no-whole-archive")
    set(build_id_flag "--build-id")
  endif()
else()
  set(whole_archive_flag "--whole-archive")
  set(no_whole_archive_flag "--no-whole-archive")
  set(build_id_flag "")
endif()

target_link_libraries( ${NODE_EXECUTABLE_NAME}
        PRIVATE appbase version
        PRIVATE -Wl,${whole_archive_flag} login_plugin               -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} history_plugin             -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} state_history_plugin       -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} trace_api_plugin           -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} history_api_plugin         -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} chain_api_plugin           -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} net_plugin                 -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} net_api_plugin             -Wl,${no_whole_archive_flag}
#        PRIVATE -Wl,${whole_archive_flag} faucet_testnet_plugin      -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} txn_test_gen_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} db_size_api_plugin         -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} producer_api_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} test_control_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} test_control_api_plugin    -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${build_id_flag}
        PRIVATE chain_plugin http_plugin producer_plugin http_client_plugin
        PRIVATE eosio_chain fc ${CMAKE_DL_LIBS} ${PLATFORM_SPECIFIC_LIBS} )

## Add logic for ".SO" file creation by VICTOR
target_link_libraries( ${NODE_SO_NAME}
        PRIVATE appbase version
        PRIVATE -Wl,${whole_archive_flag} login_plugin               -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} history_plugin             -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} state_history_plugin       -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} trace_api_plugin           -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} history_api_plugin         -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} chain_api_plugin           -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} net_plugin                 -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} net_api_plugin             -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} txn_test_gen_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} db_size_api_plugin         -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} producer_api_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} test_control_plugin        -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${whole_archive_flag} test_control_api_plugin    -Wl,${no_whole_archive_flag}
        PRIVATE -Wl,${build_id_flag}
        PRIVATE chain_plugin http_plugin producer_plugin http_client_plugin
        PRIVATE eosio_chain fc ${CMAKE_DL_LIBS} ${PLATFORM_SPECIFIC_LIBS}
        OpenSSL_1_1_0h::Crypto OpenSSL_1_1_0h::SSL)

if(BUILD_MONGO_DB_PLUGIN)
  target_link_libraries( ${NODE_EXECUTABLE_NAME} PRIVATE -Wl,${whole_archive_flag} mongo_db_plugin -Wl,${no_whole_archive_flag} )
  ## Add logic for ".SO" file creation by VICTOR
  target_link_libraries( ${NODE_SO_NAME} PRIVATE -Wl,${whole_archive_flag} mongo_db_plugin -Wl,${no_whole_archive_flag} )
endif()

include(additionalPlugins)

copy_bin( ${NODE_EXECUTABLE_NAME} )

## Add logic for "nodeos" file copy by VICTOR
set_property(TARGET ${NODE_EXECUTABLE_NAME}
        PROPERTY
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../tests/regtest/shadow-eos/bin")
## Add logic for ".SO" file creation by VICTOR
MESSAGE(STATUS "CMAKE_INSTALL_PREFIX = ${CMAKE_INSTALL_PREFIX}")
set_property(TARGET "${NODE_SO_NAME}"
        PROPERTY
        LIBRARY_OUTPUT_DIRECTORY "${CMAKE_INSTALL_PREFIX}/plugins")
set_property(TARGET "${NODE_SO_NAME}"
        PROPERTY
        OUTPUT_NAME "NODEOS_2.0.7")

install( TARGETS
   ${NODE_EXECUTABLE_NAME} RUNTIME DESTINATION ${CMAKE_INSTALL_FULL_BINDIR} COMPONENT base
)
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/log/eosio
        DIRECTORY_PERMISSIONS OWNER_READ
                              OWNER_WRITE
                              OWNER_EXECUTE
                              GROUP_READ
                              GROUP_WRITE
                              GROUP_EXECUTE
                              WORLD_READ
                              WORLD_EXECUTE
)
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/lib/eosio
        DIRECTORY_PERMISSIONS OWNER_READ
                              OWNER_WRITE
                              OWNER_EXECUTE
                              GROUP_READ
                              GROUP_WRITE
                              GROUP_EXECUTE
                              WORLD_READ
                              WORLD_EXECUTE
			)
