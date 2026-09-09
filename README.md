Its a kinda pointless fork (its more of a guide, there isn't really a true patch, and this can get outdated if any proton fork decides it to implement it)

## Dependencies
- Any RTX 30x series (20x series hasn't been well tested in general)
- Open-Nvidia drivers installed, this has been tested on ```NVIDIA-SMI 610.57.04 KMD Version: 610.57.04 CUDA UMD Version: 13.3```
- NVLibs installed into the prefix, or enabled through a Proton fork (This whole thing has been tested on proton-cachyos-11.0-20260703-slr, also will be using this method)
- You NEED a real Windows 10/11 ```version.dll``` file, the proxy will NOT work unless you replace your WINE's version.dll. This repo will contain one, you can use your own file, or extract one from an ISO or DLL website.
- Not all games has been tested, some may reports failures, some may work weird, some not. Its something you have to test for itself and report it. In my case I tested it on Far Far West and WuKong benchmark, both worked well.

## Installation
Due to the nature of this ""fork"" (its more of a guide) I will not put any of the DLSSG patch files, you will download everything from the real [source](https://github.com/sdli1995/dlssg_for_sm86) the only file available here will be a version.dll extracted from a Classic 7 installation (Windows 10 IoT Enterprise LTSC 2021)

# BEFORE DOING ANYTHING FIRST OPEN THE GAME AND LET IT CREATE A PREFIX!!! THEN DO EVERYTHING ELSE
1. After opening the game and closing it, clone the dlssg repo ```git clone https://github.com/sdli1995/dlssg_for_sm86 && cd dlssg_for_sm86```
2. Copy both the ```version.dll``` and ```dlssg_sm86.ini``` to the actual game rendering executable (on unreal based games this mean go to more folders)
3. You will then clone this repo or download the release file, and copy the ```version.dll``` to the system32 prefix folder, an example would be (in this case for Black Myth Wukong benchmark would be; ```/compatdata/3132990/pfx/drive_c/windows/system32``` remember that the prefix folder will always be named as the ID for the game, you can use SteamDB for that)
4. After all of this, you will use this launch option ```PROTON_NVIDIA_LIBS=1 WINEDLLOVERRIDES="version=n,b" %command%``` its VERY important that you use both, specially ```PROTON_NVIDIA_LIBS=1``` or the proxy will NOT work.
