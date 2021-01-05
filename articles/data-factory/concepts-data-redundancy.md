---
title: Redondance des données dans Azure Data Factory | Microsoft Docs
description: En savoir plus sur les mécanismes de redondance des métadonnées dans Azure Data Factory
services: data-factory
documentationcenter: ''
author: nabhishek
manager: weehyongtok
ms.reviewer: abnarain
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: abnarain
ms.openlocfilehash: 7f95adc264ed91e75eef668b43f674ddeb7d9e89
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350022"
---
# <a name="azure-data-factory-data-redundancy"></a>**Redondance des données dans Azure Data Factory**

Les données d’Azure Data Factory incluent des métadonnées (pipeline, jeux de données, services liés, runtime d’intégration et déclencheurs) et des données de surveillance (pipeline, déclencheur et exécutions d’activités). 

Dans toutes les régions (à l’exception des régions Brésil Sud et Asie Sud-Est), les données d’Azure Data Factory sont stockées et répliquées dans la [région jumelée](../best-practices-availability-paired-regions.md#azure-regional-pairs) afin de protéger contre la perte de métadonnées. En cas de défaillance du centre de données régional, Microsoft peut déclencher un basculement régional de votre instance Azure Data Factory. Dans la plupart des cas, aucune action n’est requise de votre part. Une fois le basculement géré par Microsoft terminé, vous serez en mesure d’accéder à votre Azure Data Factory dans la région de basculement. 

En raison des exigences relatives à la résidence des données dans les régions Brésil Sud et Asie Sud-Est, les données d’Azure Data Factory sont stockées dans la [région locale uniquement](../storage/common/storage-redundancy.md#locally-redundant-storage). Pour la région Asie Sud-Est, toutes les données sont stockées à Singapour. Pour la région Brésil Sud, toutes les données sont stockées au Brésil. Si la région est perdue en raison d’un sinistre important, Microsoft ne peut pas récupérer les données de votre Azure Data Factory.  

> [!NOTE]
> Le basculement géré par Microsoft ne s’applique pas au runtime d’intégration auto-hébergé (SHIR), car cette infrastructure est généralement gérée par le client. Si le SHIR est configuré sur une machine virtuelle Azure, il est recommandé de tirer parti d’[Azure Site Recovery](../site-recovery/site-recovery-overview.md) pour gérer le [basculement de machine virtuelle Azure](../site-recovery/azure-to-azure-architecture.md) vers une autre région.



## <a name="using-source-control-in-azure-data-factory"></a>**Utilisation du contrôle de code source dans Azure Data Factory**

Pour vous assurer que vous êtes en mesure d’effectuer le suivi et l’audit des modifications apportées aux métadonnées de votre Azure Data Factory, vous devez envisager de configurer le contrôle de code source pour votre Azure Data Factory. Cela vous permet également d’accéder à vos fichiers JSON de métadonnées pour les pipelines, les jeux de données, les services liés et le déclencheur. Azure Data Factory vous permet d’utiliser un dépôt Git différent (Azure DevOps et GitHub). 

 Découvrez comment configurer le [contrôle de code source dans Azure Data Factory](./source-control.md). 

> [!NOTE]
> En cas de sinistre (perte de région), une nouvelle fabrique de données peut être approvisionnée manuellement ou automatiquement. Une fois la fabrique de données créée, vous pouvez restaurer vos pipelines, jeux de données et services liés JSON à partir du dépôt Git existant. 



## <a name="data-stores"></a>**Magasins de données**

Azure Data Factory vous permet de déplacer des données entre des magasins de données situés localement et dans le cloud. Pour garantir la continuité d’activité de vos magasins de données, reportez-vous aux recommandations en matière de continuité d’activité pour chacun de ceux-ci. 

 

## <a name="see-also"></a>Voir aussi

- [Paires régionales Azure](../best-practices-availability-paired-regions.md)
- [Résidence des données dans Azure](https://azure.microsoft.com/global-infrastructure/data-residency/)