![cpp insights logo](artwork/logo_cppinsights.png)
# C++ Insights - See your source code with the eyes of a compiler.

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![download](https://img.shields.io/badge/latest-download-blue.svg)](https://github.com/andreasfertig/cppinsights/releases) [![Build Status](https://github.com/andreasfertig/cppinsights/workflows/ci/badge.svg)](https://github.com/andreasfertig/cppinsights/actions/)
[![codecov](https://codecov.io/gh/andreasfertig/cppinsights/branch/main/graph/badge.svg)](https://codecov.io/gh/andreasfertig/cppinsights)
[![Try online](https://img.shields.io/badge/try-online-blue.svg)](https://cppinsights.io)
[![Documentation](https://img.shields.io/badge/view-documentation-blue)](https://docs.cppinsights.io)
[![patreon](https://img.shields.io/badge/patreon-support-orange.svg)](https://www.patreon.com/cppinsights)

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io#https://github.com/andreasfertig/cppinsights)

## Contents

- [What](#what)
- [Why](#why)
- [Building](#building)
- [Usage](#usage)
- [Get Involved](#get-involved)
- [Support](#support)


## What

[C++ Insights](https://cppinsights.io) is a [Clang](https://clang.llvm.org)-based tool that does a source-to-source
transformation. The goal of C++ Insights is to make things visible that normally and intentionally happen behind the scenes.
It's about the magic the compiler does for us to make things work.

Take this piece of code, for example:

```.cpp
class Base {
};

class Derived : public Base {
};

int main() {
  Derived d;

  Derived d2 = d;

  d2 = d;

  Base& b = d;
}
```

Nothing special, and of course, it compiles. This is the compiler's view on it:

```.cpp
class Base
{
  public:
  // inline constexpr Base() noexcept = default;
  // inline constexpr Base(const Base &) noexcept = default;
  // inline constexpr Base & operator=(const Base &) noexcept = default;
};



class Derived : public Base
{
  public:
  // inline constexpr Derived() noexcept = default;
  // inline constexpr Derived(const Derived &) noexcept = default;
  // inline constexpr Derived & operator=(const Derived &) noexcept = default;
};



int main()
{
  Derived d;
  Derived d2 = Derived(d);
  d2.operator=(d);
  Base & b = static_cast<Base&>(d);
  return 0;
}
```

You can see all the compiler-provided special member functions and the upcast from `Derived` to `Base`.

## Why

[C++ Insights](https://cppinsights.io) is a [Clang](https://clang.llvm.org)-based tool that does a source-to-source transformation.
The goal of C++ Insights is to make things visible that normally and intentionally happen behind the scenes. It's about the magic the compiler does
for us to make things work. Or looking through the classes of a compiler.

In 2017, I started looking into some new things we got with C++11, C++14, and C++17. Amazing things like lambdas, range-based for-loops,
and structured bindings. I put it together in a talk. You can find the [slides](https://andreasfertig.com/talks/dl/afertig-ndcolo-2017-fast-and-small.pdf)
and a [video](https://youtu.be/Bt7KzFxcbgc) online.

However, all that research and some of my training and teaching got me to start thinking about how it would be if we could see with the eyes
of the compiler. Sure, there is an AST dump, at least for Clang. We can see what code the compiler generates from a C++ source snippet with
tools like Compiler Explorer. However, what we see is assembler. Neither the AST nor the Compiler Explorer output is in the language I write
code. Hence, I'm not very familiar with this output. Plus, when teaching students C++, showing an AST and explaining that it is all there was
not quite satisfying for me.

I started to write a Clang-based tool that can transform a range-based for-loop into the compiler-internal version. Then, I did the same
for structured bindings and lambdas. In the end, I did much more than initially planned. It shows where operators are
invoked and places in which the compiler does some casting. C++ Insights can deduce the type behind `auto` or `decltype`. The goal
is to produce compilable code. However, this is not possible in all places.

You can see, for example, the transformation of a [lambda](https://cppinsights.io/s/e4e19791), [range-based for-loop](https://cppinsights.io/s/0cddd172), or [auto](https://cppinsights.io/s/6c61d601). Of course, you can transform any other C++ snippet.

See yourself. C++ Insights is available online: [cppinsights.io](https://cppinsights.io).

Still, there is work to do.

I do not claim to get all the things right. I'm also working on supporting features from new standards, like C++20, at the moment.
Please remember that C++ Insights is based on Clang and its understanding of the AST.


I did a couple of talks about C++ Insights since I released C++ Insights. For example, at C++ now. Here are the [slides](https://andreasfertig.com/talks/dl/afertig-2021-cppnow-cpp-insights.pdf) and the [video](https://youtu.be/p-8wndrTaTs).


## Building

C++ Insights can be built inside or outside the Clang source tree.

### Building on Windows

See [Readme_Windows.md](Readme_Windows.md)

### Building on Arch Linux

To build with `extra/clang` use the following extra flags: `-DINSIGHTS_USE_SYSTEM_INCLUDES=off -DCLANG_LINK_CLANG_DYLIB=on -DLLVM_LINK_LLVM_DYLIB=on`

See https://github.com/andreasfertig/cppinsights/issues/186 for an explanation of why `INSIGHTS_USE_SYSTEM_INCLUDES` needs to be turned off.

`extra/clang` and `extra/llvm` provide `/usr/lib/{libclangAST.so,libLLVM*.a,libLLVM.so}`. `libclangAST.so` needs `libLLVM.so` and there would be a conflict if `libLLVM*.a` (instead of `libLLVM.so`) are linked. See https://bugs.archlinux.org/task/60512


### Building outside Clang

You need to have a Clang installation in the search path.

```
git clone https://github.com/andreasfertig/cppinsights.git
mkdir build && cd build
cmake -G"Ninja" ../cppinsights
ninja
```
The resulting binary (insights) can be found in the `build` folder.

### Building inside Clang

The easiest way to build C++ Insights inside the Clang source tree is using the `LLVM_EXTERNAL_PROJECTS` option.

```
git clone https://github.com/llvm/llvm-project.git
git clone https://github.com/andreasfertig/cppinsights.git

mkdir build
cd build
cmake -G Ninja -D=CMAKE_BUILD_TYPE=Release -DLLVM_EXTERNAL_PROJECTS=cppinsights -DLLVM_EXTERNAL_CPPINSIGHTS_SOURCE_DIR=<PATH/TO/cppinsights>  [INSIGHTS CMAKE OPTIONS] ../llvm-project/llvm

ninja
```



### cmake options

There are a couple of options that can be enabled with [cmake](https://cmake.org):

| Option              | Description                | Default |
|---------------------|:---------------------------| --------|
| INSIGHTS_STRIP      | Strip insight after build  | ON      |
| INSIGHTS_STATIC     | Use static linking         | OFF     |
| INSIGHTS_COVERAGE   | Enable code coverage       | OFF     |
| INSIGHTS_USE_LIBCPP | Use libc++ for tests       | OFF     |
| DEBUG               | Enable debug               | OFF     |

### Building for ARM on macOS

It seems best to supply the architecture during configuration:

```
cmake -DCMAKE_OSX_ARCHITECTURES=arm64 ../cppinsights
```


### Use it with [Cevelop](https://www.cevelop.com)

```
git clone https://github.com/andreasfertig/cppinsights.git
mkdir build_eclipse
cd build_eclipse
cmake -G"Eclipse CDT4 - Unix Makefiles" ../cppinsights/
```

Then, in [Cevelop](https://www.cevelop.com) Import -> General -> Existing Project into Workspace. Select `build_eclipse`. Enjoy editing with
[Cevelop](https://www.cevelop.com).

## Usage

Using C++ Insights is fairly simple:

```
insights <YOUR_CPP_FILE> -- -std=c++17
```

Things get complicated when it comes to the system-include paths. These paths are hard-coded in the binary, which seems
to come from the compiler C++ Insights was built with. To help with that, check out [scripts/getinclude.py](scripts/getinclude.py). The script tries to
collect the system-include paths from the compiler. Without an option, `getinclude.py` uses `g++`. You can also pass another compiler
as a first argument.

Here is an example:

```
./scripts/getinclude.py
-isystem/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../include/c++/v1 -isystem/usr/local/include -isystem/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/7.3.0/include -isystem/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include -isystem/usr/include
```

The script can be used together with C++ Insights:

```
insights <YOUR_CPP_FILE> -- -std=c++17 `./scripts/getinclude.py`
```


### Custom GCC installation

In case you have a custom build of the GCC compiler, for example, gcc-11.2.0, and _NOT_ installed in the compiler in the default system path, then after building, Clang fails to find the correct `libstdc++` path (GCC's STL). If you run into this situation, you can use "`--gcc-toolchain=/path/GCC-1x.x.x/installed/path`" to tell Clang/C++ Insights the location of the STL:

```
./cppinsights Insights.cpp -- --gcc-toolchain=${GCC_11_2_0_INSTALL_PATH} -std=c++20
```

Here "`${GCC_11_2_0_INSTALL_PATH}`" is the installation directory of your customized-built GCC. The option for Clang is described [here](https://clang.llvm.org/docs/ClangCommandLineReference.html#cmdoption-clang-gcc-toolchain).


### Ready to use Docker container

There is also another GitHub project that sets up a docker container with the latest C++ Insights version in it: [C++
Insights - Docker](https://github.com/andreasfertig/cppinsights-docker)


## Plugins / Extensions / Packages

### C++ Insights @ Vim

A plugin for Vim is available at
[here](https://github.com/Freed-Wu/cppinsights.vim).

### C++ Insights @ Neovim

A plugin for Neovim is available at
[here](https://github.com/Freed-Wu/cppinsights.nvim).

### C++ Insights @ VSCode

An extension for Visual Studio Code is available at the VS Code marketplace: [C++
Insights - VSCode Extension](https://marketplace.visualstudio.com/items?itemName=devtbi.vscode-cppinsights).


### C++ Insights @ brew

At least for macOS, you can install C++ Insights via Homebrew thanks to [this formular](https://formulae.brew.sh/formula/cppinsights):

```
brew install cppinsights
```

## Bash autocomplete

There is a bash autocomplete script. It autocompletes the C++ Insights options as well as the Clang options provided after `--`.
You can download it with the following commands:

```
cd <YOUR_BASH_COMPLETION.D>
curl -o insights https://github.com/andreasfertig/cppinsights/tree/main/scripts/bash-autocomplete.sh
```


## Compatibility

I aim for the repository to compile with the latest version of Clang and at least the one before. The website tries to
stay close to the latest release of Clang. However, due to certain issues (building Clang for Windows), the website's
version is often delayed by a few months.


## C++ Insights @ YouTube

I created a [YouTube](https://youtube.com/@andreas_fertig) channel where I release a new video each month. In
these videos, I use C++ Insights to show and explain certain C++ constructs, and sometimes I explain C++ Insights as well.


## ToDo's

See [TODO](TODO.md).


## Get Involved
+ Report bugs/issues by submitting a [GitHub issue](https://github.com/andreasfertig/cppinsights/issues).
+ Submit contributions using [pull requests](https://github.com/andreasfertig/cppinsights/pulls). See [Contributing](CONTRIBUTING.md)

## Support

If you like to support the project, consider [submitting](CONTRIBUTING.md) a patch. Another alternative is to become a [GitHub Sponsor](https://github.com/sponsors/andreasfertig) or a [Patreon](https://www.patreon.com/cppinsights) supporter.


