# configure 
add_library(Configure STATIC 
"src/Configure.cpp")

add_library(Demo::Configure ALIAS Configure)

target_link_libraries(Configure PRIVATE 
Boost::filesystem fmt::fmt spdlog::spdlog Demo::Logger 
)

target_include_directories(Configure
    PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
)

# protobuf_config
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS config.proto) 

# Print path to generated files
message ("PROTO_SRCS = ${PROTO_SRCS}")
message ("PROTO_HDRS = ${PROTO_HDRS}")

add_library(protobuf_config STATIC
    ${PROTO_SRCS}
    ${PROTO_HDRS}
)

add_library(Demo::protobuf_config ALIAS protobuf_config)

target_link_libraries(protobuf_config
    PUBLIC
    protobuf::protobuf
)
