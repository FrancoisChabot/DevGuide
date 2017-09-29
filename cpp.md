# My C++ checklist

Example projects:
- [Abulafia](https://github.com/FrancoisChabot/abulafia)

## For all projects

### .clang-format

There should be a .clang-format file living at the top of the project directory. Furthermore the contribution guide should be
providing instructions for quickly processing all the source. For reference:

```bash
find include -iname *.h | xargs clang-format -i -style=file
find tests -iname *.h -o -iname *.cpp | xargs clang-format -i -style=file
find examples -iname *.h -o -iname *.cpp | xargs clang-format -i -style=file
```
### third_party top-level directory

Every single external dependency, submodule or copied, goes in there.

### Unit testing with googletest

- Add [googletest](https://github.com/google/googletest/) as a git submodule (in `third_party/`)
- All tests live under a `tests` top-level directory
- gtest is supplemented with a memoy-leak detector

### Continous integration

[travis-ci](travis-ci.org) for unix-like platforms, and/or [Appveyor](https://www.appveyor.com/) for windows build. Remember to put status badges in the readme!

### CMakeLists.txt

I've wrestled with too many build systems to count. I feel using cmake is still, to this day, the best way to have a project that's **actually** portable.

### Header Sanitizer

Have a process to ensure that every single header file includes the proper dependencies, and does not rely accidently on pre-inclusion.

Using a glob means that new headers won't get picked up until cmake rebuilds itself, but as long as continuous integration is in place, it does not matter.

```cmake
# List of abulafia headers
FILE(GLOB_RECURSE
  PROJECT_HEADERS
  RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h
)

# Create the header sanitizer targets
foreach(HEADER_FILE ${PROJECT_HEADERS})
  get_filename_component(TGT_DIR ${HEADER_FILE} DIRECTORY)
  file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/${TGT_DIR})
  file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/${HEADER_FILE}.cpp "#include \"${HEADER_FILE}\"")

  list(APPEND HEADER_SAN_SRC ${CMAKE_CURRENT_BINARY_DIR}/${HEADER_FILE}.cpp)
endforeach()

add_library(header_sanitizer ${HEADER_SAN_SRC})
```

## Header-only projects

### The library should live in `include/project_name`

Not only that, but that directory should **only** contain the public headers of the project.

### There should be a top-level include

It should live at 'include/project_name/project_name.h`

### There should be a script to generate a single-file version of the library

This is really usefull so that tools like godbolt.org can use the library.

The following script does the job fine for simple cases: [Link](https://github.com/FrancoisChabot/header_compiler)
