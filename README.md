# Deploy Module to IoT Device

## Key Concepts
- [Azure IoT Hub](https://learn.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub "Azure IoT Hub")
- [Azure IoT Edge](https://learn.microsoft.com/en-us/azure/iot-edge/about-iot-edge?view=iotedge-1.4 "Azure IoT Edge")

## Prerequisites
- An Azure subscription, create a resource group and IoT hub
- Git Bash, WSL

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
