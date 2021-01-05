---
title: Intégrer un groupe d’administration à Azure Security Center
description: Découvrez comment utiliser une définition Azure Policy fournie afin d’activer Azure Security Center pour tous les abonnements d’un groupe d’administration.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/10/2020
ms.author: memildin
ms.openlocfilehash: 4ecd436b548c29c520a7538970d4d703cc8488d2
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93027565"
---
# <a name="enable-security-center-on-all-subscriptions-in-a-management-group"></a>Activer Security Center sur tous les abonnements d’un groupe d’administration

Vous pouvez utiliser Azure Policy pour activer Azure Security Center sur tous les abonnements Azure au sein du même groupe d’administration. Cela est plus pratique que d’accéder aux abonnements individuellement à partir du portail et fonctionne même s’ils appartiennent à des propriétaires différents. 

Pour intégrer un groupe d’administration et tous ses abonnements :

1. En tant qu’utilisateur disposant des autorisations **Administration de sécurité** , ouvrez Azure Policy et recherchez la définition **Activer Azure Security Center sur votre abonnement**.

    :::image type="content" source="./media/security-center-get-started/enable-security-center-policy.png" alt-text="Définition Azure Policy Activer Azure Security Center sur votre abonnement":::

1. Sélectionnez **Affecter** et veillez à définir l’étendue sur le niveau du groupe d’administration.

    :::image type="content" source="./media/security-center-get-started/assign-policy.png" alt-text="Affectation de la définition Activer Azure Security Center sur votre abonnement":::

    > [!TIP]
    > Hormis l’étendue, il n’existe aucun paramètre obligatoire.

1. Sélectionnez **Créer une tâche de correction** afin que soient intégrés tous les abonnements existants pour lesquels Security Center n’est pas activé.

    :::image type="content" source="./media/security-center-get-started/remediation-task.png" alt-text="Création d’une tâche de correction pour la définition Azure Policy Activer Azure Security Center sur votre abonnement":::

1. Quand la définition est assignée, elle effectue les opérations suivantes :

    1. Elle détecte tous les abonnements du groupe d’administration qui ne sont pas encore inscrits auprès de Security Center.
    1. Elle marque ces abonnements comme étant « non conformes ».
    1. Elle marque comme « conformes » tous les abonnements inscrits (qu’ils aient ou non Azure Defender activé ou désactivé).

    La tâche de correction active ensuite Security Center, gratuitement, sur les abonnements non conformes.

> [!IMPORTANT]
> La définition de stratégie active uniquement Security Center sur les **abonnements** existants. Pour inscrire les nouveaux abonnements, ouvrez l’onglet Conformité, sélectionnez les abonnements non conformes appropriés et créez une tâche de correction. Répétez cette étape si vous souhaitez superviser un ou plusieurs nouveaux abonnements avec Security Center.

## <a name="optional-modifications"></a>Modifications facultatives

Vous pouvez modifier la définition Azure Policy de plusieurs façons : 

- **Définir la conformité différemment**  : la stratégie fournie classifie comme « non conformes » tous les abonnements du groupe d’administration qui ne sont pas encore inscrits auprès de Security Center. Vous pouvez choisir de la définir sur tous les abonnements sans Azure Defender.

    La définition fournie définit *un* des paramètres de tarif ci-dessous comme étant conforme. Cela signifie qu’un abonnement défini sur « standard » ou « free » (gratuit) est conforme.

    > [!TIP]
    > Quand un plan Azure Defender est activé, il est décrit comme étant associé au paramètre « Standard ». Quand il est désactivé, il est « Gratuit ». [Découvrez-en plus sur les plans Azure Defender](security-center-pricing.md).

    ```
    "existenceCondition": {
        "anyof": [
            {
                "field": "microsoft.security/pricings/pricingTier",
                "equals": "standard"
            },
            {
                "field": "microsoft.security/pricings/pricingTier",
                "equals": "free"
            }
        ]
    },
    ```

    Si vous le remplacez par ce qui suit, seuls les abonnements définis sur « standard » sont classifiés comme étant conformes :

    ```
    "existenceCondition": {
          {
            "field": "microsoft.security/pricings/pricingTier",
            "equals": "standard"
          },
    },
    ```

- **Définir des plans Azure Defender à appliquer lors de l’activation de Security Center**  : la stratégie fournie active Security Center sans aucun des plans Azure Defender facultatifs. Vous pouvez choisir d’activer un ou plusieurs d’entre eux.

    La section `deployment` de la définition fournie a un paramètre `pricingTier`. Par défaut, ce paramètre est défini sur `free`, mais vous pouvez le modifier. 


## <a name="next-steps"></a>Étapes suivantes :

Maintenant que vous avez intégré un groupe d’administration entier, activez les protections avancées d’Azure Defender. 

> [!div class="nextstepaction"]
> [Activation d’Azure Defender](security-center-pricing.md)
