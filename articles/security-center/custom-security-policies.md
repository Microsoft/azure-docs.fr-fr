---
title: Créer des stratégies de sécurité personnalisées dans Azure Security Center | Microsoft Docs
description: Définitions de stratégie Azure personnalisées surveillées dans Azure Security Center.
services: security-center
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 12/03/2020
ms.author: memildin
ms.openlocfilehash: 8d2b43ab57ea7a3b1dc1d13bcdea9932ccecb9dc
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96559029"
---
# <a name="using-custom-security-policies"></a>Utilisation de stratégies de sécurité personnalisées

Pour vous aider à sécuriser vos systèmes et votre environnement, Azure Security Center génère des recommandations de sécurité. Ces recommandations sont basées sur les meilleures pratiques du secteur, qui sont incorporées à la stratégie de sécurité par défaut générique fournie à tous les clients. Elles peuvent également provenir des connaissances que Security Center a des normes et réglementations du secteur.

Avec cette fonctionnalité, vous pouvez ajouter vos propres initiatives *personnalisées*. Vous recevez ensuite des recommandations si votre environnement ne suit pas les stratégies que vous créez. Toutes les initiatives personnalisées que vous créez apparaîtront à côté des initiatives intégrées dans le tableau de bord de conformité à la réglementation, comme décrit dans le tutoriel [Améliorer votre conformité aux normes](security-center-compliance-dashboard.md).

Comme nous l’avons vu dans la [documentation Azure Policy](../governance/policy/concepts/definition-structure.md#definition-location), quand vous spécifiez un emplacement pour votre initiative personnalisée, il doit correspondre à un groupe d’administration ou à un abonnement. 

## <a name="to-add-a-custom-initiative-to-your-subscription"></a>Pour ajouter une initiative personnalisée à votre abonnement 

1. Dans la barre latérale de Security Center, ouvrez la page **Stratégie de sécurité**.

1. Sélectionnez un abonnement ou un groupe d’administration auquel vous voulez ajouter une initiative personnalisée.

    [![Sélection d’un abonnement pour lequel vous allez créer votre stratégie personnalisée](media/custom-security-policies/custom-policy-selecting-a-subscription.png)](media/custom-security-policies/custom-policy-selecting-a-subscription.png#lightbox)

    > [!NOTE]
    > Vous devez ajouter des standards personnalisés au niveau de l’abonnement (ou à un niveau supérieur) pour qu’ils soient évalués et affichés dans Security Center. 
    >
    > Lorsque vous ajoutez une norme personnalisée, elle attribue une *initiative* à cette étendue. Nous vous recommandons donc de sélectionner l’étendue la plus vaste requise pour cette attribution.

1. Dans la page Stratégie de sécurité, sous vos initiatives personnalisées, cliquez sur **Ajouter une initiative personnalisée**.

    [![Cliquez sur Ajouter une initiative personnalisée](media/custom-security-policies/custom-policy-add-initiative.png)](media/custom-security-policies/custom-policy-add-initiative.png#lightbox)

    La page suivante apparaît :

    ![Créer ou ajouter une stratégie](media/custom-security-policies/create-or-add-custom-policy.png)

1. Dans la page Ajouter des initiatives personnalisées, passez en revue la liste de stratégies personnalisées déjà créées dans votre organisation. Si vous en voyez une que vous souhaitez attribuer à votre abonnement, cliquez sur **Ajouter**. Si aucune initiative de la liste ne répond à vos besoins, ignorez cette étape.

1. Pour créer une nouvelle initiative personnalisée :

    1. Cliquez sur **Créer une nouvelle**.
    1. Entrez l’emplacement et le nom de la définition.
    1. Sélectionnez les stratégies à inclure, puis cliquez sur **Ajouter**.
    1. Entrez les paramètres souhaités.
    1. Cliquez sur **Enregistrer**.
    1. Dans la page Ajouter des initiatives personnalisées, cliquez sur Actualiser. Votre nouvelle initiative apparaîtra comme étant disponible.
    1. Cliquez sur **Ajouter** et affectez-la à votre abonnement.

    > [!NOTE]
    > La création d’initiatives nécessite les informations d’identification du propriétaire de l’abonnement. Pour plus d’informations sur les rôles Azure, consultez [Autorisations dans Azure Security Center](security-center-permissions.md).

    Votre nouvelle initiative est appliquée et vous pouvez visualiser son impact de deux façons :

    * Dans la barre latérale de Security Center, sous Stratégie et conformité, sélectionnez **Conformité avec la réglementation**. Le tableau de bord de conformité s'ouvre pour montrer votre nouvelle initiative personnalisée à côté des initiatives intégrées.
    
    * Vous commencerez ensuite à recevoir des recommandations si votre environnement ne suit pas les stratégies que vous avez définies.

1. Pour afficher les recommandations qui en résultent pour votre stratégie, cliquez sur **Recommandations** dans la barre latérale pour ouvrir la page de recommandations. Les recommandations apparaissent avec une étiquette « Personnalisée » et sont disponibles dans un délai d’une heure environ.

    [![Recommandations personnalisées](media/custom-security-policies/custom-policy-recommendations.png)](media/custom-security-policies/custom-policy-recommendations-in-context.png#lightbox)

## <a name="enhance-your-custom-recommendations-with-detailed-information"></a>Améliorez vos recommandations personnalisées grâce à des informations détaillées

Les recommandations intégrées fournies avec Azure Security Center incluent des détails tels que des niveaux de gravité et des instructions de correction. Si vous souhaitez ajouter ce type d’informations à vos recommandations personnalisées afin qu’elles apparaissent dans le portail Azure ou à l’endroit où vous accédez à vos recommandations, vous devez utiliser l’API REST. 

Les deux types d’informations que vous pouvez ajouter sont les suivants :

- **RemediationDescription** : chaîne
- **Severity** – Enum [Low, Medium, High]

Les métadonnées doivent être ajoutées à la définition de stratégie pour une stratégie qui fait partie de l’initiative personnalisée. Elles doivent figurer dans la propriété « securityCenter », comme indiqué ci-dessous :

```json
 "metadata": {
    "securityCenter": {
        "RemediationDescription": "Custom description goes here",
        "Severity": "High"
    },
```

Voici un exemple de stratégie personnalisée incluant la propriété metadata/securityCenter :

  ```json
  {
"properties": {
    "displayName": "Security - ERvNet - AuditRGLock",
    "policyType": "Custom",
    "mode": "All",
    "description": "Audit required resource groups lock",
    "metadata": {
        "securityCenter": {
            "RemediationDescription": "Resource Group locks can be set via Azure Portal -> Resource Group -> Locks",
            "Severity": "High"
        }
    },
    "parameters": {
        "expressRouteLockLevel": {
            "type": "String",
            "metadata": {
                "displayName": "Lock level",
                "description": "Required lock level for ExpressRoute resource groups."
            },
            "allowedValues": [
                "CanNotDelete",
                "ReadOnly"
            ]
        }
    },
    "policyRule": {
        "if": {
            "field": "type",
            "equals": "Microsoft.Resources/subscriptions/resourceGroups"
        },
        "then": {
            "effect": "auditIfNotExists",
            "details": {
                "type": "Microsoft.Authorization/locks",
                "existenceCondition": {
                    "field": "Microsoft.Authorization/locks/level",
                    "equals": "[parameters('expressRouteLockLevel')]"
                }
            }
        }
    }
}
}
  ```

Pour obtenir un autre exemple d’utilisation de la propriété securityCenter, consultez [cette section de la documentation de l’API REST](/rest/api/securitycenter/assessmentsmetadata/createinsubscription#examples).


## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à créer des stratégies de sécurité personnalisées. 

Pour d’autres informations connexes, consultez les articles suivants : 

- [La vue d’ensemble des stratégies de sécurité](tutorial-security-policy.md)
- [Liste des stratégies de sécurité intégrées](./policy-reference.md)