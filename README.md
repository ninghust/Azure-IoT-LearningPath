# Deploy Module to IoT Device

## Key Concepts
- [Azure IoT Hub](https://learn.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub "Azure IoT Hub")
- [Azure IoT Edge](https://learn.microsoft.com/en-us/azure/iot-edge/about-iot-edge?view=iotedge-1.4 "Azure IoT Edge")

## Prerequisites
- An Azure subscription, create a resource group and IoT hub
- Git Bash, WSL
- Azure CLI and install IoT extension

## Create IoT Edge Device
In this section, we need two steps:
1. [Create Virtual Machine in the Azure Portal](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal "Create Virtual Machine in the Azure Portal")
2. [Register and Provision IoT Edge Device](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu "Register and Provision IoT Edge Device")

### Create VM
- **Authentication type:** SSH public key
- **Username:** azureuser
- **SSH public key source:** Generate new key pair
- **Key pair name:** XXX (define by self)
- **Public inbound ports:** Allow selected ports
- **Select inbound ports:** HTTP(80),SSH(22)  *(only recommended for testing)*

When the Generate new key pair window opens, select **Download private key and create resource**. Remember where the .pem file was downloaded and will need the path to it in the next step.

```bash
chmod 400 ~/Downloads/myKey.pem   //set read-only permission on the .pem file
ssh -i ~/Downloads/myKey.pem azureuser@10.111.12.123    //ssh to created VM
```

>❤️Tip: The SSH key you created can be used the next time your create a VM in Azure. Just select the **Use a key stored in Azure** for **SSH public key source** the next time you create a VM.

### Register and Provision IoT Edge Device

1. [Register your device](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#register-your-device "Register your device")
2. [View registered devices and retrieve provisioning information](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#view-registered-devices-and-retrieve-provisioning-information "View registered devices and retrieve provisioning information")
3. [Install IoT Edge](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#install-iot-edge "Install IoT Edge")
   1. [Install a container engine](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#install-a-container-engine "Install a container engine")
   2. [Install the IoT Edge runtime](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#install-the-iot-edge-runtime "Install the IoT Edge runtime")


4. [Provision the device with its cloud identity](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#provision-the-device-with-its-cloud-identity "Provision the device with its cloud identity")
5. [Verify successful configuration](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-provision-single-device-linux-symmetric?view=iotedge-1.4&tabs=azure-portal%2Cubuntu#verify-successful-configuration "Verify successful configuration")
```bash
sudo iotedge system status  //Check to see that the IoT Edge system service is running.
sudo iotedge system logs  //Retrieve the service logs if toubleshooting
sudo iotedge check //Verify configuration and connection status of the device
sudo iotedge list //View all the modules running on your IoT Edge device
```
>❤️Tip: use `sudo` in front of the commands
