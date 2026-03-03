# Git - create patches

------

## Create Kernel Patches

- find patches with author

  ```bash
  tony@tony-dell:/mnt/F/rk3562t/ms-rk3562t_13_0100_main/kernel-5.10$ git log --oneline --author="rock-chips.com"
  2e10ecb11b4b (tag: android-13.0-mid-rkr5) Merge commit '52f971ee6e023d89d24f9e3cd145d86d707e459c'
  52f971ee6e02 arm64: dts: rockchip: rk3562: Enable viLKsvPwrActive for soc bus
  d692435db7b0 mtd: spi-nor: esmt: Support New devices
  00cd81ba441d mtd: spi-nor: fmsh: Support New devices
  :
  
  # print out coloum 1 to get patches only
  tony@tony-dell:/mnt/F/rk3562t/ms-rk3562t_13_0100_main/kernel-5.10$ awk '{print $1}' rock-chips-patches.txt  
  ```

- - - 

  - 