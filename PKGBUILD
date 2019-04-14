# Maintainer: Lone_Wolf <lonewolf@xs4all.nl>
# Contributor: yurikoles <root@yurikoles.com>
# Contributor: bearoso <bearoso@gmail.com>
# Contributor: Luchesar V. ILIEV <luchesar%2eiliev%40gmail%2ecom>
# Contributor: Anders Bergh <anders@archlinuxppc.org>
# Contributor: Armin K. <krejzi at email dot com>
# Contributor: Christian Babeux <christian.babeux@0x80.ca>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Evangelos Foutras <evangelos@foutrelis.com>
# Contributor: Hesiod (https://github.com/hesiod)
# Contributor: Roberto Alsina <ralsina@kde.org>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Tomas Lindquist Olsen <tomas@famolsen.dk>
# Contributor: Tomas Wilhelmsson <tomas.wilhelmsson@gmail.com>


pkgname=('llvm-git' 'llvm-libs-git')
pkgver=9.0.0_r314160.12886f04eae
pkgrel=1
_ocaml_ver=4.07.1
arch=('x86_64')
url="https://llvm.org/"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=(   'git' 'cmake' 'ninja' 'libffi' 'libedit' 'ncurses' 'libxml2' 'python-sphinx'
                            "ocaml=$_ocaml_ver" 'ocaml-ctypes' 'ocaml-findlib'
                            'python-sphinx' 'python-recommonmark' 'swig' 'python')

source=("llvm-project::git+https://github.com/llvm/llvm-project.git"
              'llvm-config.h'
              'enable-SSP-and-PIE-by-default.patch')
sha256sums=('SKIP'
            '597dc5968c695bbdbb0eac9e8eb5117fcd2773bc91edf5ec103ecffffab8bc48'
            '58f86da25eb230ed6d423b5b61870cbf3bef88f38103ca676a2c7f34b2372171')

_python_optimize() {
  python -m compileall "$@"
  python -O -m compileall "$@"
  python -OO -m compileall "$@"
}

pkgver() {
    cd llvm-project/llvm

    # This will almost match the output of `llvm-config --version` when the
    # LLVM_APPEND_VC_REV cmake flag is turned on. The only difference is
    # dash being replaced with underscore because of Pacman requirements.
    local _pkgver=$(awk -F 'MAJOR |MINOR |PATCH |)' \
            'BEGIN { ORS="." ; i=0 } \
             /set\(LLVM_VERSION_/ { print $2 ; i++ ; if (i==2) ORS="" } \
             END { print "\n" }' \
             CMakeLists.txt)_r$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
    echo "$_pkgver"
}

prepare() {
    if [  -d _build ]; then
        rm -rf _build
    fi
    mkdir _build
    cd llvm-project
    pushd clang
    patch -Np1 -i "$srcdir"/enable-SSP-and-PIE-by-default.patch
    popd
    # llvm-project contains a lot of stuff, remove parts that aren't used by this package
    rm -rf debuginfo-tests libclc libcxx libcxxabi libunwind llgo openmp parallel-libs pstl
    
    # llvm cmake uses things automagickally when they're at certain places in the sourcetree
    # TODO try building as external projects to avoid moving sourcetree around
    mv clang llvm/tools/clang
    mv clang-tools-extra llvm/tools/clang/tools/extra
    mv compiler-rt llvm/projects/compiler-rt
    mv lld llvm/tools/lld
    mv lldb llvm/tools/lldb
    mv polly llvm/tools/polly
}

build() {

    cd _build
    cmake "$srcdir"/llvm-project/llvm  -G Ninja \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DLLVM_APPEND_VC_REV=ON \
        -DLLVM_HOST_TRIPLE=$CHOST \
        -DLLVM_ENABLE_RTTI=ON \
        -DLLVM_ENABLE_FFI=ON \
        -DFFI_INCLUDE_DIR:PATH="$(pkg-config --variable=includedir libffi)" \
        -DLLVM_BUILD_LLVM_DYLIB=ON \
        -DLLVM_LINK_LLVM_DYLIB=ON \
        -DLLVM_INSTALL_UTILS=ON \
        -DLLVM_BUILD_TESTS=ON \
        -DLLVM_BUILD_DOCS=ON \
        -DLLVM_ENABLE_DOXYGEN=OFF \
        -DLLVM_ENABLE_SPHINX=ON \
        -DSPHINX_OUTPUT_HTML:BOOL=OFF \
        -DSPHINX_WARNINGS_AS_ERRORS=OFF \
        -DLLVM_BINUTILS_INCDIR=/usr/include \
        -DLLVM_VERSION_SUFFIX="" \
        -DPOLLY_ENABLE_GPGPU_CODEGEN=ON \
        -DLINK_POLLY_INTO_TOOLS=ON \
        -DCMAKE_POLICY_DEFAULT_CMP0075=NEW
    ninja "$MAKEFLAGS" all ocaml_doc
}

check() {
    cd _build
    # Dirty fix for unittests failing because the shared lib is not in the library path.
    # Also, disable the LLVM tests on i686 as they seem to fail too often there.
    [[ "$CARCH" == "i686" ]] || LD_LIBRARY_PATH="$srcdir"/_build/lib ninja check
    ninja check-clang
    ninja check-polly
    ninja check-lld
    ninja check-lldb
}

package_llvm-git() {
    pkgdesc="LLVM development version. includes clang and many other tools"
    depends=('llvm-libs-git' 'perl')
    optdepends=( 'python: for scripts'
                            'python-setuptools: for using lit (LLVM Integrated Tester)'
                            'ocaml: for ocaml support')
    # yes, I know polly is not in official repos. It just feels cleaner to list it
    provides=('llvm' 'compiler-rt' 'clang' 'lld' 'lldb' 'polly' 'llvm-ocaml'
                      'compiler-rt-git' 'clang-git' 'lld-git' 'lldb-git' 'polly-git' 'llvm-ocaml-git'
                      # legacy provides
                      'llvm-svn' 'compiler-rt-svn' 'clang-svn' 'lld-svn' 'lldb-svn' 'polly-svn' 'llvm-ocaml-svn' 
                      )
    conflicts=('llvm' 'compiler-rt' 'clang' 'lld' 'lldb' 'polly' 'llvm-ocaml')
    
    pushd _build

    DESTDIR="$pkgdir" ninja install
    
    popd
    # Clean up conflicting files
    # TODO: This should probably be discussed with upstream.
    rm -rf "${pkgdir}/usr/lib/python3.7/site-packages/six.py"
    # Include lit for running lit-based tests in other projects
    pushd llvm-project/llvm/utils/lit
    python setup.py install --root="$pkgdir" -O1
    popd

    
    # Move analyzer scripts out of /usr/libexec
    mv "$pkgdir"/usr/libexec/{ccc,c++}-analyzer "$pkgdir"/usr/lib/clang/
    rmdir "$pkgdir"/usr/libexec
    sed -i 's|libexec|lib/clang|' "$pkgdir"/usr/bin/scan-build

    # The runtime libraries go into llvm-libs
    mv -f "$pkgdir"/usr/lib/lib{LLVM,LTO}*.so* "$srcdir"
    mv -f "$pkgdir"/usr/lib/LLVMgold.so "$srcdir"

    if [[ $CARCH == x86_64 ]]; then
        # Needed for multilib (https://bugs.archlinux.org/task/29951)
        # Header stub is taken from Fedora
        mv "$pkgdir"/usr/include/llvm/Config/llvm-config{,-64}.h
        cp "$srcdir"/llvm-config.h "$pkgdir"/usr/include/llvm/Config/llvm-config.h
    fi

    cd llvm-project/llvm
    # Install Python bindings and optimize them
    cp -a bindings/python/llvm  "$pkgdir"/usr/lib/python3.7/site-packages/
    cp -a tools/clang/bindings/python/clang  "$pkgdir"/usr/lib/python3.7/site-packages/
    _python_optimize "$pkgdir"/usr/lib/python3.7/site-packages

    #optimize other python files except 2 problem cases
    _python_optimize "$pkgdir"/usr/share -x 'clang-include-fixer|run-find-all-symbols'
  
    install -Dm644 LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/llvm-LICENSE
    install -Dm644 tools/clang/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/clang-LICENSE
    install -Dm644 tools/clang/tools/extra/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/clang-tools-extra-LICENSE
    install -Dm644 projects/compiler-rt/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/compiler-rt-LICENSE
    install -Dm644 tools/lld/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/lld-LICENSE
    install -Dm644 tools/lldb/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/lldb-LICENSE
    install -Dm644 tools/polly/LICENSE.txt "$pkgdir"/usr/share/licenses/$pkgname/polly-LICENSE
}

package_llvm-libs-git() {
    pkgdesc="runtime libraries for llvm-git"
    depends=('gcc-libs' 'zlib' 'libffi' 'libedit' 'ncurses' 'libxml2')
    provides=('llvm-libs' 'llvm-libs-svn')
    conflicts=('llvm-libs')

    install -d "$pkgdir"/usr/lib
    cp -P \
        "$srcdir"/lib{LLVM,LTO}*.so* \
        "$srcdir"/LLVMgold.so \
        "$pkgdir"/usr/lib/

    # Symlink LLVMgold.so from /usr/lib/bfd-plugins
    # https://bugs.archlinux.org/task/28479
    install -d "$pkgdir"/usr/lib/bfd-plugins
    ln -s ../LLVMgold.so "$pkgdir"/usr/lib/bfd-plugins/LLVMgold.so

    cd llvm-project/llvm
    install -Dm644 LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/llvm-LICENSE
    install -Dm644 tools/clang/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/clang-LICENSE
    install -Dm644 tools/clang/tools/extra/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/clang-tools-extra-LICENSE
    install -Dm644 projects/compiler-rt/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/compiler-rt-LICENSE
    install -Dm644 tools/lld/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/lld-LICENSE
    install -Dm644 tools/lldb/LICENSE.TXT "$pkgdir"/usr/share/licenses/$pkgname/lldb-LICENSE
    install -Dm644 tools/polly/LICENSE.txt "$pkgdir"/usr/share/licenses/$pkgname/polly-LICENSE
}

# vim:set ts=2 sw=2 et:
