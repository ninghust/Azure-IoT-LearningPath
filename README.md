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

## Configure a Deployment manifest
1. [Deployment manifest example](https://github.com/ninghust/Azure-IoT-LearningPath/blob/main/edge/deployment.template.json "Deployment manifest example")
>file name: XXX.template.json

2. Generate a deployment manifest (. json)
Formatting using VS code **Azure IoT tool extension** to generate or (build deployment.template.json), the formatted manifest will be under ./config

## Deploy Module to Single Device via Azure CLI
1. Make sure in correct subscription
```bash
   az account show --output table  //Show current subscription
   az account set --subscription "My Demos" //change the active subscription using the subscription name
   az account set --subscription "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" //change the active subscription using the subscription ID
```
2. Deploy module
```bash
az iot edge deployment create -d firstTryDeployment -n smoketestiothub --content deployment.amd64.json --target-condition "deviceId='ningtest'"
// firstTryDeployment: deployment ID   smoketestiothub: IoT hub name deployment.amd64.json: deployment manifest
```
3. Verify the deployment created
![image](https://user-images.githubusercontent.com/113426179/195991592-0d4f5d35-4896-41a7-b690-26212330f326.png)

4. Check whether the module is deployed: Verify that the module is deployed in your IoT Hub in the Azure portal. Select your device, select **Set Modules** and the module should be listed in the **IoT Edge Modules** section.

> 💡[Troubleshoot your IoT Edge device](https://learn.microsoft.com/en-us/azure/iot-edge/troubleshoot?view=iotedge-1.4 "Troubleshoot your IoT Edge device")


## Build a Pipeline to deploy modules
 - [Create a Service connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml#create-a-service-connection "Create a Service connection")
 - Create Azure Pipeline YAML File
   - Sample code: [deploy-iot-modules-to-edge-server.yml](https://github.com/ninghust/Azure-IoT-LearningPath/blob/main/pipeline/deploy-iot-modules-to-edge-server.yml "deploy-iot-modules-to-edge-server.yml")

- Create Azure Pipeline with Existing YAML File and Run
- 💡Troubleshooting:
  1. Service connection is must-have. 'azureSubscription' is refer to service connection name **NOT** subscription name
     ```yaml
     - task: AzureCLI@2
      inputs:
        azureSubscription: $(AzureCNSCName)
        scriptType: 'pscore'
        scriptLocation: 'inlineScript'
        failOnStandardError: false
        inlineScript: |
          az extension add --name azure-iot;
          az iot edge deployment create -n '$(IOTHUB_NAME)' --content $(System.ArtifactsDirectory)/DeploymentManifest/config/deployment.json --target-condition "deviceId='$(EDGE_DEVICE_ID)'" --priority '1' -d '$(mydate)$(Build.Buildid)'```
  2. When running pipeline, the following error happens:
     ```bash
     Start generating deployment manifest...
     /home/vsts/work/1/s/devops/edge /home/vsts/work/1/s
     ##[error]/usr/lib/python3/dist-packages/requests/__init__.py:89: RequestsDependencyWarning: urllib3 (1.26.12) or chardet (3.0.4) doesn't match a supported version!
     ``` 
      Upgrade Python: Use Python 3.8 to resolve it
      ```yaml
      - task: UsePythonVersion@0
        displayName: 'Use Python 3.8'
        inputs:
          versionSpec: '3.8'
      ```
  

>❤️Useful Links: 
>- [Continuous integration and continuous deployment to Azure IoT Edge devices](https://learn.microsoft.com/en-us/azure/iot-edge/how-to-continuous-integration-continuous-deployment?view=iotedge-1.4 "Continuous integration and continuous deployment to Azure IoT Edge devices")
>- [Specify jobs in your pipeline](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/phases?view=azure-devops&tabs=yaml "Specify jobs in your pipeline")
>- [Add & use variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml "Add & use variable groups")


    
