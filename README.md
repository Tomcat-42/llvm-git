# llvm-git custom PKGBUILD

A fork of the "official"
[llvm-git aur package](https://aur.archlinux.org/packages/llvm-git) that also
installs the latest
[libc++, libc++abi and libunwind](https://libcxx.llvm.org/index.html).

## NOTE

Everything that was build against the official repo
[llvm](https://archlinux.org/packages/extra/x86_64/llvm/) package **WILL
BREAK**. This include the mesa drivers, our dear **Xorg** and others.

I suggest you to install [llvm16](https://aur.archlinux.org/packages/llvm16) and
[llvm16-libs](https://aur.archlinux.org/packages/llvm16) to have the current
`LLVM*16*.so*` libs available.

Also, some packages that depends on `libunwind.so.8` could be affected. I made a
symlink from `libunwind.so.1.0` to `libunwind.so.8` that should suffice. **If
some package break because libunwind, you know the reason.**
