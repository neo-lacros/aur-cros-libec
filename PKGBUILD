# Maintainer: Shadichy <shadichy@blisslabs.org>

pkgname=cros-libec
pkgver=0.0.1
pkgrel=1
pkgdesc='ChromiumOS Embedded Controller library (libec)'
arch=('x86_64')
url='https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/libec'
license=('BSD-3-Clause')
depends=('cros-libchrome' 'cros-libbrillo' 'gcc-libs' 'glibc' 'libusb')
makedepends=('clang' 'cros-common-mk' 'git' 'gn' 'lld' 'llvm' 'compiler-rt' 'ninja' 'pkgconf' 'python' 'abseil-cpp')
options=('!lto')

_local_src_dir=""

_resolve_local_src() {
  _local_src_dir=""
  local base=""
  [[ -n "${CROS_LOCAL_SRC:-}" ]] || return 0
  base="$(cd "${startdir:-$PWD}" && cd "${CROS_LOCAL_SRC}" 2>/dev/null && pwd)" || return 0
  [[ -d "${base}/tree/src/platform2/libec" ]] && _local_src_dir="${base}/tree/src/platform2"
  [[ -z "${_local_src_dir}" && -d "${base}/src/platform2/libec" ]] && _local_src_dir="${base}/src/platform2"
  [[ -z "${_local_src_dir}" && -d "${base}/platform2/libec" ]] && _local_src_dir="${base}/platform2"
  return 0
}

_resolve_local_src
if [[ -n "${_local_src_dir}" ]]; then
  source=(
    '0001-arch-portability.patch'
    '0002-fourcc-cstdint.patch'
  )
  sha256sums=(
    'c1b5bc9c89c1fac6f13ddcc641d2aa20f109d077a304187e15a85b5fd62c48fb'
    '492fecaa1976b1cd525f857e2809ad98a1d3404514c7b9bb7b9510f395375598'
  )
else
  source=(
    'platform2::git+https://chromium.googlesource.com/chromiumos/platform2.git#branch=main'
    '0001-arch-portability.patch'
    '0002-fourcc-cstdint.patch'
  )
  sha256sums=(
    'SKIP'
    'c1b5bc9c89c1fac6f13ddcc641d2aa20f109d077a304187e15a85b5fd62c48fb'
    '492fecaa1976b1cd525f857e2809ad98a1d3404514c7b9bb7b9510f395375598'
  )
fi

prepare() {
  local src_dir="${srcdir}/platform2"
  _resolve_local_src
  [[ -z "${_local_src_dir}" ]] || src_dir="${_local_src_dir}"
  rm -rf "${srcdir}/work"
  mkdir -p "${srcdir}/work"
  install -Dm644 "${src_dir}/LICENSE" "${srcdir}/work/LICENSE"
  tar -C "${src_dir}/libec" --exclude='./.git' --exclude='__pycache__' --exclude='*.pyc' --exclude='*.pyo' -cf - . | tar -C "${srcdir}/work" -xf -
  cd "${srcdir}/work"
  patch -Np1 -i "${srcdir}/0001-arch-portability.patch"
  patch -Np1 -i "${srcdir}/0002-fourcc-cstdint.patch"

  _ec_src=""
  if [[ -n "${CROS_LOCAL_SRC:-}" ]]; then
    for _cand in "${CROS_LOCAL_SRC}/tree/src/platform/ec" "${CROS_LOCAL_SRC}/src/platform/ec" "${CROS_LOCAL_SRC}/platform/ec"; do
      if [[ -f "${_cand}/include/ec_commands.h" ]]; then _ec_src="${_cand}"; break; fi
    done
  fi
  if [[ -n "${_ec_src}" && ! -f /usr/include/chromeos/ec/ec_commands.h ]]; then
    mkdir -p "${srcdir}/work/ec-headers/chromeos/ec"
    cp "${_ec_src}/include/ec_commands.h" "${srcdir}/work/ec-headers/chromeos/ec/ec_commands.h"
    cp "${_ec_src}/include/panic_defs.h" "${srcdir}/work/ec-headers/chromeos/ec/panic_defs.h"
    cp "${_ec_src}/util/cros_ec_dev.h" "${srcdir}/work/ec-headers/chromeos/ec/cros_ec_dev.h"
  fi
}

build() {
  cd "${srcdir}/work"
  export CC=clang
  export CXX=clang++
  export AR=llvm-ar
  if [[ -d "${srcdir}/work/ec-headers" ]]; then
    export CPPFLAGS="${CPPFLAGS:-} -I${srcdir}/work/ec-headers"
  fi
  rm -rf "${srcdir}/out"
  platform2-arch-build \
    --pkg libec \
    --src "${srcdir}/work" \
    --out "${srcdir}/out" \
    --libdir /usr/lib \
    --use-flags "" \
    --action deviterate \
    -- libec libec_pc
}

package() {
  install -Dm755 "${srcdir}/out/out/Default/lib/libec.so" \
    "${pkgdir}/usr/lib/libec.so"
  install -Dm644 "${srcdir}/out/out/Default/obj/libec/libec.pc" \
    "${pkgdir}/usr/lib/pkgconfig/libec.pc"
  install -Dm644 "${srcdir}/work/LICENSE" \
    "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
