# Meson build file
#
# SPDX-FileCopyrightText: 2021 Comcast Cable Communications Management, LLC
# SPDX-License-Identifier: Apache-2.0


project('cutils', 'c',
        version: '2.1.3',
        license: ['Apache-2.0', 'MIT', 'Unlicense'],
        default_options: ['c_std=c99',
                          'b_coverage=true'])


if not meson.is_subproject()
  # We want strict flags
  warning_level = 3   # -Wall
  werror = true
endif

################################################################################
# Common variable definitions
################################################################################

inc_base = 'include/'+meson.project_name()

################################################################################
# Generate the version header file
################################################################################

subdir('include/'+meson.project_name())

install_headers([headers, ver_h], subdir: meson.project_name())

################################################################################
# Define the libraries
################################################################################

inc = include_directories(inc_base)

sources = ['src/base64.c',
           'src/file.c',
           'src/hashmap.c',
           'src/hashmap_crc.c',
           'src/memory.c',
           'src/must.c',
           'src/nl_ctype.c',
           'src/printf.c',
           'src/strings.c',
           'src/xxd.c']

libcutils = library(meson.project_name(),
                    sources,
                    include_directories: inc,
                    install: true)

################################################################################
# Define the tests
################################################################################

if not meson.is_subproject()
  test_args = ['-fprofile-arcs', '-ftest-coverage', '-O0']

  cunit_dep = dependency('cunit')

  tests = [['test base64',            'test_base64'],
           ['test file',              'test_file'],
           ['test hashmap',           'test_hashmap'],
           ['test hashmap collision', 'test_hashmap_collision'],
           ['test memory',            'test_memory'],
           ['test printf',            'test_printf'],
           ['test nl_strings',        'test_nl_strings'],
           ['test strings',           'test_strings'],
           ['test xxd',               'test_xxd']]

  foreach test : tests
    test(test[0],
         executable(test[1], ['tests/'+test[1]+'.c'],
                    include_directories: inc,
                    dependencies: cunit_dep,
                    install: false,
                    link_args: test_args,
                    link_with: libcutils))
  endforeach

  test('test must',
       executable('test_must', ['tests/test_must.c', 'src/must.c'],
                  include_directories: inc,
                  dependencies: cunit_dep,
                  install: false,
                  link_args: [test_args, '-D__cutils_abort=__test_abort']),
       should_fail: true)

  add_test_setup('valgrind',
                 is_default: true,
                 exe_wrapper: [ 'valgrind',
                                '--leak-check=full',
                                '--error-exitcode=1' ],
                 timeout_multiplier: 2)
endif

################################################################################
# Add the license check target
################################################################################

reuse_bin = find_program('reuse', required: false)
if reuse_bin.found()
  custom_target('license',
                command: [reuse_bin, 'lint'],
                build_always_stale: true,
                output: ['license'])
endif

################################################################################
# Add the coverity check target
################################################################################

alias_target('coverity', libcutils )

################################################################################
# External/consumer definitions
################################################################################

libcutils_dep = declare_dependency(include_directories: ['include'],
                                          link_with: libcutils)

if meson.version().version_compare('>=0.54.0')
  meson.override_dependency(meson.project_name(), libcutils_dep)
endif
