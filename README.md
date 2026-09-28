# tapo-h200-root
An easy way to get root access on your TP-Link Tapo H200 hub.

> [!NOTE]  
> **Work in Progress**: This repository is brand new and in active development.

## Why Root?

* **S200B / S200D Dimmers:** Achieve instant response times with both Tapo and non-Tapo light bulbs.
* **The h5relay tool:** Gain access to a protected webpage hosted directly on the hub, which provides:
  * An embedded video player.
  * An API method list for the hub and its supported cameras.
  * Debugging and firmware upgrade tools.
  * The ability to seamlessly disconnect from (enabling local-only mode) or bind to the TP-Link cloud at any time.

  <details>
  <summary>Screenshots</summary>
  
  <img width="1517" height="266" alt="Camera discovery" src="https://github.com/user-attachments/assets/add04c16-564a-41b1-a6e8-9ffb1ae5dd4c" />
  <img width="510" height="214" alt="Upgrade page" src="https://github.com/user-attachments/assets/6d89c7a4-9ec5-4815-8cab-6b417cbe5ef8" />
    
  [![api-methods](https://github.com/user-attachments/assets/46ac41f1-adba-422b-959c-d8d2d12d09af)](https://github.com/user-attachments/assets/46ac41f1-adba-422b-959c-d8d2d12d09af)
  
  </details>

## Prerequisites

* **A Matter server/controller** with BDX Synchronous (Bulk Data Exchange) streaming and custom OTA file upload support.
  * **Recommended options:**
    * [chip-tool](https://project-chip.github.io/connectedhomeip-doc/development_controllers/chip-tool/chip_tool_guide.html#installation) (refer to the [OTA Provider manual](https://project-chip.github.io/connectedhomeip-doc/guides/updating-matter-device-guide.html))
    * [ioBroker Matter Adapter](https://github.com/ioBroker/ioBroker.matter)
  * **Not recommended:** Avoid using [matterjs-server](https://github.com/matter-js/matterjs-server) due to issues with BDX streaming (tested on v1.4.0).

**What we will do:**  
We are going to install **TP-Link Debug Firmware** for the hub: [H200-DEBUG-up-ver1-7-99-P1[20260515-rel61602]-signed_1779703201504.bin](http://download.tplinkcloud.com/firmware/assigned/H200-DEBUG-up-ver1-7-99-P1[20260515-rel61602]-signed_1779703201504.bin)

<details>
<summary><b>Differences between debug and release versions</b></summary><br>

The debug version is identical to the standard non-debug release, [H200-up-ver1-7-1-P1[20260515-rel61602]-signed_1779778332718.bin](http://download.tplinkcloud.com/firmware/H200-up-ver1-7-1-P1[20260515-rel61602]-signed_1779778332718.bin). The only differences are the inclusion of an always-on Telnet service, extra logging, and the enabled `h5relay` tool. See the included `.diff` file.
</details>
   
## Installation

### 1. Get ota_image_tool.py
```bash
git clone --depth 1 https://github.com/project-chip/connectedhomeip.git
```

### 2. Download the firmware
```bash
curl -g -O 'http://download.tplinkcloud.com/firmware/assigned/H200-DEBUG-up-ver1-7-99-P1[20260515-rel61602]-signed_1779703201504.bin'
```

### 3. Run the tool to wrap the TP-Link .bin file
```bash
python3 connectedhomeip/src/app/ota_image_tool.py create \
  --vendor-id 0x1392 \
  --product-id 0x0402 \
  --version 99999 \
  --version-str "1.7.99 Build 260515 rel.61602" \
  --digest-algorithm sha256 \
  'H200-DEBUG-up-ver1-7-99-P1[20260515-rel61602]-signed_1779703201504.bin' \
  tapo_h200.ota
```

### 4. Check the .ota header just in case
```bash
python3 connectedhomeip/src/app/ota_image_tool.py show tapo_h200.ota
```
Expected output:
```text
Magic: 1beef11e
Total Size: 6212686
Header Size: 92
Header TLV:
  [0] Vendor Id: 5010 (0x1392)
  [1] Product Id: 1026 (0x402)
  [2] Version: 99999 (0x1869f)
  [3] Version String: 1.7.99 Build 260515 rel.61602
  [4] Payload Size: 6212578 (0x5ecbe2)
  [8] Digest Type: 1 (0x1)
  [9] Digest: 2a351138726d1c8f45d5881d9d0028c85f5f98e620429f9201910e21857d5e71
```

### 5. Install the firmware
Refer to the manual for your Matter server/controller and install `tapo_h200.ota` as an update.

> [!NOTE]
> The version we specified in the `.ota` header does not match the installed firmware version. Because of this, your Matter server will report a "failed installation" regardless of whether the installation was successful or not.

### 6. Obtain root shell
If successful, you can connect to the hub via telnet:
```
telnet <hub_ip>
```
>[!NOTE]
>Don't forget to use the `df` command to check that there is free space in the “/etc/rwdir” directory, and don't fill it up more than 80%. The hub cannot clear it on its own. Also, the console will prompt you to set a password using the `passwd` command to enable SSH, but ignore this as the firmware does not include an SSH server.

### To be continued
