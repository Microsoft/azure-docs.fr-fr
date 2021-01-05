---
title: Déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro au moyen de modèles
description: Explique comment créer et gérer des machines virtuelles sur un appareil Azure Stack Edge Pro à l’aide de modèles.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/16/2020
ms.author: alkohli
ms.openlocfilehash: 93df80cd6fcd6f5553ea509a4778a155299bb057
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449071"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-templates"></a>Déployer des machines virtuelles sur votre appareil Azure Stack Edge Pro avec GPU au moyen de modèles

Ce tutoriel explique comment créer et gérer une machine virtuelle sur votre appareil Azure Stack Edge Pro à l’aide de modèles. Ces modèles sont des fichiers JSON (JavaScript Object Notation) qui définissent l’infrastructure et la configuration d’une machine virtuelle. Y sont spécifiées les ressources à déployer et les propriétés de ces ressources.

Les modèles sont flexibles dans différents environnements, car ils peuvent prendre des paramètres en entrée à l’exécution à partir d’un fichier. La structure d’affectation de noms standard est `TemplateName.json` pour le modèle et `TemplateName.parameters.json` pour le fichier de paramètres. Pour plus d’informations sur les modèles ARM, consultez [En quoi consistent les modèles Azure Resource Manager ?](../azure-resource-manager/templates/overview.md).

Dans ce tutoriel, nous allons utiliser des exemples de modèles pré-écrits pour créer des ressources. Il ne sera pas nécessaire de retoucher le fichier de modèle. Vous pourrez modifier uniquement les fichiers `.parameters.json` pour personnaliser le déploiement sur votre ordinateur. 

## <a name="vm-deployment-workflow"></a>Workflow du déploiement de machine virtuelle

Pour déployer des machines virtuelles Azure Stack Edge Pro sur plusieurs appareils, vous pouvez utiliser un seul disque dur virtuel préparé avec Sysprep pour tout votre parc, et le même modèle pour le déploiement. Il suffit d’apporter simplement des modifications mineures aux paramètres de ce modèle pour chaque emplacement de déploiement (à la main comme ici, ou bien par programme). 

Voici une synthèse globale du workflow de déploiement à l’aide de modèles :

1. **Configuration des prérequis**. Il existe trois types de prérequis : l’appareil, le client et la machine virtuelle.

    1. **Prérequis liés à l’appareil**

        1. Connectez-vous à Azure Resource Manager sur l’appareil.
        2. Activez le calcul au moyen de l’interface utilisateur locale.

    2. **Prérequis pour le client**

        1. Téléchargez les modèles de machine virtuelle et les fichiers associés sur le client.
        1. Configurez TLS 1.2 sur le client (facultatif).
        1. [Téléchargez et installez l’Explorateur Stockage Microsoft Azure](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409) sur votre client.

    3. **Prérequis liés à la machine virtuelle**

        1. Créez un groupe de ressources dans l’emplacement de l’appareil qui contiendra toutes les ressources de machine virtuelle.
        1. Créez un compte de stockage afin de charger le disque dur virtuel utilisé pour créer l’image de machine virtuelle.
        1. Ajoutez l’URI du compte de stockage local au nom DNS ou au fichier hosts sur le client qui accède à votre appareil.
        1. Installez le certificat de Stockage Blob sur l’appareil ainsi que sur le client local qui accède à votre appareil. Installez le certificat de Stockage Blob sur l’Explorateur Stockage (facultatif).
        1. Créez un disque dur virtuel et chargez-le sur le compte de stockage créé précédemment.

2. **Création d’une machine virtuelle à partir de modèles** :

    1. Créez une image de machine virtuelle et un réseau virtuel à l’aide du fichier de paramètres `CreateImageAndVnet.parameters.json` et du modèle de déploiement `CreateImageAndVnet.json`.
    1. Créez une machine virtuelle avec les ressources créées précédemment à l’aide du fichier de paramètres `CreateVM.parameters.json` et du modèle de déploiement `CreateVM.json`.

## <a name="device-prerequisites"></a>Prérequis liés à l’appareil

Configurez ces prérequis sur votre appareil Azure Stack Edge Pro.

[!INCLUDE [azure-stack-edge-gateway-deploy-virtual-machine-prerequisites](../../includes/azure-stack-edge-gateway-deploy-virtual-machine-prerequisites.md)]

## <a name="client-prerequisites"></a>Prérequis pour le client

Configurez ces prérequis sur le client qui sera utilisé pour accéder à l’appareil Azure Stack Edge Pro.

1. [Téléchargez l’Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/) si vous l’utilisez pour charger un disque dur virtuel. Vous pouvez également télécharger AzCopy pour cela. Vous devrez peut-être configurer TLS 1.2 sur votre ordinateur client si vous exécutez des versions antérieures d’AzCopy. 
1. [Téléchargez les modèles de machine virtuelle et les fichiers de paramètres](https://aka.ms/ase-vm-templates) sur votre ordinateur client. Décompressez-les dans le répertoire qui vous servira de répertoire de travail.


## <a name="vm-prerequisites"></a>Prérequis liés à la machine virtuelle

Configurez ces prérequis pour créer les ressources qui seront nécessaires à la création de machines virtuelles. 

    
### <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzureRmResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel sont déployées et gérées les ressources Azure, comme le compte de stockage, le disque et le disque managé.

> [!IMPORTANT]
> Toutes les ressources sont créées au même emplacement que l’appareil, à savoir **DBELocal**.

```powershell
New-AzureRmResourceGroup -Name <Resource group name> -Location DBELocal
```

Voici un exemple de sortie obtenue.

```powershell
PS C:\windows\system32> New-AzureRmResourceGroup -Name myasegpurgvm -Location DBELocal

ResourceGroupName : myasegpurgvm
Location          : dbelocal
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/DDF9FC44-E990-42F8-9A91-5A6A5CC472DB/resourceGroups/myasegpurgvm

PS C:\windows\system32>
```

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage en utilisant le groupe de ressources créé à l’étape précédente. Il s’agit d’un **compte de stockage local** qui sera utilisé pour charger l’image de disque virtuel de la machine virtuelle.

```powershell
New-AzureRmStorageAccount -Name <Storage account name> -ResourceGroupName <Resource group name> -Location DBELocal -SkuName Standard_LRS
```

> [!NOTE]
> Seuls les comptes de stockage locaux comme le stockage localement redondant (Standard_LRS ou Premium_LRS) peuvent être créés par le biais d’Azure Resource Manager. Pour créer des comptes de stockage hiérarchisés, consultez les étapes décrites dans [Ajouter des comptes de stockage à Azure Stack Edge Pro et se connecter à ces comptes](azure-stack-edge-j-series-deploy-add-storage-accounts.md).

Voici un exemple de sortie obtenue.

```powershell
PS C:\windows\system32> New-AzureRmStorageAccount -Name myasegpusavm -ResourceGroupName myasegpurgvm -Location DBELocal -SkuName Standard_LRS

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime
------------------ ----------------- -------- -------     ----    ---------- ------------
myasegpusavm       myasegpurgvm      DBELocal StandardLRS Storage            7/29/2020 10:11:16 PM

PS C:\windows\system32>
```

Pour obtenir la clé du compte de stockage, exécutez la commande `Get-AzureRmStorageAccountKey`. Voici un exemple de sortie de cette commande.

```powershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasegpurgvm
Name: myasegpusavm

KeyName    Value                                                  Permissions   
-------     -----                                                   --
key1 GsCm7QriXurqfqx211oKdfQ1C9Hyu5ZutP6Xl0dqlNNhxLxDesDej591M8y7ykSPN4fY9vmVpgc4ftkwAO7KQ== 11 
key2 7vnVMJUwJXlxkXXOyVO4NfqbW5e/5hZ+VOs+C/h/ReeoszeV+qoyuBitgnWjiDPNdH4+lSm1/ZjvoBWsQ1klqQ== ll
```

### <a name="add-blob-uri-to-hosts-file"></a>Ajout de l’URI d’objet blob au fichier hosts

Vérifiez que vous avez déjà ajouté l’URI d’objet blob dans le fichier hosts du client que vous utilisez pour vous connecter au Stockage Blob. **Exécutez le Bloc-notes en tant qu’administrateur** et ajoutez l’entrée suivante pour l’URI de l’objet blob dans `C:\windows\system32\drivers\etc\hosts` :

`<Device IP> <storage account name>.blob.<Device name>.<DNS domain>`

Dans un environnement classique, votre DNS est configuré de telle sorte que tous les comptes de stockage pointent vers l’appareil Azure Stack Edge Pro avec une entrée `*.blob.devicename.domainname.com`.

### <a name="optional-install-certificates"></a>Installation des certificats (facultatif)

Ignorez cette étape si vous vous connectez au moyen de l’Explorateur Stockage par *HTTP*. Si vous utilisez *HTTPS*, vous devez installer les certificats correspondants dans l’Explorateur Stockage. Dans ce cas, installez le certificat du point de terminaison d’objet blob. Pour plus d’informations, découvrez comment créer et charger des certificats dans [Gestion des certificats](azure-stack-edge-j-series-manage-certificates.md). 

### <a name="create-and-upload-a-vhd"></a>Créer et charger un disque dur virtuel

Vérifiez que vous disposez d’une image de disque virtuel pouvant servir au chargement à l’étape suivante. Suivez la procédure [Création d’une image de machine virtuelle](azure-stack-edge-j-series-create-virtual-machine-image.md). 

Copiez les images de disque à utiliser dans des objets blob de pages du compte de stockage local que vous avez créé lors des étapes précédentes. Vous pouvez utiliser un outil comme [l’Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/) ou [AzCopy pour charger le disque dur virtuel dans le compte de stockage](azure-stack-edge-j-series-deploy-virtual-machine-powershell.md#upload-a-vhd) déjà créé. 

### <a name="use-storage-explorer-for-upload"></a>Chargement à l’aide de l’Explorateur Stockage

1. Ouvrez l’Explorateur de stockage. Accédez à **Modifier** et vérifiez que l’application est définie sur **Cibler les API Azure Stack**.

    ![Définition des API Azure Stack comme cibles](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/set-target-apis-1.png)

2. Installez le certificat client au format PEM. Accédez à **Modifier > Certificats SSL > Importer des certificats**. Pointez vers le certificat client.

    ![Importation du certificat du point de terminaison de Stockage Blob](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/import-blob-storage-endpoint-certificate-1.png)

    - Si vous utilisez des certificats générés par l’appareil, téléchargez le certificat `.cer` du point de terminaison de Stockage Blob et convertissez-le au format `.pem` . Exécutez la commande suivante : 
    
        ```powershell
        PS C:\windows\system32> Certutil -encode 'C:\myasegpu1_Blob storage (1).cer' .\blobstoragecert.pem
        Input Length = 1380
        Output Length = 1954
        CertUtil: -encode command completed successfully.
        ```
    - Si vous apportez votre propre certificat, utilisez le certificat racine de chaîne de signature au format `.pem`.

3. Après avoir importé le certificat, redémarrez l’Explorateur Stockage pour que les modifications prennent effet.

    ![Redémarrez l’Explorateur de stockage](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/restart-storage-explorer-1.png)

4. Dans le volet gauche, cliquez avec le bouton droit sur **Comptes de stockage**, puis sélectionnez **Se connecter à Stockage Azure**. 

    ![Se connecter à Stockage Azure 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-1.png)

5. Sélectionnez **Utiliser le nom et la clé d’un compte de stockage**. Sélectionnez **Suivant**.

    ![Se connecter à Stockage Azure 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-azure-storage-2.png)

6. Dans **Se connecter avec un nom et une clé**, indiquez le **Nom d’affichage**, le **Nom du compte de stockage** et la **Clé de compte** Stockage Azure. Sélectionnez **Autre** domaine de stockage, puis spécifiez la chaîne de connexion `<device name>.<DNS domain>` . Si vous n’avez pas installé de certificat dans l’Explorateur Stockage, activez l’option **Utiliser HTTP**. Sélectionnez **Suivant**.

    ![Connexion avec un nom et une clé](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/connect-name-key-1.png)

7. Consultez la **Synthèse de connexion** et sélectionnez **Se connecter**.

8. Le compte de stockage s’affiche dans le volet gauche. Sélectionnez et développez le compte de stockage. Sélectionnez **Conteneurs d’objets blob**, cliquez avec le bouton droit, puis sélectionnez **Créer un conteneur d’objets blob**. Donnez un nom à votre conteneur d’objets blob.

9. Sélectionnez le conteneur que vous venez de créer, puis sélectionnez **Charger > Charger des fichiers** dans le volet droit. 

    ![Chargement d’un fichier de disque dur virtuel 1](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-1.png)

10. Recherchez le disque dur virtuel que vous souhaitez charger et pointez vers celui-ci dans **Fichiers sélectionnés**. Sélectionnez le **Type d’objet blob** **Objet blob de pages**, puis **Charger**.

    ![Chargement d’un fichier de disque dur virtuel 2](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-2.png)

11. Une fois le disque dur virtuel chargé dans le conteneur d’objets blob, sélectionnez le disque dur virtuel, cliquez avec le bouton droit, puis sélectionnez **Propriétés**. 

    ![Chargement d’un fichier de disque dur virtuel 3](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/upload-vhd-file-3.png)

12. Copiez et enregistrez **l’URI**, car vous en aurez besoin par la suite.

    ![Copie de l’URI](media/azure-stack-edge-gpu-deploy-virtual-machine-templates/copy-uri-1.png)

<!--### Use AzCopy for upload

Before you use AzCopy, make sure that the [AzCopy is configured correctly](#configure-azcopy) for use with the blob storage REST API version that you are using with your Azure Stack Edge Pro device.


```powershell
AzCopy /Source:<sourceDirectoryForVHD> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Y /S /V /NC:32  /BlobType:page /destType:blob 
```

> ![NOTE]
> Set `BlobType` to page for creating a managed disk out of VHD. Set `BlobType` to block when writing to tiered storage accounts using AzCopy.

You can download the disk images from the marketplace. For detailed steps, go to [Get the virtual disk image from Azure marketplace](azure-stack-edge-j-series-create-virtual-machine-image.md).

A sample output using AzCopy 7.3 is shown below. For more information on this command, go to [Upload VHD file to storage account using AzCopy](../devtest-labs/devtest-lab-upload-vhd-using-azcopy.md).


```powershell
AzCopy /Source:\\hcsfs\scratch\vm_vhds\linux\ /Dest:http://sa191113014333.blob.dbe-1dcmhq2.microsoftdatabox.com/vmimages /DestKey:gJKoyX2Amg0Zytd1ogA1kQ2xqudMHn7ljcDtkJRHwMZbMK== /Y /S /V /NC:32 /BlobType:page /destType:blob /z:2e7d7d27-c983-410c-b4aa-b0aa668af0c6
```-->

## <a name="create-image-and-vnet-for-your-vm"></a>Création d’une image et d’un réseau virtuel pour la machine virtuelle

Pour créer une image et un réseau virtuel pour votre machine virtuelle, vous devez modifier le fichier de paramètres `CreateImageAndVnet.parameters.json`, puis déployer le modèle `CreateImageAndVnet.json` qui utilise ce fichier.


### <a name="edit-parameters-file"></a>Modification du fichier de paramètres

Le fichier `CreateImageAndVnet.parameters.json` prend les paramètres suivants : 

```json
"parameters": {
        "osType": {
              "value": "<Operating system corresponding to the VHD you upload can be Windows or Linux>"
        },
        "imageName": {
            "value": "<Name for the VM image>"
        },
        "imageUri": {
              "value": "<Path to the VHD that you uploaded in the Storage account>"
        },
        "vnetName": {
            "value": "<Name for the virtual network where you will deploy the VM>"
        },
        "subnetName": {
            "value": "<Name for the subnet for the VNet>"
        },
        "addressPrefix": {
            "value": "<Address prefix for the virtual network>"
        },
        "subnetPrefix": {
            "value": "<Subnet prefix for the subnet for the Vnet>"
        }
    }
```

Modifiez le fichier `CreateImageAndVnet.parameters.json` de façon à inclure les éléments suivants pour votre appareil Azure Stack Edge Pro :

1. Indiquez le type de système d’exploitation correspondant au disque dur virtuel que vous allez charger : Windows ou Linux.

    ```json
    "parameters": {
            "osType": {
              "value": "Windows"
            },
    ```

2. Remplacez l’URI par celui de l’image que vous avez chargée à l’étape précédente :

    ```json
    "imageUri": {
        "value": "https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd"
        },
    ```
    Si vous utilisez *HTTP* avec l’Explorateur Stockage, remplacez-le par un URI *HTTPS* .

3. Modifiez `addressPrefix` et `subnetPrefix`. Dans l’interface utilisateur locale de votre appareil, accédez à la page **Réseau**. Recherchez le port que vous avez activé pour le calcul. Récupérez l’adresse IP du réseau de base et ajoutez le masque de sous-réseau afin de créer la notation CIDR. Pour cela, si vous disposez d’un sous-réseau 255.255.255.0 standard, remplacez le dernier numéro de l’adresse IP par 0 et ajoutez /24 à la fin. Par conséquent, 10.126.68.0 avec un masque de sous-réseau 255.255.255.0 devient 10.126.68.0/24. 
    
    ```json
    "addressPrefix": {
                "value": "10.126.68.0/24"
            },
            "subnetPrefix": {
                "value": "10.126.68.0/24"
            }
    ```  

4. Indiquez le nom unique de l’image, le nom du réseau virtuel et le nom du sous-réseau comme paramètres.

    Voici l’exemple JSON utilisé dans cet article.

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
      "parameters": {
        "osType": {
          "value": "Windows"
        },
        "imageName": {
          "value": "image1"
        },
        "imageUri": {
          "value": "https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd"
        },
        "vnetName": {
          "value": "vnet1"
        },
        "subnetName": {
          "value": "subnet1"
        },
        "addressPrefix": {
          "value": "10.126.68.0/24"
        },
        "subnetPrefix": {
          "value": "10.126.68.0/24"
        }
      }
    }
    ```
5. Enregistrez le fichier de paramètres.


### <a name="deploy-template"></a>Déployer un modèle 

Déployez le modèle `CreateImageAndVnet.json`. Ce modèle déploie les ressources de réseau virtuel et d’image qui serviront à créer les machines virtuelles à l’étape suivante.

> [!NOTE]
> Si vous recevez une erreur d’authentification lorsque vous déployez le modèle, ce peut être dû au fait que vos informations d’identification Azure ont expiré pour cette session. Réexécutez la commande `login-AzureRM` pour vous reconnecter à Azure Resource Manager sur votre appareil Azure Stack Edge Pro.

1. Exécutez la commande suivante : 
    
    ```powershell
    $templateFile = "Path to CreateImageAndVnet.json"
    $templateParameterFile = "Path to CreateImageAndVnet.parameters.json"
    $RGName = "<Name of your resource group>"
    New-AzureRmResourceGroupDeployment `
        -ResourceGroupName $RGName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $templateParameterFile `
        -Name "<Name for your deployment>"
    ```

2. Vérifiez que les ressources d’image et de réseau virtuel sont correctement provisionnées. Voici un exemple de sortie d’une image et d’un réseau virtuel dont la création a réussi.
    
    ```powershell
    PS C:\07-30-2020> login-AzureRMAccount -EnvironmentName aztest1 -TenantId c0257de7-538f-415c-993a-1b87a031879d
    
    Account               SubscriptionName              TenantId                             Environment
    -------               ----------------              --------                             -----------
    EdgeArmUser@localhost Default Provider Subscription c0257de7-538f-415c-993a-1b87a031879d aztest1
    
    PS C:\07-30-2020> $templateFile = "C:\07-30-2020\CreateImageAndVnet.json"
    PS C:\07-30-2020> $templateParameterFile = "C:\07-30-2020\CreateImageAndVnet.parameters.json"
    PS C:\07-30-2020> $RGName = "myasegpurgvm"
    PS C:\07-30-2020> New-AzureRmResourceGroupDeployment `
    >>     -ResourceGroupName $RGName `
    >>     -TemplateFile $templateFile `
    >>     -TemplateParameterFile $templateParameterFile `
    >>     -Name "Deployment1"
    
    DeploymentName          : Deployment1
    ResourceGroupName       : myasegpurgvm
    ProvisioningState       : Succeeded
    Timestamp               : 7/30/2020 5:53:32 PM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name             Type                       Value
                              ===============  =========================  ==========
                              osType           String                     Windows
                              imageName        String                     image1
                              imageUri         String
                              https://myasegpusavm.blob.myasegpu1.wdshcsso.com/windows/WindowsServer2016Datacenter.vhd
                              vnetName         String                     vnet1
                              subnetName       String                     subnet1
                              addressPrefix    String                     10.126.68.0/24
                              subnetPrefix     String                     10.126.68.0/24
    
    Outputs                 :
    DeploymentDebugLogLevel :
    
    PS C:\07-30-2020>
    ```
    
## <a name="create-vm"></a>Créer une machine virtuelle

### <a name="edit-parameters-file-to-create-vm"></a>Modification du fichier de paramètres de façon à créer une machine virtuelle
 
Pour créer une machine virtuelle, utilisez le fichier de paramètres `CreateVM.parameters.json`. Il prend les paramètres suivants.
    
```json
"vmName": {
            "value": "<Name for your VM>"
        },
        "adminUsername": {
            "value": "<Username to log into the VM>"
        },
        "Password": {
            "value": "<Password to log into the VM>"
        },
        "imageName": {
            "value": "<Name for your image>"
        },
        "vmSize": {
            "value": "<A supported size for your VM>"
        },
        "vnetName": {
            "value": "<Name for the virtual network you created earlier>"
        },
        "subnetName": {
            "value": "<Name for the subnet you created earlier>"
        },
        "nicName": {
            "value": "<Name for the network interface>"
        },
        "privateIPAddress": {
            "value": "<Private IP address, enter a static IP in the subnet created earlier or leave empty to assign DHCP>"
        },
        "IPConfigName": {
            "value": "<Name for the ipconfig associated with the network interface>"
        }
```    

Affectez les paramètres nécessaires dans `CreateVM.parameters.json` pour votre appareil Azure Stack Edge Pro.

1. Indiquez un nom unique, un nom d’interface réseau et un nom ipconfig. 
1. Entrez un nom d’utilisateur, un mot de passe et une taille de machine virtuelle prise en charge.
1. Donnez le même nom pour **VnetName**, **subnetName** et **ImageName** que dans les paramètres de `CreateImageAndVnet.parameters.json`. Par exemple, si vous avez respectivement donné les noms **vnet1**, **subnet1** et **image1**, conservez ces valeurs pour les paramètres de ce modèle.
1. À présent, il vous faut une adresse IP statique à attribuer à la machine virtuelle qui se trouve dans le sous-réseau défini ci-dessus. Remplacez **PrivateIPAddress** par cette adresse dans le fichier de paramètres. Pour que la machine virtuelle récupère une adresse IP auprès de votre serveur DHCP local, laissez la valeur `privateIPAddress` vide.  
    
    ```json
    "privateIPAddress": {
                "value": "5.5.153.200"
            },
    ```
    
4. Enregistrez le fichier de paramètres.

    Voici l’exemple JSON utilisé dans cet article.
    
    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "vmName": {
                "value": "mywindowsvm"
            },
            "adminUsername": {
                "value": "Administrator"
            },
            "Password": {
                "value": "Password1"
            },
            "imageName": {
                "value": "image1"
            },
            "vmSize": {
                "value": "Standard_D1_v2"
            },
            "vnetName": {
                "value": "vnet1"
            },
            "subnetName": {
                "value": "subnet1"
            },
            "nicName": {
                "value": "nic1"
            },
            "privateIPAddress": {
                "value": "10.126.68.186"
            },
            "IPConfigName": {
                "value": "ipconfig1"
            }
        }
    }
    ```      

### <a name="deploy-template-to-create-vm"></a>Déploiement d’un modèle pour créer une machine virtuelle

Déployez le modèle de création de machine virtuelle `CreateVM.json`. Ce modèle crée une interface réseau à partir du réseau virtuel existant et une machine virtuelle à partir de l’image déployée.

1. Exécutez la commande suivante : 
    
    ```powershell
    Command:
        
        $templateFile = "<Path to CreateVM.json>"
        $templateParameterFile = "<Path to CreateVM.parameters.json>"
        $RGName = "<Resource group name>"
             
        New-AzureRmResourceGroupDeployment `
            -ResourceGroupName $RGName `
            -TemplateFile $templateFile `
            -TemplateParameterFile $templateParameterFile `
            -Name "<DeploymentName>"
    ```   

    La création de la machine virtuelle prend entre 15 et 20 minutes. Voici un exemple de sortie d’une machine virtuelle dont la création a réussi.
    
    ```powershell
    PS C:\07-30-2020> $templateFile = "C:\07-30-2020\CreateWindowsVM.json"
        PS C:\07-30-2020> $templateParameterFile = "C:\07-30-2020\CreateWindowsVM.parameters.json"
        PS C:\07-30-2020> $RGName = "myasegpurgvm"
        PS C:\07-30-2020> New-AzureRmResourceGroupDeployment `
        >>     -ResourceGroupName $RGName `
        >>     -TemplateFile $templateFile `
        >>     -TemplateParameterFile $templateParameterFile `
        >>     -Name "Deployment2"    
        
        DeploymentName          : Deployment2
        ResourceGroupName       : myasegpurgvm
        ProvisioningState       : Succeeded
        Timestamp               : 7/30/2020 6:21:09 PM
        Mode                    : Incremental
        TemplateLink            :
        Parameters              :
                                  Name             Type                       Value
                                  ===============  =========================  ==========
                                  vmName           String                     MyWindowsVM
                                  adminUsername    String                     Administrator
                                  password         String                     Password1
                                  imageName        String                     image1
                                  vmSize           String                     Standard_D1_v2
                                  vnetName         String                     vnet1
                                  subnetName       String                     subnet1
                                  nicName          String                     Nic1
                                  ipConfigName     String                     ipconfig1
                                  privateIPAddress  String                    10.126.68.186
        
        Outputs                 :
        DeploymentDebugLogLevel :    
        
        PS C:\07-30-2020>
    ```   

    Vous pouvez également exécuter la commande `New-AzureRmResourceGroupDeployment` de façon asynchrone avec le paramètre `–AsJob`. Voici un exemple de sortie lorsque l’applet de commande s’exécute en arrière-plan. Vous pouvez ensuite interroger l’état du travail créé à l’aide de l’applet de commande `Get-Job`.

    ```powershell   
    PS C:\WINDOWS\system32> New-AzureRmResourceGroupDeployment `
    >>     -ResourceGroupName $RGName `
    >>     -TemplateFile $templateFile `
    >>     -TemplateParameterFile $templateParameterFile `
    >>     -Name "Deployment2" `
    >>     -AsJob
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    2      Long Running... AzureLongRun... Running       True            localhost            New-AzureRmResourceGro...
     
    PS C:\WINDOWS\system32> Get-Job -Id 2
     
    Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
    --     ----            -------------   -----         -----------     --------             -------
    ```

7. Vérifiez que la machine virtuelle est correctement provisionnée. Exécutez la commande suivante :

    `Get-AzureRmVm`


## <a name="connect-to-a-vm"></a>Se connecter à une machine virtuelle

Selon que vous avez créé une machine virtuelle Windows ou Linux, les étapes de connexion peuvent être différentes.

### <a name="connect-to-windows-vm"></a>Se connecter à une machine virtuelle Windows

Suivez ces étapes pour vous connecter à une machine virtuelle Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

### <a name="connect-to-linux-vm"></a>Se connecter à une machine virtuelle Linux

Suivez ces étapes pour vous connecter à une machine virtuelle Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

<!--## Manage VM

The following section describes some of the common operations around the VM that you will create on your Azure Stack Edge Pro device.

[!INCLUDE [azure-stack-edge-gateway-manage-vm](../../includes/azure-stack-edge-gateway-manage-vm.md)]-->


## <a name="supported-vm-sizes"></a>Tailles des machines virtuelles prises en charge

[!INCLUDE [azure-stack-edge-gateway-supported-vm-sizes](../../includes/azure-stack-edge-gateway-supported-vm-sizes.md)]

## <a name="unsupported-vm-operations-and-cmdlets"></a>Opérations et cmdlets de machine virtuelle non prises en charge

Ni les extensions, ni les groupes identiques, ni les groupes à haute disponibilité, ni les instantanés ne sont pris en charge.

<!--## Configure AzCopy

When you install the latest version of AzCopy, you will need to configure AzCopy to ensure that it matches the blob storage REST API version of your Azure Stack Edge Pro device.

On the client used to access your Azure Stack Edge Pro device, set up a global variable to match the blob storage REST API version.

### On Windows client 

`$Env:AZCOPY_DEFAULT_SERVICE_API_VERSION = "2017-11-09"`

### On Linux client

`export AZCOPY_DEFAULT_SERVICE_API_VERSION=2017-11-09`

To verify if the environment variable for AzCopy was set correctly, take the following steps:

1. Run "azcopy env".
2. Find `AZCOPY_DEFAULT_SERVICE_API_VERSION` parameter. This should have the value you set in the preceding steps.-->


## <a name="next-steps"></a>Étapes suivantes

[Applets de commande Azure Resource Manager](/powershell/module/azurerm.resources/?view=azurermps-6.13.0)