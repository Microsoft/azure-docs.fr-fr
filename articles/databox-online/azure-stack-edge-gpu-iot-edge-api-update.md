---
title: Impact de la mise à jour de janvier 2021 d’Azure Stack Edge | Microsoft Docs
description: Cet exemple décrit l’impact de la gestion des rôles IoT Edge sur les appareils Azure Stack Edge après l’installation de la mise à jour de janvier 2021.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: overview
ms.date: 10/26/2020
ms.author: alkohli
ms.openlocfilehash: f16f33e9aadcc01427602a1bd81f81cb0710e4dd
ms.sourcegitcommit: 1d6ec4b6f60b7d9759269ce55b00c5ac5fb57d32
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94578737"
---
# <a name="iot-edge-role-management-changes-for-your-azure-stack-edge-device"></a>Modifications de la gestion des rôles IoT Edge pour votre appareil Azure Stack Edge

Pour gérer les rôles Azure IoT Edge sur votre appareil Azure Stack Edge, utilisez la version mise à jour de l’API, du SDK et d’Azure PowerShell, qui est prévue pour janvier 2021. 

Cet article décrit en détail les modifications que vous devez apporter lorsque vous utilisez la version la plus récente.

La mise à jour de janvier 2021 sera uniquement disponible pour les appareils Azure Stack Edge Pro - GPU, Azure Stack Edge Pro R et Azure Stack Edge Mini R. Les informations contenues dans cet article concernent uniquement ces appareils.

> [!NOTE]
> Vous n’êtes pas obligé d’effectuer la mise à niveau vers la version de janvier 2021. Si vous continuez à utiliser votre version actuelle, cela n’aura aucun impact sur la gestion des rôles IoT Edge. Toutefois, pour bénéficier des nouvelles fonctionnalités et contribuer à réduire les risques de sécurité, nous vous recommandons d’installer la version plus récente. 

## <a name="iot-edge-role-management-changes"></a>Modifications de la gestion des rôles IoT Edge

Après avoir installé la mise à jour facultative de janvier 2021 sur votre appareil Azure Stack Edge, vous devez utiliser la dernière version de l’API, du SDK et des applets de commande PowerShell pour gérer les rôles IoT Edge.

Les modifications suivantes sont nécessaires uniquement si vous appliquez la mise à jour de janvier 2021 :

- Si vous utilisez actuellement la version&nbsp;2019-08-01 de l’API de gestion des rôles, mettez-la à niveau vers la version de l’API qui sera publiée en janvier 2021. 
- Si vous utilisez la gestion des rôles via le SDK version&nbsp;1.0.0, effectuez une mise à niveau vers la version qui sera publiée en janvier 2021.
- Si vous utilisez la gestion des rôles avec des applets de commande Azure PowerShell (préversion) telles que `Get-AzStackEdgeRole`, `New-AzStackEdgeRole`, `Set-AzStackEdgeRole`ou `Remove-AzStackEdgeRole`, attendez que les nouvelles applets de commande soient publiées en février 2021.

## <a name="api-usage"></a>Utilisation de l’API

Si vous gérez les rôles IoT Edge par le biais de l’API, vous devez utiliser la nouvelle version de l’API (2020-12-01) qui sera publiée ultérieurement. Si vous utilisez l’API Rôle actuelle, une fois que vous aurez installé la version du logiciel de l’appareil à venir, vous devrez passer au rôle Kubernetes PUT, GET ou DELETE, puis à l’API du module complémentaire IoT PUT.

### <a name="for-the-put-method"></a>Pour la méthode PUT

#### <a name="the-current-http-request"></a>La requête HTTP actuelle 

- Les appels d’API sont effectués à cet URI : « https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/IoTRole1?api-version=2019-08-01  »

- Le corps de la requête se présente comme suit :

    ```json
    {
        "kind": "IOT",
        "properties": {
            "hostPlatform": "Linux",
            "ioTDeviceDetails": {
                "deviceId": "iotdevice",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {
                        "connectionString": {
                            "value": "Encrypted<<HostName=iothub.azure-devices.net;DeviceId=iotDevice;SharedAccessKey=2C750FscEas3JmQ8Bnui5yQWZPyml0/UiRt1bQwd8=>>",
                            "encryptionCertThumbprint": "348586569999244",
                            "encryptionAlgorithm": "AES256"
                        }
                    }
                }
            },
            "ioTEdgeDeviceDetails": {
                "deviceId": "iotEdge",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {
                        "connectionString": {
                            "value": "Encrypted<<HostName=iothub.azure-devices.net;DeviceId=iotEdge;SharedAccessKey=2C750FscEas3JmQ8Bnui5yQWZPyml0/UiRt1bQwd8=>>",
                            "encryptionCertThumbprint": "1245475856069999244",
                            "encryptionAlgorithm": "AES256"
                        }
                    }
                }
            },
            "shareMappings": [],
            "roleStatus": "Enabled"
        }
    }
    ```

#### <a name="the-upcoming-http-request"></a>La requête HTTP à venir 

- Les appels d’API pour le rôle Kubernetes sont effectués à l’URI suivant : 

  'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/KubernetesRole1?api-version=2020-12-01'

    Le corps de la requête doit ressembler à ceci :

    ```json
    {
        "kind": "Kubernetes",
        "properties": {
            "hostPlatform": "Linux",
            "kubernetesClusterInfo": {
                "version": "v1.17.3"
            },
            "kubernetesRoleResources": {
                "storage": {
                    "endpoints": []
                },
                "compute": {
                    "vmProfile": "DS1_v2"
                }
            }
        }
    }
    ```

- Les appels d’API pour le module complémentaire IoT Edge sont effectués à l’URI suivant : 

   'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/KubernetesRole1/addons/iotaddon?api-version=2020-12-01'

    Le corps de la requête doit ressembler à ceci :

    ```json
    {
        "kind": "IoT",
        "properties": {
            "ioTDeviceDetails": {
                "deviceId": "iotdevice",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {
                        "connectionString": {
                            "value": "Encrypted<<HostName=iothub.azure-devices.net;DeviceId=iotDevice;SharedAccessKey=2C750FscEas3JmQ8Bnui5yQWZPyml0/UiRt1bQwd8=>>",
                            "encryptionCertThumbprint": "348586569999244",
                            "encryptionAlgorithm": "AES256"
                        }
                    }
                }
            },
            "ioTEdgeDeviceDetails": {
                "deviceId": "iotEdge",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {
                        "connectionString": {
                            "value": "Encrypted<<HostName=iothub.azure-devices.net;DeviceId=iotEdge;SharedAccessKey=2C750FscEas3JmQ8Bnui5yQWZPyml0/UiRt1bQwd8=>>",
                            "encryptionCertThumbprint": "1245475856069999244",
                            "encryptionAlgorithm": "AES256"
                        }
                    }
                }
            }
        }
    }
    ```


### <a name="for-the-get-method"></a>Pour la méthode GET

#### <a name="the-current-http-response"></a>La réponse HTTP actuelle

- Les appels d’API sont effectués à l’URI suivant : 

   'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/IoTRole1?api-version=2019-08-01'

- Le corps de la réponse se présente comme suit :

    ```json
        "kind": "IOT",
        "properties": {
            "hostPlatform": "Linux",
            "ioTDeviceDetails": {
                "deviceId": "iotdevice",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {}
                }
            },
            "ioTEdgeDeviceDetails": {
                "deviceId": "iotEdge",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {}
                }
            },
            "shareMappings": [],
            "roleStatus": "Enabled"
        },
        "id": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/IoTRole1",
        "name": "IoTRole1",
        "type": "dataBoxEdgeDevices/roles"
    }    
    ```

#### <a name="the-upcoming-http-response"></a>La réponse HTTP à venir

- Les appels d’API sont effectués à l’URI suivant :  

   'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/KubernetesRole1/addons/iotaddon?api-version=2020-12-01'

- Le corps de la réponse se présente comme suit : 

    ```json
    {
        "kind": "IoT",
        "properties": {
            "provisioningState": "Creating",
            "ioTDeviceDetails": {
                "deviceId": "iotdevice",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {}
                }
            },
            "ioTEdgeDeviceDetails": {
                "deviceId": "iotEdge",
                "ioTHostHub": "iothub.azure-devices.net",
                "ioTHostHubId": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/Microsoft.Devices/IotHubs/testrxiothub",
                "authentication": {
                    "symmetricKey": {}
                }
            },
            "version": "0.1.0-beta10"
        },
        "id": "/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/res1/roles/kubernetesRole/addons/iotName",
        "name": " iotName",
        "type": "Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/addon",
    }
    ```

### <a name="for-the-delete-method"></a>Pour la méthode DELETE

### <a name="the-current-api-calls"></a>Les appels d’API actuels

Les appels d’API sont effectués à l’URI suivant :  

'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/IoTRole1?api-version=2019-08-01'

### <a name="the-upcoming-api-calls"></a>Les appels d’API à venir

Les appels d’API sont effectués à l’URI suivant :  

'https://management.azure.com/subscriptions/4385cf00-2d3a-425a-832f-f4285b1c9dce/resourceGroups/GroupForEdgeAutomation/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/testedgedevice/roles/KubernetesRole1/addons/iotaddon?api-version=2020-12-01'

## <a name="sdk-usage"></a>Utilisation du Kit de développement logiciel (SDK)

Si vous utilisez le SDK, une fois que vous aurez installé la mise à jour de janvier 2021, vous devrez modifier la façon dont vous configurez le rôle IoT Edge, comme illustré dans l’exemple suivant. Vous devrez ensuite télécharger et installer le package NuGet à venir pour passer au nouveau SDK, comme illustré dans l’exemple suivant.

### <a name="the-current-sdk-sample"></a>L’exemple de SDK actuel

```csharp
var iotRoleStatus = "Enabled";
var iotHostPlatform = "Linux";
var id = $@"/subscriptions/546ec571-2d7f-426f-9cd8-0d695fa7edba/resourceGroups/resourceGroup/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/deviceName/roles/iotrole"; 
var name = "iotrole";
var type = "Microsoft.DataBoxEdge/dataBoxEdgeDevices/role";
var iotRoleName = "iotrole";
var ioTDeviceDetails = new IoTDeviceInfo(...);
var ioTEdgeDeviceDetails = new IoTDeviceInfo(...);
var ioTEdgeAgentInfo = new IoTEdgeAgentInfo(...);
var shareMappings = new List<MountPointMap>(...);

var role = new IoTRole(roleStatus, 
    hostPlatform, 
    shareMappings, 
    ioTDeviceDetails, 
    ioTEdgeDeviceDetails, 
    ioTEdgeAgentInfo, 
    id, 
    name, 
    type);

DataBoxEdgeManagementClient.Roles.CreateOrUpdate(deviceName, iotRoleName, role, resourceGroup);
```


### <a name="the-new-sdk-sample"></a>Le nouvel exemple de SDK

```csharp
var k8sRoleStatus = "Enabled";
var k8sHostPlatform = "Linux";
var k8sId = $@"/subscriptions/546ec571-2d7f-426f-9cd8-0d695fa7edba/resourceGroups/resourceGroup/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/deviceName/roles/KubernetesRole"; 
var k8sRoleName = "KubernetesRole";
var k8sClusterVersion = "v1.17.3"; //Final values will be updated here around January 2021
var k8sVmProfile = "DS1_v2"; //Final values will be updated here around January 2021
var type = "Microsoft.DataBoxEdge/dataBoxEdgeDevices/role";
var k8sRole = new KubernetesRole(
    roleStatus,
    hostPlatform,
    shareMappings,
    k8sClusterVersion,
    k8sVmProfile,
    k8sId,
    k8sRoleName,
    type
);
DataBoxEdgeManagementClient.Roles.CreateOrUpdate(deviceName, k8sRoleName, k8sRole, resourceGroup); //Final usage will be updated here around January 2021

var ioTId = $@"/subscriptions/546ec571-2d7f-426f-9cd8-0d695fa7edba/resourceGroups/resourceGroup/providers/Microsoft.DataBoxEdge/dataBoxEdgeDevices/deviceName/roles/KubernetesRole/addons/iotaddon";
var ioTAddonName = "iotaddon";
var ioTAddonType = "Microsoft.DataBoxEdge/dataBoxEdgeDevices/roles/addons";
var addon = new IoTAddon(
    ioTDeviceDetails, 
    ioTEdgeDeviceDetails, 
    ioTEdgeAgentInfo, 
    ioTId, 
    ioTAddonName, 
    ioTAddonType);
DataBoxEdgeManagementClient.AddOns.CreateOrUpdate(deviceName, k8sRoleName, addonName, addon, resourceGroup); //Final usage will be updated here around January 2021
```

## <a name="cmdlet-usage"></a>Utilisation des applets de commande

Si vous utilisez actuellement l’applet de commande `Get-AzStackEdgeRole`, `New-AzStackEdgeRole`, `Set-AzStackEdgeRole` ou `Remove-AzStackEdgeRole`, vous devrez attendre la nouvelle version prévue pour février 2021.

## <a name="frequently-asked-questions"></a>Forum aux questions

**J’utilise Azure Stack Edge Pro – FPGA. La mise à jour de janvier 2021 a-t-elle un impact sur le modèle FPGA ?**

Non. La mise à jour de janvier 2021 s’applique uniquement aux appareils Azure Stack Edge - FPGA, Azure Stack Edge Pro R et Azure Stack Edge Mini R. Azure Stack Edge Pro - FPGA n’est pas affecté par cette mise à jour, et aucune modification de la gestion des rôles IoT Edge n’est nécessaire.

**Une fois que j’aurai mis à jour Azure Stack Edge Pro - GPU vers le nouveau logiciel d’appareil en janvier 2021, les services existants seront-ils affectés ?**

Non. Vos services configurés ne seront pas affectés par l’installation de la mise à jour d’appareil de janvier 2021.

**Quelles sont les principales modifications apportées à l’API, au SDK ou à l’applet de commande de gestion IoT Edge ?**

IoT Edge est un module complémentaire disponible avec le rôle Kubernetes. Vous devez donc d’abord vérifier que Kubernetes est configuré, puis effectuer la configuration d’IoT Edge.


## <a name="next-steps"></a>Étapes suivantes

- [Découvrez comment appliquer des mises à jour](azure-stack-edge-gpu-install-update.md)

