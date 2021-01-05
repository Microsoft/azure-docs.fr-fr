---
title: Migrer des machines virtuelles AWS vers Azure avec Azure Migrate
description: Cet article décrit les options pour la migration d’instances AWS vers Azure, et recommande Azure Migrate.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/27/2019
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 83d8106b9f206da9095c37b6179bdda8001b439c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009056"
---
# <a name="migrate-amazon-web-services-aws-vms-to-azure"></a>Migrer des machines virtuelles Amazon Web Services (AWS) vers Azure

Cet article décrit les options de migration d’instances Amazon Web Services (AWS) vers Azure.

## <a name="migrate-with-azure-migrate"></a>Migrer avec Azure Migrate

Nous vous recommandons de migrer les instances AWS EC2 vers Azure à l’aide du service [Azure Migrate](../migrate/migrate-services-overview.md). Azure Migrate est spécialement conçu pour la migration du serveur. Azure Migrate fournit un hub centralisé pour la découverte, l’évaluation et la migration des machines locales vers Azure.

[Découvrez comment](../migrate/tutorial-migrate-aws-virtual-machines.md) migrer des instances AWS avec Azure Migrate. 


## <a name="migrate-with-site-recovery"></a>Migrer avec Site Recovery

Site Recovery ne doit être utilisé que pour la récupération d’urgence, pas pour la migration.

Si vous utilisez déjà Azure Site Recovery et souhaitez continuer à l’utiliser pour la migration AWS, procédez comme pour configurer la [récupération d’urgence de machines physiques](physical-azure-disaster-recovery.md).


> [!NOTE]
> Lorsque vous opérez un basculement à des fins de récupération d’urgence, lors de la dernière étape, vous validez le basculement. Lorsque vous migrez des instances AWS, l’option **Valider** n’est pas pertinente. Au lieu de cela, vous sélectionnez l’option **Terminer la migration**. 

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Passez en revue les questions courantes](../migrate/resources-faq.md) sur Azure Migrate.
