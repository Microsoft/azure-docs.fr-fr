---
title: 'Tutoriel : Effectuer un déploiement sur un réseau virtuel existant à l’aide de PowerShell - Module de sécurité matériel (HSM) dédié Azure | Microsoft Docs'
description: Tutoriel expliquant comment déployer un module de sécurité matériel dédié (HSM) à l'aide de PowerShell sur un réseau virtuel existant
services: dedicated-hsm
documentationcenter: na
author: msmbaldwin
manager: rkarlin
editor: ''
ms.service: key-vault
ms.topic: tutorial
ms.custom: mvc, seodec18, devx-track-azurepowershell
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/14/2020
ms.author: johndaw
ms.openlocfilehash: ee431df89128a516e3a1cabeb43b5cbe9e356dae
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92927853"
---
# <a name="tutorial--deploying-hsms-into-an-existing-virtual-network-using-powershell"></a>Tutoriel : Déploiement de modules HSM sur un réseau virtuel existant à l’aide de PowerShell

Le service HSM dédié d’Azure fournit un appareil physique à usage exclusif du client, avec un contrôle administratif complet et une responsabilité complète de la gestion. Étant donné qu’il fournit du matériel physique, Microsoft doit contrôler la façon dont ces appareils sont alloués afin de vérifier que la capacité est gérée de manière efficace. Par conséquent, dans un abonnement Azure, le service HSM dédié n’est pas visible pour le provisionnement des ressources. Tous les clients Azure qui nécessitent un accès au service HSM dédié doivent d’abord contacter leur responsable de compte Microsoft afin d’effectuer une demande d’inscription au service HSM dédié. Ce n’est qu’une fois le processus terminé que le provisionnement est possible.
Ce tutoriel montre un processus de provisionnement typique où le client :

- Dispose déjà d’un réseau virtuel
- Dispose d’une machine virtuelle
- Doit ajouter des ressources HSM dans l’environnement existant

Voici une architecture typique de déploiement multirégion et haute disponibilité :

![déploiement multirégion](media/tutorial-deploy-hsm-powershell/high-availability.png)

Ce tutoriel concerne l’intégration d’une paire de modules de sécurité matériels (HSM) et l’intégration de la passerelle ExpressRoute nécessaire (voir le sous-réseau 1 ci-dessus) à un réseau virtuel existant (voir le réseau virtuel 1 ci-dessus).  Toutes les autres ressources sont des ressources Azure standard. Le même processus d’intégration peut être utilisé pour les modules HSM situés sur le sous-réseau 4 du réseau virtuel 3 ci-dessus.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Le service HSM dédié d’Azure n’est pas disponible dans le portail Azure. Par conséquent, toutes les interactions avec le service se font via la ligne de commande ou PowerShell. Ce tutoriel utilise PowerShell dans Azure Cloud Shell. Si vous débutez avec PowerShell, suivez les instructions fournies ici : [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

Il est supposé que :

- Vous avez suivi le processus d’inscription au service HSM dédié d’Azure et que vous avez reçu l’approbation nécessaire pour utiliser ce service. Si ce n’est pas le cas, contactez votre responsable de compte Microsoft. 
- Vous avez créé un groupe de ressources pour ces ressources, et que les nouvelles ressources déployées dans ce tutoriel devront être ajoutées à ce groupe.
- Vous avez déjà créé le réseau virtuel, le sous-réseau et les machines virtuelles nécessaires d’après le diagramme ci-dessus, et que vous souhaitez intégrer deux modules HSM à ce déploiement.

Toutes les instructions ci-dessous supposent que vous avez déjà ouvert le portail Azure ainsi que Cloud Shell (sélectionnez \>\_ en haut à droite du portail).

## <a name="provisioning-a-dedicated-hsm"></a>Provisionnement d’un module HSM dédié

Le provisionnement des modules HSM et leur intégration à un réseau virtuel existant via la passerelle ExpressRoute seront vérifiés à l’aide de l’outil en ligne de commande ssh afin de garantir l’accessibilité et la disponibilité de base du module HSM en vue d’une configuration supplémentaire. Les commandes suivantes utilisent un modèle Resource Manager pour créer les ressources HSM et les ressources réseau associées.

### <a name="validating-feature-registration"></a>Vérification de la fonctionnalité d’inscription

Comme mentionné plus haut, toute activité de provisionnement nécessite que le service HSM dédié soit inscrit dans votre abonnement. Pour vérifier si c’est le cas, exécutez la commande PowerShell suivante dans l’interpréteur de commandes cloud du portail Azure. 

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.HardwareSecurityModules -FeatureName AzureDedicatedHsm
```

Pour continuer, vous devez attendre que la commande retourne l’état « Registered » (Inscrit) comme indiqué ci-dessous.  Si vous n’êtes pas inscrit à ce service, contactez votre responsable de compte Microsoft.

![État de l’abonnement](media/tutorial-deploy-hsm-powershell/subscription-status.png)

### <a name="creating-hsm-resources"></a>Création de ressources HSM

Un modèle HSM est provisionné dans le réseau virtuel d’un client. Cela nécessite l’ajout d’un sous-réseau. Pour permettre la communication entre le réseau virtuel et l’appareil physique, vous devez utiliser une passerelle ExpressRoute. En outre, vous devez utiliser une machine virtuelle pour accéder au module HSM à l’aide du logiciel client Gemalto. Ces ressources ont été rassemblées dans un fichier de modèle, avec le fichier de paramètres correspondant, pour une plus grande facilité d’utilisation. Pour obtenir ces fichiers, contactez Microsoft directement à l’adresse HSMrequest@Microsoft.com.

Une fois que vous avez reçu ces fichiers, vous devez insérer les noms choisis pour vos ressources dans le fichier de paramètres. Les noms doivent être insérés là où la valeur est vide ("value": "").

- `namingInfix` Préfixe des noms de ressources HSM
- `ExistingVirtualNetworkName` Nom du réseau virtuel utilisé pour les modules HSM
- `DedicatedHsmResourceName1` Nom de la ressource HSM dans le tampon de centre de données 1
- `DedicatedHsmResourceName2` Nom de la ressource HSM dans le tampon de centre de données 2
- `hsmSubnetRange` Plage d’adresses IP du sous-réseau pour les modules HSM
- `ERSubnetRange` Plage d’adresses IP du sous-réseau pour la passerelle de réseau virtuel

Voici un exemple de ces modifications :

```json
{
"$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "namingInfix": {
      "value": "MyHSM"
    },
    "ExistingVirtualNetworkName": {
      "value": "MyHSM-vnet"
    },
    "DedicatedHsmResourceName1": {
      "value": "HSM1"
    },
    "DedicatedHsmResourceName2": {
      "value": "HSM2"
    },
    "hsmSubnetRange": {
      "value": "10.0.2.0/24"
    },
    "ERSubnetRange": {
      "value": "10.0.255.0/26"
    },
  }
}
```

Le fichier de modèle Resource Manager associé crée six ressources à partir des informations suivantes :

- Un sous-réseau pour les modules HSM dans le réseau virtuel spécifié
- Un sous-réseau pour la passerelle de réseau virtuel 
- Une passerelle de réseau virtuel qui connecte le réseau virtuel aux modules HSM
- Une adresse IP publique pour la passerelle
- Un module HSM dans le tampon 1
- Un module HSM dans le tampon 2

Une fois que les valeurs de paramètres sont définies, les fichiers doivent être chargés sur le partage de fichiers Cloud Shell du portail Azure en vue de leur utilisation. Dans le portail Azure, cliquez sur le symbole Cloud Shell « \>\_ » situé en haut à droite pour transformer la partie inférieure de l’écran en un environnement de commande. Les options de cet environnement sont BASH et PowerShell, et vous devez sélectionner BASH si ce n’est déjà fait.

Une option de chargement/téléchargement est disponible dans la barre d’outils de l’interface de commande. Vous devez l’utiliser pour charger le fichier de modèle et le fichier de paramètres sur votre partage de fichiers :

![Partage de fichiers](media/tutorial-deploy-hsm-powershell/file-share.png)

Une fois les fichiers chargés, vous êtes prêt à créer des ressources.
Avant de créer des ressources HSM, vous devez avoir mis en place certaines ressources. Vous devez disposer d’un réseau virtuel avec des plages de sous-réseau pour le calcul, les modules HSM et la passerelle. Les commandes suivantes sont des exemples permettant de créer ce réseau virtuel.

```powershell
$compute = New-AzVirtualNetworkSubnetConfig `
  -Name compute `
  -AddressPrefix 10.2.0.0/24
```

```powershell
$delegation = New-AzDelegation `
  -Name "myDelegation" `
  -ServiceName "Microsoft.HardwareSecurityModules/dedicatedHSMs"

```

```powershell
$hsmsubnet = New-AzVirtualNetworkSubnetConfig ` 
  -Name hsmsubnet ` 
  -AddressPrefix 10.2.1.0/24 ` 
  -Delegation $delegation 

```

```powershell

$gwsubnet= New-AzVirtualNetworkSubnetConfig `
  -Name GatewaySubnet `
  -AddressPrefix 10.2.255.0/26

```

```powershell

New-AzVirtualNetwork `
  -Name myHSM-vnet `
  -ResourceGroupName myRG `
  -Location westus `
  -AddressPrefix 10.2.0.0/16 `
  -Subnet $compute, $hsmsubnet, $gwsubnet

```

>[!NOTE]
>La chose la plus importante à noter concernant la configuration du réseau virtuel est que le sous-réseau du module HSM doit avoir des délégations définies sur « Microsoft.HardwareSecurityModules/dedicatedHSMs ».  Si ce n’est pas le cas, le provisionnement HSM ne fonctionnera pas.

Une fois que tous les prérequis sont réunis, exécutez la commande suivante pour utiliser le modèle Resource Manager, en vérifiant bien que vous avez remplacé les valeurs par vos noms uniques (au moins celui du groupe de ressources) :

```powershell

New-AzResourceGroupDeployment -ResourceGroupName myRG `
     -TemplateFile .\Deploy-2HSM-toVNET-Template.json `
     -TemplateParameterFile .\Deploy-2HSM-toVNET-Params.json `
     -Name HSMdeploy -Verbose

```

L’exécution de cette commande prend environ 20 minutes. L’option « -verbose » garantit que l’état sera affiché en permanence.

![état du provisionnement](media/tutorial-deploy-hsm-powershell/progress-status.png)

Lorsque l’exécution est terminée (ce qui est indiqué par "provisioningState": "Succeeded"), vous pouvez vous connecter à la machine virtuelle existante et utiliser SSH pour garantir la disponibilité du module HSM.

## <a name="verifying-the-deployment"></a>Vérification du déploiement

Pour vérifier que les modules ont été provisionnés et pour voir les attributs des modules, exécutez le jeu de commandes suivant. Vérifiez que le groupe de ressources est défini correctement et que le nom de la ressource correspond exactement à celui du fichier de paramètres.

```powershell

$subid = (Get-AzContext).Subscription.Id
$resourceGroupName = "myRG"
$resourceName = "HSM1"  
Get-AzResource -Resourceid /subscriptions/$subId/resourceGroups/$resourceGroupName/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/$resourceName

```

![état du provisionnement](media/tutorial-deploy-hsm-powershell/progress-status2.png)

Vous pouvez également voir les ressources à l’aide de l’[Explorateur de ressources Azure](https://resources.azure.com/).   Dans l’Explorateur, développez « Abonnements » sur la gauche, développez votre abonnement pour HSM dédié, développez « Groupes de ressources », développez le groupe de ressources que vous avez utilisé, puis sélectionnez l’élément « Ressources ».

## <a name="testing-the-deployment"></a>Test du déploiement

Le test du déploiement consiste à se connecter à une machine virtuelle qui peut accéder aux modules HSM, puis à se connecter directement au module HSM. Ces actions permettent de vérifier que le module HSM est accessible.
L’outil SSH est utilisé pour la connexion à la machine virtuelle. La commande sera similaire à celle qui suit, sauf qu’elle contiendra le nom de l’administrateur et le nom DNS que vous avez spécifiés dans le paramètre.

`ssh adminuser@hsmlinuxvm.westus.cloudapp.azure.com`

Le mot de passe à utiliser est celui du fichier de paramètres.
Une fois connecté à la machine virtuelle Linux, vous pouvez vous connecter au module HSM à l’aide de l’adresse IP privée fournie dans le portail pour la ressource \<prefix>hsm_vnic.

```powershell

(Get-AzResource -ResourceGroupName myRG -Name HSMdeploy -ExpandProperties).Properties.networkProfile.networkInterfaces.privateIpAddress

```
Une fois que vous disposez de l’adresse IP, exécutez la commande suivante :

`ssh tenantadmin@<ip address of HSM>`

En cas de réussite, un mot de passe vous est demandé. Le mot de passe par défaut est PASSWORD. Le module HSM vous invitera à changer votre mot de passe. Choisissez un mot de passe fort et utilisez le mécanisme choisi par votre organisation pour stocker le mot de passe et éviter toute perte de données.  

>[!IMPORTANT]
>Si vous perdez ce mot de passe, le module HSM doit être réinitialisé, ce qui signifie que vous perdrez vos clés.

Lorsque vous êtes connecté au module HSM à l’aide de ssh, exécutez la commande suivante pour vérifier que le module est opérationnel.

`hsm show`

La sortie doit ressembler à ceci :

![Capture d’écran montrant la sortie de la commande hsm show.](media/tutorial-deploy-hsm-powershell/output.png)

À ce stade, vous avez alloué toutes les ressources pour un déploiement haute disponibilité à deux HSM, et vous avez vérifié que l’accès était possible et que l’état était opérationnel. Toute configuration ou tests supplémentaires doivent être effectués au niveau du module HSM. Pour cela, vous devez suivre les instructions du chapitre 7 du guide d’administration Gemalto Luna Network HSM 7 pour initialiser le module HSM et créer des partitions. L’ensemble des logiciels et de la documentation peuvent être téléchargés directement sur le site Gemalto une fois que vous vous êtes inscrit dans le portail de support client Gemalto et que vous disposez d’un ID client. Téléchargez le logiciel client version 7.2 pour obtenir tous les composants nécessaires.

## <a name="delete-or-clean-up-resources"></a>Supprimer ou nettoyer des ressources

Si vous n’avez plus besoin du module HSM, vous pouvez le supprimer en tant que ressource, ce qui lui permettra de retourner au pool des modules libres. Lorsque vous supprimez un module, le problème évident qui se pose est celui des données client sensibles qui s’y trouvent. La meilleure façon de « réinitialiser » un appareil consiste à entrer un mot de passe d’administrateur HSM incorrect à trois reprises (remarque : il ne s’agit pas de l’administrateur d’appliance, mais de l’administrateur HSM). En guise de mesure de sécurité pour protéger le matériel de clé, l’appareil ne peut pas être supprimé en tant que ressource Azure tant qu’il n’est pas à l’état réinitialisé.

> [!NOTE]
> Si vous rencontrez un problème avec la configuration d’un module Gemalto, vous devez contacter le [support technique Gemalto](https://safenet.gemalto.com/technical-support/).

Si vous souhaitez supprimer la ressource HSM dans Azure, vous pouvez utiliser la commande suivante en remplaçant les variables « $ » par vos paramètres uniques :

```powershell

$subid = (Get-AzContext).Subscription.Id
$resourceGroupName = "myRG" 
$resourceName = "HSMdeploy"  
Remove-AzResource -Resourceid /subscriptions/$subId/resourceGroups/$resourceGroupName/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/$resourceName 

```

## <a name="next-steps"></a>Étapes suivantes

Les étapes décrites dans ce tutoriel vous ont permis de provisionner les ressources HSM dédié et de les rendre disponibles dans votre réseau virtuel. Vous pouvez maintenant compléter ce déploiement avec d’autres ressources, selon les besoins de votre architecture de déploiement. Pour plus d’informations sur la planification de votre déploiement, consultez les documents Concepts. La conception recommandée comprend deux modules HSM dans une région primaire pour la disponibilité au niveau du rack, et deux modules HSM dans une région secondaire pour la disponibilité régionale. Le fichier de modèle utilisé dans ce tutoriel peut facilement être utilisé comme base pour un déploiement à deux HSM. Toutefois, vous devez modifier ses paramètres pour répondre à vos besoins.

* [Haute disponibilité](high-availability.md)
* [Sécurité physique](physical-security.md)
* [Mise en réseau](networking.md)
* [Surveillance](monitoring.md)
* [Prise en charge](supportability.md)
