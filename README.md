# HOWTO: Install AMD ROCM / HIP on Pop!_OS (and use it with Blender)

Pop!_OS is not natively supported by ROCm and uses an unsupported kernel >=6.2 (as of June 2023).
This guide shows how to install and use it with the Blender Cycles renderer. 

Probably works also with other distributions which use newish kernels.

Environment:
- ROCm versions tested: 5.4 and 5.5
- Blender versions tested: 3.5 and 3.6
- Tested GPU: Vega 7 (Integrated Graphics in Ryzen 7 4800H APU)

## Step-by-step

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

    ![image](https://user-images.githubusercontent.com/18579177/232140758-0a78c6e1-0fee-4d45-a2cf-0075c9922e43.png)

   With my GPU there is no identifier string, but activating the combobox works nevertheless.
