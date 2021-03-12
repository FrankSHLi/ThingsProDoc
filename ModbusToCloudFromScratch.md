# 1. System Overview

![](./Image/overview.png)

# 2. Configure Device - Part 1

## 2.1 Make Sure Applications are Ready

```sh
watch appman app ls
```

Once all the applications are ready, connect our computer directly to LAN2 and change the computer's IP to 192.168.4.100, then we can login to the web GUI directly by https://192.168.4.127:8443. The default username/password is admin/admin@123.

## 2.2 Setup Network (default: dhcp on eth0)

![](./Image/network.png)

## 2.3 Sync Time

![](./Image/time.png)

## 2.4 Enable SSH

![](./Image/ssh.png)

> The default username/password for SSH is moxa/moxa and default ip for LAN2 is 192.168.4.127.

# 3. Setup IoT Edge

## 3.1 Prepare IoT Edge Deployment

- Create Deployment

    ![](./Image/deployment1.png)

- Name and Label

    ![](./Image/deployment2.png)

- Modules

    - Create ThingsPro Edge Module

        ![](./Image/deployment3.png)
        ![](./Image/deployment4.png)

        - Image URI:

            ```
            moxa2019/thingspro-agent:2.1.1-929-armhf
            ```

    - Fix IoT Edge default modules' version (required) and protocol (optional):

        ![](./Image/deployment5.png)
        ![](./Image/deployment6.png)

        - Version: (Required)

            ```
            mcr.microsoft.com/azureiotedge-agent:1.0.9.4
            mcr.microsoft.com/azureiotedge-hub:1.0.9.4
            ```

        - Name: (Optional)

            ```
            UpstreamProtocol
            ```

        - Value: (Optional)

            ```
            MqttWs
            ```

    ![](./Image/deployment7.png)
    - Container Create Options:

        ```
        {
            "HostConfig": {
                "Binds": [
                    "/var/thingspro/apps/cloud/data/setting/:/var/thingspro/cloud/setting/",
                    "/run/:/host/run/",
                    "/var/thingspro/data/:/var/thingspro/data/"
                ]
            }
        }
        ```

    ![](./Image/deployment8.png)

- Routes

    ![](./Image/deployment9.png)

    - NAME:
        ```
        route
        ```

    - VALUE:
        ```
        FROM /messages/* INTO $upstream
        ```

- Target Devices

    ![](./Image/deployment10.png)

    - Priority
        ```
        10
        ```

    - Target Condition:
        ```
        tags.project='demo'
        ```

- Review + create

    ![](./Image/deployment11.png)

## 3.2 Provision to IoT Hub

### 3.2.1 Provision Tool

- Modify Configuration File
    ```
    {
        "steps": [
            {
            "target": "Predefined",
            "description": "",
            "path": "",
            "method": "provision iot edge using dps",
            "post": {
                "scope": "{Service Endpoint of DPS}",
                "keyName": "{Shared Access Policy}",
                "key": "{Shared Access keys}",
                "scopeId": "{ID Scope of DPS}",
                "iotHubHostName": "{Target IoT Hub}",
                "initialTwin": {
                "properties": {},
                "tags": {
                    "{Key}": "{Value}"
                }
                },
                "generateDownstreamCertificate": true,
                "enableIoTEdge": true
            }
            }
        ]
    }
    ```

- Device Discovery

    ![](./Image/prov1.png)

- Select Target Devices

    ![](./Image/prov2.png)

- Provide Device Credential and Specify Provision Configuration

    ![](./Image/prov3.png)

- Provision

    ![](./Image/prov4.png)

- Check the Provision Result and Azure DPS

    ![](./Image/prov5.png)
    ![](./Image/prov6.png)

### 3.2.2 Check AIE Application from GUI

![](./Image/Image8.png)

> The sample provisioning utility creates the enrollment on DPS for each devices, generates the downstream certificate and enables IoT Edge service.

> We recommand users to create their own version of provisoning utility/service, since there should be more tasks to be finished during the provisioning process, such as changing default password.

# 4. Configure Device - Part 2

## 4.1 Modbus Setting

The modbus settings can be configured through ThingsPro Edge web GUI by clicking the **Modbus Master** tag from the side menu.

![](./Image/modbus1.png)

### 4.1.1 Add a Modbus TCP Device

![](./Image/modbus2.png)
![](./Image/modbus3.png)

- Basic Device Settings

    ![](./Image/modbus4.png)

- Add Commands (Tags) to the Device

    Here we are manually adding commands to the device, ThingsPro Edge also supports importing commands through a preconfigured csv file.

    ![](./Image/modbus5.png)
    ![](./Image/modbus6.png)
    ![](./Image/modbus7.png)
    ![](./Image/modbus8.png)
    ![](./Image/modbus9.png)
    ![](./Image/modbus10.png)

- Verify the Device and Command Settings

![](./Image/modbus11.png)

> The configurations won't take effect before we apply them. 

### 4.1.2 Apply Changes to Modbus Application

![](./Image/modbus12.png)
![](./Image/modbus13.png)

### 4.1.3 Check Current Tag Data

- Query the latest tag values

    ```sh
    curl "https://127.0.0.1:8443/api/v1/tags/monitor/modbus_tcp_master/ioLogik-E1242?tags=AI_0_resetMaxValue,DI_0_counterOverflowFlag,AI_0_mode,AI_0_scaledValue,status&streamInterval=1000" \
     -X GET -H "Content-Type:application/json" \
     -H "mx-api-token: $(cat /var/thingspro/data/mx-api-token)" -k
    ```

- Subscribe to tag changes

    ```sh
    docker exec -it tagservice_server_1 taghubd sub --all
    ```

## 4.2 Message Upload

### 4.2.1 Creating Message Group

After we've successfully configured the gateway to poll data from the ioLogik, now we send those data to Azure IoT Hub. To make it cost-effective, we are sending messages per minute. Having multiple message groups is supported by ThingsPro Edge.

- Create Message Group

    ![](./Image/D2CMessage1.png)
    ![](./Image/D2CMessage2.png)

- Setting Topic Name and Policies

    ![](./Image/D2CMessage3.png)

- Select Commands (Tags)

    ![](./Image/D2CMessage4.png)

- [Optional] Enable Custom Payload

    ![](./Image/D2CMessage5.png)

- Submit

    ![](./Image/D2CMessage6.png)

### 4.2.2 Monitor Uploaded Messages

Now the messages are being sent to IoT Hub. We can monitor the messages that are being sent by [**IoT Explorer**](https://github.com/Azure/azure-iot-explorer/releases).

- Provide IoT Hub Connection String

    ![](./Image/IoTExplorer1.png)

- Select the IoT Edge we are using

    ![](./Image/IoTExplorer2.png)

- Start Monitoring Telemetry Data

    ![](./Image/IoTExplorer3.png)

- Verify the Message Content

    ![](./Image/IoTExplorer4.png)
