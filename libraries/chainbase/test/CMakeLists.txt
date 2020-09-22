file(GLOB UNIT_TESTS "*.cpp")
list(APPEND CMAKE_MODULE_PATH "/home/mong/CLionProjects/blockchain-sim/shadow/cmake")
include(ShadowTools) 

add_shadow_plugin( chainbase_test ${UNIT_TESTS}  )
target_link_libraries( chainbase_test  chainbase Boost::unit_test_framework ${OPENSSL_LIBRARIES} ${PLATFORM_SPECIFIC_LIBS} )

