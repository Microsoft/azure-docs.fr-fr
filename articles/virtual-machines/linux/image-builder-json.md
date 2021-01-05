---
title: Créer un modèle de générateur d’images Azure (préversion)
description: Découvrez comment créer un modèle à utiliser avec le générateur d’images Azure.
author: danielsollondon
ms.author: danis
ms.date: 08/13/2020
ms.topic: reference
ms.service: virtual-machines
ms.subservice: imaging
ms.reviewer: cynthn
ms.openlocfilehash: 43f33093010aa6a70d02c58e9faa34f7f0e2dfee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91307277"
---
# <a name="preview-create-an-azure-image-builder-template"></a>Aperçu : Créer un modèle de générateur d’images Azure 

Le générateur d’images Azure utilise un fichier .json pour transmettre des informations au service du générateur d’images. Dans cet article, nous allons vous présenter les sections du fichier json pour que vous puissiez créer le vôtre. Pour voir des exemples de fichiers .json complets, consultez [GitHub sur le générateur d’images Azure](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).

Voici le format de modèle de base :

```json
 { 
    "type": "Microsoft.VirtualMachineImages/imageTemplates", 
    "apiVersion": "2020-02-14", 
    "location": "<region>", 
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
     },
    "identity":{},           
    "dependsOn": [], 
    "properties": { 
        "buildTimeoutInMinutes": <minutes>, 
        "vmProfile": 
            {
            "vmSize": "<vmSize>",
            "osDiskSizeGB": <sizeInGB>,
            "vnetConfig": {
                "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
                }
            },
        "source": {}, 
        "customize": {}, 
        "distribute": {} 
      } 
 } 
```



## <a name="type-and-api-version"></a>Type et version de l’API

`type` est le type de ressource, qui doit être `"Microsoft.VirtualMachineImages/imageTemplates"`. `apiVersion` change au fil du temps à mesure que l’API change, mais doit être `"2020-02-14"` pour la préversion.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2020-02-14",
```

## <a name="location"></a>Emplacement

L’emplacement est la région dans laquelle l’image personnalisée sera créée. Pour le générateur d’images en préversion, les régions suivantes sont prises en charge :

- USA Est
- USA Est 2
- Centre-USA Ouest
- USA Ouest
- USA Ouest 2
- Europe Nord
- Europe Ouest


```json
    "location": "<region>",
```
## <a name="vmprofile"></a>vmProfile
Par défaut, Image Builder utilise une machine virtuelle de build « Standard_D1_v2 » mais vous pouvez modifier cela. Par exemple, si vous souhaitez personnaliser une image pour une machine virtuelle GPU, vous avez besoin d’une taille de machine virtuelle GPU. Ce paramètre est facultatif.

```json
 {
    "vmSize": "Standard_D1_v2"
 },
```

## <a name="osdisksizegb"></a>osDiskSizeGB

Par défaut, Image Builder ne modifie pas la taille de l’image et utilise la taille de l’image source. La taille du disque du système d’exploitation (Win et Linux) peut **seulement** être augmentée. Cette opération est facultative ; la valeur 0 indique de conserver la taille de l’image source. Vous ne pouvez pas choisir une taille de disque du système d’exploitation inférieure à celle de l’image source.

```json
 {
    "osDiskSizeGB": 100
 },
```

## <a name="vnetconfig"></a>vnetConfig
Si vous ne spécifiez pas de propriétés de réseau virtuel, le générateur d’images créera son propre réseau virtuel, sa propre adresse IP publique et son propre groupe de sécurité réseau. L’adresse IP publique est utilisée par le service pour communiquer avec la machine virtuelle de build. Toutefois, si vous ne souhaitez pas une adresse IP publique ou si vous souhaitez que le générateur d’images ait accès à vos ressources de réseau virtuel existantes, comme les serveurs de configuration (DSC, Chef, Puppet, Ansible), les partages de fichiers, etc., vous pouvez spécifier un réseau virtuel. Pour plus d’informations, consultez la [documentation relative à la mise en réseau](image-builder-networking.md) (facultatif).

```json
    "vnetConfig": {
        "subnetId": "/subscriptions/<subscriptionID>/resourceGroups/<vnetRgName>/providers/Microsoft.Network/virtualNetworks/<vnetName>/subnets/<subnetName>"
        }
    }
```
## <a name="tags"></a>Balises

Il s’agit de paires clé/valeur que vous pouvez spécifier pour l’image générée.

## <a name="depends-on-optional"></a>Dépend de (facultatif)

Cette section facultative peut être utilisée pour s’assurer que les dépendances sont terminées avant de continuer. 

```json
    "dependsOn": [],
```

Pour plus d’informations, consultez [Définir les dépendances des ressources](../../azure-resource-manager/templates/define-resource-dependency.md#dependson).

## <a name="identity"></a>Identité

(Obligatoire) Pour qu’Image Builder soit autorisé à lire/écrire des images et à lire des scripts provenant du Stockage Azure, vous devez créer une identité affectée par l’utilisateur Azure disposant d’autorisations sur les différentes ressources. Pour plus d’informations sur le fonctionnement des autorisations d’Image Builder et sur les étapes pertinentes, consultez la [documentation](image-builder-user-assigned-identity.md).


```json
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<imgBuilderId>": {}
        }
        },
```


Prise en charge par Image Builder d’une identité affectée par l’utilisateur :
* Prise en charge d’une seule identité
* Non-prise en charge des noms de domaine personnalisés

Pour en savoir plus, consultez [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md).
Pour plus d’informations sur le déploiement de cette fonctionnalité, consultez [Configurer des identités managées pour ressources Azure sur une machine virtuelle Azure en utilisant Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity).

## <a name="properties-source"></a>Propriétés : source

La section `source` fournit des informations sur l’image source qui sera utilisée par le générateur d’images. Le générateur Image Builder prend en charge uniquement en mode natif la création d’images Hyper-V de génération 1 (Gen1) dans Azure Shared Image Gallery (SIG) ou Managed Image. Si vous souhaitez créer des images de génération 2 (Gen2), vous devez utiliser une image source Gen2 et la distribuer sur le disque dur virtuel. Après quoi, vous devez créer une image managée à partir du disque dur virtuel et l’injecter dans le SIG en tant qu’image Gen2.

L’API nécessite un « SourceType » qui définit la source pour la génération d’image. Il en existe actuellement trois types :
- PlatformImage : indique que l’image source est une image de la Place de marché.
- ManagedImag : utilisez cette option au démarrage à partir d’une image managée classique.
- SharedImageVersion : cette option s’applique lorsque vous utilisez une version d’image dans une galerie d’images partagées comme source.


> [!NOTE]
> Lors de l’utilisation d’images personnalisées Windows existantes, vous pouvez exécuter la commande Sysprep jusqu’à 8 fois sur une même image Windows. Pour plus d’informations, consultez la documentation [sysprep](/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation#limits-on-how-many-times-you-can-run-sysprep).

### <a name="platformimage-source"></a>Source PlatformImage 
Azure Image Builder prend en charge les images Windows Server et client, ainsi que les images de la Place de marché Azure pour Linux. Pour la liste complète, voir [ici](../windows/image-builder-overview.md#os-support). 

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "UbuntuServer",
                "sku": "18.04-LTS",
                "version": "latest"
        },
```


Ces propriétés sont les mêmes que celles utilisées pour créer des machines virtuelles, à l’aide d’AZ CLI. Exécutez la commande ci-dessous pour obtenir les propriétés : 
 
```azurecli-interactive
az vm image list -l westus -f UbuntuServer -p Canonical --output table –-all 
```

Vous pouvez utiliser ’latest’ dans la version ; la version est évaluée lors de la génération de l’image, et non lors de l’envoi du modèle. Si vous utilisez cette fonctionnalité avec la destination Galerie d’images partagées, vous pouvez éviter de soumettre à nouveau le modèle, puis réexécuter la génération de l’image par intervalles afin que vos images soient recréées à partir des images les plus récentes.

#### <a name="support-for-market-place-plan-information"></a>Prise en charge des informations sur les plans de la Place de marché
Vous pouvez également spécifier des informations de plan, par exemple :
```json
    "source": {
        "type": "PlatformImage",
        "publisher": "RedHat",
        "offer": "rhel-byos",
        "sku": "rhel-lvm75",
        "version": "latest",
        "planInfo": {
            "planName": "rhel-lvm75",
            "planProduct": "rhel-byos",
            "planPublisher": "redhat"
       }
```
### <a name="managedimage-source"></a>Source ManagedImage

Définit l’image source comme une image managée existante d’un disque dur virtuel généralisé ou d’une machine virtuelle.

> [!NOTE]
> L’image managée source doit provenir d’un système d’exploitation pris en charge et se trouver dans la même région que votre modèle Azure VM Image Builder. 

```json
        "source": { 
            "type": "ManagedImage", 
                "imageId": "/subscriptions/<subscriptionId>/resourceGroups/{destinationResourceGroupName}/providers/Microsoft.Compute/images/<imageName>"
        }
```

`imageId` doit être l’ID de ressource de l’image managée. Utilisez `az image list` pour répertorier les images disponibles.


### <a name="sharedimageversion-source"></a>Source SharedImageVersion
Définit l’image source comme une version d’image existante dans une galerie d’images partagées.

> [!NOTE]
> L’image managée source doit provenir d’un système d’exploitation pris en charge et se trouver dans la même région que votre modèle Azure VM Image Builder. Si ce n’est pas le cas, veuillez répliquer la version de l’image dans la région du modèle Image Builder.


```json
        "source": { 
            "type": "SharedImageVersion", 
            "imageVersionID": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/p  roviders/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageDefinitionName/versions/<imageVersion>" 
   } 
```

`imageVersionId` doit être l’ID de ressource de la version d’image. Utilisez [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list) pour répertorier les versions d’image.


## <a name="properties-buildtimeoutinminutes"></a>Propriétés : buildTimeoutInMinutes

Par défaut, Image Builder s’exécutera pendant 240 minutes. Après cela, il expire et s’arrête, que la génération de l’image soit terminée ou non. Si le délai d’expiration est atteint, une erreur semblable à celle-ci s’affiche :

```text
[ERROR] Failed while waiting for packerizer: Timeout waiting for microservice to
[ERROR] complete: 'context deadline exceeded'
```

Si vous ne spécifiez pas de valeur buildTimeoutInMinutes ou si vous lui affectez la valeur 0, la valeur par défaut est utilisée. Vous pouvez augmenter ou diminuer la valeur, jusqu’à la valeur maximale de 960 min (16 heures). Pour Windows, nous vous déconseillons de définir cette valeur en dessous de 60 minutes. Si vous constatez que vous avez atteint le délai d’expiration, consultez les [journaux](image-builder-troubleshoot.md#customization-log) pour voir si l’étape de personnalisation attend par exemple une entrée de l’utilisateur. 

Si vous avez besoin de plus de temps pour que les personnalisations se terminent, définissez ce dont vous avez besoin, avec une faible surcharge. Toutefois, ne définissez pas une valeur trop élevée, car vous devrez peut-être attendre qu’il expire avant d’afficher une erreur. 


## <a name="properties-customize"></a>Propriétés : personnaliser

Le générateur d’images prend en charge plusieurs « personnalisateurs ». Les personnalisateurs sont des fonctions utilisées pour personnaliser votre image, telles que l’exécution de scripts ou le redémarrage de serveurs. 

Lorsque vous utilisez `customize` : 
- Vous pouvez utiliser plusieurs personnalisateurs, mais ils doivent avoir une valeur `name` unique.
- Les personnalisateurs sont exécutés dans l’ordre spécifié dans le modèle.
- En cas d’échec d’un personnalisateur, l’ensemble du composant de personnalisation échoue et renvoie une erreur.
- Il est vivement recommandé de tester rigoureusement le script avant de l’utiliser dans un modèle. Le débogage du script sur votre propre machine virtuelle en sera simplifié.
- Ne placez pas de données sensibles dans les scripts. 
- Les emplacements de script doivent être accessibles publiquement, sauf si vous utilisez [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

```json
        "customize": [
            {
                "type": "Shell",
                "name": "<name>",
                "scriptUri": "<path to script>",
                "sha256Checksum": "<sha256 checksum>"
            },
            {
                "type": "Shell",
                "name": "<name>",
                "inline": [
                    "<command to run inline>",
                ]
            }

        ],
```     

 
La section de personnalisation est un tableau. Le générateur d’images Azure s’exécute via les personnalisateurs dans un ordre séquentiel. L’échec d’un personnalisateur entraîne un échec du processus de génération. 

> [!NOTE]
> Les commandes inline peuvent être affichées dans la définition du modèle d’image et par le Support Microsoft pour résoudre un cas de support. Les informations sensibles doivent être déplacées dans des scripts dans le Stockage Azure, où l’accès exige une authentification.
 
### <a name="shell-customizer"></a>Personnalisateur de l’interpréteur de commandes

Le personnalisateur de l’interpréteur de commandes prend en charge l’exécution de scripts shell. Ceux-ci doivent donc être accessibles publiquement pour que le générateur d’images puisse y accéder.

```json
    "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "scriptUri": "<link to script>",
            "sha256Checksum": "<sha256 checksum>"       
        }, 
    ], 
        "customize": [ 
        { 
            "type": "Shell", 
            "name": "<name>", 
            "inline": "<commands to run>"
        }, 
    ], 
```

Système d’exploitation pris en charge : Linux 
 
Propriétés de personnalisation :

- **type** - Shell 
- **name** - Nom pour le suivi de la personnalisation 
- **scriptUri** - URI vers l’emplacement du fichier 
- **inline** - Tableau de commandes d’interpréteur de commandes, séparées par des virgules.
- **sha256Checksum** - Valeur de la somme de contrôle sha256 du fichier. Vous la générez localement, puis Image Builder vérifie la somme de contrôle et valide.
    * Pour générer la somme de contrôle sha256, à l’aide d’un terminal sur Mac/Linux, exécutez la commande : `sha256sum <fileName>`


Pour que les commandes s’exécutent avec des privilèges de super utilisateur, elles doivent être précédées du préfixe `sudo`.

> [!NOTE]
> Les commandes inline sont stockées au sein de la définition du modèle d’image. Vous pouvez les voir lorsque vous videz la définition de l’image, et elles sont également accessibles au Support Microsoft dans le cadre de la résolution d’un cas de support. Il est fortement recommandé de déplacer les commandes et les valeurs sensibles dans des scripts et d’utiliser une identité d’utilisateur pour l’authentification auprès du Stockage Azure.

### <a name="windows-restart-customizer"></a>Personnalisateur de redémarrage Windows 
Le personnalisateur de redémarrage vous permet de redémarrer une machine virtuelle Windows et d’attendre qu’elle revienne en ligne, vous permettant ainsi d’installer un logiciel qui nécessite un redémarrage.  

```json 
     "customize": [ 

            {
                "type": "WindowsRestart",
                "restartCommand": "shutdown /r /f /t 0", 
                "restartCheckCommand": "echo Azure-Image-Builder-Restarted-the-VM  > c:\\buildArtifacts\\azureImageBuilderRestart.txt",
                "restartTimeout": "5m"
            }
  
        ],
```

Système d’exploitation pris en charge : Windows
 
Propriétés de personnalisation :
- **Type** : WindowsRestart
- **restartCommand** - Commande pour exécuter le redémarrage (facultatif). Par défaut, il s’agit de `'shutdown /r /f /t 0 /c \"packer restart\"'`.
- **restartCheckCommand** - Commande pour vérifier si le redémarrage a réussi (facultatif). 
- **restartTimeout** - Délai d’expiration de redémarrage spécifié sous forme de chaîne de magnitude et d’unité. Par exemple, `5m` (5 minutes) ou `2h` (2 heures). La valeur par défaut est : '5m'

### <a name="linux-restart"></a>Redémarrage de Linux  
Bien qu’il n’existe aucun personnalisateur de redémarrage de Linux, si vous installez des pilotes ou composants nécessitant un redémarrage, vous pouvez les installer et appeler un redémarrage à l’aide du personnalisateur de l’interpréteur de commandes. Il existe un délai d’expiration SSH de 20 minutes pour la machine virtuelle de build.

### <a name="powershell-customizer"></a>Personnalisateur PowerShell 
Le personnalisateur PowerShell prend en charge l’exécution de scripts PowerShell et de commande en ligne. Les scripts doivent être accessibles publiquement pour que le générateur d’images puisse y accéder.

```json 
     "customize": [
        { 
             "type": "PowerShell",
             "name":   "<name>",  
             "scriptUri": "<path to script>",
             "runElevated": "<true false>",
             "sha256Checksum": "<sha256 checksum>" 
        },  
        { 
             "type": "PowerShell", 
             "name": "<name>", 
             "inline": "<PowerShell syntax to run>", 
             "validExitCodes": "<exit code>",
             "runElevated": "<true or false>" 
         } 
    ], 
```

Système d’exploitation pris en charge : Windows et Linux

Propriétés de personnalisation :

- **type** - PowerShell.
- **scriptUri** - URI vers l’emplacement du fichier de script PowerShell. 
- **inline** - Commandes en ligne à exécuter, séparées par des virgules.
- **validExitCodes** - Facultatif. Des codes valides peuvent être retournés par le script/la commande en ligne, permettant ainsi d’éviter le signalement d’un échec du script/de la commande en ligne.
- **runElevated** - Facultatif, booléen avec prise en charge de l’exécution de commandes et de scripts avec des autorisations élevées.
- **sha256Checksum** - Valeur de la somme de contrôle sha256 du fichier. Vous la générez localement, puis Image Builder vérifie la somme de contrôle et valide.
    * Pour générer la somme de contrôle sha256, utiliser une commande PowerShell [Get-Hash](/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-6) sur Windows


### <a name="file-customizer"></a>Personnalisateur de fichier

Le personnalisateur de fichier permet aux générateurs d’images de télécharger un fichier d’un GitHub ou Stockage Azure. Si vous disposez d’un pipeline de build d’image qui s’appuie sur des artefacts, vous pouvez définir le personnalisateur de fichier à télécharger du partage de build et déplacer les artefacts dans l’image.  

```json
     "customize": [ 
         { 
            "type": "File", 
             "name": "<name>", 
             "sourceUri": "<source location>",
             "destination": "<destination>",
             "sha256Checksum": "<sha256 checksum>"
         }
     ]
```

Système d’exploitation pris en charge : Linux et Windows 

Propriétés du personnalisateur de fichier :

- **sourceUri** - Point de terminaison de stockage accessible, il peut s’agir de GitHub ou de Stockage Azure. Vous pouvez uniquement télécharger un fichier, pas un répertoire complet. Si vous devez télécharger un répertoire, utilisez un fichier compressé, puis décompressez-le à l’aide du personnalisateur de l’interpréteur de commandes ou PowerShell. 
- **destination** - Nom de destination complet et nom du fichier. Le chemin d’accès et les sous-répertoires référencés doivent exister, utilisez les personnalisateurs de l’interpréteur de commandes ou PowerShell pour les définir au préalable. Vous pouvez utiliser les personnalisateurs de script pour créer le chemin d’accès. 

Cela est pris en charge par les répertoires Windows et les chemins d’accès Linux, mais à quelques différences près : 
- Système d’exploitation Linux - Le seul chemin dans lequel le générateur d’images peut écrire est /tmp.
- Windows - Aucune restriction de chemin d’accès, mais le chemin d’accès doit exister.
 
 
Si une erreur se produit lors de la tentative de téléchargement du fichier, ou de son placement dans un répertoire spécifié, l’étape de personnalisation échoue, et sera consignée dans le fichier customization.log.

> [!NOTE]
> le Personnalisateur de fichier est uniquement adapté au téléchargement de fichiers de petite taille (moins de 20 Mo); Pour le téléchargement de fichiers plus volumineux, utilisez un script ou une commande incluse, le code d’utilisation pour télécharger des fichiers, tel que `wget` ou `curl` pour Linux et `Invoke-WebRequest` pour Windows.

Les fichiers dans le personnalisateur de fichier peuvent être téléchargés depuis le Stockage Azure à l’aide de [MSI](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage).

### <a name="windows-update-customizer"></a>Personnalisateur de Windows Update
Ce personnalisateur est basé sur [le provisionneur Windows Update de la communauté](https://packer.io/docs/provisioners/community-supported.html) pour Packer, un projet open source géré par la communauté Packer. Microsoft teste et valide le provisionneur à l’aide du service Image Builder et prend en charge l’examen des problèmes rencontrés, et travaille à la résolution des problèmes, mais le projet open source n’est pas officiellement pris en charge par Microsoft. Pour obtenir une documentation détaillée et une aide sur le provisionneur de Windows Update, consultez le référentiel du projet.

```json
     "customize": [
            {
                "type": "WindowsUpdate",
                "searchCriteria": "IsInstalled=0",
                "filters": [
                    "exclude:$_.Title -like '*Preview*'",
                    "include:$true"
                            ],
                "updateLimit": 20
            }
               ], 
OS support: Windows
```

Propriétés de personnalisation :
- **type** : WindowsUpdate.
- **searchCriteria** : facultatif, définit les types de mises à jour installées (recommandées, importantes, etc.), BrowseOnly = 0 et IsInstalled = 0 (recommandé) sont les valeurs par défaut.
- **filters** : facultatif, vous permet de spécifier un filtre pour inclure ou exclure des mises à jour.
- **updateLimit** : facultatif, définit le nombre de mises à jour pouvant être installées, par défaut 1 000.
 
> [!NOTE]
> Le personnalisateur de Windows Update peut échouer si des redémarrages de Windows sont en suspens ou si des installations d’applications sont en cours. En général, vous pouvez voir cette erreur dans le fichier customization.log, `System.Runtime.InteropServices.COMException (0x80240016): Exception from HRESULT: 0x80240016`. Nous vous recommandons vivement d’envisager l’ajout au redémarrage de Windows et/ou de laisser aux applications suffisamment de temps pour accomplir leurs installations à l’aide de la commande [sleep] ou de commandes d’attente (https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/start-sleep?view=powershell-7) ) dans les commandes ou scripts Inline avant d’exécuter Windows Update.

### <a name="generalize"></a>Généraliser 
Le générateur d’images Azure exécute également du code de « déprovisionnement » à la fin de chaque phase de personnalisation d’image, pour « généraliser » l’image. La généralisation est un processus dans lequel l’image est configurée pour pouvoir être réutilisée afin de créer plusieurs machines virtuelles. Pour les machines virtuelles Windows, le générateur d’images Azure utilise Sysprep. Pour Linux, le générateur d’images Azure exécute « waagent-deprovision ». 

Les commandes du générateur d’images utilisées pour généraliser peuvent ne pas convenir à chaque situation. Le générateur d’images Azure vous permet ainsi de personnaliser cette commande si nécessaire. 

Si vous migrez une personnalisation existante et que vous utilisez différentes commandes Sysprep/waagent, vous pouvez utiliser les commandes génériques du générateur d’images. De plus, si la création de la machine virtuelle échoue, utilisez vos propres commandes Sysprep ou waagent.

Si le générateur d’images Azure crée une image personnalisée Windows avec succès et que vous créez une machine virtuelle à partir de celle-ci, recherchez la machine virtuelle dont la création échoue ou ne se termine pas correctement. Vous devrez consulter la documentation de Windows Server Sysprep ou ouvrir une demande de support auprès de l’équipe du support technique du service client Windows Server Sysprep Services, qui peut résoudre les problèmes et vous conseiller sur l’utilisation correcte de Sysprep.


#### <a name="default-sysprep-command"></a>Commande Sysprep par défaut
```powershell
Write-Output '>>> Waiting for GA Service (RdAgent) to start ...'
while ((Get-Service RdAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureTelemetryService) to start ...'
while ((Get-Service WindowsAzureTelemetryService) -and ((Get-Service WindowsAzureTelemetryService).Status -ne 'Running')) { Start-Sleep -s 5 }
Write-Output '>>> Waiting for GA Service (WindowsAzureGuestAgent) to start ...'
while ((Get-Service WindowsAzureGuestAgent).Status -ne 'Running') { Start-Sleep -s 5 }
Write-Output '>>> Sysprepping VM ...'
if( Test-Path $Env:SystemRoot\system32\Sysprep\unattend.xml ) {
  Remove-Item $Env:SystemRoot\system32\Sysprep\unattend.xml -Force
}
& $Env:SystemRoot\System32\Sysprep\Sysprep.exe /oobe /generalize /quiet /quit
while($true) {
  $imageState = (Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\State).ImageState
  Write-Output $imageState
  if ($imageState -eq 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { break }
  Start-Sleep -s 5
}
Write-Output '>>> Sysprep complete ...'
```
#### <a name="default-linux-deprovision-command"></a>Commande de déprovisionnement Linux par défaut

```bash
/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync
```

#### <a name="overriding-the-commands"></a>Remplacement de commandes
Pour remplacer les commandes, utilisez les fournisseurs de script PowerShell ou de l’interpréteur de commandes pour créer les fichiers de commandes avec le nom de fichier exact, puis placez-les dans les répertoires appropriés :

* Windows : c:\DeprovisioningScript.ps1
* Linux : /tmp/DeprovisioningScript.sh

Le générateur d’images lira ces commandes, celles-ci sont écrites dans les journaux AIB, « customization.log ». Consultez la section de [dépannage](image-builder-troubleshoot.md#customization-log) pour savoir comment collecter les journaux.
 
## <a name="properties-distribute"></a>Propriétés : distribuer

Le générateur d’images Azure prend en charge trois cibles de distribution : 

- **managedImage** - Image managée.
- **sharedImage** - Galerie d’images partagées.
- **VHD** - Disque dur virtuel dans un compte de stockage.

Vous pouvez distribuer une image sur les deux types de cibles dans la même configuration.

> [!NOTE]
> La commande sysprep AIB par défaut n’inclut pas « /mode:vm », mais cela peut être nécessaire lors de la création d’images pour lesquelles le rôle HyperV est installé. Si vous devez ajouter cet argument de commande, vous devez écraser la commande sysprep.

Comme vous pouvez avoir plusieurs cibles sur lesquelles distribuer, le générateur d’images gère un état pour chaque cible de distribution accessible en interrogeant `runOutputName`.  `runOutputName` est un objet que vous pouvez interroger après la distribution pour plus d’informations sur cette distribution. Par exemple, vous pouvez interroger l’emplacement du disque dur virtuel, ou des régions dans lesquelles la version de l’image a été répliquée ou la version de l’image SIG créée. Il s’agit d’une propriété de chaque cible de distribution. `runOutputName` doit être unique pour chaque cible de distribution. Voici un exemple qui interroge une distribution de la Shared Image Gallery :

```bash
subscriptionID=<subcriptionID>
imageResourceGroup=<resourceGroup of image template>
runOutputName=<runOutputName>

az resource show \
        --ids "/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/$runOutputName"  \
        --api-version=2020-02-14
```

Sortie :
```json
{
  "id": "/subscriptions/xxxxxx/resourcegroups/rheltest/providers/Microsoft.VirtualMachineImages/imageTemplates/ImageTemplateLinuxRHEL77/runOutputs/rhel77",
  "identity": null,
  "kind": null,
  "location": null,
  "managedBy": null,
  "name": "rhel77",
  "plan": null,
  "properties": {
    "artifactId": "/subscriptions/xxxxxx/resourceGroups/aibDevOpsImg/providers/Microsoft.Compute/galleries/devOpsSIG/images/rhel/versions/0.24105.52755",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "rheltest",
  "sku": null,
  "tags": null,
  "type": "Microsoft.VirtualMachineImages/imageTemplates/runOutputs"
}
```

### <a name="distribute-managedimage"></a>Distribuer : managedImage

La sortie de l’image correspond à une ressource d’image managée.

```json
{
       "type":"managedImage",
       "imageId": "<resource ID>",
       "location": "<region>",
       "runOutputName": "<name>",
       "artifactTags": {
            "<name": "<value>",
            "<name>": "<value>"
        }
}
```
 
Propriétés de distribution :
- **type** - managedImage 
- **imageId** - ID de ressource de l’image de destination, format attendu : /subscriptions/\<subscriptionId>/resourceGroups/\<destinationResourceGroupName>/providers/Microsoft.Compute/images/\<imageName>
- **location** - Emplacement de l’image managée.  
- **runOutputName** - Nom unique d’identification de la distribution.  
- **artifactTags** - Facultatif, balises de paire de valeur de clé spécifiées par l’utilisateur.
 
 
> [!NOTE]
> Le groupe de ressources de destination doit exister.
> Si vous souhaitez distribuer l’image dans une autre région, le temps de déploiement est prolongé. 

### <a name="distribute-sharedimage"></a>Distribuer : sharedImage 
La galerie d’images partagées Azure est un nouveau service de gestion des images qui permet de gérer la réplication de la région d’image, le contrôle de version et le partage d’images personnalisées. Le générateur d’images Azure prend en charge la distribution avec ce service, vous pouvez donc distribuer des images dans des régions prises en charge par les galeries d’images partagées. 
 
Une galerie d’images partagées est constituée des éléments suivants : 
 
- Galerie : conteneur pour plusieurs images partagées. Une galerie est déployée dans une région.
- Définitions d’image : regroupement logique d’images. 
- Versions d’image : il s’agit d’un type d’image utilisé pour le déploiement d’une machine virtuelle ou d’un groupe identique. Des versions d’image peuvent être répliquées vers d’autres régions où des machines virtuelles doivent être déployées.
 
Avant de pouvoir distribuer dans la galerie d’images, vous devez créer une galerie et une définition d’image, consultez [Images partagées](../shared-images-cli.md). 

```json
{
    "type": "SharedImage",
    "galleryImageId": "<resource ID>",
    "runOutputName": "<name>",
    "artifactTags": {
        "<name>": "<value>",
        "<name>": "<value>"
    },
    "replicationRegions": [
        "<region where the gallery is deployed>",
        "<region>"
    ]
}
``` 

Propriétés de distribution de galeries d’images partagées :

- **type** - sharedImage  
- **galleryImageId** – ID de la galerie d’images partagées, qui peut être spécifiée dans deux formats :
    * Contrôle de version automatique – Image Builder génère automatiquement un numéro de version monotone, ce qui est utile pour continuer à reconstruire des images à partir du même modèle. Le format est `/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<sharedImageGalleryName>/images/<imageGalleryName>`.
    * Contrôle de version explicite – Vous pouvez transmettre le numéro de version qu’Image Builder devra utiliser. Le format est `/subscriptions/<subscriptionID>/resourceGroups/<rgName>/providers/Microsoft.Compute/galleries/<sharedImageGalName>/images/<imageDefName>/versions/<version e.g. 1.1.1>`.

- **runOutputName** - Nom unique d’identification de la distribution.  
- **artifactTags** - Facultatif, balises de paire de valeur de clé spécifiées par l’utilisateur.
- **replicationRegions** - Tableau de régions pour la réplication. Une des régions doit être la région où la galerie est déployée. L’ajout de régions entraîne une augmentation du temps de build, car le build ne se termine pas tant que la réplication n’est pas achevée.
- **excludeFromLatest** (facultatif) – Cette propriété permet d’indiquer que la version de l’image créée ne doit pas être utilisée comme dernière version dans la définition SIG. La valeur par défaut est « false ».
- **storageAccountType** (facultatif) – AIB permet de spécifier les types de stockage suivants pour la version de l’image à créer :
    * « Standard_LRS »
    * « Standard_ZRS »


> [!NOTE]
> Si le modèle d’image et la référence `image definition` ne se trouvent pas au même emplacement, la création des images prendra plus de temps. À l’heure actuelle, Image Builder ne comporte pas de paramètre `location` pour la ressource de version de l’image. Nous récupérons celui de la valeur `image definition` parente. Prenons par exemple une définition d’image se trouvant dans USA Ouest et supposons que vous souhaitiez répliquer la version de l’image vers USA Est : un objet blob est copié dans USA Ouest ; à partir de là, une ressource de version de l’image est créée dans USA Ouest, puis répliquée vers USA Est. Pour éviter le temps de réplication supplémentaire, veillez à ce que `image definition` et le modèle d’image se trouvent au même emplacement.


### <a name="distribute-vhd"></a>Distribuer : Disque dur virtuel (VHD)  
Vous pouvez générer sur un disque dur virtuel. Vous pouvez ensuite copier le disque dur virtuel et l’utiliser pour publier sur la Place de marché Azure ou l’utiliser avec Azure Stack.  

```json
{ 
    "type": "VHD",
    "runOutputName": "<VHD name>",
    "tags": {
        "<name": "<value>",
        "<name>": "<value>"
    }
}
```
 
Système d’exploitation pris en charge : Windows et Linux

Distribuer des paramètres de disque dur virtuel :

- **type** - Disque dur virtuel.
- **runOutputName** - Nom unique d’identification de la distribution.  
- **tags** - Facultatif, balises de paire de valeur de clé spécifiées par l’utilisateur.
 
Le générateur d’images Azure ne permet pas à l’utilisateur de spécifier un emplacement de compte de stockage, mais vous pouvez interroger l’état de `runOutputs` pour obtenir l’emplacement.  

```azurecli-interactive
az resource show \
   --ids "/subscriptions/$subscriptionId/resourcegroups/<imageResourceGroup>/providers/Microsoft.VirtualMachineImages/imageTemplates/<imageTemplateName>/runOutputs/<runOutputName>"  | grep artifactUri 
```

> [!NOTE]
> Lorsque le disque dur virtuel est créé, copiez-le dès que possible dans un autre emplacement. Le disque dur virtuel est stocké dans un compte de stockage dans le groupe de ressources temporaire créé lors de l’envoi du modèle d’image au service de générateur d’images Azure. Si vous supprimez le modèle d’image, vous perdez le disque dur virtuel. 

## <a name="image-template-operations"></a>Opérations de modèle d’image

### <a name="starting-an-image-build"></a>Lancement d’un build d’image
Pour lancer un build, vous devez appeler « Run » sur la ressource de modèle d’image. Voici des exemples de commandes `run` :

```PowerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Run -Force
```


```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Run 
```

### <a name="cancelling-an-image-build"></a>Annulation d’un build d’image
Si vous exécutez un build d’image qui vous semble incorrect, en attente d’une entrée d’utilisateur ou susceptible de ne jamais aboutir, vous pouvez l’annuler.

Le build peut être annulé à tout moment. Si la phase de distribution a commencé, l’annulation reste possible, mais vous devrez nettoyer toutes les images non terminées. La commande Cancel n’attend pas la fin de l’annulation. Surveillez la progression de l’annulation dans `lastrunstatus.runstate` à l’aide de ces [commandes](image-builder-troubleshoot.md#customization-log) d’état.


Exemples de commandes `cancel` :

```powerShell
Invoke-AzResourceAction -ResourceName $imageTemplateName -ResourceGroupName $imageResourceGroup -ResourceType Microsoft.VirtualMachineImages/imageTemplates -ApiVersion "2020-02-14" -Action Cancel -Force
```

```bash
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateLinux01 \
     --action Cancel 
```

## <a name="next-steps"></a>Étapes suivantes

Il existe des exemples de fichiers .json pour différents scénarios dans le [GitHub de générateur d’images Azure](https://github.com/danielsollondon/azvmimagebuilder).
