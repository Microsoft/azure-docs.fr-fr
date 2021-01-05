---
title: Azure Hybrid Benefit et machines virtuelles Linux
description: Découvrez comment Azure Hybrid Benefit peut vous aider à réaliser des économies sur vos machines virtuelles Linux s’exécutant sur Azure.
services: virtual-machines
documentationcenter: ''
author: mathapli
manager: westonh
ms.service: virtual-machines-linux
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 09/22/2020
ms.author: mathapli
ms.openlocfilehash: 1bc108f76ac35b13474de18d473f5728dbad9d23
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560014"
---
# <a name="how-azure-hybrid-benefit-applies-for-linux-virtual-machines"></a>Application d’Azure Hybrid Benefit aux machines virtuelles Linux

Azure Hybrid Benefit est un avantage de licence qui vous permet de réduire considérablement les coûts d’exploitation de vos machines virtuelles Red Hat Enterprise Linux (RHEL) et SUSE Linux Enterprise Server (SLES) dans le cloud. Grâce à cet avantage, vous payez uniquement les coûts d’infrastructure de votre machine virtuelle, car votre abonnement RHEL ou SLES couvre les frais liés aux logiciels. L’avantage s’applique à toutes les images PAYG (paiement à l’accès) de la place de marché RHEL et SLES.

Azure Hybrid Benefit pour machines virtuelles Linux est désormais accessible au public.

## <a name="benefit-description"></a>Description de l’avantage

Azure Hybrid Benefit vous permet de migrer vos serveurs locaux RHEL et SLES vers Azure en convertissant les machines virtuelles PAYG RHEL et SLES existantes sur Azure pour la facturation BYOS (Apportez votre propre abonnement). En règle générale, les machines virtuelles déployées à partir d’images PAYG sur Azure génèrent à la fois des frais d’infrastructure et des frais de logiciel. Avec Azure Hybrid Benefit, les machines virtuelles PAYG peuvent être converties en modèle de facturation BYOS sans redéploiement, ce qui vous permet d’éviter tout risque de temps d’arrêt.

:::image type="content" source="./media/ahb-linux/azure-hybrid-benefit-cost.png" alt-text="Visualisation des coûts Azure Hybrid Benefit sur les machines virtuelles Linux.":::

Après avoir activé l’avantage sur une machine virtuelle RHEL ou SLES, vous ne serez plus facturé pour les frais de logiciel supplémentaires généralement encourus sur une machine virtuelle PAYG. Au lieu de cela, votre machine virtuelle commencera à accumuler des frais BYOS, qui comprennent uniquement les frais de matériel de calcul, et non les frais de logiciel.

Vous pouvez également choisir de convertir une machine virtuelle sur laquelle l’avantage a été activé pour revenir à un modèle de facturation PAYG.

## <a name="scope-of-azure-hybrid-benefit-eligibility-for-linux-vms"></a>Étendue de l’éligibilité Azure Hybrid Benefit pour les machines virtuelles Linux

Azure Hybrid Benefit est disponible pour toutes les images PAYG RHEL et SLES de Place de marché Azure. L’avantage n’est pas encore disponible pour les images BYOS RHEL ou SLES ou pour les images personnalisées de Place de marché Azure.

Les instances réservées, les instances d’Azure Dedicated Host et les avantages hybrides SQL ne sont pas éligibles à Azure Hybrid Benefit si vous utilisez déjà l’avantage avec des machines virtuelles Linux.

## <a name="get-started"></a>Bien démarrer

### <a name="red-hat-customers"></a>Clients Red Hat

Azure Hybrid Benefit pour RHEL est disponible pour les clients Red Hat qui répondent à ces deux critères :

- Avoir des abonnements RHEL actifs ou inutilisés qui peuvent être utilisés dans Azure
- Avoir activé un ou plusieurs de ces abonnements pour une utilisation dans Azure avec le programme [Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access)

Pour commencer à utiliser l’avantage pour Red Hat :

1. Activez un ou plusieurs de vos abonnements RHEL éligibles pour être utilisés dans Azure à l’aide de l’[interface client de Red Hat Cloud Access](https://access.redhat.com/management/cloud).

   Les abonnements Azure que vous fournissez pendant le processus d’activation de Red Hat Cloud Access seront alors autorisés à utiliser la fonctionnalité Azure Hybrid Benefit.
1. Appliquez Azure Hybrid Benefit à l’une de vos machines virtuelles PAYG RHEL existantes et à toutes les nouvelles machines virtuelles RHEL que vous déployez à partir d’images PAYG de Place de marché Azure.
1. Effectuez les [étapes recommandées suivantes](https://access.redhat.com/articles/5419341) pour configurer des sources de mise à jour pour les machines virtuelles RHEL et tenir compte des consignes de conformité des abonnements RHEL.


### <a name="suse-customers"></a>Clients SUSE

Pour commencer à utiliser l’avantage pour SUSE :

1. Inscrivez-vous au programme de cloud public SUSE.
1. Appliquez l’avantage à vos machines virtuelles existantes via Azure CLI.
1. Inscrivez vos machines virtuelles recevant l’avantage auprès d’une source de mises à jour distincte.


## <a name="enable-and-disable-the-benefit-in-the-azure-cli"></a>Activez et désactivez l’avantage dans Azure CLI.

Vous pouvez utiliser la commande `az vm update` pour mettre à jour des machines virtuelles existantes. Pour les machines virtuelles RHEL, exécutez la commande avec le paramètre `--license-type` défini sur `RHEL_BYOS`. Pour les machines virtuelles SLES, exécutez la commande avec le paramètre `--license-type` défini sur `SLES_BYOS`.

### <a name="cli-example-to-enable-the-benefit"></a>Exemple CLI pour activer l’avantage
```azurecli
# This will enable the benefit on a RHEL VM
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS

# This will enable the benefit on a SLES VM
az vm update -g myResourceGroup -n myVmName --license-type SLES_BYOS
```
### <a name="cli-example-to-disable-the-benefit"></a>Exemple CLI pour désactiver l’avantage
Pour désactiver l’avantage, utilisez la valeur `None` pour `--license-type` :

```azurecli
# This will disable the benefit on a VM
az vm update -g myResourceGroup -n myVmName --license-type None
```

### <a name="cli-example-to-enable-the-benefit-on-a-large-number-of-vms"></a>Exemple CLI pour activer l’avantage sur un grand nombre de machines virtuelles
Pour activer l’avantage sur un grand nombre de machines virtuelles, vous pouvez utiliser le paramètre `--ids` dans Azure CLI :

```azurecli
# This will enable the benefit on a RHEL VM. In this example, ids.txt is an
# existing text file that contains a delimited list of resource IDs corresponding
# to the VMs using the benefit
az vm update -g myResourceGroup -n myVmName --license-type RHEL_BYOS --ids $(cat ids.txt)
```

Les exemples suivants illustrent deux méthodes d’obtention d’une liste d’ID de ressource : l’une au niveau du groupe de ressources, l’autre au niveau de l’abonnement.

```azurecli
# To get a list of all the resource IDs in a resource group:
$(az vm list -g MyResourceGroup --query "[].id" -o tsv)

# To get a list of all the resource IDs of VMs in a subscription:
az vm list -o json | jq '.[] | {VMName: .name, ResourceID: .id}'
```

## <a name="apply-the-azure-hybrid-benefit-at-vm-create-time"></a>Appliquer Azure Hybrid Benefit au moment de la création de la machine virtuelle
En plus d’appliquer Azure Hybrid Benefit aux machines virtuelles de paiement à l’utilisation existantes, vous pouvez appeler l’avantage au moment de la création de la machine virtuelle. Les avantages de cette méthode sont triples :
- Vous pouvez approvisionner des machines virtuelles PAYG et BYOS en utilisant la même image et le même processus.
- Cela permet de changer de mode de licence à l’avenir, ce qui n’est pas possible avec une image BYOS uniquement ou si vous apportez votre propre machine virtuelle.
- Par défaut, la machine virtuelle sera connectée à Red Hat Update Infrastructure (RHUI) afin de garantir qu’elle reste à jour et sécurisée. Vous pouvez modifier le mécanisme de mise à jour à tout moment après le déploiement.

## <a name="check-the-azure-hybrid-benefit-status-of-a-vm"></a>Vérifier l’état Azure Hybrid Benefit d’une machine virtuelle
Vous pouvez afficher l’état Azure Hybrid Benefit d’une machine virtuelle à l’aide d’Azure CLI ou d’Azure Instance Metadata Service.

### <a name="azure-cli"></a>Azure CLI

Vous pouvez utiliser la commande `az vm get-instance-view` à cet effet. Recherchez un champ `licenseType` dans la réponse. Si le champ `licenseType` existe et que la valeur est `RHEL_BYOS` ou `SLES_BYOS`, l’avantage est activé sur votre machine virtuelle.

```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVm
```

### <a name="azure-instance-metadata-service"></a>Service de métadonnées d’instance Azure

À partir de la machine virtuelle elle-même, vous pouvez interroger les métadonnées attestées dans Azure Instance Metadata Service pour déterminer la valeur de la propriété `licenseType` de la machine virtuelle. Une propriété `licenseType` ayant la valeur `RHEL_BYOS` ou `SLES_BYOS` indique que l’avantage est activé sur votre machine virtuelle. [En savoir plus sur les métadonnées attestées](./instance-metadata-service.md#attested-data).

## <a name="compliance"></a>Conformité

### <a name="red-hat"></a>Red Hat

Les clients qui utilisent Azure Hybrid Benefit pour RHEL acceptent les [conditions légales](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Cloud_Software_Subscription_Agreement_for_Microsoft_Azure.pdf) standard et la [déclaration de confidentialité](http://www.redhat.com/licenses/cloud_CSSA/Red_Hat_Privacy_Statement_for_Microsoft_Azure.pdf) associées aux offres RHEL de Place de marché Azure.

Les clients utilisant Azure Hybrid Benefit pour RHEL peuvent fournir des mises à jour et des patchs de logiciel à ces machines virtuelles de trois façons :

- [Red Hat Update Infrastructure](../workloads/redhat/redhat-rhui.md) (option par défaut)
- Red Hat Satellite Server
- Red Hat Subscription Manager

Les clients qui choisissent l’option RHUI peuvent continuer à utiliser RHUI comme principale source de mise à jour pour leurs machines virtuelles RHEL Azure Hybrid Benefit sans avoir à joindre d’abonnements RHEL à ces machines virtuelles. Les clients qui choisissent l’option RHUI sont tenus de veiller à la conformité de l’abonnement RHEL.

Les clients qui choisissent Red Hat Satellite Server ou Red Hat Subscription Manager doivent supprimer la configuration RHUI, puis joindre un abonnement RHEL avec Cloud Access à leurs machines virtuelles RHEL Azure Hybrid Benefit.  

Pour plus d’informations sur la conformité des abonnements Red Hat, les mises à jour de logiciel et les sources des machines virtuelles RHEL Azure Hybrid Benefit, consultez [l’article Red Hat relatif à l’utilisation des abonnements RHEL avec Azure Hybrid Benefit](https://access.redhat.com/articles/5419341).

### <a name="suse"></a>SUSE

Pour utiliser Azure Hybrid Benefit pour vos machines virtuelles SLES, vous devez d’abord être inscrit au [programme de cloud public SUSE](https://www.suse.com/media/guide/suse_public_cloud_service_provider_program_overview.pdf). Une fois que vous avez acheté des abonnements SUSE, vous devez inscrire vos machines virtuelles qui utilisent ces abonnements auprès de votre propre source de mises à jour. Utilisez le centre de clients SUSE, le serveur Subscription Management Tool ou SUSE Manager pour cette inscription.

## <a name="frequently-asked-questions"></a>Forum aux questions
*Q : Puis-je utiliser un type de licence `RHEL_BYOS` avec une image SLES ou inversement ?*

A : Non, c’est impossible. Si vous essayez d’entrer un type de licence ne correspondant pas à la distribution en cours d’exécution sur votre machine virtuelle, les métadonnées de facturation ne seront pas mises à jour. Toutefois, si vous entrez accidentellement un type de licence incorrect, la mise à jour de votre machine virtuelle vers le type de licence qui convient activera l’avantage.

*Q : Je me suis inscrit au programme Red Hat Cloud Access, mais je ne peux toujours pas activer l’avantage sur mes machines virtuelles RHEL. Que dois-je faire ?*

A : L’inscription au programme Red Hat Cloud Access peut mettre un certain temps à se propager entre Red Hat et Azure. Si l’erreur persiste après un jour ouvrable, contactez le support technique de Microsoft.

*Q : J’ai déployé une machine virtuelle à l’aide d’une image de référence BYOS RHEL. Puis-je convertir la facturation de ces images de BYOS à PAYG ?*

A : Non, c’est impossible. Azure Hybrid Benefit prend en charge la conversion uniquement sur les images de paiement à l’utilisation.

*Q : J’ai chargé ma propre image RHEL à partir d’un dossier local (via Azure Migrate, Azure Site Recovery ou autre) vers Azure. Puis-je convertir la facturation de ces images de BYOS à PAYG ?*

A : Non, c’est impossible. La capacité Azure Hybrid Benefit est actuellement disponible uniquement pour les images RHEL et SLES de Place de marché Azure. 

*Q : J’ai chargé ma propre image RHEL à partir d’un dossier local (via Azure Migrate, Azure Site Recovery ou autre) vers Azure. Dois-je faire quelque chose pour tirer parti d’Azure Hybrid Benefit ?*

A : Non, vous n’avez rien à faire. Les images RHEL que vous chargez sont déjà considérées comme des images BYOS, et seuls les coûts d’infrastructure Azure vous sont facturés. Vous êtes responsable des coûts des abonnements RHEL, comme vous l’êtes pour vos environnements locaux. 

*Q : Puis-je utiliser Azure Hybrid Benefit sur des machines virtuelles déployées à partir d’images SAP RHEL et SLES de Place de marché Azure ?*

R : Oui, c’est possible. Vous pouvez utiliser le type de licence `RHEL_BYOS` pour les machines virtuelles RHEL et `SLES_BYOS` afin de convertir des machines virtuelles déployées à partir d’images SAP RHEL et SLES de Place de marché Azure.

*Q : Puis-je utiliser Azure Hybrid Benefit sur des groupes de machines virtuelles identiques pour RHEL et SLES ?*

A : Non, c’est impossible. Les groupes de machines virtuelles identiques n’entrent pas actuellement dans le champ d’application d’Azure Hybrid Benefit pour RHEL et SLES.

*Q : Puis-je utiliser Azure Hybrid Benefit sur des instances réservées pour RHEL et SLES ?*

A : Non, c’est impossible. Les instances réservées n’entrent pas actuellement dans le champ d’application d’Azure Hybrid Benefit pour RHEL et SLES.

*Q : Puis-je utiliser Azure Hybrid Benefit sur une machine virtuelle déployée pour SQL Server sur des images RHEL ?*

A : Non, c’est impossible. Aucune prise en charge de ces éléments n’est prévue.
 

## <a name="common-problems"></a>Problèmes courants
Cette section répertorie les problèmes courants que vous pouvez rencontrer et les étapes d’atténuation.

| Error | Limitation des risques |
| ----- | ---------- |
| « Impossible de terminer l’action, car les informations dont nous disposons indique que vous n’avez pas réussi à activer Red Hat Cloud Access sur votre abonnement Azure... » | Pour utiliser l’avantage avec des machines virtuelles RHEL, vous devez d’abord [inscrire vos abonnements Azure auprès de Red Hat Cloud Access](https://access.redhat.com/management/cloud).

## <a name="next-steps"></a>Étapes suivantes
* [Découvrez comment créer et mettre à jour des machines virtuelles et ajouter des types de licences (RHEL_BYOS, SLES_BYOS) pour Azure Hybrid Benefit à l’aide d’Azure CLI.](/cli/azure/vm?preserve-view=true&view=azure-cli-latest)
