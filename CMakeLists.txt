cmake_minimum_required(VERSION 2.8.10)
project(Mutilate)
message("lele multilate")
set(mutilate_SRCS
        mutilate.cc
        cmdline.cc
        log.cc
        distributions.cc
        util.cc
        Connection.cc
        Protocol.cc
        Generator.cc)

add_executable(mutilate ${mutilate_SRCS})

target_link_libraries(mutilate event pthread rt zmq)

set(test_generator_SRCS
        TestGenerator.cc
        log.cc
        Generator.cc)
add_executable(test_generator ${test_generator_SRCS})

target_link_libraries(test_generator event pthread rt zmq)
