Good variant of using non MSVC compiler on windows and separating build environment.  
\* WSL can be installed on Win10+ from build 19401.


# 1. [Install WSL](https://learn.microsoft.com/en-en/windows/wsl/install)  

1. Run Powershell as administrator
2. Check distributions in wsl: ```wsl --list --online``` 
3. Install distribution (by default Ubuntu will be installed): ```wsl --install -d Ubuntu 22.04 LTS```  
4. Reboot PC  
5. Run WSL: ```wsl --distribution Ubuntu-22.04``` (check names with ```wsl --list```)  
   First run will take time: all files will be unpacked and saved.

\* You can check linux filesystem by open windows explorer and go to \\\\wsl$.  
** You can move your wsl data to another location: ```wsl --export``` and ```--import```.  
*** [Basic wsl commands](https://learn.microsoft.com/en-en/windows/wsl/basic-commands#install)


# 2. Install VS Code
1. [Download VS Code and install](https://code.visualstudio.com/download)
2. [Install extension wsl](https://code.visualstudio.com/docs/remote/wsl)
3. Connect to linux via wsl extension
4. Install extensions for WSL: LINUX, not for LOCAL  
   Possible list of extensions: ```cmake tools, c/c++, git graph, git lens, back & forth```

# 3. Setup linux, install packets

```sudo apt get install git cmake build-essential ninja-build```


# Extra
## [Building clang](https://apt.llvm.org):
```
wget https://apt.llvm.org/llvm.sh  
chmod +x llvm.sh  
sudo ./llvm.sh
```

Set clang to default compiler instead of gcc:  
```
sudo update-alternatives --install /usr/bin/cc cc /usr/bin/clang-17 30  
sudo update-alternatives --install /usr/bin/c++ c++ /usr/bin/clang++-17 30
```


[Sharing Git credentials between Windows and WSL](https://code.visualstudio.com/docs/remote/troubleshooting#_sharing-git-credentials-between-windows-and-wsl)


## [Build and install perf](https://gist.github.com/abel0b/b1881e41b9e1c4b16d84e5e083c38a13)

On windows:  
```wsl --update```  
On wsl 2:  
```
sudo apt update  
sudo apt install flex bison  
sudo apt install libdwarf-dev libelf-dev libnuma-dev libunwind-dev \  
libnewt-dev libdwarf++0 libelf++0 libdw-dev libbfb0-dev \  
systemtap-sdt-dev libssl-dev libperl-dev python-dev-is-python3 \  
binutils-dev libiberty-dev libzstd-dev libcap-dev libbabeltrace-dev  
git clone https://github.com/microsoft/WSL2-Linux-Kernel --depth 1  
cd WSL2-Linux-Kernel/tools/perf  
make -j8 # parallel build  
sudo cp perf /usr/local/bin
```

## [Install instruction for (bcc)bpf](https://github.com/iovisor/bcc/blob/master/INSTALL.md#ubuntu---source)
