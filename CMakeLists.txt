cmake_minimum_required(VERSION 3.4.3)

if (POLICY CMP0048)
  # Silence CMP0048 warning about missing project VERSION.
  cmake_policy(SET CMP0048 NEW)
endif()

project(include-what-you-use)

if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  message(STATUS "IWYU: out-of-tree configuration")

  find_package(LLVM CONFIG REQUIRED)
  find_package(Clang CONFIG REQUIRED)

  list(APPEND CMAKE_MODULE_PATH ${LLVM_DIR})
  include(AddLLVM)
  include(HandleLLVMOptions)
else()
  message(STATUS "IWYU: in-tree configuration")
endif()

message(STATUS "IWYU: configuring for LLVM ${LLVM_VERSION}...")

add_definitions(${LLVM_DEFINITIONS})
include_directories(
  ${LLVM_INCLUDE_DIRS}
  ${CLANG_INCLUDE_DIRS}
  )

get_target_property(clang_binary clang LOCATION)
message(STATUS "clang is installed to ${clang_binary}")
get_filename_component(clang_binary_dir ${clang_binary} DIRECTORY)
get_filename_component(clang_install_prefix ${clang_binary_dir} DIRECTORY)
get_filename_component(clang_real_install_prefix ${clang_install_prefix} REALPATH)
get_filename_component(this_install_prefix ${CMAKE_INSTALL_PREFIX} REALPATH)
message(STATUS "compare ${this_install_prefix} to ${clang_real_install_prefix}")
if(NOT (${this_install_prefix} STREQUAL ${clang_real_install_prefix}))
  message(STATUS "need to fix headers")
  message(STATUS "clang would look for its headers in ${clang_real_install_prefix}/lib/clang/${LLVM_VERSION}/include")
  message(STATUS "iwyu thus needs the headers in ${CMAKE_INSTALL_PREFIX}/lib/clang/${LLVM_VERSION}/include")
  if(EXISTS ${CMAKE_INSTALL_PREFIX}/lib/clang/${LLVM_VERSION}/include)
    message(STATUS "somebody already did it for us")
  else()
    message(STATUS "work to do")
    install(DIRECTORY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/clang/${LLVM_VERSION})
    install(CODE "
    EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E create_symlink ${clang_real_install_prefix}/lib/clang/${LLVM_VERSION}/include ${CMAKE_INSTALL_PREFIX}/lib/clang/${LLVM_VERSION}/include)
    ")
  endif()
else()
  message(STATUS "all fine")
endif()

add_llvm_executable(include-what-you-use
  iwyu.cc
  iwyu_ast_util.cc
  iwyu_cache.cc
  iwyu_driver.cc
  iwyu_getopt.cc
  iwyu_globals.cc
  iwyu_include_picker.cc
  iwyu_lexer_utils.cc
  iwyu_location_util.cc
  iwyu_output.cc
  iwyu_path_util.cc
  iwyu_preprocessor.cc
  iwyu_verrs.cc
  )

if (MINGW)
  # Work around 'too many sections' error with MINGW/GCC
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wa,-mbig-obj")
endif()

if (MSVC)
  # Disable warnings for IWYU, and disable exceptions in MSVC's STL.
  add_definitions(
    -wd4722 # Suppress ''destructor'' : destructor never returns, potential memory leak
    -D_HAS_EXCEPTIONS=0
    )

  # Put project in solution folder
  set_target_properties(include-what-you-use
    PROPERTIES FOLDER "Clang executables"
    )
endif()

# Prefer dynamic library if it exists.
if (TARGET LLVM)
  message(STATUS "IWYU: Linking LLVM dynamically")
  set(IWYU_LLVM_LIBS LLVM)
else()
  message(STATUS "IWYU: Linking LLVM statically")
  set(IWYU_LLVM_LIBS
    LLVMSupport
    LLVMX86AsmParser
    )
endif()

target_link_libraries(include-what-you-use
  clangFrontend
  clangSerialization
  clangDriver
  clangParse
  clangSema
  clangAnalysis
  clangAST
  clangBasic
  clangLex
)

# LLVM dependencies.
target_link_libraries(include-what-you-use
  LLVMX86AsmParser # MC, MCParser, Support, X86CodeGen, X86Desc, X86Info
  LLVMX86CodeGen # Analysis, AsmPrinter, CodeGen, Core, MC, Support, Target, 
                 # X86AsmPrinter, X86Desc, X86Info, X86Utils
  LLVMX86Desc # MC, MCDisassembler, Object, Support, X86AsmPrinter, X86Info
  LLVMX86AsmPrinter # MC, Support, X86Utils
  LLVMX86Info # Support
  LLVMX86Utils # Core, Support
  LLVMCodeGen # Analysis, Core, MC, Scalar, Support, Target, TransformUtils
  LLVMipo
  LLVMScalarOpts
  LLVMInstCombine
  LLVMTransformUtils
  LLVMTarget # Analysis, MC, Core, Support
  LLVMAnalysis # Core, Support
  LLVMOption # Support
  LLVMMCDisassembler # MC, Support
  LLVMMCParser # MC, Support
  LLVMMC # Object, Support
  LLVMProfileData # Core, Support, Object
  LLVMObject # BitReader, Core, Support
  LLVMBitReader # Core, Support
  LLVMCore # Support
  LLVMSupport
)

# Platform dependencies.
if (WIN32)
  target_link_libraries(include-what-you-use
    shlwapi
  )
elseif( UNIX )
  include(FindCurses)
  target_link_libraries(include-what-you-use
    pthread
    z
    ${CURSES_LIBRARIES}
    ${CMAKE_DL_LIBS}
  )
else()
  message(STATUS "Warning: IWYU Git version info not found, DO NOT release "
                 "from this build tree!")
endif()
add_definitions(-DIWYU_GIT_REV="${IWYU_GIT_REV}")

# Install programs
install(TARGETS include-what-you-use RUNTIME DESTINATION bin)
install(PROGRAMS fix_includes.py iwyu_tool.py DESTINATION bin)

# Install mapping files
file(GLOB MAPPING_FILES *.imp)
install(FILES ${MAPPING_FILES} DESTINATION share/include-what-you-use)
