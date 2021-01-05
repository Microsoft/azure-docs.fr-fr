---
title: "Tutoriel : Effectuer un déploiement sur un réseau virtuel existant à l’aide d'Azure CLI - Module de sécurité matériel (HSM) dédié Azure | Microsoft Docs"
description: Tutoriel expliquant comment déployer un module de sécurité matériel (HSM) à l'aide d'Azure CLI sur un réseau virtuel existant
services: dedicated-hsm
documentationcenter: na
author: msmbaldwin
manager: rkarlin
editor: ''
ms.service: key-vault
ms.topic: tutorial
ms.custom: mvc, seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/20/2020
ms.author: mbaldwin
ms.openlocfilehash: b6f4610887092b1dac5cdc85622739318d5921d7
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96852232"
---
# <a name="tutorial-deploying-hsms-into-an-existing-virtual-network-using-the-azure-cli"></a>Tutoriel : Déploiement de modules HSM sur un réseau virtuel existant à l’aide d’Azure CLI

Le service HSM dédié d’Azure fournit un appareil physique à usage exclusif du client, avec un contrôle administratif complet et une responsabilité complète de la gestion. L’utilisation d’appareils physiques nécessite que Microsoft contrôle l’allocation des appareils afin de garantir que la capacité est gérée de manière efficace. Par conséquent, dans un abonnement Azure, le service HSM dédié n’est pas visible pour le provisionnement des ressources. Tous les clients Azure qui nécessitent un accès au service HSM dédié doivent d’abord contacter leur responsable de compte Microsoft afin d’effectuer une demande d’inscription au service HSM dédié. Ce n’est qu’une fois le processus terminé que le provisionnement est possible. 

Ce tutoriel montre un processus de provisionnement typique où le client :

- Dispose déjà d’un réseau virtuel
- Dispose d’une machine virtuelle
- Doit ajouter des ressources HSM dans l’environnement existant

Voici une architecture typique de déploiement multirégion et haute disponibilité :

![déploiement multirégion](media/tutorial-deploy-hsm-cli/high-availability-architecture.png)

Ce tutoriel concerne l’intégration d’une paire de modules de sécurité matériels (HSM) et l’intégration de la passerelle ExpressRoute nécessaire (voir le sous-réseau 1 ci-dessus) à un réseau virtuel existant (voir le réseau virtuel 1 ci-dessus).  Toutes les autres ressources sont des ressources Azure standard. Le même processus d’intégration peut être utilisé pour les modules HSM situés sur le sous-réseau 4 du réseau virtuel 3 ci-dessus.

## <a name="prerequisites"></a>Prérequis

Le service Azure HSM dédié n’est pas disponible dans le portail Azure. Toute interaction avec le service est effectuée via la ligne de commande ou PowerShell. Ce tutoriel utilise l’interface de ligne de commande (CLI) Azure Cloud Shell. Si vous débutez avec Azure CLI, suivez les instructions fournies ici : [Bien démarrer avec Azure CLI 2.0](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest&preserve-view=true).

Il est supposé que :

- Vous avez suivi le processus d’inscription du service HSM dédié Azure.
- Vous avez reçu l’approbation pour utiliser ce service. Si ce n’est pas le cas, contactez votre responsable de compte Microsoft.
- Vous avez créé un groupe de ressources pour ces ressources, et que les nouvelles ressources déployées dans ce tutoriel devront être ajoutées à ce groupe.
- Vous avez déjà créé le réseau virtuel, le sous-réseau et les machines virtuelles nécessaires d’après le diagramme ci-dessus, et que vous souhaitez intégrer deux modules HSM à ce déploiement.

Toutes les instructions ci-dessous supposent que vous avez déjà accédé au portail Azure et que vous avez ouvert Cloud Shell (sélectionnez \>\_ en haut à droite du portail).

## <a name="provisioning-a-dedicated-hsm"></a>Provisionnement d’un module HSM dédié

Le provisionnement des modules HSM et leur intégration à un réseau virtuel existant via la passerelle ExpressRoute seront vérifiés à l’aide de ssh. Cette vérification a pour but de garantir l’accessibilité et la disponibilité de base du module HSM en vue d’une configuration supplémentaire.

### <a name="validating-feature-registration"></a>Vérification de la fonctionnalité d’inscription

Comme mentionné plus haut, toute activité de provisionnement nécessite que le service HSM dédié soit inscrit dans votre abonnement. Pour vérifier si c’est le cas, exécutez les commandes suivantes dans le Cloud Shell du portail Azure.

```azurecli
az feature show \
   --namespace Microsoft.HardwareSecurityModules \
   --name AzureDedicatedHSM
```

Les commandes doivent retourner l’état « Registered » (Inscrit) comme indiqué ci-dessous. Si les commandes ne retournent pas « Registered », vous devez vous inscrire au service. Pour cela, contactez votre responsable de compte Microsoft.

![État de l’abonnement](media/tutorial-deploy-hsm-cli/subscription-status.png)

### <a name="creating-hsm-resources"></a>Création de ressources HSM

Avant de créer des ressources HSM, vous devez disposer de ressources prérequises. Vous devez disposer d’un réseau virtuel avec des plages de sous-réseau pour le calcul, les modules HSM et la passerelle. Les commandes suivantes sont des exemples permettant de créer ce réseau virtuel.

```azurecli
az network vnet create \
  --name myHSM-vnet \
  --resource-group myRG \
  --address-prefix 10.2.0.0/16 \
  --subnet-name compute \
  --subnet-prefix 10.2.0.0/24
```

```azurecli
az network vnet subnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name hsmsubnet \
  --address-prefixes 10.2.1.0/24 \
  --delegations Microsoft.HardwareSecurityModules/dedicatedHSMs
```

```azurecli
az network vnet subnet create \
  --vnet-name myHSM-vnet \
  --resource-group myRG \
  --name GatewaySubnet \
  --address-prefixes 10.2.255.0/26
```

>[!NOTE]
>La chose la plus importante à noter concernant la configuration du réseau virtuel est que le sous-réseau du module HSM doit avoir des délégations définies sur « Microsoft.HardwareSecurityModules/dedicatedHSMs ».  Si ce n’est pas le cas, le provisionnement HSM ne fonctionnera pas.

Après avoir configuré votre réseau, utilisez ces commandes Azure CLI pour provisionner vos HSM.

1. Utilisez la commande [az dedicated-hsm create](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_create) pour provisionner le premier HSM. Le HSM est nommé hsm1. Remplacez votre abonnement :

   ```azurecli
   az dedicated-hsm create --location westus --name hsm1 --resource-group myRG --network-profile-network-interfaces \
        /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet
   ```

   Ce déploiement prend environ 25 à 30 minutes, la majeure partie de ce temps étant consacré aux modules HSM.

1. Pour voir un HSM actuel, exécutez la commande [az dedicated-hsm show](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_show) :

   ```azurecli
   az dedicated-hsm show --resource group myRG --name hsm1
   ```

1. Provisionnez le deuxième HSM à l’aide de cette commande :

   ```azurecli
   az dedicated-hsm create --location westus --name hsm2 --resource-group myRG --network-profile-network-interfaces \
        /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/MyHSM-vnet/subnets/MyHSM-vnet
   ```

1. Exécutez la commande [az dedicated-hsm list](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_list) pour voir les détails de vos HSM actuels :

   ```azurecli
   az dedicated-hsm list --resource-group myRG
   ```

D’autres commandes peuvent être utiles. Utilisez la commande [az dedicated-hsm update](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_update) pour mettre à jour un HSM :

```azurecli
az dedicated-hsm update --resource-group myRG –name hsm1
```

Pour supprimer un HSM, utilisez la commande [az dedicated-hsm delete](/cli/azure/ext/hardware-security-modules/dedicated-hsm#ext_hardware_security_modules_az_dedicated_hsm_delete) :

```azurecli
az dedicated-hsm delete --resource-group myRG –name hsm1
```

## <a name="verifying-the-deployment"></a>Vérification du déploiement

Pour vérifier que les modules ont été provisionnés et pour voir les attributs des modules, exécutez le jeu de commandes suivant. Vérifiez que le groupe de ressources est défini correctement et que le nom de la ressource correspond exactement à celui du fichier de paramètres.

```azurecli
subid=$(az account show --query id --output tsv)
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM1
az resource show \
   --ids /subscriptions/$subid/resourceGroups/myRG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSM2
```

La sortie ressemble à ceci :

```json
{
    "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/HSMl",
    "identity": null,
    "kind": null,
    "location": "westus",
    "managedBy": null,
    "name": "HSM1",
    "plan": null,
    "properties": {
        "networkProfile": {
            "networkInterfaces": [
            {
            "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.Network/networkInterfaces/HSMl_HSMnic", "privatelpAddress": "10.0.2.5",
            "resourceGroup": "HSM-RG"
            }
            L
            "subnet": {
                "id": n/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/HSM-RG/providers/Microsoft.Network/virtualNetworks/demo-vnet/subnets/hsmsubnet", "resourceGroup": "HSM-RG"
            }
        },
        "provisioningState": "Succeeded",
        "stampld": "stampl",
        "statusMessage": "The Dedicated HSM device is provisioned successfully and ready to use."
    },
    "resourceGroup": "HSM-RG",
    "sku": {
        "capacity": null,
        "family": null,
        "model": null,
        "name": "SafeNet Luna Network HSM A790",
        "size": null,
        "tier": null
    },
    "tags": {
        "Environment": "prod",
        "resourceType": "Hsm"
    },
    "type": "Microsoft.HardwareSecurityModules/dedicatedHSMs"
}
```

Vous pouvez également voir les ressources à l’aide de l’[Explorateur de ressources Azure](https://resources.azure.com/).   Une fois dans l’Explorateur, développez « Abonnements » sur la gauche, votre abonnement pour HSM dédié, « Groupes de ressources », le groupe de ressources que vous avez utilisé, puis sélectionnez l’élément « Ressources ».

## <a name="testing-the-deployment"></a>Test du déploiement

Le test du déploiement consiste à se connecter à une machine virtuelle qui peut accéder aux modules HSM, puis à se connecter directement au module HSM. Ces actions permettent de vérifier que le module HSM est accessible.
L’outil SSH est utilisé pour la connexion à la machine virtuelle. La commande sera similaire à celle qui suit, sauf qu’elle contiendra le nom de l’administrateur et le nom DNS que vous avez spécifiés dans le paramètre.

`ssh adminuser@hsmlinuxvm.westus.cloudapp.azure.com`

L’adresse IP de la machine virtuelle peut également être utilisée à la place du nom DNS dans la commande ci-dessus. Si la commande réussit, vous êtes invité à entrer un mot de passe. Une fois connecté à la machine virtuelle, vous pouvez vous connecter au module HSM à l’aide de l’adresse IP privée située dans le portail pour la ressource d’interface réseau associée au module HSM.

![liste des composants](media/tutorial-deploy-hsm-cli/resources.png)

>[!NOTE]
>Notez la case à cocher « Afficher les types masqués » qui, quand elle est cochée, affiche les ressources HSM.

Dans la capture d’écran ci-dessus, un clic sur « HSM1_HSMnic » ou « HSM2_HSMnic » affiche l’adresse IP privée appropriée. Sinon, la commande `az resource show` utilisée ci-dessus permet également d’identifier l’adresse IP appropriée. 

Une fois que vous avez obtenu la bonne adresse IP, exécutez la commande suivante en remplaçant cette adresse :

`ssh tenantadmin@10.0.2.4`

En cas de réussite, un mot de passe vous est demandé. Le mot de passe par défaut est PASSWORD. Le module HSM vous invitera à changer votre mot de passe. Choisissez un mot de passe fort et utilisez le mécanisme choisi par votre organisation pour stocker le mot de passe et éviter toute perte de données.

>[!IMPORTANT]
>Si vous perdez ce mot de passe, le module HSM doit être réinitialisé, ce qui signifie que vous perdrez vos clés.

Lorsque vous êtes connecté au module HSM à l’aide de ssh, exécutez la commande suivante pour vérifier que le module est opérationnel.

`hsm show`

La sortie doit ressembler à ceci :

![Capture d’écran qui montre la sortie dans la fenêtre PowerShell.](media/tutorial-deploy-hsm-cli/hsm-show-output.png)

À ce stade, vous avez alloué toutes les ressources pour un déploiement haute disponibilité à deux HSM, et vous avez vérifié que l’accès était possible et que l’état était opérationnel. Toute configuration ou tests supplémentaires doivent être effectués au niveau du module HSM. Pour cela, vous devez suivre les instructions du chapitre 7 du guide d’administration Thales Luna Network HSM 7 pour initialiser le module HSM et créer des partitions. L’ensemble des logiciels et de la documentation peuvent être téléchargés directement sur le site Thales une fois que vous vous êtes inscrit dans le portail de support client Thales et que vous disposez d’un ID client. Téléchargez le logiciel client version 7.2 pour obtenir tous les composants nécessaires.

## <a name="delete-or-clean-up-resources"></a>Supprimer ou nettoyer des ressources

Si vous n’avez plus besoin du module HSM, vous pouvez le supprimer en tant que ressource, ce qui lui permettra de retourner au pool des modules libres. Lorsque vous supprimez un module, le problème évident qui se pose est celui des données client sensibles qui s’y trouvent. La meilleure façon de « réinitialiser » un appareil consiste à entrer un mot de passe d’administrateur HSM incorrect à trois reprises (remarque : il ne s’agit pas de l’administrateur d’appliance, mais de l’administrateur HSM). En guise de mesure de sécurité pour protéger le matériel de clé, l’appareil ne peut pas être supprimé en tant que ressource Azure tant qu’il n’est pas à l’état réinitialisé.

> [!NOTE]
> Si vous rencontrez un problème avec la configuration d’un module Thales, vous devez contacter le [support technique Thales](https://safenet.gemalto.com/technical-support/).

Si vous n’avez plus besoin des ressources du groupe de ressources, vous pouvez les supprimer toutes avec la commande suivante :

```azurecli
az group delete \
   --resource-group myRG \
   --name HSMdeploy \
   --verbose
```

## <a name="next-steps"></a>Étapes suivantes

Les étapes de ce tutoriel vous ont permis de provisionner des ressources HSM dédié. En outre, vous disposez désormais d’un réseau virtuel avec les modules HSM nécessaires et d’autres composants réseau permettant de communiquer avec le module HSM.  Vous pouvez maintenant compléter ce déploiement avec d’autres ressources, selon les besoins de votre architecture de déploiement. Pour plus d’informations sur la planification de votre déploiement, consultez les documents Concepts.
La conception recommandée comprend deux modules HSM dans une région primaire pour la disponibilité au niveau du rack, et deux modules HSM dans une région secondaire pour la disponibilité régionale. 

* [Haute disponibilité](high-availability.md)
* [Sécurité physique](physical-security.md)
* [Mise en réseau](networking.md)
* [Prise en charge](supportability.md)
* [Surveillance](monitoring.md)
