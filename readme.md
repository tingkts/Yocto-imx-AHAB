### Porting imx Encrypted AHAB on imx8qm imx-5.15.71_2.2.1 yocto

#### AHAB / Encrypted AHAB ：  i.MX High Assurance Boot

The NXP document ：

- [AN12312 : Secure Boot on AHAB Supported Devices](https://www.nxp.com/webapp/Download?colCode=AN12312&location=null)

- [AN13994 : i.MX Encrypted Boot on AHAB-Enabled Devices](https://www.nxp.com/webapp/Download?colCode=AN13994&location=null)

- [IMX8QMSRM : Security Reference Manual for i.MX8QuadMax Application Processors](https://www.nxp.com/webapp/sps/download/mod_download.jsp?colCode=IMX8QMSRM&location=null&appType=moderated)



#### Porting guide ： imx8qm imx-5.15.71_2.2.1 yocto

The NXP SOP `u-boot-imx/doc/imx/ahab`： the step by step guilde how to enable encrypted AHAB boot progress.

```
⭐ This SOP is a very clear step-by-step instruction document, just follow it.

yocto-5.15.71_2.2.1-bsp/build_mx8mpevk_xwayland_imx-image-full/workspace/sources/u-boot-imx/doc/imx/ahab
.
├── csf_examples
│   ├── csf_boot_image_sgk.txt
│   ├── csf_boot_image.txt
│   ├── csf_enc_boot_image.txt
│   ├── csf_linux_img.txt
│   └── csf_uboot_atf.txt
├── guides
│   ├── mx8_mx8x_encrypted_boot.txt
│   ├── mx8_mx8x_secure_boot.txt
│   ├── mx8_mx8x_spl_secure_boot.txt
│   ├── mx8ulp_secure_boot.txt
│   └── sign_os_cntr.txt
└── introduction_ahab.txt
```


[imx-mkimage](https://github.com/nxp-imx/imx-mkimage) : i.MX Mkimage Bootloader Tool. used to build boot/kernel imaage, or there is the recipe `imx-boot` in the imx 5.15.71 _ 2.2.1 yocto bsp for convenient use.


[cst-3.4.0](https://www.nxp.com/webapp/sps/download/license.jsp?colCode=IMX_CST_TOOL_NEW&appType=file2&DOWNLOAD_ID=null) : the nxp Code Signing Tool, used to sign and encrypt boot/kernel image





Concept & Steps：

１　 u-boot-imx enable AHAB／Encrypted AHAB support

```
    // AHAB support
    CONFIG_AHAB_BOOT=y

    // Encrypted AHAB support
    CONFIG_CMD_DEKBLOB=y
    CONFIG_IMX_SECO_DEK_ENCAP=y
    CONFIG_FAT_WRITE=y
```


2　 follow `introduction_ahab.txt` to build SRK Table and its Hash code.

```
    // the PKI tree :

                             ┌─────────┐
                             │   CA    │
                             └────┬────┘
                                  │
                                  │
         ┌───────────────┬────────┴────────┬───────────────┐
         │               │                 │               │
         v               v                 v               v
    ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐
    │  SRK1  │       │  SRK2  │       │  SRK3  │       │  SRK4  │
    └────┬───┘       └───┬────┘       └────┬───┘       └───┬────┘
         │               │                 │               │
         v               v                 v               v
    ┌────────┐       ┌────────┐       ┌────────┐       ┌────────┐
    │  SGK1  │       │  SGK2  │       │  SGK3  │       │  SGK4  │
    └────────┘       └────────┘       └────────┘       └────────┘
```


```
    // the output signature keys :

    cst-3.4.0/crts/CA1_sha384_secp384r1_v3_ca_crt.der
    cst-3.4.0/crts/CA1_sha384_secp384r1_v3_ca_crt.pem

    cst-3.4.0/crts/SRK1_sha384_secp384r1_v3_ca_crt.der
    cst-3.4.0/crts/SRK1_sha384_secp384r1_v3_ca_crt.pem
    cst-3.4.0/crts/SRK1_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SRK1_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SRK2_sha384_secp384r1_v3_ca_crt.der
    cst-3.4.0/crts/SRK2_sha384_secp384r1_v3_ca_crt.pem
    cst-3.4.0/crts/SRK2_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SRK2_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SRK3_sha384_secp384r1_v3_ca_crt.der
    cst-3.4.0/crts/SRK3_sha384_secp384r1_v3_ca_crt.pem
    cst-3.4.0/crts/SRK3_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SRK3_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SRK4_sha384_secp384r1_v3_ca_crt.der
    cst-3.4.0/crts/SRK4_sha384_secp384r1_v3_ca_crt.pem
    cst-3.4.0/crts/SRK4_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SRK4_sha384_secp384r1_v3_usr_crt.pem

    cst-3.4.0/crts/SGK1_1_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SGK1_1_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SGK2_1_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SGK2_1_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SGK3_1_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SGK3_1_sha384_secp384r1_v3_usr_crt.pem
    cst-3.4.0/crts/SGK4_1_sha384_secp384r1_v3_usr_crt.der
    cst-3.4.0/crts/SGK4_1_sha384_secp384r1_v3_usr_crt.pem

    cst-3.4.0/crts/SRK_1_2_3_4_table.bin    // it's the final SRK_Table
    cst-3.4.0/crts/SRK_1_2_3_4_fuse.bin     // it's the Hash code of SRK_Table
```

&emsp;&emsp;SRK Table： combine into the Signature Block of the final signed image

&emsp;&emsp;SRK Table Hash： program into SoC eFuse　　　！！！ must be careful tha eFause is OTP, it's irreversible.


3　follow `mx8_mx8x_spl_secure_boot.txt`　`mx8_mx8x_encrypted_boot.txt` to build signed encrypted imx boot image.

&emsp;&emsp;&emsp;&emsp;　there are 3 image container in the boot image, flash_spl.bin.

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1st container : SECO FW

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;2nd container : SC FW, SPL          &nbsp;&nbsp;&nbsp; _signed, encrypted_

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;3rd container : u-boot, ATF         &nbsp;&nbsp;&nbsp; _signed, encrypted_

```
        // boot image layout, flash_spl.bin

             +---------------------------+ ---------
             |   1st Container header    |   ^
             |       and signature       |   |
             +---------------------------+   |
             | Padding for 1kB alignment |   |
             +---------------------------+   |
             |   2nd Container header    |   |
             |       and signature       |   |
             +---------------------------+   |
             |          Padding          |   |  Authenticated at
             +---------------------------+   |   SCU ROM Level
             |          SECO FW          |   |
             +---------------------------+   |
             |          Padding          |   |
             +---------------------------+   |
             |      SCU FW + DCD Table   |   |
             +---------------------------+   |
             |       Cortex-M Image      |   |
             +---------------------------+   |
             |         SPL Image         |   v
             +---------------------------+ ---------
             |   3rd Container header    |   ^
             |       and signature       |   |
             +---------------------------+   |
             |          Padding          |   | Authenticated
             +---------------------------+   |  at SPL Level
             |     U-Boot Proper IMG     |   |
             +---------------------------+   |
             |    ARM Trusted FW (ATF)   |   v
             +---------------------------+ ---------

      // container header, signature block, decrypted key dek_blob.bin

             +----------------------------+
             |                            |  ^
             |                            |  |
             |      Container header      |  |
             |                            |  |
             |                            |  |
             +---+------------------------+  |
             | S | Signature block header |  | Signed
             | i +------------------------+  |
             | g |                        |  |
             | n |                        |  |
             | a |        SRK table       |  |
             | t |                        |  |
             | u |                        |  v
             | r +------------------------+
             | e |       Signature        |
             |   +------------------------+
             | B |                        |
             | l |        SGK Key         |
             | o | Certificate (optional) |
             | c |                        |
             | k +------------------------+
             |   |        DEK Blob        |
             +---+------------------------+

```

```
　　　 3.1 build, sign 3rd image container, u-boot-atf-container.img

            // build 3rd image container
            $ cd ${imx-mkimage}
            $ make SOC=iMX8QM u-boot-atf-container.img

            // sign 3rd image container
            $ cd ${cst-3.4.0}/linux64/bin
            $ cp ${imx-mkimage}/iMX8QM/u-boot-atf-container.img .
            $ cp csf_uboot_atf.txt .
            $ ./cst -i csf_uboot_atf.txt -o signed_u-boot-atf-container.img

　　　 3.2 build, sign, encrypt whole boot image, flash.bin (or called flash_spl.bin, naming is arbitrary)

            // build boot image, flash.bin
            $ cp signed_u-boot-atf-container.img ${imx-mkimage}/iMX8QM/u-boot-atf-container.img
            $ make SOC=iMX8QM flash_spl // output file is flash.bin

            // sign, encrypt boot image
            $ cd ${cst-3.4.0}/linux64/bin
            $ cp ${imx-mkimage}/iMX8QM/flash.bin .
            $ cp csf_boot_image.txt .
            $ ./cst -i csf_boot_image.txt -o signed-encrypted-flash.bin  // will also output decrypt key of dek.bin

　　　 3.3 make dek_blob.bin, and append into signed-encrypted-flash.bin

            U-Boot prompt:
            => mmc list
                FSL_SDHC: 1 (SD)
                FSL_SDHC: 2
            => fatload mmc 1:1 0x80280000 dek.bin
            => dek_blob 0x80280000 0x80280100 128
            => fatwrite mmc 1:1 0x80280100 dek_blob.bin 0x48

            copy output dek_blob.bin back to  ${cst-3.4.0}, then
            insert DEK Blob into container signature block:

            // need to modify offset "0x7c0" by the log printed by above cst command
            $ dd if=dek_blob.bin of=signed-encrypted-flash.bin bs=1 seek=$((0x7c0)) conv=notrunc

　　　 3.4 all done, now ready to flash boot image, signed-encrypted-flash.bin

          uuu signed-encrypted-flash.bin
```

4　follow `sign_os_cntr.txt`　`mx8_mx8x_encrypted_boot.txt` to build signed encrypted kernel & dtb image, this step is smailiar wuth 3 above.


```
      4.1 build, sign, encrypt kernel+dtb iamge, must be naming as os_cntr_signed.bin

          // build kernel+dtb image
          $ cp Image imx8qm-mek.dtb ${imx-mkimage}/iMX8QM
          $ make SOC=iMX8QM flash_kernel // ouput file is also flash.bin

          // sign, encrypt kernel+dtb image
          $ cd ${cst-3.4.0}/linux64/bin
          $ cp ${imx-mkimage}/iMX8QM/flash.bin flash_os.bin
          $ cp csf_linux_img.txt .
          $ ./cst -i csf_linux_img.txt -o signed-encrypted-flash_os.bin // will also output decrypt key of dek_os.bin

      4.2. make dek_os_blob.bin, and append into signed-encrypted-flash_os.bin

         U-Boot prompt:
         => mmc list
             FSL_SDHC: 1 (SD)
             FSL_SDHC: 2
         => fatload mmc 1:1 0x80280000 dek_os.bin
         => dek_blob 0x80280000 0x80280100 128
         => fatwrite mmc 1:1 0x80280100 dek_blob_os.bin 0x48

         // need to modify offset "0x340" by the log printed by above cst command
         $ dd if=dek_blob_os.bin of=enc_flash_os.bin bs=1 seek=$((0x340)) conv=notrunc

      4.3 must be rename enc_flash_os.bin to os_cntr_signed.bin

      4.4 all done, now ready to put os_cntr_signed.bin into the emmc boot partition (emmc partition 1, in the imx8qm is mmc 0: 1)

        // mount emmc partition 1
        $ rm Image
        $ rm imx8qm-mek.dtb or imx8qm-mx8mpevk.dtb
        $ cp os_cntr_signed.bin .


         // or use uuu command
        $ sudo ./uuu ucmd setenv fastboot_buffer ${cntr_addr}
        $ sudo ./uuu download -i os_cntr_signed.bin
        $ sudo ./uuu FB: ucmd fatwrite mmc 0:1 ${fastboot_buffer} os_cntr_signed.bin <size in hex>
```



</br>
</br>
</br>

[GitHub - nxp-imx/uboot-imx: i.MX U-Boot](https://github.com/nxp-imx/uboot-imx)　│　[u-boot-imx/doc/imx/ahab](https://github.com/nxp-imx/uboot-imx/tree/lf_v2022.04/doc/imx/ahab)

[csf_uboot_atf.txt](./csf_uboot_atf.txt)　│　[csf_boot_image.txt](./csf_boot_image.txt)　│　[csf_linux_img.txt](./csf_linux_img.txt)

