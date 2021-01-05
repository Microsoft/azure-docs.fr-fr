---
title: PowerShell - Guide pratique pour créer des images de machines virtuelles avec Packer
description: Découvrez comment utiliser Packer et PowerShell pour créer des images de machines virtuelles dans Azure
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 08/05/2020
ms.author: cynthn
ms.openlocfilehash: 16d0a4ea2353778b16803bd9b92a7f8af6e4ba88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91325910"
---
# <a name="powershell-how-to-use-packer-to-create-virtual-machine-images-in-azure"></a>PowerShell : Comment utiliser Packer pour créer des images de machines virtuelles dans Azure
Chaque machine virtuelle dans Azure est créée à partir d’une image qui définit la distribution Windows et la version du système d’exploitation. Les images peuvent inclure des configurations et des applications pré-installées. La Place de marché Microsoft Azure fournit de nombreuses images internes et de tiers pour les systèmes d’exploitation et environnements d’application les plus courants. Vous pouvez également créer vos propres images personnalisées selon vos besoins. Cet article explique comment utiliser l’outil open source [Packer](https://www.packer.io/) pour définir et générer des images personnalisées dans Azure.

Cet article a été testé pour la dernière fois le 5/8/2020 à l’aide de [Packer](https://www.packer.io/docs/install) version 1.6.1.

> [!NOTE]
> Azure propose désormais un service, le générateur d’images Azure (préversion), pour définir et créer vos propres images personnalisées. Le générateur d’images Azure repose sur Packer. Vous pouvez donc même utiliser vos scripts d’approvisionnement de shell Packer existants. Pour vous familiariser avec le générateur d’images Azure, voir [Créer une machine virtuelle Windows avec le générateur d’images Azure](image-builder.md).

## <a name="create-azure-resource-group"></a>Créer un groupe de ressources Azure
Pendant le processus de génération, Packer crée des ressources Azure temporaires lorsqu’il génère la machine virtuelle source. Pour capturer cette machine virtuelle source afin de l’utiliser en tant qu’image, vous devez définir un groupe de ressources. La sortie du processus de génération Packer est stockée dans ce groupe de ressources.

Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé *myPackerGroup* à l’emplacement *eastus* :

```azurepowershell
$rgName = "myPackerGroup"
$location = "East US"
New-AzResourceGroup -Name $rgName -Location $location
```

## <a name="create-azure-credentials"></a>Créer des informations d’identification Azure
Packer s’authentifie auprès d’Azure à l’aide d’un principal de service. Un principal de service Azure est une identité de sécurité que vous pouvez utiliser avec des applications, des services et des outils d’automatisation comme Packer. Vous contrôlez et vous définissez les opérations que le principal du service est autorisé à effectuer dans Azure.

Créez un principal de service avec [New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal). La valeur de `-DisplayName` doit être unique ; remplacez-la par votre propre valeur, si nécessaire.  

```azurepowershell
$sp = New-AzADServicePrincipal -DisplayName "PackerSP$(Get-Random)"
$BSTR = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sp.Secret)
$plainPassword = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($BSTR)
```

Générez ensuite le mot de passe et l’ID d’application.

```powershell
$plainPassword
$sp.ApplicationId
```


Pour vous authentifier auprès d’Azure, vous devez également obtenir vos ID client et d’abonnement Azure à l’aide de la commande [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) :

```powershell
Get-AzSubscription
```


## <a name="define-packer-template"></a>Définition du modèle Packer
Pour générer les images, vous devez créer un modèle en tant que fichier JSON. Dans le modèle, vous définissez des générateurs et des fournisseurs pour mener à bien le processus de génération réel. Packer intègre un [générateur pour Azure](https://www.packer.io/docs/builders/azure.html) qui vous permet de définir des ressources Azure, telles que les informations d’identification du principal de service créées à l’étape précédente.

Créez un fichier nommé *windows.json* et collez le contenu suivant : Saisissez vos propres valeurs comme suit :

| Paramètre                           | Emplacement |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | Affichage de l’ID du principal de service avec `$sp.applicationId` |
| *client_secret*                     | Affichage du mot de passe généré automatiquement avec `$plainPassword` |
| *tenant_id*                         | Sortie de la commande `$sub.TenantId` |
| *subscription_id*                   | Sortie de la commande `$sub.SubscriptionId` |
| *managed_image_resource_group_name* | Nom du groupe de ressources créé lors de la première étape |
| *managed_image_name*                | Nom de l’image de disque géré créée |

```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "client_secret": "ppppppp-pppp-pppp-pppp-ppppppppppp",
    "tenant_id": "zzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz",
    "subscription_id": "yyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyy",

    "managed_image_resource_group_name": "myPackerGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Windows",
    "image_publisher": "MicrosoftWindowsServer",
    "image_offer": "WindowsServer",
    "image_sku": "2016-Datacenter",

    "communicator": "winrm",
    "winrm_use_ssl": true,
    "winrm_insecure": true,
    "winrm_timeout": "5m",
    "winrm_username": "packer",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "build_resource_group_name": "myPackerGroup",
    "vm_size": "Standard_D2_v2"
  }],
  "provisioners": [{
    "type": "powershell",
    "inline": [
      "Add-WindowsFeature Web-Server",
      "while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }",
      "while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }",
      "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
      "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
    ]
  }]
}
```

Ce modèle génère une machine virtuelle Windows Server 2016, installe IIS, puis généralise la machine virtuelle avec Sysprep. L’installation d’IIS montre comment utiliser le fournisseur PowerShell pour exécuter des commandes supplémentaires. L’image finale de Packer inclut l’installation et la configuration du logiciel requis.

L’agent invité Windows participe au processus Sysprep. L’agent doit être entièrement installé pour que la machine virtuelle puisse faire l’objet du processus Sysprep. Pour s’en assurer, tous les services de l’agent doivent être en cours d’exécution avant d’exécuter sysprep.exe. L’extrait de code JSON précédent montre une façon de le faire dans le fournisseur PowerShell. Cet extrait de code est uniquement requis si la machine virtuelle est configurée pour installer l’agent, ce qui correspond à la configuration par défaut.


## <a name="build-packer-image"></a>Génération de l’image Packer
Si Packer n’est pas encore installé sur votre ordinateur local, [suivez les instructions d’installation de Packer](https://learn.hashicorp.com/packer/getting-started/install).

Générez l’image en ouvrant une invite de commandes et en spécifiant votre fichier de modèle Packer de la manière suivante :

```
./packer build windows.json
```

Voici un exemple de la sortie des commandes précédentes :

```bash
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> task : Image deployment
==> azure-arm:  ->> dept : Engineering
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the certificate’s URL ...
==> azure-arm:  -> Key Vault Name        : ‘pkrkvpq0mthtbtt’
==> azure-arm:  -> Key Vault Secret Name : ‘packerKeyVaultSecret’
==> azure-arm:  -> Certificate URL       : ‘https://pkrkvpq0mthtbtt.vault.azure.net/secrets/packerKeyVaultSecret/8c7bd823e4fa44e1abb747636128adbb'
==> azure-arm: Setting the certificate’s URL ...
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.55.35’
==> azure-arm: Waiting for WinRM to become available...
==> azure-arm: Connected to WinRM!
==> azure-arm: Provisioning with Powershell...
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-powershell-provisioner902510110
    azure-arm: #< CLIXML
    azure-arm:
    azure-arm: Success Restart Needed Exit Code      Feature Result
    azure-arm: ------- -------------- ---------      --------------
    azure-arm: True    No             Success        {Common HTTP Features, Default Document, D...
    azure-arm: <Objs Version=“1.1.0.1” xmlns=“http://schemas.microsoft.com/powershell/2004/04"><Obj S=“progress” RefId=“0"><TN RefId=“0”><T>System.Management.Automation.PSCustomObject</T><T>System.Object</T></TN><MS><I64 N=“SourceId”>1</I64><PR N=“Record”><AV>Preparing modules for first use.</AV><AI>0</AI><Nil /><PI>-1</PI><PC>-1</PC><T>Completed</T><SR>-1</SR><SD> </SD></PR></MS></Obj></Objs>
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-pq0mthtbtt/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Compute Name              : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

La génération de la machine virtuelle, l’exécution des fournisseurs et le nettoyage du déploiement par Packer ne prennent que quelques minutes.


## <a name="create-a-vm-from-the-packer-image"></a>Créer une machine virtuelle à partir de l’image de Packer
Vous pouvez à présent créer une machine virtuelle à partir de votre image à l’aide de la commande [New-AzVM](/powershell/module/az.compute/new-azvm). Si elles n’existent pas déjà, les ressources réseau requises sont créées. À l’invite, entrez un nom d’utilisateur d’administration et un mot de passe à créer sur la machine virtuelle. L’exemple suivant permet de créer une machine virtuelle nommée *myVM* à partir de *myPackerImage* :

```powershell
New-AzVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Image "myPackerImage"
```

Si vous souhaitez créer des machines virtuelles dans un autre groupe de ressources ou dans une autre région que votre image Packer, spécifiez l’ID de l’image plutôt que son nom. Pour obtenir l’ID d’image, exécutez la commande [Get-AzImage](/powershell/module/az.compute/get-azimage).

La création de la machine virtuelle à partir de votre image Packer ne nécessite que quelques minutes.


## <a name="test-vm-and-webserver"></a>Tester la machine virtuelle et le serveur web
Obtenez l’adresse IP publique de votre machine virtuelle avec [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant obtient l’adresse IP pour *myPublicIP* créée précédemment :

```powershell
Get-AzPublicIPAddress `
    -ResourceGroupName $rgName `
    -Name "myPublicIPAddress" | select "IpAddress"
```

Pour voir votre machine virtuelle, qui inclut l’installation d’IIS à partir du fournisseur Packer, entrez l’adresse IP publique dans un navigateur web.

![Site IIS par défaut](./media/build-image-with-packer/iis.png) 


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez également utiliser les scripts d’approvisionnement Packer existants avec le [générateur d’images Azure](image-builder.md).
