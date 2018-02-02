option(LINK_WITH_PTHREAD "Explicitly link mbed TLS library to pthread." OFF)

set(src_crypto
    mbedtls/library/aes.c
    mbedtls/library/aesni.c
    mbedtls/library/arc4.c
    mbedtls/library/asn1parse.c
    mbedtls/library/asn1write.c
    mbedtls/library/base64.c
    mbedtls/library/bignum.c
    mbedtls/library/blowfish.c
    mbedtls/library/camellia.c
    mbedtls/library/ccm.c
    mbedtls/library/cipher.c
    mbedtls/library/cipher_wrap.c
    mbedtls/library/cmac.c
    mbedtls/library/ctr_drbg.c
    mbedtls/library/des.c
    mbedtls/library/dhm.c
    mbedtls/library/ecdh.c
    mbedtls/library/ecdsa.c
    mbedtls/library/ecjpake.c
    mbedtls/library/ecp.c
    mbedtls/library/ecp_curves.c
    mbedtls/library/entropy.c
    mbedtls/library/entropy_poll.c
    mbedtls/library/error.c
    mbedtls/library/gcm.c
    mbedtls/library/havege.c
    mbedtls/library/hmac_drbg.c
    mbedtls/library/md.c
    mbedtls/library/md2.c
    mbedtls/library/md4.c
    mbedtls/library/md5.c
    mbedtls/library/md_wrap.c
    mbedtls/library/memory_buffer_alloc.c
    mbedtls/library/oid.c
    mbedtls/library/padlock.c
    mbedtls/library/pem.c
    mbedtls/library/pk.c
    mbedtls/library/pk_wrap.c
    mbedtls/library/pkcs12.c
    mbedtls/library/pkcs5.c
    mbedtls/library/pkparse.c
    mbedtls/library/pkwrite.c
    mbedtls/library/platform.c
    mbedtls/library/ripemd160.c
    mbedtls/library/rsa.c
    mbedtls/library/rsa_internal.c
    mbedtls/library/sha1.c
    mbedtls/library/sha256.c
    mbedtls/library/sha512.c
    mbedtls/library/threading.c
    mbedtls/library/timing.c
    mbedtls/library/version.c
    mbedtls/library/version_features.c
    mbedtls/library/xtea.c
)

set(src_x509
    mbedtls/library/certs.c
    mbedtls/library/pkcs11.c
    mbedtls/library/x509.c
    mbedtls/library/x509_create.c
    mbedtls/library/x509_crl.c
    mbedtls/library/x509_crt.c
    mbedtls/library/x509_csr.c
    mbedtls/library/x509write_crt.c
    mbedtls/library/x509write_csr.c
)

set(src_tls
    mbedtls/library/debug.c
    mbedtls/library/net_sockets.c
    mbedtls/library/ssl_cache.c
    mbedtls/library/ssl_ciphersuites.c
    mbedtls/library/ssl_cli.c
    mbedtls/library/ssl_cookie.c
    mbedtls/library/ssl_srv.c
    mbedtls/library/ssl_ticket.c
    mbedtls/library/ssl_tls.c
)

if(CMAKE_COMPILER_IS_GNUCC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wmissing-declarations -Wmissing-prototypes")
endif(CMAKE_COMPILER_IS_GNUCC)

if(CMAKE_COMPILER_IS_CLANG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wmissing-declarations -Wmissing-prototypes -Wdocumentation -Wno-documentation-deprecated-sync -Wunreachable-code")
endif(CMAKE_COMPILER_IS_CLANG)

if(WIN32)
    set(libs ${libs} ws2_32)
endif(WIN32)

if(USE_PKCS11_HELPER_LIBRARY)
    set(libs ${libs} pkcs11-helper)
endif(USE_PKCS11_HELPER_LIBRARY)

if(ENABLE_ZLIB_SUPPORT)
    set(libs ${libs} ${ZLIB_LIBRARIES})
endif(ENABLE_ZLIB_SUPPORT)

if(LINK_WITH_PTHREAD)
    set(libs ${libs} pthread)
endif()

# if (NOT USE_OBJECT_MBEDTLS_LIBRARY AND NOT USE_SHARED_MBEDTLS_LIBRARY)
#     message(FATAL_ERROR "Need to choose static or shared mbedtls build!")
# endif(NOT USE_OBJECT_MBEDTLS_LIBRARY AND NOT USE_SHARED_MBEDTLS_LIBRARY)

set(mbedtls_object_target "mbedtls")
set(mbedx509_object_target "mbedx509")
set(mbedcrypto_object_target "mbedcrypto")

add_library(${mbedcrypto_object_target} OBJECT ${src_crypto})
set_target_properties(${mbedcrypto_object_target} PROPERTIES OUTPUT_NAME mbedcrypto)
target_link_libraries(${mbedcrypto_object_target} ${libs})

add_library(${mbedx509_object_target} OBJECT ${src_x509})
set_target_properties(${mbedx509_object_target} PROPERTIES OUTPUT_NAME mbedx509)
target_link_libraries(${mbedx509_object_target} ${libs} ${mbedcrypto_object_target})

add_library(${mbedtls_object_target} OBJECT ${src_tls})
set_target_properties(${mbedtls_object_target} PROPERTIES OUTPUT_NAME mbedtls)
target_link_libraries(${mbedtls_object_target} ${libs} ${mbedx509_object_target})

install(TARGETS ${mbedtls_object_target} ${mbedx509_object_target} ${mbedcrypto_object_target}
        DESTINATION ${LIB_INSTALL_DIR}
        PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
