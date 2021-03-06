---
title: Comparer Azure Migrate et Site Recovery pour la migration vers Azure
description: Résume les avantages de l’utilisation d’Azure Migrate pour la migration, au lieu de Site Recovery.
services: site-recovery
ms.service: site-recovery
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: c4d30b8b21bf3a0ea27467d2120b04a107f5a73f
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581413"
---
# <a name="migrating-to-azure"></a>Migration vers Azure

Pour la migration, nous vous recommandons d’utiliser le service Azure Migrate plutôt que le service Azure Site Recovery pour migrer des machines virtuelles et des serveurs vers Azure. [Apprenez-en davantage](../migrate/migrate-services-overview.md) sur Azure Migrate.


## <a name="why-use-azure-migrate"></a>Pourquoi utiliser Azure Migrate ?

L’utilisation d’Azure Migrate pour la migration offre un certain nombre d’avantages :
 
 
- Azure Migrate fournit un hub centralisé pour la découverte, l’évaluation et la migration vers Azure.
- L’utilisation d’Azure Migrate fournit une interopérabilité et une extensibilité future avec les outils Azure Migrate, les autres services Azure et les outils tiers.
- L’outil Azure Migrate : Migration de serveurs est conçu spécialement pour la migration de serveurs vers Azure. Il est optimisé pour la migration. Vous n’avez pas besoin d’en savoir plus sur les concepts et les scénarios qui ne sont pas directement pertinents pour la migration. 
- Il n’y a pas de frais d’utilisation de l’outil pour la migration pendant 180 jours, à partir du moment où la réplication est démarrée pour une machine virtuelle. Cela vous donne le temps de terminer la migration. Vous payez uniquement pour les ressources de stockage et de réseau utilisées pour la réplication, et pour les frais de calcul consommés pendant les migrations de test.
- Azure Migrate prend en charge tous les scénarios de migration pris en charge par Site Recovery. En outre, pour les machines virtuelles VMware, Azure Migrate fournit une option de migration sans agent.
- Nous donnons la priorité aux nouvelles fonctionnalités de migration pour l’outil Azure Migrate : Migration de serveurs uniquement. Ces fonctionnalités ne sont pas ciblées pour Site Recovery.

## <a name="when-to-use-site-recovery"></a>Quand utiliser Azure Site Recovery ?

Site Recovery doit être utilisé :

- Pour la récupération d’urgence de machines locales vers Azure.
- Pour la récupération d’urgence de machines virtuelles Azure entre des régions Azure.

Bien que nous vous recommandons d’utiliser Azure Migrate pour migrer des serveurs locaux vers Azure, si vous avez déjà commencé votre migration avec Site Recovery, vous pouvez continuer à l’utiliser pour terminer votre migration.  

## <a name="next-steps"></a>Étapes suivantes

> [Passez en revue les questions courantes](../migrate/resources-faq.md) sur Azure Migrate.
