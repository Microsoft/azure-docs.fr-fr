---
title: Migrer des machines virtuelles vers Resource Manager à l’aide d’Azure CLI
description: Cet article décrit pas à pas la procédure de migration de ressources prise en charge par la plateforme de l’environnement Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.custom: devx-track-azurecli
ms.openlocfilehash: a6f6783819d97ec97b93f86b687cf3093d6f4209
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94904623"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-cli"></a>Migration de ressources IaaS d’un environnement Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure

> [!IMPORTANT]
> Aujourd’hui, environ 90 % des machines virtuelles IaaS utilisent [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). Depuis le 28 février 2020, les machines virtuelles classiques sont dépréciées. Elles seront entièrement mises hors service le 1er mars 2023. [Apprenez-en davantage]( https://aka.ms/classicvmretirement) sur cette dépréciation et [la façon dont elle vous concerne](classic-vm-deprecation.md#how-does-this-affect-me).

Ces étapes vous montrent comment utiliser les commandes de l’interface de ligne de commande Azure (CLI) pour migrer des ressources d’infrastructure en tant que service (IaaS) à partir du modèle de déploiement Classic vers le modèle de déploiement Azure Resource Manager. Cet article nécessite [Azure Classic CLI](/cli/azure/install-classic-cli). Étant donné que l’interface Azure CLI est applicable uniquement pour les ressources Azure Resource Manager, elle ne peut pas être utilisée pour cette migration.

> [!NOTE]
> Toutes les opérations décrites ici sont idempotentes. Si vous rencontrez un problème autre qu’une fonctionnalité non prise en charge ou qu’une erreur de configuration, nous vous recommandons de réexécuter la procédure de préparation, d’abandon ou de validation. La plateforme tentera une nouvelle fois l’opération.
> 
> 

<br>
Voici un diagramme de flux permettant d’identifier l’ordre dans lequel les étapes doivent être exécutées lors d’un processus de migration.

![Screenshot that shows the migration steps](./media/migration-classic-resource-manager/migration-flow.png)

## <a name="step-1-prepare-for-migration"></a>Étape 1 : Préparation de la migration
Voici quelques bonnes pratiques recommandées lorsque vous évaluez la migration de ressources IaaS d’un environnement Classic vers Resource Manager.

* Parcourez la liste des [configurations ou fonctionnalités non prises en charge](migration-classic-resource-manager-overview.md). Si vous avez des machines virtuelles qui utilisent des configurations ou fonctionnalités non prises en charge, nous vous conseillons d’attendre que leur prise en charge soit annoncée. Vous pouvez également supprimer cette fonctionnalité ou modifier cette configuration pour permettre la migration si cela répond à vos besoins.
* Si vous avez déjà des scripts automatisés qui déploient votre infrastructure et vos applications, essayez de créer une configuration de test similaire à l’aide de ces scripts pour la migration. Vous pouvez également configurer des environnements de test à l’aide du portail Azure.

> [!IMPORTANT]
> Les passerelles d’application ne sont actuellement pas prises en charge pour la migration de déploiement classique vers Resource Manager. Pour migrer un réseau virtuel classique avec une passerelle d’application, supprimez la passerelle avant d’exécuter une opération de préparation pour déplacer le réseau. Après avoir effectué la migration, reconnectez la passerelle dans Azure Resource Manager. 
>
>Les passerelles ExpressRoute se connectant à des circuits ExpressRoute dans un autre abonnement ne peuvent pas être migrées automatiquement. Dans ce cas, supprimez la passerelle ExpressRoute, migrez le réseau virtuel et recréez la passerelle. Pour plus d’informations, cnsultez [Migrer des circuits ExpressRoute et les réseaux virtuels associés du modèle de déploiement classique au modèle de déploiement Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md).
> 
> 

## <a name="step-2-set-your-subscription-and-register-the-provider"></a>Étape 2 : Étape 2 : Définition de votre abonnement et inscription du fournisseur
Pour les scénarios de migration, vous devez configurer votre environnement à la fois pour l’environnement Classic et pour Resource Manager. [Installez l’interface de ligne de commande Azure](/cli/azure/install-classic-cli) et [sélectionnez votre abonnement](/cli/azure/authenticate-azure-cli).

Connectez-vous à votre compte.

```azurecli
azure login
```

Sélectionnez votre abonnement Azure à l’aide de la commande suivante.

```azurecli
azure account set "<azure-subscription-name>"
```

> [!NOTE]
> L’inscription constitue une étape unique, mais elle doit être effectuée une seule fois avant de tenter la migration. Sans vous inscrire, vous verrez le message suivant 
> 
> *BadRequest : L'abonnement n'est pas inscrit pour la migration.* 
> 
> 

Inscrivez-vous auprès du fournisseur de ressources de migration à l’aide de la commande suivante. Notez que cette commande expire dans certains cas. Toutefois, l’inscription sera réussie.

```azurecli
azure provider register Microsoft.ClassicInfrastructureMigrate
```

Veuillez patienter cinq minutes le temps que l’inscription se termine. Vous pouvez vérifier l’état de l’approbation à l’aide de la commande suivante. Assurez-vous que RegistrationState est `Registered` avant de continuer.

```azurecli
azure provider show Microsoft.ClassicInfrastructureMigrate
```

Passez ensuite l’interface de ligne de commande en mode `asm` .

```azurecli
azure config mode asm
```

## <a name="step-3-make-sure-you-have-enough-azure-resource-manager-virtual-machine-vcpus-in-the-azure-region-of-your-current-deployment-or-vnet"></a>Étape 3 : vérifier le nombre de processeurs virtuels de machines virtuelles Azure Resource Manager dans la région Azure de votre déploiement ou réseau virtuel actuel
Pour cette étape, vous devrez passer au mode `arm` . Pour ce faire, utilisez la commande suivante.

```azurecli
azure config mode arm
```

Vous pouvez utiliser la commande CLI suivante pour vérifier la quantité de processeurs virtuels dont vous disposez actuellement dans Azure Resource Manager. Pour en savoir plus sur les quotas de processeurs virtuels, consultez [Limites et Azure Resource Manager](../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

```azurecli
azure vm list-usage -l "<Your VNET or Deployment's Azure region"
```

Une fois que vous avez procédé à la vérification de cette étape, vous pouvez revenir au mode `asm` .

```azurecli
azure config mode asm
```

## <a name="step-4-option-1---migrate-virtual-machines-in-a-cloud-service"></a>Étape 4 : Option 1 – Migration de machines virtuelles dans un service cloud
Obtenez la liste des services cloud à l’aide de la commande suivante, puis choisissez le service cloud que vous voulez migrer. Notez que vous obtiendrez un message d’erreur si les machines virtuelles du service cloud sont dans un réseau virtuel ou si elles ont des rôles Web/de travail.

```azurecli
azure service list
```

Exécutez la commande suivante pour obtenir le nom du déploiement du service cloud à partir d’une sortie détaillée. Dans la plupart des cas, le nom du déploiement est le même que le nom de service Cloud.

```azurecli
azure service show <serviceName> -vv
```

La première étape consiste à valider si vous pouvez migrer le service cloud à l’aide des commandes suivantes :

```shell
azure service deployment validate-migration <serviceName> <deploymentName> new "" "" ""
```

Préparez les machines virtuelles dans le service cloud pour la migration. Vous disposez de deux options.

Si vous voulez migrer les machines virtuelles vers un réseau virtuel créé à partir d’une plateforme, utilisez la commande suivante.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> new "" "" ""
```

Si vous voulez procéder à leur migration vers un réseau virtuel existant dans le modèle de déploiement Resource Manager, utilisez la commande suivante.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> existing <destinationVNETResourceGroupName> <subnetName> <vnetName>
```

Une fois l’opération de préparation réussie, vous pouvez consulter la sortie détaillée pour voir l’état de la migration des machines virtuelles et vous assurer que leur état est `Prepared` .

```azurecli
azure vm show <vmName> -vv
```

Vérifiez la configuration pour les ressources préparées à l’aide de l’interface de ligne de commande ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante.

```azurecli
azure service deployment abort-migration <serviceName> <deploymentName>
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante.

```azurecli
azure service deployment commit-migration <serviceName> <deploymentName>
```

## <a name="step-4-option-2----migrate-virtual-machines-in-a-virtual-network"></a>Étape 4 : Option 2 – Migration de machines virtuelles dans un réseau virtuel
Sélectionnez le réseau virtuel dont vous souhaitez effectuer la migration. Sachez que, si le réseau virtuel contient des rôles Web/de travail ou des machines virtuelles avec des configurations non prises en charge, vous obtiendrez un message d’erreur pour la validation.

Récupérez tous les réseaux virtuels de l’abonnement à l’aide de la commande suivante.

```azurecli
azure network vnet list
```

Vous obtenez un résultat semblable à ce qui suit :

![Capture d’écran de la ligne de commande avec le nom de réseau virtuel entier en surbrillance.](./media/virtual-machines-linux-cli-migration-classic-resource-manager/vnet.png)

Dans l’exemple ci-dessus, **virtualNetworkName** est le nom complet **« Group classicubuntu16 classicubuntu16 »** .

Vérifiez d’abord que vous pouvez migrer le réseau virtuel à l’aide de la commande suivante :

```shell
azure network vnet validate-migration <virtualNetworkName>
```

Préparez le réseau virtuel de votre choix pour la migration à l’aide de la commande suivante.

```azurecli
azure network vnet prepare-migration <virtualNetworkName>
```

Vérifiez la configuration pour les machines virtuelles préparées à l’aide de l’interface de ligne de commande ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante.

```azurecli
azure network vnet abort-migration <virtualNetworkName>
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante.

```azurecli
azure network vnet commit-migration <virtualNetworkName>
```

## <a name="step-5-migrate-a-storage-account"></a>Étape 5 : Migrer un compte de stockage
Une fois que vous avez terminé la migration des machines virtuelles, nous vous recommandons de migrer le compte de stockage.

Préparer le compte de stockage pour la migration à l’aide de la commande suivante

```azurecli
azure storage account prepare-migration <storageAccountName>
```

Vérifiez la configuration pour le compte de stockage préparé à l’aide de l’interface de ligne de commande ou du portail Azure. Si vous n’êtes pas prêt pour la migration et que vous souhaitez revenir à l’ancien état, utilisez la commande suivante.

```azurecli
azure storage account abort-migration <storageAccountName>
```

Si la configuration préparée semble correcte, vous pouvez continuer et valider les ressources à l’aide de la commande suivante.

```azurecli
azure storage account commit-migration <storageAccountName>
```

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de la migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planification de la migration des ressources IaaS d’Azure Classic vers Azure Resource Manager](migration-classic-resource-manager-plan.md)
* [Faire migrer des ressources IaaS Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Outils de la communauté pour aider à la migration de ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-community-tools.md)
* [Passer en revue les erreurs courantes de migration](migration-classic-resource-manager-errors.md)
* [Passez en revue les questions fréquemment posées sur la migration des ressources IaaS de Classic vers Azure Resource Manager](migration-classic-resource-manager-faq.md)
