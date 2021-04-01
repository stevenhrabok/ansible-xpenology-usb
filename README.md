# Ansible Playbook - Xpenology USB Creator


- [Ansible Playbook - Xpenology USB Creator](#ansible-playbook---xpenology-usb-creator)
  * [Introduction](#introduction)
  * [Supported OS](#supported-os)
  * [Prerequisites](#prerequisites)
  * [Collect Required Files](#collect-required-files)
  * [Playbook Instructions](#playbook-instructions)
    + [Initial data collection](#initial-data-collection)
    + [Running Playbook](#running-playbook)
      - [Command for users with passwordless sudo access](#command-for-users-with-passwordless-sudo-access)
      - [Command for users with password required sudo access](#command-for-users-with-password-required-sudo-access)
      - [Initial Vars Prompts](#initial-vars-prompts)
      - [Additional prompts during run](#additional-prompts-during-run)
        * [Selecting USB Device Prompt](#selecting-usb-device-prompt)
        * [Manual Changes Prompt](#manual-changes-prompt)
  * [Playbook Completion](#playbook-completion)


## Introduction
This ansible playbook automates the process of creating a Bootable Xpenology USB Device.

Please read all necessary documentation for setting up Xpenology before reading this guide. Try the start-here guides below:
https://xpenology.com/forum/forum/83-faq-start-here/


## Supported OS
Currently this playbook only supports Mac OSX (Darwin).
In future other Linux distributions may be added or if other individuals want to contribute.
Windows does not support ansible (outside of Windows Subsystem for Linux) and will not be included.


## Prerequisites
The following prerequisites are required to run this playbook
- sudo (root access by sudo)
  - https://support.apple.com/en-ca/HT204012
- homebrew (installed, required to install 7zip)
  - https://brew.sh/
- ansible (installed, can be installed using brew)
  - https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
  - https://formulae.brew.sh/formula/ansible


## Collect Required Files
**this README is using Synology model 3617, adjust accordingly for the model you want to use**

Place all of the files in the `xpenology` folder found in this project.

Download the Synology PAT file for the version of Xpenology/Synology to be installed/upgraded.
https://archive.synology.com/download/Os/DSM


Download the Xpenology Boot Loader (synoboot.img) zip file specific to the version of Xpenology/Synology to be installed/upgraded.
https://xpenology.com/forum/forum/31-loaders/


Optional: Download the Xpenology Extra Drivers (extra.lzma) zip file specific to the version of Xpenology/Synology to be installed/upgraded.
**Extra Drivers are only needed if your devices (nic, raid, etc) are not supported by the current boot loader files**
https://xpenology.com/forum/forum/91-additional-compiled-modules/


Place downloaded files in `xpenology` folder of this project.
Example:
```
.
├── README.md
├── xpenology
│   ├── README.md
│   ├── DSM_DS3617xs_25426.pat
│   ├── extra3617_v0.11.2_test.zip
│   └── synoboot_3617.zip
└── xpenology-usb.yml
```


## Playbook Instructions
The instructions provided herein will guide you through the process of running the playbook and what values(variables) will be requested from the user.


### Initial data collection
The playbook will prompt for values specific to your Xpenology hardware to configure the grub.cfg.
The following values are required:
- Serial number for the model being used, use existing or generate a new one
  - Format Example: 1129ODN024917
  - Note these are not official Synology serial numbers
  - Generator: https://xpenogen.github.io/serial_generator/index.html
- MAC address of NIC on your Xpenology hardware
  - Format Example: 90:2B:34:AC:9F:C4
  - Can be found in bios, use a bootable USB Linux distribution, or physical nic mac information on card label if available
- Filenames in the xpenology folder
  - Bootloader/Synoboot Filename
    - Format Example: synoboot_3617.zip
  - Extra Drivers Filename
    - Leave blank if not required
    - Format Example: extra3617_v0.11.2_test.zip
  - Synology PAT Filename
    - Format Example: DSM_DS3617xs_25426.pat 

Optional: You can edit the `vars_prompt` section at the top of the `xpenology-usb.yml` file and populate the `default` field with the contents of your values for easier typing or copy/paste use. This also helps if you plan to run multiple times or like to retain your device settings for future use. (Keep xpenology_extra_drivers_filename default blank if not being used).
```yaml
  vars_prompt:
    - name: xpenology_serial_number
      prompt: "Enter a generated serial number (https://xpenogen.github.io/serial_generator/index.html)"
      default: "1230ODN022844"
      private: no
    - name: xpenology_nic_mac_address
      prompt: "Enter the nic mac address from your xpenology device (Format example: 90:2B:34:AC:9F:C4)"
      default: "90:2B:34:AC:9F:C4"
      private: no
    - name: xpenology_synoboot_filename
      prompt: "Enter the xpenology synoboot zip file name (File should be placed in xpenology folder)"
      default: "synoboot_3617.zip"
      private: no
    - name: xpenology_extra_drivers_filename
      prompt: "Enter the xpenology extra drivers zip file name (Leave blank if none required. File should be placed in xpenology folder)"
      default: "extra3617_v0.11.2_test.zip"
      private: no
    - name: synology_pat_filename
      prompt: "Enter the synology pat file name (File should be placed in xpenology folder)"
      default: "DSM_DS3617xs_25426.pat"
      private: no
```


### Running Playbook
**Make sure you've inserted your USB device you would like to install the bootloader on**
**This will destroy all data on the USB device, please be sure you read all the prompts**

![ansible-xpenology-usb](ansible-xpenology-usb-20210331.svg)

When running `ansible-playbook` there are arguments that may need to be specified depending on your sudo permissions.

#### Command for users with passwordless sudo access
`ansible-playbook xpenology-usb.yml`

#### Command for users with password required sudo access
`ansible-playbook -K xpenology-usb.yml` (note: `-K` can be replaced with the expanded equivalent `--ask-become-pass`)

Once the command has been run you will be prompted for your sudo password (your user password) called `BECOME password:` in ansible (because it will "become" a user).
```
ansible-playbook xpenology-usb.yml -K
BECOME password: 
```

#### Initial Vars Prompts
On initial playbook run you will see some `[WARNING]` messages, these can be ignored.

Enter the values requested by the prompts, if you populated the defaults you can hit enter to accept the default.

```
ansible-playbook xpenology-usb.yml -K
BECOME password:
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
Enter a generated serial number (https://xpenogen.github.io/serial_generator/index.html) [1230ODN022844]: 
Enter the nic mac address from your xpenology device (Format example: 90:2B:34:AC:9F:C4) [90:2B:34:AC:9F:C4]: 
Enter the xpenology synoboot zip file name (File should be placed in xpenology folder) [synoboot_3617.zip]: 
Enter the xpenology extra drivers zip file name (Leave blank if none required. File should be placed in xpenology folder) [extra3617_v0.11.2_test.zip]: 
Enter the synology pat file name (File should be placed in xpenology folder) [DSM_DS3617xs_25426.pat]: 

PLAY [localhost] ******************************************************************************************************************
```

#### Additional prompts during run
There are additional prompts during the playbook run, please read carefully before continuing through the prompt.

##### Selecting USB Device Prompt
When the playbook scans for external devices it will collect all external devices it can find and list them in JSON output. Select the correct device_id that matches the USB device inserted.

In this example the device is a SanDisk USB Drive, but there are multiple devices listed including a controller, make sure to select the correct device (disk2 in this case)

Example:
```json
TASK [Select Device] **************************************************************************************************************
[Select Device]
Available USB Devices

[
    {
        "device_id": "disk2",
        "device_manufacturer": "SanDisk",
        "device_name": "Ultra Fit",
        "device_size": "61.51 GB",
        "product_id": "0x5583",
        "removable_media": "yes",
        "vendor_id": "0x0781  (SanDisk Corporation)",
        "volumes": [
            "disk2s1",
            "disk2s2",
            "disk2s3"
        ]
    },
    {
        "device_id": null,
        "device_manufacturer": "Broadcom Corp.",
        "device_name": "Bluetooth USB Host Controller",
        "device_size": null,
        "product_id": "0x8290",
        "removable_media": null,
        "vendor_id": "apple_vendor_id",
        "volumes": null
    }
]

Please enter the external physical device id from list above to use ("diskX" or "quit" to exit)
:
disk2
```

##### Manual Changes Prompt
There is a prompt that will give the user an opportunity to make manual changes. Open an additional terminal and add files or change configuration in the mounted USB partitions in `xpenology/usb_part(1|2)`. This is also an opportunity to verify the changes made by the playbook.

Since Xpenology can have many customizations, it would be a challenge to include them all, this allows users to make additional changes that may be unique to their hardware before the USB drive is unmounted and ejected.

```
TASK [Manual Changes] *************************************************************************************************************
[Manual Changes]
Since this playbook only covers basic Xpenology configuration you now have the opportunity to make more advanced changes
Now is the time to open another terminal and make any additional changes before ansible ejects the USB device
You can find the mounted partitions in xpenology/usb_partX folders - requires sudo access
  - add any additional config changes to grub.cfg
  - add any additional files not addressed in the playbook
  - any additional changes you would like

Press Enter to continue or ctrl+c and then c, to abort a playbook press ctrl+c and then a
:
```

You'll find the file structure of the xpenology folder is as follows (Note, partition 3 is not mounted):
```
.
└── xpenology
    ├── dsm
    ├── extra_drivers
    ├── synoboot
    ├── usb_part1
    ├── usb_part2
    └── usb_part3
```


## Playbook Completion
Once the playbook has completed its run the USB Drive will be umounted and ejected. Additionally all the extracted files and folders will be cleaned up.

Now you can insert the USB device into your Xpenology hardware and perform the install/upgrade.