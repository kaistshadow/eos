file(GLOB UNIT_TESTS "*.cpp")
add_library( chainbase_test SHARED ${UNIT_TESTS}  )
target_link_libraries( chainbase_test  chainbase Boost::unit_test_framework ${OPENSSL_LIBRARIES} ${PLATFORM_SPECIFIC_LIBS} )

