cmake_minimum_required (VERSION 2.6)
project(CodeDB)

option(USE_RE2 "Use Google's re2 regular expression library." ON)

if(CMAKE_COMPILER_IS_GNUCC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -Wall")
endif()

# Set up boost include and library path
find_package(Boost 1.46.0 REQUIRED COMPONENTS system filesystem thread)
include_directories(${Boost_INCLUDE_DIRS})
link_directories(${BOOST_LIBRARYDIR})

if(NOT MSVC)
    find_package(Threads)
    set(LIBS ${Boost_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
endif()

# Set up re2
if(WIN32)
  # re2 is not available on windows
  set(USE_RE2 OFF)
endif()

if(USE_RE2)

  find_path    (RE2_INC re2.h   PATHS /usr/local/include /opt/local/include /usr/include PATH_SUFFIXES re2)
  find_library (RE2_LIB re2     PATHS /usr/local/lib /opt/local/lib /usr/lib)

  set(LIBS ${LIBS} ${RE2_LIB})

  include_directories (${RE2_INC})

  add_definitions("-DCDB_USE_RE2")
endif()

file(GLOB SOURCES "src/*.cpp")

file(GLOB SNAPPY_SRC "ext/snappy/*.cc")
include_directories("ext/snappy")

add_executable(cdb ${SOURCES} ${SNAPPY_SRC})
target_link_libraries(cdb ${LIBS})

## Packaging
install(TARGETS cdb RUNTIME DESTINATION bin)

set(CPACK_PACKAGE_EXECUTABLES cdb)

if(WIN32)
  # TODO: date
  set(CPACK_PACKAGE_VERSION "20120408")
  set(CPACK_GENERATOR zip)
else()
  execute_process(COMMAND date +%Y%m%d
                  OUTPUT_VARIABLE CPACK_PACKAGE_VERSION
                  OUTPUT_STRIP_TRAILING_WHITESPACE)

  set(CPACK_GENERATOR TBZ2)
endif()
include(CPack)
