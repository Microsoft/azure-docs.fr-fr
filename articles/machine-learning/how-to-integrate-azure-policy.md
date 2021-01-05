---
title: Auditer et gérer la conformité à la stratégie
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser Azure Policy pour utiliser des stratégies intégrées pour Azure Machine Learning afin de vous assurer que vos espaces de travail sont conformes à vos besoins.
author: jhirono
ms.author: jhirono
ms.date: 09/15/2020
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.openlocfilehash: 22901c4e8409fc4846c1566a57b2679f4fa92396
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94444558"
---
# <a name="audit-and-manage-azure-machine-learning-using-azure-policy"></a>Auditer et gérer Azure Machine Learning à l'aide d'Azure Policy

[Azure Policy](../governance/policy/index.yml) est un outil de gouvernance qui vous permet de vous assurer que les ressources Azure sont conformes à vos stratégies. Avec Azure Machine Learning, vous pouvez attribuer les stratégies suivantes :

* **Clé gérée par le client** : auditez ou appliquez une valeur indiquant si les espaces de travail doivent utiliser une clé gérée par le client.
* **Liaison privée** : procédez à un audit pour déterminer si les espaces de travail utilisent un point de terminaison privé pour communiquer avec un réseau virtuel.

Les stratégies peuvent être définies sur des étendues différentes, par exemple au niveau de l'abonnement ou du groupe de ressources. Pour plus d'informations, consultez la [documentation relative à Azure Policy](../governance/policy/overview.md).

## <a name="built-in-policies"></a>Stratégies prédéfinies

Azure Machine Learning fournit un ensemble de stratégies que vous pouvez utiliser dans des scénarios courants. Vous pouvez attribuer ces définitions de stratégie à votre abonnement existant ou les utiliser comme base pour créer vos propres définitions personnalisées. Pour obtenir la liste complète des stratégies intégrées pour Azure Machine Learning, consultez [Stratégies intégrées pour Azure Machine Learning](../governance/policy/samples/built-in-policies.md#machine-learning).

Pour consulter les définitions de stratégie intégrées en rapport avec Azure Machine Learning, procédez comme suit :

1. Accédez à __Azure Policy__ sur le [portail Azure](https://portal.azure.com).
1. Sélectionnez __Définitions__.
1. Dans le champ __Type__, sélectionnez _Intégré_ et dans le champ __Catégorie__, sélectionnez __Machine Learning__.

De là, vous pouvez sélectionner les définitions de stratégie à consulter. Lorsque vous consultez une définition, vous pouvez utiliser le lien __Attribuer__ pour attribuer la stratégie à une étendue spécifique et configurer les paramètres de cette stratégie. Pour plus d'informations, consultez [Attribuer une stratégie - portail](../governance/policy/assign-policy-portal.md).

Vous pouvez également attribuer des stratégies à l'aide d'[Azure PowerShell](../governance/policy/assign-policy-powershell.md), d'[Azure CLI](../governance/policy/assign-policy-azurecli.md) et de [modèles](../governance/policy/assign-policy-template.md).

## <a name="workspaces-encryption-with-customer-managed-key"></a>Chiffrement des espaces de travail à l'aide d'une clé gérée par le client

Détermine si les espaces de travail doivent être chiffrés à l’aide d’une clé gérée par le client, ou si une clé gérée par Microsoft doit être utilisée pour chiffrer les métriques et les métadonnées. Pour plus d’informations sur l’utilisation d’une clé gérée par le client, consultez la section [Azure Cosmos DB](concept-data-encryption.md#azure-cosmos-db) de l’article consacré au chiffrement des données.

Pour configurer cette stratégie, définissez le paramètre d'effet sur __audit__ ou __deny__. Si le paramètre est défini sur __audit__, vous pouvez créer des espaces de travail sans clé gérée par le client ; un événement d’avertissement est alors créé dans le journal d’activité.

Si la stratégie est définie sur __deny__, vous ne pouvez pas créer d’espace de travail, sauf si une clé gérée par le client est spécifiée. Toute tentative de création d’un espace de travail sans clé gérée par le client génère une erreur semblable à `Resource 'clustername' was disallowed by policy` et crée une erreur dans le journal d’activité. L'identificateur de la stratégie est également renvoyé dans le cadre de cette erreur.

## <a name="workspaces-should-use-private-link"></a>Les espaces de travail doivent utiliser Private Link

Détermine si les espaces de travail doivent utiliser Azure Private Link pour communiquer avec le Réseau virtuel Azure. Pour plus d'informations sur l'utilisation de Private Link, consultez [Configurer Private Link pour un espace de travail](how-to-configure-private-link.md).

Pour configurer cette stratégie, définissez le paramètre d'effet sur __audit__. Si vous créez un espace de travail sans utiliser Private Link, un événement d'avertissement est créé dans le journal d'activité.

## <a name="next-steps"></a>Étapes suivantes

* [Documentation Azure Policy](../governance/policy/overview.md)
* [Stratégies intégrées pour Azure Machine Learning](policy-reference.md)
* [Utiliser des stratégies de sécurité avec Azure Security Center](../security-center/tutorial-security-policy.md)