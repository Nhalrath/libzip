project('libzip', 'c', version: '1.11.3')

cc = meson.get_compiler('c')
config = configuration_data()

libzip_version = meson.project_version().split('.')
libzip_version_major = libzip_version[0]
libzip_version_minor = libzip_version[1]
libzip_version_micro = libzip_version[2]

config.set('PACKAGE', meson.project_name())
config.set('PACKAGE_VERSION', meson.project_version())
config.set('PACKAGE_VERSION_MAJOR', libzip_version_major)
config.set('PACKAGE_VERSION_MINOR', libzip_version_minor)
config.set('PACKAGE_VERSION_MICRO', libzip_version_micro)
config.set('VERSION', meson.project_version())

do_install = not meson.is_subproject()

use_crypto = false
use_commoncrypto = false
use_gnutls = false
use_mbedtls = false
use_openssl = false
use_windows_crypto = false

use_bzip2 = false
use_lzma = false
use_zstd = false

zlib_dep = dependency('zlib', version: '>=1.3.1', required: true)
zstd_dep = dependency('libzstd', version: '>=1.4.0', required: get_option('enable_zstd'))

libzip_deps = [zlib_dep]
libzip_cflags = []

stdint_includes = ''

if get_option('default_library') == 'static'
  config.set('ZIP_STATIC', true)
endif

# TODO: Crypto

if cc.get_id() == 'msvc'
  libzip_cflags += ['-D_CRT_SECURE_NO_WARNINGS', '-D_CRT_NONSTDC_NO_DEPRECATE']
endif

if host_machine.system() == 'windows'
  libzip_deps += cc.find_library('advapi32')
endif

# Checks

check_functions = [
  '_close',
  '_dup',
  '_fdopen',
  '_fileno',
  '_fseeki64',
  '_fstat64',
  '_setmode',
  '_stat64',
  '_strdup',
  '_strtoi64',
  '_strtoui64',
  '_unlink',
  'arc4random',
  'clonefile',
  'explicit_bzero',
  'explicit_memset',
  'fchmod',
  'fileno',
  'fseeko',
  'ftello',
  'getprogname',
  'GetSecurityInfo',
  'memcpy_s',
  'random',
  'setmode',
  'strdup',
  'strerror_s',
  'strerrorlen_s',
  'stricmp',
  'strncpy_s',
  'strtoll',
  'stroull'
]

foreach fn : check_functions
  config.set('HAVE_@0@'.format(fn.to_upper()), cc.has_function(fn))
endforeach

check_symbols = [
  ['_snprintf', 'stdio.h'],
  ['_snprintf_s', 'stdio.h'],
  ['_snwprintf_s', 'stdio.h'],
  ['_stricmp', 'string.h'],
  ['localtime_r', 'time.h'],
  ['localtime_s', 'time.h'],
  ['snprintf', 'stdio.h'],
  ['snprintf_s', 'stdio.h'],
  ['strcasecmp', 'strings.h'],
]

foreach sym_pair : check_symbols
  sym = sym_pair[0]
  header = sym_pair[1]
  config.set('HAVE_@0@'.format(sym.to_upper()), cc.has_header_symbol(header, sym))
endforeach

# TODO: Check for FTS library

have_fts = (
  cc.has_header('sys/types.h') and
  cc.has_header('sys/stat.h') and
  cc.has_header('fts.h')
)

if (have_fts)
  config.set('HAVE_FTS_H', have_fts)
  config.set('HAVE_FTS_OPEN', cc.has_function('fts_open'))
endif

check_headers = [
  'stdbool.h',
  'strings.h',
  'unistd.h'
]

foreach header : check_headers
  config.set('HAVE_@0@'.format(header.underscorify().to_upper()), cc.has_header(header))
endforeach

check_zlib_headers = [
  'inttypes.h',
  'stdint.h',
  'sys/types.h'
]

foreach header : check_zlib_headers
  if cc.has_header(header)
    stdint_includes = '@0@#include <@1@>\n'.format(stdint_includes, header)
    config.set('HAVE_@0@_LIBZIP'.format(header.underscorify().to_upper()), true)
  endif
endforeach

check_types = [
  '__int8',
  'int8_t',
  'uint8_t',
  '__int16',
  'int16_t',
  'uint16_t',
  '__int32',
  'int32_t',
  'uint32_t',
  '__int64',
  'int64_t',
  'uint64_t',
  'short',
  'int',
  'long long',
  'off_t',
  'size_t'
]

foreach type : check_types
  size = cc.sizeof(type, prefix: stdint_includes)
  name = type.underscorify().to_upper()
  config.set('@0@_LIBZIP'.format(name), size)
  config.set('HAVE_@0@_LIBZIP'.format(name), size > -1)
endforeach

config.set('HAVE_FICLONERANGE', cc.has_header_symbol('linux/fs.h', 'FICLONERANGE'))
config.set('WORDS_BIGENDIAN', host_machine.endian() == 'big')

# fixed size integral types

if config.get('HAVE_STDINT_H_LIBZIP')
  config.set('LIBZIP_TYPES_INCLUDE', '#include <stdint.h>')
elif config.get('HAVE_INTTYPES_H_LIBZIP')
  config.set('LIBZIP_TYPES_INCLUDE', '#include <inttypes.h>')
elif config.get('HAVE_SYS_TYPES_H_LIBZIP')
  config.set('LIBZIP_TYPES_INCLUDE', '#include <sys/types.h>')
endif

if config.get('HAVE_INT8_T_LIBZIP')
  config.set('ZIP_INT8_T', 'int8_t')
elif config.get('HAVE__INT8_LIBZIP')
  config.set('ZIP_INT8_T', '__int8')
else
  config.set('ZIP_INT8_T', 'signed char')
endif

if config.get('HAVE_UINT8_T_LIBZIP')
  config.set('ZIP_UINT8_T', 'uint8_t')
elif config.get('HAVE__UINT8_LIBZIP')
  config.set('ZIP_UINT8_T', '__uint8')
else
  config.set('ZIP_UINT8_T', 'unsigned char')
endif

if config.get('HAVE_INT16_T_LIBZIP')
  config.set('ZIP_INT16_T', 'int16_t')
elif config.get('HAVE__INT16_LIBZIP')
  config.set('ZIP_INT16_T', '__int16')
elif config.get('SHORT_LIBZIP') == 2
  config.set('ZIP_INT16_T', 'short')
endif

if config.get('HAVE_UINT16_T_LIBZIP')
  config.set('ZIP_UINT16_T', 'uint16_t')
elif config.get('HAVE__UINT16_LIBZIP')
  config.set('ZIP_UINT16_T', '__uint16')
elif config.get('SHORT_LIBZIP') == 2
  config.set('ZIP_UINT16_T', 'unsigned short')
endif

if config.get('HAVE_INT32_T_LIBZIP')
  config.set('ZIP_INT32_T', 'int32_t')
elif config.get('HAVE__INT32_LIBZIP')
  config.set('ZIP_INT32_T', '__int32')
elif config.get('INT_LIBZIP') == 4
  config.set('ZIP_INT32_T', 'int')
elif config.get('LONG_LIBZIP') == 4
  config.set('ZIP_INT32_T', 'long')
endif

if config.get('HAVE_UINT32_T_LIBZIP')
  config.set('ZIP_UINT32_T', 'uint32_t')
elif config.get('HAVE__INT32_LIBZIP')
  config.set('ZIP_UINT32_T', 'unsigned __int32')
elif config.get('INT_LIBZIP') == 4
  config.set('ZIP_UINT32_T', 'unsigned int')
elif config.get('LONG_LIBZIP') == 4
  config.set('ZIP_UINT32_T', 'unsigned long')
endif

if config.get('HAVE_INT64_T_LIBZIP')
  config.set('ZIP_INT64_T', 'int64_t')
elif config.get('HAVE__INT64_LIBZIP')
  config.set('ZIP_INT64_T', '__int64')
elif config.get('LONG_LIBZIP') == 8
  config.set('ZIP_INT64_T', 'long')
elif config.get('LONG_LONG_LIBZIP') == 8
  config.set('ZIP_INT64_T', 'long long')
endif

if config.get('HAVE_UINT64_T_LIBZIP')
  config.set('ZIP_UINT64_T', 'uint64_t')
elif config.get('HAVE__INT64_LIBZIP')
  config.set('ZIP_UINT64_T', 'unsigned __int64')
elif config.get('LONG_LIBZIP') == 4
  config.set('ZIP_UINT64_T', 'unsigned long')
elif config.get('LONG_LONG_LIBZIP') == 4
  config.set('ZIP_UINT64_T', 'unsigned long long')
endif

config.set('SIZEOF_OFF_T', config.get('OFF_T_LIBZIP'))

# config file template

config_template = files('config.h.in')
zipconf_template = files('zipconf.h.in')

subdir('lib')
