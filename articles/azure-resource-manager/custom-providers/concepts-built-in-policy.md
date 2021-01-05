---
title: Déployer des associations à l’aide de stratégies
description: Découvrez comment déployer des associations pour un fournisseur personnalisé à l’aide du service Azure Policy.
author: msHich
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: hich
ms.openlocfilehash: 00cd1d39c0110aac9ea96f73127e83197976c95a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82190127"
---
# <a name="deploy-associations-for-a-custom-provider-using-azure-policy"></a>Déployer des associations pour un fournisseur personnalisé à l’aide d’Azure Policy

Les stratégies Azure peuvent être utilisées pour déployer des associations dans le but d’associer des ressources à un fournisseur personnalisé. Cet article aborde une stratégie intégrée permettant de déployer des associations, et explique comment utiliser cette stratégie.

## <a name="built-in-policy-to-deploy-associations"></a>Stratégie intégrée pour déployer des associations

Le déploiement d’associations pour un fournisseur personnalisé est une stratégie intégrée qui peut être utilisée pour déployer une association dans le but d’associer une ressource à un fournisseur personnalisé. La stratégie accepte trois paramètres :

- ID de fournisseur personnalisé - Il s’agit de l’ID de ressource du fournisseur personnalisé auquel des ressources doivent être associées.
- Types de ressources à associer - Ces types de ressources constituent la liste des types de ressources qui doivent être associés au fournisseur personnalisé. Vous pouvez associer plusieurs types de ressources à un fournisseur personnalisé à l’aide d’une même stratégie.
- Préfixe du nom d’association - Cette chaîne correspond au préfixe qui doit être ajouté au nom de la ressource d’association en cours de création. La valeur par défaut est « DeployedByPolicy ».

La stratégie utilise l’évaluation DeployIfNotExists. Elle est exécutée après qu’un fournisseur de ressources a traité une requête de création ou de mise à jour de ressource, et après que l’évaluation a retourné un code d’état indiquant une réussite. Après cela, la ressource d’association est déployée à l’aide d’un déploiement de modèle.
Pour plus d’informations sur les associations, consultez [Intégration des ressources des fournisseurs personnalisés Azure](./concepts-resource-onboarding.md).

## <a name="how-to-use-the-deploy-associations-built-in-policy"></a>Utiliser la stratégie intégrée de déploiement des associations 

### <a name="prerequisites"></a>Prérequis
Si le fournisseur personnalisé a besoin d’autorisations sur l’étendue de la stratégie pour effectuer une action, le déploiement de stratégie de la ressource d’association ne fonctionnera pas si vous n’accordez pas les autorisations nécessaires.

### <a name="policy-assignment"></a>Affectation de rôle
Pour utiliser la stratégie intégrée, créez une attribution de stratégie, puis attribuez la stratégie Déployer des associations pour un fournisseur personnalisé. La stratégie va alors identifier les ressources non conformes et déployer une association pour ces ressources.

![Attribuer une stratégie intégrée](media/concepts-built-in-policy/assign-builtin-policy-customprovider.png)

## <a name="getting-help"></a>Obtenir de l’aide

Si vous avez des questions sur le développement de fournisseurs de ressources personnalisés Azure, postez-les sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Avant de publier une question, vérifiez si une question similaire a déjà reçu une réponse. Ajoutez le mot clé ```azure-custom-providers``` pour obtenir une réponse rapide !

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez vu comment utiliser une stratégie intégrée pour déployer des associations. Pour en savoir plus, consultez les articles suivants :

- [Concepts : Intégration des ressources des fournisseurs personnalisés Azure](./concepts-resource-onboarding.md)
- [Tutoriel : Intégration des ressources avec des fournisseurs personnalisés](./tutorial-resource-onboarding.md)
- [Tutoriel : Créer des actions et des ressources personnalisées dans Azure](./tutorial-get-started-with-custom-providers.md)
- [Démarrage rapide : Créer un fournisseur de ressources personnalisées et déployer des ressources personnalisées](./create-custom-provider.md)
- [Procédure : ajouter des actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Procédure : ajouter des ressources personnalisées à l’API REST Azure](./custom-providers-resources-endpoint-how-to.md)
