# Copyright - 2015 - Jan Christoph Uhde <Jan@UhdeJC.com>

cmake_minimum_required(VERSION 3.2.0)
project(libsnap VERSION 2.4)

if (CMAKE_HOST_WIN32)
else(CMAKE_HOST_WIN32)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_CXX_FLAGS_DEBUG
                "${CMAKE_CXX_FLAGS_DEBUG} -Wall -DNOMP -O0 -ggdb"
                )
        set(CMAKE_CXX_FLAGS_RELEASE
                "${CMAKE_CXX_FLAGS_DEBUG} -Wall -DNOMP -DNDEBUG -O3 -ggdb"
                )
    elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        set(CMAKE_CXX_FLAGS_DEBUG
                "${CMAKE_CXX_FLAGS_DEBUG} -Wall -O0 -ggdb"
                )
        set(CMAKE_CXX_FLAGS_RELEASE
                "${CMAKE_CXX_FLAGS_DEBUG} -Wall -DNDEBUG -O3 -ggdb"
                )
        find_package(OpenMP)
        if (OPENMP_FOUND)
            set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
            set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
        endif()
    endif()
endif(CMAKE_HOST_WIN32)

set(GLIB_CORE_HDR
        glib-core/base.h
        glib-core/bd.h
        glib-core/bits.h
        glib-core/blobbs.h
        glib-core/ds.h
        glib-core/dt.h
        glib-core/env.h
        glib-core/fl.h
        glib-core/gnuplot.h
        glib-core/hash.h
        glib-core/html.h
        glib-core/http.h
        glib-core/json.h
        glib-core/linalg.h
        glib-core/lx.h
        glib-core/md5.h
        glib-core/os.h
        glib-core/shash.h
        glib-core/ss.h
        glib-core/stdafx.h
        glib-core/tm.h
        glib-core/unicode.h
        glib-core/unicodestring.h
        glib-core/url.h
        glib-core/ut.h
        glib-core/wch.h
        glib-core/xfl.h
        glib-core/xmath.h
        glib-core/xml.h
        glib-core/xmlser.h
        glib-core/zipfl.h
        )

set(SNAP_CORE_HDR
        snap-core/alg.h
        snap-core/anf.h
        snap-core/bfsdfs.h
        snap-core/bignet.h
        snap-core/centr.h
        snap-core/cmty.h
        snap-core/cncom.h
        snap-core/coreper.h
        snap-core/ff.h
        snap-core/flow.h
        snap-core/gbase.h
        snap-core/ggen.h
        snap-core/ghash.h
        snap-core/gio.h
        snap-core/graph.h
        snap-core/gstat.h
        snap-core/gsvd.h
        snap-core/gviz.h
        snap-core/kcore.h
        snap-core/network.h
        snap-core/Snap.h
        snap-core/statplot.h
        snap-core/stdafx.h
        snap-core/subgraph.h
        snap-core/timenet.h
        snap-core/triad.h
        snap-core/util.h
        )

## lib
add_library(snap SHARED ./snap-core/Snap.cpp)
target_include_directories(snap PUBLIC glib-core snap-core)
install(FILES ${GLIB_CORE_HDR} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/snap/glib-core)
install(FILES ${SNAP_CORE_HDR} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/snap/snap-core)
install(TARGETS snap LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

## tutorials
if(TUTORIALS)
    file(GLOB TUTORIAL_CPP tutorials/*cpp)
    foreach(tutorial_src ${TUTORIAL_CPP})
        get_filename_component(tutorial ${tutorial_src} NAME_WE)
        add_executable(${tutorial} ${tutorial_src})
        target_link_libraries(${tutorial} snap)
    endforeach()
endif()

## tests
if(TESTS)
    file(GLOB TEST_SRC test/*cpp)
    find_package(GTest)
    if(GTEST_FOUND)
        enable_testing()
        add_executable(run-all-tests EXCLUDE_FROM_ALL ${TEST_SRC})
        target_include_directories(run-all-test PRIVATE ${GTEST_INCLUDE_DIRS})
        target_link_libraries(run-all-tests snap ${GTEST_BOTH_LIBRARIES})
        add_test(test run-all-tests)
    else()
        if(IS_DIRECTORY /usr/src/gtest)
            #if file exists - try to build form source
            enable_testing()
            add_executable(run-all-tests EXCLUDE_FROM_ALL ${TEST_SRC})
            add_subdirectory(/usr/src/gtest ${CMAKE_CURRENT_BINARY_DIR}/gtest)
            target_link_libraries(run-all-tests snap gtest)
            add_test(test run-all-tests)
        endif()
    endif()
endif()


## examples
# adding the examples is unfortunatly not easy :(
#add_subdirectory(examples)

# building this as lib is impossible as the files depend on the examples
#file(GLOB SNAP_ADV_CPP snap-adv/*cpp)
#file(GLOB GLIB_ADV_CPP glib-adv/*cpp)
#add_library(snap-adv ${SNAP_ADV_CPP} ${GLIB_ADV_CPP})
#list(REMOVE_ITEM GLIB_ADV_CPP "glib-adv/stemming.cpp")
#target_include_directories(snap-adv PUBLIC glib-adv snap-adv)