# Maintainer: Jesus Jerez <jerezmoreno (at) gmail (dot) com>

pkgname=rocblas-gfx1031
pkgver=6.4.1
pkgrel=1
pkgdesc='Next generation BLAS implementation for ROCm platform'
arch=('x86_64')
url='https://rocblas.readthedocs.io/en/latest'
license=('MIT')
depends=(
  'rocm-core'
  'hip-runtime-amd'
  'roctracer'
  'glibc'
  'gcc-libs'
  'openmp'
  'cblas'
)
makedepends=(
  'git'
  'cmake'
  'rocm-cmake'
  'python'
  'python-virtualenv'
  'python-pyaml'
  'python-wheel'
  'python-tensile'
  'python-msgpack'
  'python-joblib'
  'perl-file-which'
  'msgpack-cxx'
  'gcc-fortran'
)
_rocblas='https://github.com/ROCm/rocBLAS'
_tensile='https://github.com/ROCm/Tensile'
source=("$pkgname-$pkgver.tar.gz::$_rocblas/archive/refs/tags/rocm-$pkgver.tar.gz"
        "$pkgname-tensile-$pkgver.tar.gz::$_tensile/archive/refs/tags/rocm-$pkgver.tar.gz"
        "tensile_gfx1031.patch"
        "rocblas_gfx1031.patch"
        "asm_full_navi22.tar.xz")
sha256sums=('517950ff6b3715dee8b2bcfbdd3968c65e1910e4b8e353e148574ae08aa6dc73'
            'f96fe39fbb0d43e39b258b21d66234abf3248f8cfa6954f922618d4bb7d04c74'
            'a65eb1c3fd48268a2894c3ec8ae81c9d80e687196563200de48b115f0a34c029'
            '65b048ef75b2e6ea028dccbe2ad337587e52d13f6bbf6201394aea1d1b25d08b'
            '5cbec2447f06fbe95b854e5d5b8a654014bc1c09da9972d6e7e4c3760a6f2c53')
options=(!lto)
_dirname="$(basename "$_rocblas")-$(basename "${source[0]}" ".tar.gz")"
_tensile_dir="$(basename "$_tensile")-$(basename "${source[1]}" ".tar.gz")"
provides=(rocblas)
conflicts=(rocblas)

prepare() {
    cd ${srcdir}/Tensile-rocm-${pkgver}
    patch --forward --strip=1 --input="${srcdir}/tensile_gfx1031.patch"

    cd ${srcdir}/rocBLAS-rocm-${pkgver}
    patch --forward --strip=1 --input="${srcdir}/rocblas_gfx1031.patch"

    cd ${srcdir}/rocBLAS-rocm-${pkgver}/library/src/blas3/Tensile/Logic/asm_full
    tar -xvf ${srcdir}/asm_full_navi22.tar.xz
}

build() {
  # Compile source code for supported GPU archs in parallel
  export HIPCC_COMPILE_FLAGS_APPEND="-parallel-jobs=$(nproc)"
  export HIPCC_LINK_FLAGS_APPEND="-parallel-jobs=$(nproc)"

  # -fcf-protection is not supported by HIP, see
  # https://rocm.docs.amd.com/projects/llvm-project/en/latest/reference/rocmcc.html#support-status-of-other-clang-options
  local cmake_args=(
    -Wno-dev
    -S "$_dirname"
    -B build
    -D CMAKE_BUILD_TYPE=None
    -D CMAKE_C_COMPILER=/opt/rocm/lib/llvm/bin/amdclang
    -D CMAKE_CXX_COMPILER=/opt/rocm/lib/llvm/bin/amdclang++
    -D CMAKE_TOOLCHAIN_FILE=toolchain-linux.cmake
    -D CMAKE_CXX_FLAGS="${CXXFLAGS} -fcf-protection=none"
    -D CMAKE_INSTALL_PREFIX=/opt/rocm
    -D CMAKE_PREFIX_PATH=/opt/rocm/llvm/lib/cmake/llvm
    -D amd_comgr_DIR=/opt/rocm/lib/cmake/amd_comgr
    -D BUILD_FILE_REORG_BACKWARD_COMPATIBILITY=OFF
    -D HIP_PLATFORM=amd
    -D BLAS_LIBRARY=cblas
    -D BUILD_WITH_TENSILE=ON
    -D Tensile_LIBRARY_FORMAT=msgpack
    -D Tensile_TEST_LOCAL_PATH="$srcdir/$_tensile_dir"
    -D Tensile_COMPILER=hipcc
    -D BUILD_WITH_PIP=OFF
    # hipblaslt doesn't support all relevant targets
    -D BUILD_WITH_HIPBLASLT=OFF
  )
  cmake "${cmake_args[@]}"
  cmake --build build
}

package() {
  DESTDIR="$pkgdir" cmake --install build

  install -Dm644 "$_dirname/LICENSE.md" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
