---
title: Vue d’ensemble des machines virtuelles Windows dans Azure
description: Vue d’ensemble des machines virtuelles Windows dans Azure.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: overview
ms.date: 11/14/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 26905b746efb0bfcc877e0fa8fad797672bb6447
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96483213"
---
# <a name="windows-virtual-machines-in-azure"></a>Machines virtuelles Windows dans Azure

Les Machines Virtuelles Azure sont l’un des nombreux types de [ressources informatiques évolutives et à la demande](/azure/architecture/guide/technology-choices/compute-decision-tree) proposés par Azure. En règle générale, une machine virtuelle est mieux adaptée à vos besoins si vous devez surtout améliorer le contrôle de votre environnement informatique. Cet article vous informe sur les points à prendre en compte avant de créer une machine virtuelle, sur sa création et sur sa gestion.

Une machine virtuelle Azure vous donne la flexibilité de la virtualisation sans que vous ayez à acheter le matériel physique qui exécute la machine virtuelle ni à en assurer la maintenance. Toutefois, vous devez toujours assurer la maintenance de la machine virtuelle en effectuant des tâches comme la configuration, la mise à jour corrective et l’installation des logiciels qui s’exécutent dessus.

Les Machines Virtuelles Azure peuvent être utilisées de différentes manières. Quelques exemples :

* **Développement et test** : les machines virtuelles Azure permettent de créer rapidement et facilement un ordinateur avec des configurations spécifiques requises pour encoder et tester une application.
* **Applications dans le cloud** : la demande de votre application étant susceptible de fluctuer, il peut être économique de l’exécuter sur une machine virtuelle dans Azure. Vous payez pour des machines virtuelles supplémentaires lorsque vous en avez besoin et vous les arrêtez le reste du temps.
* **Centre de données étendu** : les machines virtuelles au sein d’un réseau virtuel Azure peuvent être facilement connectées au réseau de votre organisation.

Le nombre de machines virtuelles utilisées par votre application peut varier (montée en puissance et augmentation de la charge) pour répondre à vos besoins.

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>À quoi dois-je penser avant de créer une machine virtuelle ?
Il existe toujours une multitude de [considérations liées à la conception](/azure/architecture/reference-architectures/n-tier/windows-vm) lorsque vous générez une infrastructure d’application dans Azure. Il est important de réfléchir à ces aspects des machines virtuelles avant de commencer :

* le nom de vos ressources d’application,
* l’emplacement de stockage des ressources,
* la taille de la machine virtuelle,
* le nombre maximal de machines virtuelles qui peuvent être créées,
* le système d’exploitation de la machine virtuelle,
* la configuration de la machine virtuelle après son démarrage
* et les ressources liées dont a besoin la machine virtuelle.

### <a name="locations"></a>Emplacements
Toutes les ressources créées dans Azure sont réparties sur plusieurs [régions géographiques](https://azure.microsoft.com/regions/) dans le monde. En règle générale, la région est appelée **emplacement** lorsque vous créez une machine virtuelle. Pour une machine virtuelle, l’emplacement spécifie l’endroit où les disques durs virtuels sont stockés.

Ce tableau présente quelques moyens d’obtenir la liste des emplacements disponibles.

| Méthode | Description |
| --- | --- |
| Portail Azure |Sélectionnez un emplacement dans la liste lorsque vous créez une machine virtuelle. |
| Azure PowerShell |Utilisez la commande [Get-AzLocation](/powershell/module/az.resources/get-azlocation). |
| API REST |Utilisez l’opération [Lister les emplacements](/rest/api/resources/subscriptions). |
| Azure CLI |Utilisez l’opération [az account list-locations](/cli/azure/account?view=azure-cli-latest). |

### <a name="singapore-data-residency"></a>Résidence des données à Singapour

Dans Azure, la fonctionnalité permettant le stockage de données client dans une seule région n’est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique. Pour toutes les autres régions, les données client sont stockées dans Zone géographique. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).

## <a name="availability"></a>Disponibilité
Azure a annoncé un contrat de niveau de service de pointe pour machine virtuelle à instance unique de 99,9 % à condition de déployer la machine virtuelle avec le stockage premium pour tous les disques.  Afin que votre déploiement puisse bénéficier du contrat de niveau de service standard de 99,95 % pour les machines virtuelles, vous devez déployer au moins deux machines virtuelles exécutant votre charge de travail à l’intérieur d’un groupe à haute disponibilité. Un groupe à haute disponibilité assure que vos machines virtuelles sont réparties sur plusieurs domaines d’erreur dans les centres de données Azure et déployées sur des hôtes ayant des fenêtres de maintenance distinctes. La version complète du [contrat SLA Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) explique la disponibilité garantie d’Azure dans son ensemble.


## <a name="vm-size"></a>Taille de la machine virtuelle
La [taille](../sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) de la machine virtuelle que vous utilisez est déterminée par la charge de travail que vous souhaitez exécuter. La taille que vous choisissez détermine ensuite des facteurs comme la puissance de traitement, la mémoire et la capacité de stockage. Azure propose différentes tailles vous permettant de prendre en charge de nombreux types d'utilisation.

Azure facture un [prix horaire](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) en fonction de la taille et du système d’exploitation de la machine virtuelle. Pour les heures partielles, Azure facture uniquement les minutes utilisées. Le stockage est facturé séparément.

## <a name="vm-limits"></a>Limites des machines virtuelles
Votre abonnement comporte des [limites de quota](../../azure-resource-manager/management/azure-subscription-service-limits.md) par défaut qui peuvent avoir un impact négatif sur le déploiement d’un grand nombre de machines virtuelles pour votre projet. La limite est de 20 machines virtuelles par région et par abonnement. Les limites peuvent être augmentées en [soumettant un ticket de support demandant leur hausse](../../azure-portal/supportability/resource-manager-core-quotas-request.md).

### <a name="operating-system-disks-and-images"></a>Images et disques du système d’exploitation
Les machines virtuelles utilisent des [disques durs virtuels](../managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) pour stocker leurs données et leur système d’exploitation (SE). Les disques durs virtuels sont également utilisés pour les images à partir desquelles vous pouvez choisir d'installer un système d'exploitation. 

Azure fournit de nombreuses [images Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps?filters=virtual-machine-images%3Bwindows&page=1) à utiliser avec différentes versions et différents types de systèmes d’exploitation Windows Server. Les images Marketplace sont identifiées par l’éditeur d’images, l’offre, la référence (SKU) et la version (la version est généralement spécifiée en dernier). Seuls les systèmes d’exploitation 64 bits sont pris en charge. Pour plus d’informations sur les rôles, fonctionnalités et systèmes d’exploitation invités pris en charge, consultez la rubrique [Prise en charge du logiciel serveur Microsoft pour les machines virtuelles Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

Ce tableau présente différents moyens de trouver les informations d’une image.

| Méthode | Description |
| --- | --- |
| Portail Azure |Les valeurs sont spécifiées automatiquement pour vous lorsque vous sélectionnez une image à utiliser. |
| Azure PowerShell |[Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) -Location *location*<BR>[Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) -Location *location* -Publisher *publisherName*<BR>[Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) -Location *location* -Publisher *publisherName* -Offer *offerName* |
| API REST |[Lister les éditeurs d’images](/rest/api/compute/platformimages/platformimages-list-publishers)<BR>[Lister les offres d’images](/rest/api/compute/platformimages/platformimages-list-publisher-offers)<BR>[Lister les références d’images](/rest/api/compute/platformimages/platformimages-list-publisher-offer-skus) |
| Azure CLI |[az vm image list-publishers](/cli/azure/vm/image?view=azure-cli-latest) --emplacement *location*<BR>[az vm image list-offers](/cli/azure/vm/image?view=azure-cli-latest) --emplacement *location* --éditeur *publisherName*<BR>[az vm image list-skus](/cli/azure/vm?view=azure-cli-latest) --emplacement *location* --éditeur *publisherName* --offre *offerName*|

Vous pouvez choisir de [charger et utiliser votre propre image](upload-generalized-managed.md) ; lorsque vous procédez ainsi, le nom de l’éditeur, l’offre et la référence ne sont pas utilisés.

### <a name="extensions"></a>Extensions
Les [extensions](../extensions/features-windows.md?toc=/azure/virtual-machines/windows/toc.json) de machines virtuelles étendent les fonctionnalités de votre machine virtuelle par le biais de la configuration post-déploiement et de tâches automatisées.

Ces tâches courantes peuvent être accomplies à l’aide des extensions :

* **Exécuter des scripts personnalisés** : [l’extension de script personnalisé](../extensions/custom-script-windows.md?toc=/azure/virtual-machines/windows/toc.json) vous permet de configurer des charges de travail sur la machine virtuelle en exécutant votre script pendant l’approvisionnement de la machine virtuelle.
* **Déployer et gérer des configurations** : [l’extension de configuration d’état souhaité (DSC) PowerShell](../extensions/dsc-overview.md?toc=/azure/virtual-machines/windows/toc.json) vous permet de configurer DSC sur une machine virtuelle pour gérer les environnements et les configurations.
* **Collecter les données de diagnostic** : [l’extension des diagnostics Azure](../extensions/diagnostics-template.md?toc=/azure/virtual-machines/windows/toc.json) vous permet de configurer la machine virtuelle de sorte qu’elle collecte des données de diagnostics utilisées pour surveiller l’intégrité de votre application.

### <a name="related-resources"></a>Ressources associées
Les ressources de cette table sont utilisées par la machine virtuelle et doivent exister ou être créées lors de sa création.

| Ressource | Obligatoire | Description |
| --- | --- | --- |
| [Groupe de ressources](../../azure-resource-manager/management/overview.md) |Oui |La machine virtuelle doit être contenue dans un groupe de ressources. |
| [Compte de stockage](../../storage/common/storage-account-create.md) |Oui |La machine virtuelle doit stocker ses disques durs virtuels dans le compte de stockage. |
| [Réseau virtuel](../../virtual-network/virtual-networks-overview.md) |Oui |La machine virtuelle doit faire partie d’un réseau virtuel. |
| [Adresse IP publique](../../virtual-network/public-ip-addresses.md) |Non |La machine virtuelle peut avoir une adresse IP publique pour être accessible à distance. |
| [Interface réseau](../../virtual-network/virtual-network-network-interface.md) |Oui |La machine virtuelle a besoin de l’interface réseau pour communiquer sur le réseau. |
| [Disques de données](attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) |Non |La machine virtuelle peut comprendre des disques de données pour développer ses capacités de stockage. |


## <a name="data-residency"></a>Résidence des données

Dans Azure, la fonctionnalité permettant le stockage de données client dans une seule région n’est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique et la région Brésil Sud (État de Sao Paulo) de la zone géographique Brésil. Pour toutes les autres régions, les données client sont stockées dans Zone géographique. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).


## <a name="next-steps"></a>Étapes suivantes

Créer votre première machine virtuelle

- [Portail](quick-create-portal.md)
- [PowerShell](quick-create-powershell.md)
- [Azure CLI](quick-create-cli.md)