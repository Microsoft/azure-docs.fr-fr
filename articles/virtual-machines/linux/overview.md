---
title: Vue d’ensemble des machines virtuelles Linux dans Azure
description: Vue d’ensemble des machines virtuelles Linux dans Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: overview
ms.workload: infrastructure
ms.date: 11/14/2019
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 54982189a5da584c7daf66855ffb655e403a455a
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500510"
---
# <a name="linux-virtual-machines-in-azure"></a>Machines virtuelles Linux dans Azure

Les Machines Virtuelles Azure sont l’un des nombreux types de [ressources informatiques évolutives et à la demande](/azure/architecture/guide/technology-choices/compute-decision-tree) proposés par Azure. En règle générale, une machine virtuelle est mieux adaptée à vos besoins si vous devez surtout améliorer le contrôle de votre environnement informatique. Cet article vous informe sur les points à prendre en compte avant de créer une machine virtuelle, sur sa création et sur sa gestion.

Une machine virtuelle Azure vous donne la flexibilité de la virtualisation sans que vous ayez à acheter le matériel physique qui exécute la machine virtuelle ni à en assurer la maintenance. Toutefois, vous devez toujours assurer la maintenance de la machine virtuelle en effectuant des tâches comme la configuration, la mise à jour corrective et l’installation des logiciels qui s’exécutent dessus.

Les Machines Virtuelles Azure peuvent être utilisées de différentes manières. Quelques exemples :

* **Développement et test** : les machines virtuelles Azure permettent de créer rapidement et facilement un ordinateur avec des configurations spécifiques requises pour encoder et tester une application.
* **Applications dans le cloud** : la demande de votre application étant susceptible de fluctuer, il peut être économique de l’exécuter sur une machine virtuelle dans Azure. Vous payez pour des machines virtuelles supplémentaires lorsque vous en avez besoin et vous les arrêtez le reste du temps.
* **Centre de données étendu** : les machines virtuelles au sein d’un réseau virtuel Azure peuvent être facilement connectées au réseau de votre organisation.

Le nombre de machines virtuelles utilisées par votre application peut varier (montée en puissance et augmentation de la charge) pour répondre à vos besoins.

## <a name="what-do-i-need-to-think-about-before-creating-a-vm"></a>À quoi dois-je penser avant de créer une machine virtuelle ?
Il existe toujours une multitude de [considérations liées à la conception](/azure/architecture/reference-architectures/n-tier/linux-vm) lorsque vous générez une infrastructure d’application dans Azure. Il est important de réfléchir à ces aspects des machines virtuelles avant de commencer :

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
La [taille](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) de la machine virtuelle que vous utilisez est déterminée par la charge de travail que vous souhaitez exécuter. La taille que vous choisissez détermine ensuite des facteurs comme la puissance de traitement, la mémoire et la capacité de stockage. Azure propose différentes tailles vous permettant de prendre en charge de nombreux types d'utilisation.

Azure facture un [prix horaire](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) en fonction de la taille et du système d’exploitation de la machine virtuelle. Pour les heures partielles, Azure facture uniquement les minutes utilisées. Le stockage est facturé séparément.

## <a name="vm-limits"></a>Limites des machines virtuelles
Votre abonnement comporte des [limites de quota](../../azure-resource-manager/management/azure-subscription-service-limits.md) par défaut qui peuvent avoir un impact négatif sur le déploiement d’un grand nombre de machines virtuelles pour votre projet. La limite est de 20 machines virtuelles par région et par abonnement. Les limites peuvent être augmentées en [soumettant un ticket de support demandant leur hausse](../../azure-portal/supportability/resource-manager-core-quotas-request.md).

## <a name="managed-disks"></a>Disques managés

La fonctionnalité Disques managés se charge de la création et de la gestion du compte de stockage Azure en arrière-plan, éliminant les préoccupations liées aux limites d’extensibilité du compte de stockage. Vous spécifiez la taille du disque et le niveau de performances (Standard ou Premium) et Azure crée et gère le disque. Lorsque vous ajoutez des disques ou faites monter ou descendre en puissance la machine virtuelle, vous n’avez pas à vous soucier du stockage utilisé. Si vous créez de nouvelles machines virtuelles, [utilisez Azure CLI](quick-create-cli.md) ou le Portail Azure pour créer des machines virtuelles avec des disques de système d’exploitation et de données managés. Si vous avez des machines virtuelles qui utilisent des disques non managés, vous pouvez [convertir vos machines virtuelles pour qu’elles soient sauvegardées avec la fonctionnalité Disques managés](convert-unmanaged-to-managed-disks.md).

Vous pouvez également gérer vos images personnalisées dans un compte de stockage par région Azure et les utiliser pour créer des centaines de machines virtuelles dans le même abonnement. Pour plus d’informations sur la fonctionnalité Disques managés, consultez [Vue d’ensemble de la fonctionnalité Disques managés](../managed-disks-overview.md).

## <a name="distributions"></a>Distributions 
Microsoft Azure prend en charge un certain nombre de distributions Linux populaires fournies et gérées par plusieurs partenaires.  Vous pouvez trouver les distributions disponibles dans la Place de marché Azure. Microsoft travaille activement avec différentes communautés Linux pour enrichir davantage la liste des [distributions Linux approuvées par Azure](endorsed-distros.md).

Si votre distribution Linux préférée n’est pas présente dans la galerie, vous pouvez « apporter votre propre machine virtuelle Linux » en [créant et chargeant un disque dur virtuel dans Azure](create-upload-generic.md).

Microsoft travaille en étroite collaboration avec des partenaires afin de garantir que les images disponibles sont mises à jour et optimisées pour un runtime Azure.  Pour plus d’informations sur les offres des partenaires Azure, consultez les liens suivants :

* Linux sur Azure : [Distributions approuvées](endorsed-distros.md)
* SUSE : [Place de marché Azure - SUSE Linux Enterprise Server](https://azuremarketplace.microsoft.com/marketplace/apps?page=1&search=suse)
* Red Hat : [Place de marché Azure - Red Hat Enterprise Linux](https://azuremarketplace.microsoft.com/marketplace/apps?search=Red%20Hat%20Enterprise%20Linux)
* Canonical - [Place de marché Azure - Ubuntu Server](https://azuremarketplace.microsoft.com/marketplace/apps/Canonical.UbuntuServer)
* Debian : [Place de marché Azure - Debian](https://azuremarketplace.microsoft.com/marketplace/apps?search=Debian&page=1)
* FreeBSD : [Place de marché Azure - FreeBSD](https://azuremarketplace.microsoft.com/marketplace/apps?search=freebsd&page=1)
* Flatcar : [Place de marché Azure - Flatcar Container Linux](https://azuremarketplace.microsoft.com/marketplace/apps?search=Flatcar&page=1)
* RancherOS - [Place de marché Azure - RancherOS](https://azuremarketplace.microsoft.com/marketplace/apps/rancher.rancheros)
* Bitnami - [Bitnami Library pour Azure](https://azure.bitnami.com/)
* Mesosphere - [Place de marché Azure - Mesosphere DC/OS sur Azure](https://azure.microsoft.com/services/kubernetes-service/mesosphere/)
* Docker - [Place de marché Azure - Images Docker](https://azuremarketplace.microsoft.com/marketplace/apps?search=docker&page=1&filters=virtual-machine-images)
* Jenkins - [Place de marché Azure - CloudBees Jenkins Platform](https://azuremarketplace.microsoft.com/marketplace/apps/cloudbees.cloudbees-core-contact)


## <a name="cloud-init"></a>Cloud-Init 

Pour obtenir une culture DevOps appropriée, toute l’infrastructure doit être du code.  Lorsque l’ensemble de l’infrastructure se trouve dans le code, elle est facile à recréer.  Azure fonctionne avec tous les principaux outils d’automatisation, comme Ansible, Chef, SaltStack et Puppet.  Azure propose également ses propres outils pour l’automatisation :

* [Les modèles Azure](create-ssh-secured-vm-from-template.md)
* [Azure `VMaccess`](../extensions/vmaccess.md)

Azure prend en charge [cloud-init](https://cloud-init.io/) sur la plupart des distributions Linux qui le prennent en charge.  Nous travaillons activement avec nos partenaires de distribution Linux afin de mettre des images compatibles cloud-init à disposition sur la Place de marché Azure. Ces images permettent à vos déploiements et configurations cloud-init de fonctionner de manière fluide avec des machines virtuelles et des groupes de machines virtuelles identiques.

* [À l’aide de cloud-init sur les machines virtuelles Linux Azure](using-cloud-init.md)

## <a name="storage"></a>Stockage
* [Introduction à Microsoft Azure Storage](../../storage/common/storage-introduction.md)
* [Ajouter un disque à une machine virtuelle Linux avec l’interface de ligne de commande Azure](add-disk.md)
* [Attachement d’un disque de données à une machine virtuelle Linux dans le portail Azure](attach-disk-portal.md)

## <a name="networking"></a>Mise en réseau
* [Présentation du réseau virtuel.](../../virtual-network/virtual-networks-overview.md)
* [Adresses IP dans Azure](../../virtual-network/public-ip-addresses.md)
* [Ouverture de ports sur une machine virtuelle Linux dans Azure](nsg-quickstart.md)
* [Créer un nom de domaine complet dans le portail Azure](../create-fqdn.md)


## <a name="data-residency"></a>Résidence des données

Dans Azure, la fonctionnalité permettant le stockage de données client dans une seule région n’est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique et la région Brésil Sud (État de Sao Paulo) de la zone géographique Brésil. Pour toutes les autres régions, les données client sont stockées dans Zone géographique. Pour plus d’informations, consultez le [Centre de gestion de la confidentialité](https://azuredatacentermap.azurewebsites.net/).


## <a name="next-steps"></a>Étapes suivantes

Créer votre première machine virtuelle

- [Portail](quick-create-portal.md)
- [Azure CLI](quick-create-cli.md)
- [PowerShell](quick-create-powershell.md)