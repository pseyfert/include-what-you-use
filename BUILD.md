config:
```sh
PATH=/cvmfs/lhcb.cern.ch/lib/bin/Linux-x86_64:${PATH} /cvmfs/lhcb.cern.ch/lib/bin/Linux-x86_64/cmake -GNinja .. -DIWYU_LLVM_ROOT_PATH=/cvmfs/sft.cern.ch/lcg/contrib/clang/6.0.0binutils/x86_64-centos7/ -DCMAKE_INSTALL_PREFIX=/var/iwyu/include-what-you-use/install -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_RULE_MESSAGES=NO -DCMAKE_EXPORT_COMPILE_COMMANDS=YES -DCMAKE_CXX_STANDARD=17 -DCMAKE_CXX_COMPILER=/cvmfs/lhcb.cern.ch/lib/bin/x86_64-centos7/lcg-clang++-6.0.0
```

build:
```sh
PATH=/cvmfs/lhcb.cern.ch/lib/bin/Linux-x86_64:${PATH} /cvmfs/lhcb.cern.ch/lib/bin/Linux-x86_64/cmake --build . --target install
```

fixup install:
```sh
mkdir -p /var/iwyu/include-what-you-use/install/lib
ln -s /cvmfs/sft.cern.ch/lcg/contrib/clang/6.0.0binutils/x86_64-centos7/lib/clang /var/iwyu/include-what-you-use/install/lib
```

run:
```
PATH=/cvmfs/lhcb.cern.ch/lib/lcg/releases/llvm/6.0.0/x86_64-centos7-gcc7-opt/bin:/cvmfs/lhcb.cern.ch/lib/lcg/releases/gcc/7.3.0/x86_64-centos7/bin:/cvmfs/lhcb.cern.ch/lib/lcg/releases/binutils/2.28/x86_64-centos7/bin${PATH:+:$PATH} LD_LIBRARY_PATH=/cvmfs/lhcb.cern.ch/lib/lcg/releases/llvm/6.0.0/x86_64-centos7-gcc7-opt/lib64:/cvmfs/lhcb.cern.ch/lib/lcg/releases/gcc/7.3.0/x86_64-centos7/lib64:/cvmfs/lhcb.cern.ch/lib/lcg/releases/binutils/2.28/x86_64-centos7/lib${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} /var/iwyu/include-what-you-use/install/bin/include-what-you-use --gcc-toolchain=/cvmfs/lhcb.cern.ch/lib/lcg/releases/gcc/7.3.0/x86_64-centos7
```
