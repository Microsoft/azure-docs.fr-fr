---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/31/2020
ms.author: alkohli
ms.openlocfilehash: 3a17e73c66c2296cc36b24e3b0a8abfcab00e46a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89419393"
---
Avant de pouvoir déployer des machines virtuelles sur votre appareil Azure Stack Edge, vous devez configurer votre client pour qu’il se connecte à l’appareil via Azure Resource Manager sur Azure PowerShell. Pour obtenir des instructions détaillées, accédez à [Se connecter à Azure Resource Manager sur votre appareil Azure Stack Edge](../articles/databox-online/azure-stack-edge-j-series-connect-resource-manager.md).


Assurez-vous de pouvoir effectuer les étapes suivantes pour accéder à l’appareil à partir de votre client (vous avez effectué cette configuration lors de la connexion à Azure Resource Manager, vous vérifiez simplement que la configuration est correcte) : 

1. Vérifiez que la communication avec Azure Resource Manager est possible. Tapez :     

    ```powershell
    Add-AzureRmEnvironment -Name <Environment Name> -ARMEndpoint "https://management.<appliance name>.<DNSDomain>"
    ```

1. Appelez les API d’appareil local pour l’authentification. Tapez : 

    `login-AzureRMAccount -EnvironmentName <Environment Name>`

    Indiquez le nom d’utilisateur *EdgeARMuser* et le mot de passe utilisés pour vous connecter via Azure Resource Manager.

1. Si vous avez configuré **Calcul** pour Kubernetes, vous pouvez ignorer cette étape. Assurez-vous maintenant que vous avez activé une interface réseau pour le calcul. Dans l’interface utilisateur locale, accédez aux paramètres **Calcul**. Sélectionnez l’interface réseau que vous allez utiliser pour créer un commutateur virtuel. Les machines virtuelles que vous créez seront attachées à un commutateur virtuel, lui-même attaché à ce port et au réseau associé. Choisissez un réseau qui correspond à l’adresse IP que vous allez utiliser pour la machine virtuelle.  

    ![Activer les paramètres de calcul 1](../articles/databox-online/media/azure-stack-edge-gpu-deploy-virtual-machine-templates/enable-compute-setting.png)

    Activez le calcul sur l’interface réseau. Azure Stack Edge va ensuite créer et gérer un commutateur virtuel correspondant à cette interface réseau. N’entrez pas d’adresses IP spécifiques pour Kubernetes à ce stade. L’activation du calcul peut prendre plusieurs minutes.

    <!--If you decide to use another network interface for compute, make sure that you:
    
    - Delete all the VMs that you have deployed using Azure Resource Manager.
    
    - Delete all virtual network interfaces and the virtual network associated with this network interface. 
    
    - You can now enable another network interface for compute.-->

<!--1. You may also need to configure TLS 1.2 on your client machine if running older versions of AzCopy.--> 

