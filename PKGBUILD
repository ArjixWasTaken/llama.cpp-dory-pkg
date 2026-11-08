# Maintainer: ArjixWasTaken <me@arjix.dev>
#
# llama.cpp built from the Dory branch (ggml-org/llama.cpp#30306) for ROCm gfx1031 + Vulkan.
# Drop-in replacement for llama.cpp-gfx1031 until the PR lands upstream.

pkgname=llama.cpp-dory-gfx1031
_commit=98a74c97ea09ac72f6ceda1559808fc0a2f447ce
pkgver=pr30306.${_commit:0:7}
pkgrel=1
pkgdesc="LLM inference in C/C++ with NVIDIA Dory support (PR #30306), ROCm gfx1031 + Vulkan, socket-activated server"
arch=(x86_64)
url="https://github.com/ggml-org/llama.cpp/pull/30306"
license=(MIT)
depends=(
  curl
  gcc-libs
  glibc
  openmp
  openssl
  hip-runtime-amd
  hipblas
  rocblas
  vulkan-icd-loader
)
makedepends=(
  git
  cmake
  rocm-llvm
  rocm-cmake
  vulkan-headers
  shaderc
  spirv-headers
  glslang
)
provides=(llama.cpp llama.cpp-hip llama.cpp-vulkan llama-cpp libggml ggml ggml-rocm ggml-vulkan)
conflicts=(llama.cpp-gfx1031 llama.cpp llama.cpp-hip llama.cpp-vulkan llama-cpp libggml ggml ggml-rocm ggml-vulkan stable-diffusion.cpp)
replaces=(llama.cpp-gfx1031)
backup=(
  etc/conf.d/llama.cpp
  etc/llama.cpp/models.ini
)
options=(!debug !lto)
install=llama.cpp.install
source=(
  "llama.cpp::git+https://github.com/brianchmielNV/llama.cpp.git#commit=${_commit}"
  llama.cpp.service
  llama.cpp.socket
  llama.cpp-proxy.service
  llama.cpp.conf
  models.ini
  llama.cpp.sysusers
  llama.cpp.tmpfiles
  wait-ready
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

build() {
  # Prebuilt web UI, same version the Dory model card uses.
  export HF_UI_VERSION=b11551
  export HIPCXX=/opt/rocm/lib/llvm/bin/clang
  export HIP_PATH=/opt/rocm

  # Built in CI, so never tune for the build machine's CPU.
  # Target is a Ryzen 7 5700X (Zen 3: AVX2/FMA/F16C/BMI2, no AVX-512).
  cmake -S llama.cpp -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DBUILD_SHARED_LIBS=ON \
    -DGGML_NATIVE=OFF \
    -DGGML_AVX=ON -DGGML_AVX2=ON -DGGML_FMA=ON -DGGML_F16C=ON -DGGML_BMI2=ON \
    -DGGML_HIP=ON \
    -DGPU_TARGETS=gfx1031 \
    -DGGML_VULKAN=ON \
    -DGGML_RPC=ON \
    -DGGML_BACKEND_DL=OFF \
    -DLLAMA_OPENSSL=ON \
    -DLLAMA_BUILD_SERVER=ON \
    -DLLAMA_BUILD_TOOLS=ON \
    -DLLAMA_BUILD_EXAMPLES=ON \
    -DLLAMA_BUILD_TESTS=OFF \
    -DLLAMA_BUILD_UI=OFF \
    -DLLAMA_USE_PREBUILT_UI=ON
  cmake --build build --parallel "$(nproc)"
}

package() {
  DESTDIR="$pkgdir" cmake --install build

  install -Dm644 llama.cpp/LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  install -Dm644 llama.cpp.service       "$pkgdir/usr/lib/systemd/system/llama.cpp.service"
  install -Dm644 llama.cpp.socket        "$pkgdir/usr/lib/systemd/system/llama.cpp.socket"
  install -Dm644 llama.cpp-proxy.service "$pkgdir/usr/lib/systemd/system/llama.cpp-proxy.service"
  install -Dm644 llama.cpp.sysusers      "$pkgdir/usr/lib/sysusers.d/llama.cpp.conf"
  install -Dm644 llama.cpp.tmpfiles      "$pkgdir/usr/lib/tmpfiles.d/llama.cpp.conf"
  install -Dm755 wait-ready              "$pkgdir/usr/lib/llama.cpp/wait-ready"
  install -Dm644 llama.cpp.conf          "$pkgdir/etc/conf.d/llama.cpp"
  install -Dm644 models.ini              "$pkgdir/etc/llama.cpp/models.ini"
}
