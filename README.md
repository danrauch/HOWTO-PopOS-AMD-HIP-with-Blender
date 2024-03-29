# HOWTO: Install AMD ROCM / HIP on Pop!_OS (and use it with Blender)

This guide shows how to install ROCm on Pop OS! 22.04 and use it with the Blender Cycles renderer. 

Probably works also with other unsupported distributions which use newish kernels.

Environment:
- Pop!_OS 22.04 LTS
- Tested kernels: 6.2 and 6.5
- ROCm versions tested: 5.4 - 5.7.1
- Blender versions tested: 3.5 - 4.0
- Tested GPU: Vega 7 (Integrated Graphics in Ryzen 7 4800H APU)

## Step-by-step

> **For ROCm versions >=5.6 and Pop OS 22.04 (current version as of July 2023) you can simply use the AMD install instructions for the very convenient "[package manager install](https://rocm.docs.amd.com/en/latest/deploy/linux/os-native/install.html)", which I highly recommend. Follow the steps for "Ubuntu / Ubuntu 22.04". You can skip to step 3 after completing this install variant.**

> **The ROCm documentation for versions 5.7.0 and 5.7.1 state that these versions do not support iGPUs, so please keep that in mind**

1. use AMD install instruction to get install script -> [link to latest](https://rocm.docs.amd.com/en/latest/deploy/linux/installer/install.html) 

    Relevent steps (for version 5.5)

    ```bash
    sudo apt update
    wget https://repo.radeon.com/amdgpu-install/5.5.1/ubuntu/jammy/amdgpu-install_5.5.50501-1_all.deb
    sudo apt install ./amdgpu-install_5.5.50501-1_all.deb
    ```

2. Use the install script
    
    First "pop" has to be added to the supported distributions of the script.
    To do so edit the install script at `/usr/bin/amdgpu-install` via a text editor and add `|pop` at around line 396:
    
    ```bash
    case "$ID" in
    ubuntu|linuxmint|debian|pop)
        ...
    ```

    Afterwards run
    
    ```bash
    sudo amdgpu-install --no-dkms
    ```

    Optionally you can specify more "usecases" here if you need ROCm for different stuff like ML. Check the AMD install instructions for more details.
    Important is "--no-dkms", to not build the kernel module (will fail for the newer kernel we use).

3. Add user to groups so /dev/kfd can be accessed

    If Blender shows an error like "hip: Invalid device" it is probably because of skipping this step.

    ```bash
    sudo usermod -a -G video $USER
    sudo usermod -a -G render $USER
    ```

    A restart might be required at this point.

    More info can be found in the [prerequisites section](https://rocm.docs.amd.com/en/latest/deploy/linux/prerequisites.html#setting-permissions-for-groups) of the official AMD documentation.

5. Test if the installation was successful and the priviledges are set by running

    ```bash
    rocminfo 
    ```
    without sudo!
    
    If you get no permission error and your GPU is detected, everything should be set up.

6. Use the tarball of Blender from the Blender download page, _not_ the flatpak version from the software center!

    For install hints look [here](https://ubuntuhandbook.org/index.php/2021/12/blender-3-0-released-install-tarball/).

    To see if the flatpak supports HIP yet, follow these issues / proposals:
    - https://github.com/flatpak/flatpak/issues/5383
    - https://github.com/flathub/org.blender.Blender/issues/121
    - https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1535
    - https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/12612

7. Set HIP device in the Blender "Preferences"->"System" tab and activate Cycles renderer with GPU compute

    ![image](https://github.com/danrauch/HOWTO-PopOS-AMD-HIP-with-Blender/assets/18579177/8ff4e23d-bae3-48b5-87c2-ed7676d73fbb)

   With older ROCm versions (<5.6) and for my GPU there is no identifier string, but activating the combobox works nevertheless.
