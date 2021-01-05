---
title: Vue d’ensemble de l’exemple de blueprint Charge de travail ASE/SQL ISO 27001
description: Vue d’ensemble et architecture de l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001.
ms.date: 11/02/2020
ms.topic: sample
ms.openlocfilehash: 4972aa09e993f8de445cf4bf581f5ad76dbca520
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420374"
---
# <a name="overview-of-the-iso-27001-app-service-environmentsql-database-workload-blueprint-sample"></a>Vue d’ensemble de l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001

L’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001 fournit une infrastructure supplémentaire à l’exemple de blueprint [Services partagés ISO 27001](../iso27001-shared/index.md).
Ce blueprint aide les clients à déployer des architectures cloud qui offrent des solutions aux scénarios qui impliquent des obligations d’accréditation ou de conformité.

Il existe deux exemples de blueprints ISO 27001, le présent exemple et [Services partagés ISO 27001](../iso27001-shared/index.md).

> [!IMPORTANT]
> Cet exemple dépend de l’infrastructure déployée par l’exemple de blueprint [Services partagés ISO 27001](../iso27001-shared/index.md). Il doit être déployé en premier.

## <a name="architecture"></a>Architecture

L’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001 déploie une plateforme en tant qu’environnement web basé sur des services. L’environnement peut servir à héberger plusieurs applications web, API web et instances SQL Database qui suivent les normes ISO 27001. Cet exemple de blueprint dépend de l’exemple de blueprint [Services partagés ISO 27001](../iso27001-shared/index.md).

:::image type="content" source="../../media/sample-iso27001-ase-sql-workload/iso27001-ase-sql-workload-blueprint-sample-design.png" alt-text="Conception de l’exemple de blueprint Charge de travail ASE/SQL ISO 27001" border="false":::

Cet environnement se compose de plusieurs services Azure utilisés pour fournir une infrastructure de charge de travail sécurisée, entièrement supervisée et prête pour les entreprises, basée sur les normes ISO 27001. Cet environnement comporte les éléments suivants :

- [Rôle Azure](../../../../role-based-access-control/overview.md) nommé DevOps disposant des droits nécessaires pour déployer et gérer des ressources dans un [Azure App Service Environment](../../../../app-service/environment/intro.md) déployé par l’exemple de blueprint
- Définitions [Azure Policy](../../../policy/overview.md) pour verrouiller les services pouvant être déployés dans l’environnement et interdire la création d’une ressource d’adresse IP publique
- Un réseau virtuel contenant un seul sous-réseau et appairé à un environnement de [services partagés](../iso27001-shared/index.md) préexistant avec forçage du trafic à travers le pare-feu des [services partagés](../iso27001-shared/index.md) Le réseau virtuel héberge les ressources suivantes :
  - Un [environnement Azure App Service Environment](../../../../app-service/environment/intro.md) utilisable pour héberger une ou plusieurs applications web, API web ou fonctions
  - Une instance [Azure Key Vault](../../../../key-vault/general/overview.md) qui utilise un point de terminaison de service de réseau virtuel pour stocker les secrets utilisés par les applications qui s’exécutent dans l’environnement de la charge de travail
  - Une instance de serveur [Azure SQL Database](../../../../azure-sql/database/sql-database-paas-overview.md) qui utilise un point de terminaison de service de réseau virtuel pour héberger les bases de données utilisées pour les applications dans l’environnement de la charge de travail

## <a name="next-steps"></a>Étapes suivantes

Vous avez parcouru la vue d’ensemble et l’architecture de l’exemple de blueprint Charge de travail App Service Environment/SQL Database ISO 27001. Continuez avec les articles suivants pour découvrir les correspondances de contrôles et la manière de déployer cet exemple :

> [!div class="nextstepaction"]
> [Blueprint Charge de travail App Service Environment/SQL Database ISO 27001 - Correspondance des contrôles](./control-mapping.md)
> [Blueprint Charge de travail App Service Environment/SQL Database ISO 27001 - Étapes de déploiement](./deploy.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
