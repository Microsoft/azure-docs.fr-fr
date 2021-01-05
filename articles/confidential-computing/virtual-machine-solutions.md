---
title: Solutions d’informatique confidentielle Azure sur les machines virtuelles
description: Découvrez les solutions d’informatique confidentielle Azure sur les machines virtuelles.
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: 8d5ce3cde8c86d66bec025c778318a192ef60b73
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94560841"
---
# <a name="solutions-on-azure-virtual-machines"></a>Solutions sur des machines virtuelles Azure

Cet article contient des informations sur le déploiement de machines virtuelles d’informatique confidentielle Azure exécutant des processeurs Intel à technologie [Intel Software Guard Extension](https://software.intel.com/sgx) (Intel SGX). 

## <a name="azure-confidential-computing-vm-sizes"></a>Tailles de machines virtuelles d’informatique confidentielle Azure

Les machines virtuelles d’informatique confidentielle Azure sont conçues pour protéger la confidentialité et l’intégrité de vos données et de votre code pendant leur traitement dans le cloud. 

La [série DCsv2](../virtual-machines/dcv2-series.md) est la famille de tailles de machines virtuelles d’informatique confidentielle la plus récente. Ces machines virtuelles prennent en charge un plus grand nombre de fonctionnalités de déploiement, ont un EPC (Enclave Page Cache) deux fois plus élevé et un plus grand choix de tailles par rapport à nos machines virtuelles de série DC. Les machines virtuelles de la [série DC](../virtual-machines/sizes-previous-gen.md#preview-dc-series) sont actuellement en préversion et seront dépréciées ; elles ne seront pas incluses dans la disponibilité générale.

Commencez le déploiement d’une machine virtuelle de série DCsv2 par le biais de la Place de marché commerciale Microsoft en suivant le [tutoriel de démarrage rapide](quick-create-marketplace.md).

### <a name="current-available-sizes-and-regions"></a>Tailles et régions actuellement disponibles

Pour obtenir une liste de toutes les tailles de machines virtuelles de calcul confidentielles disponibles dans les régions disponibles et les zones de disponibilité, exécutez la commande suivante dans [Azure CLI](/cli/azure/install-azure-cli-windows?view=azure-cli-latest) :

```azurecli-interactive
az vm list-skus `
    --size dc `
    --query "[?family=='standardDCSv2Family'].{name:name,locations:locationInfo[0].location,AZ_a:locationInfo[0].zones[0],AZ_b:locationInfo[0].zones[1],AZ_c:locationInfo[0].zones[2]}" `
    --all `
    --output table
```

Pour obtenir une vue plus détaillée des tailles ci-dessus, exécutez la commande suivante :

```azurecli-interactive
az vm list-skus `
    --size dc `
    --query "[?family=='standardDCSv2Family']"
```
### <a name="dedicated-host-requirements"></a>Configuration requise pour l’hôte dédié
Le déploiement d’une taille de machine virtuelle **Standard_DC8_v2** dans la famille de machines virtuelles de la série DCSv2 occupera l’hôte complet et celui-ci ne sera pas partagé avec d’autres locataires ou abonnements. Cette famille de références SKU de machines virtuelles fournit l’isolement dont vous pouvez avoir besoin pour répondre aux exigences réglementaires de conformité et de sécurité qui sont normalement respectées en utilisant un service d’hôte dédié. Lorsque vous choisissez la référence SKU **Standard_DC8_v2**, le serveur hôte physique alloue toutes les ressources matérielles disponibles, notamment la mémoire EPC, uniquement à votre machine virtuelle. Notez que cette fonctionnalité existe de par la conception de l’infrastructure et que toutes les fonctionnalités de **Standard_DC8_v2** seront prises en charge. Ce déploiement n’est pas identique au service [Azure Dedicated Host](../virtual-machines/dedicated-hosts.md) fourni par d’autres familles de machines virtuelles Azure.


## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

Suivez un tutoriel de démarrage rapide pour déployer une machine virtuelle de la série DCsv2 en moins de 10 minutes. 

- **Abonnement Azure** : pour déployer une instance de machine virtuelle d’informatique confidentielle, envisagez de souscrire un abonnement de paiement à l’utilisation ou d’autres options d’achat. Si vous utilisez un [compte gratuit Azure](https://azure.microsoft.com/free/), vous n’aurez pas de quota pour la quantité appropriée de cœurs de calcul Azure.

- **Tarifs et disponibilité régionale** : recherchez les prix des machines virtuelles de la série DCsv2 dans la [page de tarification des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). Pour connaître la disponibilité dans les différentes régions Azure, voir [Disponibilité des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) .


- **Quota de cœurs** : vous devrez peut-être augmenter le quota de cœurs dans votre abonnement Azure à partir de la valeur par défaut. Votre abonnement peut également limiter le nombre de cœurs que vous pouvez déployer dans certaines familles de taille de machine virtuelle, dont la série DCsv2. Pour demander une augmentation de quota, [ouvrez une demande de service clientèle en ligne](../azure-portal/supportability/per-vm-quota-requests.md) gratuitement. Notez que les limites par défaut peuvent varier en fonction de la catégorie de votre abonnement.

  > [!NOTE]
  > Si vous avez des besoins de capacité à grande échelle, contactez le support Azure. Les quotas d’Azure sont des limites de crédit et non des garanties de capacité. Quel que soit votre quota, vous êtes facturé uniquement pour les cœurs que vous utilisez.
  
- **Redimensionnement** : en raison de leur matériel spécialisé, vous pouvez uniquement redimensionner les instances d’informatique confidentielle qui appartiennent à la même famille de taille. Par exemple, vous pouvez uniquement redimensionner une machine virtuelle de la série DCsv2 d’une taille DCsv2 en une autre de cette même série. Le redimensionnement d’une taille d’informatique non confidentielle en une taille d’informatique confidentielle n’est pas pris en charge.  

- **Image** : pour assurer la prise en charge d’Intel Software Guard Extension (Intel SGX) sur les instances d’informatique confidentielle, tous les déploiements doivent être exécutés sur des images de génération 2. L’informatique confidentielle Azure prend en charge les charges de travail s’exécutant sur Ubuntu 18.04 Gen 2, Ubuntu 16.04 Gen 2, Windows Server 2019 gen2 et Windows Server 2016 Gen 2. Consultez l’article sur la [prise en charge des machines virtuelles de génération 2 sur Azure](../virtual-machines/generation-2.md) pour en savoir plus sur les scénarios pris en charge et non pris en charge. 

- **Stockage** : les disques de données de machines virtuelles d’informatique confidentielle Azure et nos disques de système d’exploitation éphémères se trouvent sur des disques NVMe. Les instances ne prennent en charge que les disques SSD Premium et SSD Standard, et non SSD Ultra ou HDD Standard. La taille de machine virtuelle **DC8_v2** ne prend pas en charge le stockage Premium. 

- **Chiffrement de disque** : les instances d’informatique confidentielle ne prennent pas en charge le chiffrement de disque pour l’instant. 

## <a name="high-availability-and-disaster-recovery-considerations"></a>Remarques relatives à la haute disponibilité et à la récupération d’urgence

Lors de l’utilisation de machines virtuelles dans Azure, vous êtes responsable de l’implémentation d’une solution de haute disponibilité et reprise d’activité après sinistre pour éviter tout temps d’arrêt. 

À l’heure actuelle, l’informatique confidentielle Azure ne prend pas en charge la redondance de zone par le biais de Zones de disponibilité. Pour bénéficier d’une disponibilité et d’une redondance optimales pour l’informatique confidentielle, utilisez des [groupes à haute disponibilité](../virtual-machines/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy). En raison des restrictions matérielles, les groupes à haute disponibilité pour les instances d’informatique confidentielle ne peuvent avoir qu’un maximum de 10 domaines de mise à jour. 

## <a name="deployment-with-azure-resource-manager-arm-template"></a>Déploiement avec des modèles Azure Resource Manager (ARM)

Azure Resource Manager est le service de déploiement et de gestion d’Azure. Il fournit une couche de gestion qui vous permet de créer, mettre à jour et supprimer des ressources dans votre abonnement Azure. Vous pouvez utiliser des fonctionnalités de gestion, telles que le contrôle d’accès, les verrous et les étiquettes, pour sécuriser et organiser vos ressources après le déploiement.

Pour en savoir plus sur les modèles ARM, consultez [Vue d’ensemble du déploiement de modèles](../azure-resource-manager/templates/overview.md).

Pour déployer une machine virtuelle de série DCsv2 dans un modèle ARM, vous utiliserez la [ressource Machine virtuelle](../virtual-machines/windows/template-description.md). Veillez à spécifier les propriétés correctes pour **vmSize** et votre **imageReference**.

### <a name="vm-size"></a>Taille de la machine virtuelle

Spécifiez l’une des tailles suivantes dans votre modèle ARM dans la ressource Machine virtuelle. Cette chaîne est spécifiée en tant que **vmSize** dans **properties**.

```json
  [
        "Standard_DC1s_v2",
        "Standard_DC2s_v2",
        "Standard_DC4s_v2",
        "Standard_DC8_v2"
      ],
```

### <a name="gen2-os-image"></a>Image de système d’exploitation Gen2

Sous **properties**, vous devrez également référencer une image sous **storageProfile**. Utilisez *une seule* des images suivantes pour votre **imageReference**.

```json
      "2019-datacenter-gensecond": {
        "offer": "WindowsServer",
        "publisher": "MicrosoftWindowsServer",
        "sku": "2019-datacenter-gensecond",
        "version": "latest"
      },
      "2016-datacenter-gensecond": {
        "offer": "WindowsServer",
        "publisher": "MicrosoftWindowsServer",
        "sku": "2016-datacenter-gensecond",
        "version": "latest"
      },
        "18_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "18_04-lts-gen2",
        "version": "latest"
      },
      "16_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "16_04-lts-gen2",
        "version": "latest"
      }
```

## <a name="next-steps"></a>Étapes suivantes 

Dans cet article, vous avez découvert les qualifications et configurations nécessaires à la création d’une machine virtuelle d’informatique confidentielle. Vous pouvez maintenant accéder à la Place de marché Microsoft Azure pour déployer une machine virtuelle de série DCsv2.

> [!div class="nextstepaction"]
> [Déployer une machine virtuelle de série DCsv2 dans la Place de marché Azure](quick-create-marketplace.md)