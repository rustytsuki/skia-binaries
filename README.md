# skia-binaries
*this is rustytsuki's skia-binaries binary build*

*For the lack of some platform like 32bit windows in the [official](https://github.com/rust-skia/skia-binaries/releases) build*

##### how to use?

just set enviorenment var SKIA_BINARIES_URL pointing to this release file before you build your project with dependencies of skia-safe.

##### how to custom build, I'll show how to build for a target of i686-pc-windows-msvc as below.

When you compile a project with dep of skia-safe, if the target binary is not in the offical build, the skia-safe will automatically build. But in windows, there's big trouble, the cargo will download the skia-bindings and skia code into cargo registry. and from your code root path. the ninja.build file may contains some path like

`../../../../../../../../../.cargo/registry/src/github.com-1ecc6299db9ec823/skia-bindings-0.58.0/skia/third_party/externals/zlib/contrib/optimizations/inffast_chunk.c`

The key is that the gn system and ninja will call GetFullPathNameA() function in windows, this api has a limited path length, your path is too long.

and if your code is not in a same drive as rust installation, you'll get some path like this:

```
build obj/src/opts/avx.SkOpts_avx.obj: cxx ../../../../../../../../../C:...
```

you have to move the source directory to the same drive as your rust installation, or vice versa.

I found a way to enable long path support as followed

`git config --global core.longpaths true`

`regedit`
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem\LongPathsEnabled`

`gpedit.msc`
`Computer Configuration > Administrative Templates > System > Filesystem > Enable Win32 long paths`

but not work. Because "GetFullPathNameA()" is the the non unicode API version: this did never support long pathes.

so I placed my source code to .cargo/registry/src/github.com-1ecc6299db9ec823, thus the ninja.build generated short path. And it does work!

but, this is an ugly solution.

##### so I decided to build skia-bindings my self.

1. you have to install the *prerequisite* what the building need in https://github.com/rust-skia/rust-skia
2. use administrator mode in the terminal
3. checkout https://github.com/rust-skia/rust-skia and switch to the tag you want to build
4. remove rust-skia\skia-bindings\src\bindings.rs if the file existed before every time you build
5. `cd rust-skia/skia-bindings`
6. `cargo +nightly-x86_64-pc-windows-msvc build --release --features=gl --target=i686-pc-windows-msvc`
7. copy rust-skia\skia-bindings\src\bindings.rs
8. cd rust-skia\target\i686-pc-windows-msvc\release\build\skia-bindings-074fe6697fee20a3\out\skia
9. put bindings.rs, skia-bindings.lib, skia.lib and other needed by features to a folder called skia-binaries
10. `tar -czvf xxx.tar.gz skia-binaries`
11. done!

