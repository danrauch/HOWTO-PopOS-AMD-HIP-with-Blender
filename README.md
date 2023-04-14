# Install ROCM and HIP on Pop OS Kernel 6.2

Probably works also with other distributions which use newish kernels.

Tested GPU: Vega 7 (Integrated Graphics in Ryzen 7 4800H APU)

## Step-by-step

1. use AMD install instruction to get install script -> [link](https://docs.amd.com/bundle/ROCm-Installation-Guide-v5.4.3/page/How_to_Install_ROCm.html) (version 5.4)

    Relevent steps (version 5.4)

    ```bash
    sudo apt-get update
    wget https://repo.radeon.com/amdgpu-install/5.4.3/ubuntu/jammy/amdgpu-install_5.4.50403-1_all.deb 
    sudo apt-get install ./amdgpu-install_5.4.50403-1_all.deb
    ```

2. Use the install script

    I'm not sure if every "usecase" is necessary (at least use "rocm" and "hip") or if the "--opencl=rocr" part is necessary.
    Important is "--no-dkms", to not build the kernel module (will fail for the newer kernel we use)

    ```bash
    sudo amdgpu-install --usecase=rocm,hip,graphics,opencl --opencl=rocr --no-dkms
    ``

3. Add user to groups so /dev/kfd can be accessed

    I'm not sure if both are necessary, but this worked for me.
    If Blender shows an error like "hip: Invalid device" it is probably because of skipping this step.

    ```bash
    sudo usermod -a -G video <your-username>
    sudo usermod -a -G render <your-username>
    ``

4. Test if the installation was successful and the priviledges are set by running

    ```bash
    rocminfo 
    ```
    without sudo!
    
    If you get no permission error and your GPU is detected, everything should be set up.

5. Use the tarball of Blender from the Blender download page, _not_ the flatpak version from the software center!

    To see if the flatpak supports HIP yet, follow these issues / proposals:
    https://github.com/flatpak/flatpak/issues/5383
    https://github.com/flathub/org.blender.Blender/issues/121
    https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1535
    https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/12612

6. Set HIP device in the Blender "Preferences"->"System" tab and activate Cycles renderer with GPU compute

    ![image](https://user-images.githubusercontent.com/18579177/232140758-0a78c6e1-0fee-4d45-a2cf-0075c9922e43.png)
