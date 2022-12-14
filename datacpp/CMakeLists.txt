## TODO: reseach how to combine conan skbuild cmake
# build with conan
if(NOT ENABLE_SKBUILD)
  add_executable(${PROJECT_NAME} src/main.cpp)
endif()

target_include_directories(${PROJECT_NAME} PUBLIC
                           "${PROJECT_BINARY_DIR}"
)

target_link_libraries(${PROJECT_NAME} PRIVATE Demo::Logger Demo::Configure)


# third party library
if(TARGET Boost::boost)
  target_link_libraries(${PROJECT_NAME} PRIVATE Boost::boost Boost::filesystem)
else()
  message(FATAL_ERROR "Boost not found")
endif()

if(TARGET Poco::Poco)
  target_link_libraries(${PROJECT_NAME} PRIVATE Poco::Poco)
else()
  message(FATAL_ERROR "Poco not found")
endif()

if(TARGET spdlog::spdlog)
  target_link_libraries(${PROJECT_NAME} PRIVATE spdlog::spdlog)
else()
  message(FATAL_ERROR "spdlog not found")
endif()

if(TARGET fmt::fmt)
    target_link_libraries(${PROJECT_NAME} PRIVATE fmt::fmt)
else()
    message(FATAL_message "fmt not found")
endif()

if(TARGET yaml-cpp::yaml-cpp)
    target_link_libraries(${PROJECT_NAME} PRIVATE yaml-cpp::yaml-cpp)
else()
    message(FATAL_message "yaml-cpp not found")
endif()

if(TARGET protobuf::protobuf)
  message("protobuf found")
  if(ENABLE_PROTOBUF_CONFIG)
    message("protobuf for config enable")
    target_link_libraries(${PROJECT_NAME} PRIVATE Demo::protobuf_config)
  else()
    message("protobuf for config disable")
  endif()
else()
  message(FATAL_message "protobuf not found")
endif()

# TODO: fix cppcheck check-config not found header file error,
# load from build dir output with \0 is wrong format and
# output xml which convert to html report.
if(TARGET cppcheck::cppcheck)
  message("using cppcheck")
  find_program(CPPCHECK cppcheck)
  # set(CPPCHECK_TEMPLATE "cppcheck: {file}:{line}:{severity}, {id}, {message}\\n{code}")
  # set(CPPCHECK_TEMPLATE_LOCATION "{file}:{line}: note: {info}\\n{code}")
  set(CPPCHECK_CHECK_DSN "${CMAKE_CURRENT_SOURCE_DIR}")
  set(CPPCHECK_BUILD_RESULT "${CMAKE_CURRENT_SOURCE_DIR}/cppcheck_result")
  list(
    APPEND CPPCHECK_CMD_OPTION
      "--enable=all"
      "--inconclusive"
      "--force"
      "--inline-suppr"
      "--verbose"
      # "--check-config"
      # "--xml"
      # "--output-file=${CPPCHECK_BUILD_RESULT}/output.xml"
      "--project=${CMAKE_BINARY_DIR}/compile_commands.json"
      "--template=gcc"
      "--suppress=missingInclude"
      # "--check-library"
      # "--addon=cert.py"
      # "--template=${CPPCHECK_TEMPLATE}"
      # "--template-location=${CPPCHECK_TEMPLATE_LOCATION}"
      "--cppcheck-build-dir=${CPPCHECK_BUILD_RESULT}"
      # unwant in release
      "-i ${CMAKE_CURRENT_SOURCE_DIR}/training"
      # TODO: fix checking config.pb.cc warning
      "-i ${CMAKE_CURRENT_SOURCE_DIR}/configuration"
      "-i ${CMAKE_BINARY_DIR}"
  )

  message("Path: ${CPPCHECK}")
  message("Current source: ${CMAKE_CURRENT_SOURCE_DIR}")
  target_link_libraries(${PROJECT_NAME} PRIVATE cppcheck::cppcheck)

  add_custom_target(cppcheck ALL COMMAND ${CPPCHECK}
    ${CPPCHECK_CMD_OPTION}
    ${CPPCHECK_CHECK_DSN} 
    # "--xml 2> output.xml"
    # "htmlreport/cppcheck-html-htmlreport --file=output.xml --report-dir=${CPPCHECK_BUILD_RESULT} --source-dir=."
    COMMENT "Cppcheck Checking ..."
    VERBATIM
  )

  message("debug value: ${CPPCHECK_CMD_OPTION}")
  message("htmlreport: ${CPPCHECK_HTMLREPORT}")
  # message("bin path: ${CPPCHECK}")
else()
  message(FATAL_ERROR "cppcheck not found")
endif()