## Requirements

* Raspberry Pi 4/5 installed with Home Assistant OS -> [Raspberry Pi - Home Assistant (home-assistant.io)](https://www.home-assistant.io/installation/raspberrypi/)
* ESP32C6 as a Matter Device that supports OpenThread
* ESP32/ESP32C3/ESP32S3 as a Matter Device tht supports BLE
* Border Router consisting of ESP32/ESP32S3 as the Host Controller and ESP32H2 as the RCP (Radio Co-Processor) that supports OpenThread
* Home Assistant Application on Android/IOS/Web Browser
* Ubuntu/WSL Computer Host
* ESP-IDF minimum v5.3.

## Set-up Matter Devices and Border Router

### Border Router

There is a Border Router development board by ESPRESSIF available on the market. On this occassion, I choose to construct my own Border Router using ESP32 as the main controller capable of Wi-Fi and BLE communications, and ESP32H2 as the Radio Co-Processor (RCP) capable of OpenThread communication.

* Follow this guide for flashing the RCP firmware to the ESP32H2 -> [esp-idf/examples/openthread/ot_rcp at master · espressif/esp-idf (github.com)](https://github.com/espressif/esp-idf/tree/master/examples/openthread/ot_rcp)
* For the ESP32 host controller, follow this guide -> [esp-thread-br/examples/basic_thread_border_router at main · espressif/esp-thread-br (github.com)](https://github.com/espressif/esp-thread-br/tree/main/examples/basic_thread_border_router)

### Generating Manufacture Partition and esp_secure_certificate

A Matter device can be flashed with manufacturer information along with secure certificate using **esp-mfg-tool**. For brief steps, you can refer to this guide [esp-matter-tools/mfg_tool at main · espressif/esp-matter-tools (github.com)](https://github.com/espressif/esp-matter-tools/tree/main/mfg_tool). Upon successful generation, your project will have **out/** folder. You will flash those files after flashing the application firmware.

![1725008397976](image/README/1725008397976.png)

### Matter Device with OpenThread Capability

Several examples are provided by the ESP-Matter, you can choose one to build and flash the firmware to the ESP32C6. For this experiment, I use [esp-matter/examples/light_switch at main · espressif/esp-matter (github.com)](https://github.com/espressif/esp-matter/tree/main/examples/light_switch). With several modification to enable the OpenThread and generating the manufacture partition and esp_secure_certificate. For setting the firmware to use the generated partition, follow this guide:

Enter the configuration by entering this command `idf.py menuconfig`.

1. Enable Device Instance Info Provider, Device Info Provider, and Use Secure Cert DAC Provider.

   `Component Config > CHIP Device Layer > Commissioning Options`

   ![1725005737654](image/README/1725005737654.png)
2. Define Partition Address for the manufacture partition. The default is nvs `0x10000` you can optionally change to `0x3E0000` (fctry)

   ![1725005925888](image/README/1725005925888.png)
3. Config ESP Matter

   `Component Config > ESP Matter` then:

   - Change DAC Provider Options to **Secure Cert**
   - Change Commissionable Data Provider Options to **Factory**
   - Change the Device Instance Infor Provider Options to **Factory**

   ![1725006948684](image/README/1725006948684.png)
4. Save and exit

Build and flash the application firmware with the folllowing command:

`idf.py build flash`

Flash the esp_secure_cert.bin to `0xd000`. with the following command `esptool.py -p /dev/ttyUSB0  write_flash 0xd000 out/fff2_8001/<generated code>/<generated code>_esp_secure_cert.bin`

![1725008492179](image/README/1725008492179.png)

Flash the factory partition to the nvs address `0x10000` with the following command `esptool.py -p /dev/ttyUSB0  write_flash 0x10000 out/fff2_8001/<generated code>`/`<generated code>-partition.bin`

![1725008694574](image/README/1725008694574.png)

Monitor the device using the following command `idf.py monitor`

### Matter Device with BLE Capability

For this experiment, I use [esp-matter/examples/light at main · espressif/esp-matter (github.com)](https://github.com/espressif/esp-matter/tree/main/examples/light) example code. This example is intended for LED devices. We also need to modify several configuration for generating the manufacture partition and esp_secure_certificate. For setting the firmware to use the generated partition, follow this guide:

Enter the configuration by entering this command `idf.py menuconfig`.

1. Enable Device Instance Info Provider, Device Info Provider, and Use Secure Cert DAC Provider.

   `Component Config > CHIP Device Layer > Commissioning Options`

   ![1725005737654](image/README/1725005737654.png)
2. Define Partition Address for the manufacture partition. The default is nvs `0x10000` you can optionally change to `0x3E0000` (fctry)

   ![1725005925888](image/README/1725005925888.png)
3. Config ESP Matter

   `Component Config > ESP Matter` then:

   - Change DAC Provider Options to **Secure Cert**
   - Change Commissionable Data Provider Options to **Factory**
   - Change the Device Instance Infor Provider Options to **Factory**

   ![1725006948684](image/README/1725006948684.png)
4. Save and exit

As the ESP32C3 Dev board that manufactured by AI-Thinker is not addressable LED, then we need to define the LED Pin. Navigate to `Component Config > Board Support Package (generic) > LED` and configure the pin as follows:

![1725007459853](image/README/1725007459853.png)

Build and flash the application firmware with the folllowing command:

`idf.py build flash`

Flash the esp_secure_cert.bin to `0xd000`. with the following command `esptool.py -p /dev/ttyUSB0  write_flash 0xd000 out/fff2_8001/<generated code>/<generated code>_esp_secure_cert.bin`

![1725008492179](image/README/1725008492179.png)

Flash the factory partition to the nvs address `0x10000` with the following command `esptool.py -p /dev/ttyUSB0  write_flash 0x10000 out/fff2_8001/<generated code>`/`<generated code>-partition.bin`

![1725008694574](image/README/1725008694574.png)

Monitor the device using the following command `idf.py monitor`

## Configuring the Border Router

Open your web browser and enter the following URL `http://homeassistant:8123/` to configure the Home Assistant. You may need to create an account for loggin in on the HA. Upon successful login you can configure the HA, navigate to `Settings > Devices & Services`

![1725009057910](image/README/1725009057910.png)

You may need to install several add-ons, including:

* Matter (BETA)
* OpenThread Border Router
* Thread

![1725009082129](image/README/1725009082129.png)

Upon installing all the add-ons, we continue to configure the ESP Border Router to the Thread Network.

![1725009277258](image/README/1725009277258.png)

Click configure and add the OpenThread network dataset. Then, click the three dots to add the recognized ESP-BORDER-ROUTER as the preferred network.

![1725009372837](image/README/1725009372837.png)

## Commission Matter Devices

The BR is ready to be used and we can add Matter Devices using Home Assistant application. Open your mobile application and navigate to `Settings > Devices & Services > Devices` and click **Add Device** and choose **Add Matter Device.** Do these steps for each Matter device.

| ![1725009794882](image/README/1725009794882.png) | ![1725009800603](image/README/1725009800603.png) | ![1725009823614](image/README/1725009823614.png) |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- |

Commission the device by scanning the QR code generated by **mfg-tool**. Follow all the steps until the device successfully commissioned.

| ![1725009843394](image/README/1725009843394.png) | ![1725009878393](image/README/1725009878393.png) | ![1725009919575](image/README/1725009919575.png) | ![1725009926737](https://file+.vscode-resource.vscode-cdn.net/d%3A/Self-Learning/Matter/Matter/image/README/1725009926737.png)  |
| ---------------------------------------------- | ---------------------------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |

At this point your dashboard will look like this and you can control the devices.

| ![1725009966835](https://file+.vscode-resource.vscode-cdn.net/d%3A/Self-Learning/Matter/Matter/image/README/1725009966835.png) | ![1725010006634](https://file+.vscode-resource.vscode-cdn.net/d%3A/Self-Learning/Matter/Matter/image/README/1725010006634.png) | ![1725010152213](image/README/1725010152213.png) |
| ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |

Optionally, you can also add automation to trigger the LED when the switch is pressed. You can do this by navigating to **Automation Page** and configure as follows:

![1725010199093](image/README/1725010199093.png)

You can also read each device information by navigating to `Settings > Devices & Services > Click Matter Device`. You will see a list of devices:

![1725010622613](image/README/1725010622613.png)

You can also see their information in more details.

![1725010642954](image/README/1725010642954.png)

![1725010648335](image/README/1725010648335.png)

# Bonus

Here's the real condition of my experiment

![1725010443828](image/README/1725010443828.png)
