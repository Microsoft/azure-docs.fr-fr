---
title: Vue d’ensemble de l’exemple de blueprint de zone d’accueil de migration du framework d’adoption du cloud
description: Vue d’ensemble et architecture de l’exemple de blueprint de zone d’accueil de migration du framework d’adoption du cloud pour Azure.
ms.date: 09/14/2020
ms.topic: sample
ms.openlocfilehash: 6b8e3484690d263a43d3824c054b28344ea07fb1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90531676"
---
# <a name="overview-of-the-microsoft-cloud-adoption-framework-for-azure-migration-landing-zone-blueprint-sample"></a>Vue d’ensemble de l’exemple de blueprint de zone d’accueil de migration du framework d’adoption de Microsoft Cloud pour Azure.

Le blueprint de zone d’accueil de migration du framework d’adoption de Microsoft Cloud pour Azure est un ensemble d’infrastructures destinées à vous aider à configurer la migration de votre première charge de travail et à gérer vos ressources cloud en adéquation avec le framework d’adoption du cloud.

L’exemple de blueprint de [base du framework d’adoption du cloud](../caf-foundation/index.md) étend cet exemple.

## <a name="architecture"></a>Architecture

L’exemple de blueprint de zone d’accueil de migration du framework d’adoption du cloud déploie dans Azure des ressources d’infrastructure de base que les organisations peuvent utiliser pour préparer leur abonnement à accueillir la migration de machines virtuelles. Il aide également à mettre en place les contrôles de gouvernance nécessaires pour gérer leurs ressources cloud. Cet exemple déploie et applique des ressources, stratégies et modèles qui permettent à une organisation de commencer à utiliser Azure en toute confiance.

:::image type="complex" source="../../media/caf-blueprints/caf-migration-landing-zone-architecture.png" alt-text="Zone d’atterrissage de migration C A F, image décrivant ce qui est installé conformément aux instructions du C A F pour la zone d’atterrissage initiale." border="false":::
   Décrit une architecture Azure qui est obtenue en déployant le blueprint Migration C A F.  Il s’applique à un abonnement avec des groupes de ressources comprenant un réseau virtuel Azure, un compte de stockage pour stocker des journaux, Log Analytics configuré pour stocker dans le compte de stockage.  Il décrit également Azure Key Vault configuré et la configuration initiale d’Azure Migrate créée.  Toutes ces infrastructures de base sont accessibles avec Azure Active Directory.     
:::image-end:::

Cet environnement se compose de plusieurs services Azure utilisés pour fournir une gouvernance sécurisée, entièrement supervisée et prête pour les entreprises. Cet environnement comporte les éléments suivants :

- Une instance [Azure Key Vault](../../../../key-vault/general/overview.md) destinée à héberger les secrets utilisés pour les certificats, clés et secret déployés dans l’environnement des services partagés
- [Log Analytics](../../../../azure-monitor/overview.md) est déployé pour s’assurer que l’ensemble des actions et services journalisent dans un emplacement central dès l’instant où vous démarrez votre migration.
- [Réseau virtuel Azure](../../../../virtual-network/virtual-networks-overview.md) fournissant un réseau isolé et des sous-réseaux pour votre machine virtuelle.
- [Projet Azure Migrate](../../../../migrate/migrate-services-overview.md) pour la découverte et l’évaluation. Nous ajoutons les outils pour l’évaluation de serveur, la migration de serveur, l’évaluation de base de données et la migration de base de données.  


Tous ces éléments se conforment aux pratiques éprouvées publiés dans [Centre des architectures Azure - Architectures de référence](/azure/architecture/reference-architectures/).

> [!NOTE]
> Le blueprint de migration du framework d’adoption du cloud constitue une zone d’accueil pour vos charges de travail. Vous devez toujours effectuer l’évaluation et la migration de vos machines virtuelles/bases de données en plus de cette architecture de base.

Pour plus d’informations, voir [Framework d’adoption de Microsoft Cloud pour Azure – Migrer](/azure/architecture/cloud-adoption/migrate/).

## <a name="next-steps"></a>Étapes suivantes

Vous avez découvert la vue d’ensemble et l’architecture de l’exemple de blueprint de zone d’accueil de migration du framework d’adoption du cloud.

> [!div class="nextstepaction"]
> [Blueprint de zone d’accueil de migration du framework d’adoption du cloud – Étapes de déploiement](./deploy.md)

Autres articles sur les blueprints et la manière de les utiliser :

- En savoir plus sur le [cycle de vie des blueprints](../../concepts/lifecycle.md)
- Comprendre comment utiliser les [paramètres statiques et dynamiques](../../concepts/parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](../../concepts/sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](../../concepts/resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../../how-to/update-existing-assignments.md).
