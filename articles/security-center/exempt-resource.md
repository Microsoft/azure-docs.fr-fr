---
title: Exempter une ressource des recommandations de sécurité et du degré de sécurisation d’Azure Security Center
description: Découvrez comment exempter une ressource des recommandations de sécurité et du degré de sécurisation
author: memildin
ms.author: memildin
ms.date: 9/22/2020
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: b7780a0ef70a89a88070d5883cc840319a67fa3d
ms.sourcegitcommit: b8a175b6391cddd5a2c92575c311cc3e8c820018
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96122365"
---
# <a name="exempt-a-resource-from-recommendations-and-secure-score"></a>Exempter une ressource des recommandations et du degré de sécurisation

Une priorité de base de chaque équipe de sécurité tente de s’assurer que les analystes peuvent se concentrer sur les tâches et les incidents importants pour l’entreprise. Security Center dispose de nombreuses fonctionnalités permettant de personnaliser les informations que vous classez par ordre de priorité et s’assurer que votre degré de sécurisation reflète précisément les décisions de sécurité prises par votre organisation. L’exemption de ressources est une fonctionnalité de ce type.

Quand vous examinez une recommandation de sécurité dans Azure Security Center, l’une des premières informations que vous examinez est la liste des ressources affectées.

Parfois, une ressource sera répertoriée et ne devrait pas être incluse. Elle a peut-être été corrigée par un processus non suivi par Security Center. Ou bien votre organisation a décidé d’accepter le risque pour cette ressource spécifique. 

Dans ce cas, vous pouvez créer une règle d’exemption et veiller à ce que la ressource ne figure plus parmi les ressources non saines à l’avenir et n’affecte pas votre degré de sécurisation. 

La ressource sera répertoriée comme non applicable, avec le motif « exemptée » et la justification de votre choix.

## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|PRÉVERSION<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Prix :|Il s’agit d’une fonctionnalité de stratégie Azure Premium proposée sans surcoût aux clients Azure Defender. Pour les autres utilisateurs, des frais pourront s’appliquer.|
|Rôles et autorisations obligatoires :|**Propriétaire de l’abonnement** ou **Contributeur de stratégie** pour créer une exemption<br>Pour créer une règle, vous devez disposer de l’autorisation de modifier des stratégies dans Azure Policy.<br>Pour plus d’informations, consultez [Autorisations Azure RBAC dans Azure Policy](../governance/policy/overview.md#azure-rbac-permissions-in-azure-policy).|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||


## <a name="create-an-exemption-rule"></a>Créer une règle d’exemption

1. Dans la liste des ressources non saines, sélectionnez le menu de points de suspension (« ... ») pour la ressource que vous souhaitez exempter.

    :::image type="content" source="./media/exempt-resource/create-exemption.png" alt-text="Option Créer une exemption du menu contextuel":::

    Le volet Créer une exemption s’ouvre.

    :::image type="content" source="./media/exempt-resource/exemption-rule-options.png" alt-text="Volet Créer une exemption":::

1. Entrez vos critères, puis sélectionnez la raison pour laquelle cette ressource doit être exemptée :
    - **Atténuée** : ce problème ne s’applique pas à la ressource, car il a été traité par un outil ou un processus différent de celui suggéré
    - **Dérogation** : acceptation du risque pour cette ressource
1. Sélectionnez **Enregistrer**.
1. Après un certain temps (jusqu’à 24 heures) :
    - La ressource n’a pas d’impact sur votre degré de sécurisation.
    - La ressource apparaît dans l’onglet **Non applicable** de la page des détails de la recommandation
    - La bande d’informations en haut de la page des détails de la recommandation indique le nombre de ressources exemptées :
        
        :::image type="content" source="./media/exempt-resource/info-banner.png" alt-text="Nombre de ressources exemptées":::

1. Pour passer en revue vos ressources exemptées, ouvrez l’onglet **Non applicable** .

    :::image type="content" source="./media/exempt-resource/modifying-exemption.png" alt-text="Modification d’une exemption":::

    La raison de chaque exemption est spécifiée dans le tableau (1).

    Pour modifier ou supprimer une exemption, sélectionnez le menu de points de suspension (« ... ») comme indiqué (2).


## <a name="review-all-of-the-exemption-rules-on-your-subscription"></a>Passer en revue toutes les règles d’exemption de votre abonnement

Les règles d’exemption utilisent une stratégie Azure afin de créer une exemption pour la ressource sur l’attribution de stratégie.

Vous pouvez utiliser Azure Policy pour suivre toute votre exemption dans la page **Exemption** :

:::image type="content" source="./media/exempt-resource/policy-page-exemption.png" alt-text="Page d’exemption d’Azure Policy":::



## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à exempter une ressource d’une recommandation afin qu’elle n’affecte pas votre degré de sécurisation. Pour plus d’informations sur le degré de sécurisation, voir :

- [Degré de sécurisation dans Azure Security Center](secure-score-security-controls.md)