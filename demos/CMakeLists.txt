# Set the platform named based on the host OS if not defined.
if( NOT DEFINED PLATFORM_NAME )
    if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )
        set( PLATFORM_NAME "posix" CACHE STRING "Port to use for building the SDK." )
    else()
        message( FATAL_ERROR "${CMAKE_SYSTEM_NAME} is not a supported platform." )
    endif()
else()
    set( PLATFORM_NAME ${PLATFORM_NAME} CACHE STRING "Port to use for building the SDK." )
endif()

set(AWS_DEMO_INCLUDE_DIRS ${CMAKE_CURRENT_LIST_DIR}/include)

# Include each subdirectory that has a CMakeLists.txt file in it
file(GLOB demo_dirs "${DEMOS_DIR}/*/*")
foreach(demo_dir IN LISTS demo_dirs)
    if(IS_DIRECTORY "${demo_dir}" AND EXISTS "${demo_dir}/CMakeLists.txt")
        add_subdirectory(${demo_dir})
    endif()
endforeach()

# Check for function fork
check_symbol_exists(fork "unistd.h" HAVE_FORK)

# Filter demos based on what packages or library exist.
if(${LIB_RT} STREQUAL "LIB_RT-NOTFOUND")
    set(librt_demos
            "http_demo_s3_download_multithreaded"
    )
    message( WARNING "rt library could not be found. Demos that use it will be excluded from the default target." )
    foreach(demo_name ${librt_demos})
        set_target_properties(${demo_name} PROPERTIES EXCLUDE_FROM_ALL true)
    endforeach()
endif()
if(NOT ${OpenSSL_FOUND})
    set(openssl_demos
            "defender_demo"
            "http_demo_basic_tls"
            "http_demo_mutual_auth"
            "http_demo_s3_download"
            "http_demo_s3_generate_presigned_url"
            "http_demo_s3_download_multithreaded"
            "http_demo_s3_upload"
            "mqtt_demo_basic_tls"
            "mqtt_demo_mutual_auth"
            "mqtt_demo_subscription_manager"
            "shadow_demo_main"
            "greengrass_demo_local_auth"
    )
    message( WARNING "OpenSSL library could not be found. Demos that use it will be excluded from the default target." )
    foreach(demo_name ${openssl_demos})
        set_target_properties(${demo_name} PROPERTIES EXCLUDE_FROM_ALL true)
    endforeach()
endif()
if(NOT HAVE_FORK)
    set(fork_demos
            "http_demo_s3_download_multithreaded"
            "jobs_demo_mosquitto"
    )
    message( WARNING "fork() could not be found. Demos that use it will be excluded from the default target." )
    foreach(demo_name ${fork_demos})
        set_target_properties(${demo_name} PROPERTIES EXCLUDE_FROM_ALL true)
    endforeach()
endif()

# Exclude MQTT version-specific demos based on MQTT_VERSION_5 option
if( MQTT_VERSION_5 )
    # Exclude MQTT 3.1.1 demos when building for MQTT 5
    set(mqtt_3_1_1_demos
            "mqtt_demo_basic_tls"
            "mqtt_demo_mutual_auth"
            "mqtt_demo_plaintext"
            "mqtt_demo_serializer"
            "mqtt_demo_subscription_manager"
    )
    message( STATUS "MQTT 5.0 enabled - excluding MQTT 3.1.1 demos from build" )
    foreach(demo_name ${mqtt_3_1_1_demos})
        if(TARGET ${demo_name})
            set_target_properties(${demo_name} PROPERTIES EXCLUDE_FROM_ALL true)
        endif()
    endforeach()
else()
    # Exclude MQTT 5 demos when building for MQTT 3.1.1 (default)
    set(mqtt_5_demos
            "mqtt_demo_mutual_auth_mqtt5"
    )
    message( STATUS "MQTT 3.1.1 enabled (default) - excluding MQTT 5 demos from build" )
    foreach(demo_name ${mqtt_5_demos})
        if(TARGET ${demo_name})
            set_target_properties(${demo_name} PROPERTIES EXCLUDE_FROM_ALL true)
        endif()
    endforeach()
endif()
