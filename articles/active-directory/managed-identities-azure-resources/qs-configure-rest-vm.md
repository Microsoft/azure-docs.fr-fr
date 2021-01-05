---
title: Configurer des identités managées sur une machine virtuelle Azure à l’aide de REST - Azure AD
description: Instructions pas à pas pour configurer des identités managées affectées par le système et l’utilisateur sur une machine virtuelle Azure en utilisant CURL pour effectuer les appels d’API REST.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: f70229fabe4331adb7740cf09cf2bf8f3e3e4617
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587517"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-rest-api-calls"></a>Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant des appels d’API REST

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent aux services Azure une identité système managée automatiquement dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code. 

Dans cet article, en utilisant CURL pour effectuer des appels au point de terminaison REST d’Azure Resource Manager, vous allez apprendre à configurer les identités managées suivantes pour les opérations de ressources Azure sur une machine virtuelle Azure :

- Activer et désactiver l’identité managée affectée par le système sur une machine virtuelle Azure
- Ajouter et supprimer une identité managée affectée par l’utilisateur sur une machine virtuelle Azure

Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour les ressources Azure, consultez [Que sont les identités managées pour les ressources Azure ?](overview.md) Pour en savoir plus sur les types d’identités managées affectées par le système et par l’utilisateur, consultez [Types d’identités managées](overview.md#managed-identity-types).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, découvrez comment activer et désactiver une identité managée affectée par le système sur une machine virtuelle Azure, en utilisant CURL pour effectuer des appels au point de terminaison REST Azure Resource Manager.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Activer une identité managée affectée par le système lors de la création d’une machine virtuelle Azure

Pour créer une machine virtuelle Azure avec l’identité managée affectée par le système sur une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur de machine virtuelle](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Créez un [groupe de ressources](../../azure-resource-manager/management/overview.md#terminology) pour l’imbrication et le déploiement de votre machine virtuelle et de ses ressources connexes, à l’aide de la commande [az group create](/cli/azure/group/#az-group-create). Vous pouvez ignorer cette étape si vous possédez déjà le groupe de ressources que vous souhaitez utiliser à la place :

   ```azurecli-interactive 
   az group create --name myResourceGroup --location westus
   ```

2. Créez une [interface réseau](/cli/azure/network/nic#az-network-nic-create) pour votre machine virtuelle :

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. À l’aide d’Azure Cloud Shell, créez une machine virtuelle avec CURL pour appeler le point de terminaison REST Azure Resource Manager. L’exemple suivant crée une machine virtuelle nommée *myVM* avec une identité managée affectée par le système, telle qu’identifiée dans le corps de la demande par la valeur `"identity":{"type":"SystemAssigned"}`. Remplacez `<ACCESS TOKEN>` par la valeur que vous avez reçue à l’étape précédente lorsque vous avez demandé un jeton d’accès du porteur et la valeur `<SUBSCRIPTION ID>` adaptée à votre environnement.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"SystemAssigned"},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"<SECURE PASSWORD STRING>"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **En-têtes de requête**
   
   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 
   
   **Corps de la demande**

   ```JSON
     {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"SystemAssigned"
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{
             "imageReference":{
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{
                "caching":"ReadWrite",
                "managedDisk":{
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }  
   ```

### <a name="enable-system-assigned-identity-on-an-existing-azure-vm"></a>Activer une identité affectée par le système sur une machine virtuelle Azure existante

Pour activer l’identité managée affectée par le système sur une machine virtuelle qui en était dépourvue initialement, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Utilisez la commande CURL suivante pour appeler le point de terminaison REST Azure Resource Manager afin d’activer l’identité managée affectée par le système sur votre machine virtuelle, telle qu’identifiée dans le corps de la demande par la valeur `{"identity":{"type":"SystemAssigned"}` pour une machine virtuelle nommée *myVM*.  Remplacez `<ACCESS TOKEN>` par la valeur que vous avez reçue à l’étape précédente lorsque vous avez demandé un jeton d’accès du porteur et la valeur `<SUBSCRIPTION ID>` adaptée à votre environnement.
   
   > [!IMPORTANT]
   > Pour éviter de supprimer des identités managées affectées par l’utilisateur existantes qui sont attribuées à la machine virtuelle, répertoriez les identités managées affectées par l’utilisateur en utilisant la commande CURL suivante : `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Si vous avez attribué des identités managées affectées par l’utilisateur à la machine virtuelle, telles qu’identifiées par la valeur `identity` dans la réponse, passez à l’étape 3 qui montre comment conserver les identités managées affectées par l’utilisateur, tout en activant l’identité managée affectée par le système sur votre machine virtuelle.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 
   
   **Corps de la demande**
    
   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned"
       }
    }
   ```

3. Pour activer l’identité managée affectée par le système sur une machine virtuelle avec des identités managées affectées par l’utilisateur existantes, vous devez ajouter `SystemAssigned` à la valeur `type`.  
   
   Par exemple, si votre machine virtuelle a des identités managées affectées par l’utilisateur `ID1` et `ID2` qui lui sont attribuées, et que vous souhaitez ajouter l’identité managée affectée par le système à la machine virtuelle, utilisez l’appel CURL suivant. Remplacez `<ACCESS TOKEN>` et `<SUBSCRIPTION ID>` par les valeurs adaptées à votre environnement.

   L’API version `2018-06-01` stocke les identités managées affectées par l’utilisateur dans la valeur `userAssignedIdentities` dans un format de dictionnaire, par opposition à la valeur `identityIds` au format de tableau utilisée dans l’API version `2017-12-01`.
   
   **API VERSION 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{  
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{  
    
             }
          }
       }
    }
   ```

   **API VERSION 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
    
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {  
       "identity":{  
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[  
             "/subscriptions/<<SUBSCRIPTION ID>>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```   

### <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Désactiver une identité managée affectée par le système d’une machine virtuelle Azure

Pour désactiver l’identité managée affectée par le système sur une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Mettez à jour la machine virtuelle à l’aide de CURL pour appeler le point de terminaison REST Azure Resource Manager afin de désactiver l’identité managée affectée par le système.  L’exemple suivant désactive une identité managée affectée par le système, telle qu’identifiée dans le corps de la demande par la valeur `{"identity":{"type":"None"}}` d’une machine virtuelle nommée *myVM*.  Remplacez `<ACCESS TOKEN>` par la valeur que vous avez reçue à l’étape précédente lorsque vous avez demandé un jeton d’accès du porteur et la valeur `<SUBSCRIPTION ID>` adaptée à votre environnement.

   > [!IMPORTANT]
   > Pour éviter de supprimer des identités managées affectées par l’utilisateur existantes qui sont attribuées à la machine virtuelle, répertoriez les identités managées affectées par l’utilisateur en utilisant la commande CURL suivante : `curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"`. Si vous avez attribué des identités managées affectées par l’utilisateur à la machine virtuelle, telles qu’identifiées par la valeur `identity` dans la réponse, passez à l’étape 3 qui montre comment conserver les identités managées affectées par l’utilisateur, tout en désactivant l’identité managée affectée par le système sur votre machine virtuelle.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {  
       "identity":{  
          "type":"None"
       }
    }
   ```

   Pour supprimer l’identité managée affectée par le système d’une machine virtuelle ayant des identités managées affectées par l’utilisateur, supprimez `SystemAssigned` de la valeur `{"identity":{"type:" "}}` tout en conservant la valeur `UserAssigned` et les valeurs de dictionnaire `userAssignedIdentities` si vous utilisez la **version de l’API du 01/06/2018**. Si vous utilisez la **version d’API 2017-12-01** ou une version antérieure, conservez le tableau `identityIds`.

## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, découvrez comment ajouter et supprimer une identité managée affectée par l’utilisateur sur une machine virtuelle Azure, en utilisant CURL pour effectuer des appels au point de terminaison REST Azure Resource Manager.

### <a name="assign-a-user-assigned-managed-identity-during-the-creation-of-an-azure-vm"></a>Attribuer une identité managée affectée par l’utilisateur lors de la création d’une machine virtuelle Azure

Pour affecter une identité managée affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Créez une [interface réseau](/cli/azure/network/nic#az-network-nic-create) pour votre machine virtuelle :

   ```azurecli-interactive
    az network nic create -g myResourceGroup --vnet-name myVnet --subnet mySubnet -n myNic
   ```

3. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ``` 

4. Créez une identité managée attribuée par l’utilisateur en vous aidant des instructions disponibles ici : [Créer une identité managée attribuée par l’utilisateur](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

5. Créez une machine virtuelle à l’aide de CURL pour appeler le point de terminaison REST Azure Resource Manager. L’exemple suivant crée une machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup* avec une identité managée affectée par l’utilisateur `ID1`, telle qu’identifiée dans le corps de la demande par la valeur `"identity":{"type":"UserAssigned"}`. Remplacez `<ACCESS TOKEN>` par la valeur que vous avez reçue à l’étape précédente lorsque vous avez demandé un jeton d’accès du porteur et la valeur `<SUBSCRIPTION ID>` adaptée à votre environnement.
 
   **API VERSION 2018-06-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
    
   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {  
       "location":"westus",
       "name":"myVM",
       "identity":{  
          "type":"UserAssigned",
          "identityIds":[  
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{  
          "hardwareProfile":{  
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{  
             "imageReference":{  
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{  
                "caching":"ReadWrite",
                "managedDisk":{  
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[  
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {  
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{  
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{  
             "networkInterfaces":[  
                {  
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{  
                      "primary":true
                   }
                }
             ]
          }
       }
    }

   ```
  
   **API VERSION 2017-12-01**

   ```bash   
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PUT -d '{"location":"westus","name":"myVM","identity":{"type":"UserAssigned","identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]},"properties":{"hardwareProfile":{"vmSize":"Standard_D2_v2"},"storageProfile":{"imageReference":{"sku":"2016-Datacenter","publisher":"MicrosoftWindowsServer","version":"latest","offer":"WindowsServer"},"osDisk":{"caching":"ReadWrite","managedDisk":{"storageAccountType":"Standard_LRS"},"name":"myVM3osdisk","createOption":"FromImage"},"dataDisks":[{"diskSizeGB":1023,"createOption":"Empty","lun":0},{"diskSizeGB":1023,"createOption":"Empty","lun":1}]},"osProfile":{"adminUsername":"azureuser","computerName":"myVM","adminPassword":"myPassword12"},"networkProfile":{"networkInterfaces":[{"id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic","properties":{"primary":true}}]}}}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "location":"westus",
       "name":"myVM",
       "identity":{
          "type":"UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       },
       "properties":{
          "hardwareProfile":{
             "vmSize":"Standard_D2_v2"
          },
          "storageProfile":{
             "imageReference":{
                "sku":"2016-Datacenter",
                "publisher":"MicrosoftWindowsServer",
                "version":"latest",
                "offer":"WindowsServer"
             },
             "osDisk":{
                "caching":"ReadWrite",
                "managedDisk":{
                   "storageAccountType":"Standard_LRS"
                },
                "name":"myVM3osdisk",
                "createOption":"FromImage"
             },
             "dataDisks":[
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":0
                },
                {
                   "diskSizeGB":1023,
                   "createOption":"Empty",
                   "lun":1
                }
             ]
          },
          "osProfile":{
             "adminUsername":"azureuser",
             "computerName":"myVM",
             "adminPassword":"myPassword12"
          },
          "networkProfile":{
             "networkInterfaces":[
                {
                   "id":"/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkInterfaces/myNic",
                   "properties":{
                      "primary":true
                   }
                }
             ]
          }
       }
    }
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Attribuer une identité managée affectée par l’utilisateur à une machine virtuelle Azure existante

Pour affecter une identité managée affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator). Aucune attribution de rôle d’annuaire Azure AD supplémentaire n’est requise.

1. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ```

2.  Créez une identité managée affectée par l’utilisateur à l’aide des instructions fournies dans [Créer une identité managée affectée par l’utilisateur](how-to-manage-ua-identity-rest.md#create-a-user-assigned-managed-identity).

3. Pour éviter de supprimer des identités managées affectées par l’utilisateur ou le système existantes qui sont attribuées à la machine virtuelle, vous devez répertorier les types d’identités affectés à la machine virtuelle en utilisant la commande CURL suivante. Si vous avez affecté des identités managées au groupe de machines virtuelles identiques, celles-ci sont répertoriées sous la valeur `identity`.

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>" 
   ```

   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.

    Si vous avez des identités managées affectées par l’utilisateur ou le système qui sont attribuées à la machine virtuelle, telle qu’identifiées par la valeur `identity` dans la réponse, passez à l’étape 5 qui montre comment conserver l’identité managée affectée par le système, tout en ajoutant une identité managée affectée par l’utilisateur sur votre machine virtuelle.

4. Si vous n’avez aucune identité managée affectée par l’utilisateur attribuée à votre machine virtuelle, utilisez la commande CURL suivante pour appeler le point de terminaison REST Azure Resource Manager afin d’attribuer la première identité managée affectée par l’utilisateur à la machine virtuelle.

   L’exemple suivant attribue une identité managée affectée par l’utilisateur, `ID1`, à une machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup*.  Remplacez `<ACCESS TOKEN>` par la valeur que vous avez reçue à l’étape précédente lorsque vous avez demandé un jeton d’accès du porteur et la valeur `<SUBSCRIPTION ID>` adaptée à votre environnement.

   **API VERSION 2018-06-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        |
 
   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             }
          }
       }
    }
   ```

   **API VERSION 2017-12-01**

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"userAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```
   
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"userAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

5. Si une identité managée existante affectée par l’utilisateur ou par le système est attribuée à votre machine virtuelle :
   
   **API VERSION 2018-06-01**

   Ajoutez l’identité managée affectée par l’utilisateur à la valeur de dictionnaire `userAssignedIdentities`.
    
   Par exemple, si des identités managées affectées par le système et par l’utilisateur `ID1` sont actuellement attribuées à votre machine virtuelle, et si vous souhaitez ajouter l’identité managée affectée par l’utilisateur `ID2` à celle-ci :

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{},"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{}}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```
   
   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1":{
    
             },
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":{
    
             }
          }
       }
    }
   ```

   **API VERSION 2017-12-01**

   Conservez les identités managées affectées par l’utilisateur que vous souhaitez garder dans la valeur de tableau `identityIds` lors de l’ajout de la nouvelle identité managée affectée par l’utilisateur.

   Par exemple, si des identités managées affectées par le système et par l’utilisateur `ID1` sont actuellement attribuées à votre machine virtuelle, et si vous souhaitez ajouter l’identité managée affectée par l’utilisateur `ID2` à celle-ci : 

   ```bash
   curl  'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned,UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1","/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned,UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1",
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2"
          ]
       }
    }
   ```   

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Supprimer une identité managée affectée par l’utilisateur d’une machine virtuelle Azure

Pour supprimer une identité affectée par l’utilisateur à une machine virtuelle, votre compte a besoin de l’affectation de rôle [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

1. Récupérez un jeton d’accès de porteur que vous allez utiliser à l’étape suivante dans l’en-tête d’autorisation pour créer votre machine virtuelle avec une identité managée affectée par le système.

   ```azurecli-interactive
   az account get-access-token
   ```

2. Pour éviter de supprimer des identités managées affectées par l’utilisateur existantes que vous souhaitez conserver sur la machine virtuelle, ou de supprimer l’identité managée affectée par le système, vous devez répertorier les identités managées en utilisant la commande CURL suivante : 

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01' -H "Authorization: Bearer <ACCESS TOKEN>"
   ```
 
   ```HTTP
   GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.Compute/virtualMachines/<VM NAME>?api-version=2018-06-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.
 
   Si vous avez affecté des identités managées à la machine virtuelle, celles-ci sont répertoriées dans la réponse sous la valeur `identity`.

   Par exemple, si des identités managées affectées par l’utilisateur `ID1` et `ID2` sont attribuées à votre machine virtuelle et que vous souhaitez uniquement garder l’élément `ID1` affecté et conserver l’identité affectée par le système :
   
   **API VERSION 2018-06-01**

   Ajoutez `null` à l’identité managée affectée par l’utilisateur que vous souhaitez supprimer :

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "userAssignedIdentities":{"/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null}}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "userAssignedIdentities":{
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID2":null
          }
       }
    }
   ```

   **API VERSION 2017-12-01**

   Conservez uniquement la ou les identités managées affectées par l’utilisateur que vous souhaitez garder dans le tableau `identityIds` :

   ```bash
   curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01' -X PATCH -d '{"identity":{"type":"SystemAssigned, UserAssigned", "identityIds":["/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"]}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
   ```

   ```HTTP
   PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2017-12-01 HTTP/1.1
   ```

   **En-têtes de requête**

   |En-tête de requête  |Description  |
   |---------|---------|
   |*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
   |*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.        | 

   **Corps de la demande**

   ```JSON
    {
       "identity":{
          "type":"SystemAssigned, UserAssigned",
          "identityIds":[
             "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/ID1"
          ]
       }
    }
   ```

Si votre machine virtuelle dispose d’identités managées affectées tant par le système que par l’utilisateur, vous pouvez supprimer toutes les identités managées affectées par l’utilisateur en choisissant de n’utiliser que l’identité managée affectée par le système en utilisant la commande suivante :

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"SystemAssigned"}}' -H "Content-Type: application/json" -H "Authorization:Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
```

**En-têtes de requête**

|En-tête de requête  |Description  |
|---------|---------|
|*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
|*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide. | 

**Corps de la demande**

```JSON
{
   "identity":{
      "type":"SystemAssigned"
   }
}
```
    
Si votre machine virtuelle a uniquement des identités managées affectées par l’utilisateur que vous souhaitez supprimer, utilisez la commande suivante :

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01' -X PATCH -d '{"identity":{"type":"None"}}' -H "Content-Type: application/json" -H Authorization:"Bearer <ACCESS TOKEN>"
```

```HTTP
PATCH https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM?api-version=2018-06-01 HTTP/1.1
```

**En-têtes de requête**

|En-tête de requête  |Description  |
|---------|---------|
|*Content-Type*     | Obligatoire. Défini sur `application/json`.        |
|*Autorisation*     | Obligatoire. Défini sur un jeton d’accès `Bearer` valide.| 

**Corps de la demande**

```JSON
{
   "identity":{
      "type":"None"
   }
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la façon de créer, de répertorier ou de supprimer des identités managées affectées par l’utilisateur en utilisant REST, voir :

- [Créer, répertorier ou supprimer une identité affectée par l’utilisateur à l’aide d’appels d’API REST](how-to-manage-ua-identity-rest.md)
