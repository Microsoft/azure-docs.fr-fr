---
title: Présentation du cycle de vie d’un blueprint
description: Découvrez le cycle de vie que traverse une définition de blueprint ainsi que les détails sur chaque phase, notamment la mise à jour et la suppression des affectations blueprint.
ms.date: 08/27/2020
ms.topic: conceptual
ms.openlocfilehash: c1de2cf869362154b112c3eef74ed1fb0db3a993
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89049704"
---
# <a name="understand-the-lifecycle-of-an-azure-blueprint"></a>Présentation du cycle de vie d’un blueprint Azure

Comme de nombreuses ressources dans Azure, les blueprints dans Azure Blueprints ont un cycle de vie normal et naturel. Ils sont créés, déployés puis supprimés quand ils ne sont plus nécessaires ou pertinents. Azure Blueprints prend en charge les opérations de cycle de vie standard. Ils s’appuient ensuite sur celles-ci pour fournir des niveaux supplémentaires d’état prenant en charge les pipelines d’intégration et de déploiement continus communs pour les organisations qui gèrent leur infrastructure en tant que code (élément clé dans DevOps).

Pour bien comprendre ce qu’est un blueprint et quelles sont les phases de son cycle de vie, étudions d’abord un cycle de vie standard :

> [!div class="checklist"]
> - Création et modification d’un blueprint
> - Publication du blueprint
> - Création et modification d’une nouvelle version du blueprint
> - Publication d’une nouvelle version du blueprint
> - Suppression d’une version spécifique du blueprint
> - Suppression du blueprint

## <a name="creating-and-editing-a-blueprint"></a>Création et modification d’un blueprint

Quand vous créez un blueprint, vous y ajoutez des artefacts, vous l’enregistrez dans un groupe d’administration ou un abonnement, et vous spécifiez un nom unique et une version unique. Le blueprint est à présent en mode **Brouillon** et ne peut pas encore être affecté. Pendant qu’il est en mode **Brouillon**, il peut continuer d’être mis à jour et modifié.

Un blueprint en mode **Brouillon** qui n’a jamais été publié affiche une icône dans la page **Définitions de blueprint** qui est différente de ceux qui ont été **publiés**. Le champ **Dernière version** indique **Brouillon** pour les blueprints qui n’ont jamais été publiés.

Vous créez et modifiez un blueprint avec le [portail Azure](../create-blueprint-portal.md#create-a-blueprint) ou l’[API REST](../create-blueprint-rest-api.md#create-a-blueprint).

## <a name="publishing-a-blueprint"></a>Publication d’un blueprint

Une fois que tous les changements planifiés ont été apportés à un blueprint en mode **Brouillon**, vous pouvez le **publier** et l’affecter. La version **publiée** du blueprint ne peut pas être modifiée. Une fois **publié**, le blueprint est affecté d’une icône différente des blueprints en mode **Brouillon**, et le numéro de version fourni est affiché dans la colonne **Dernière version**.

Vous publiez un blueprint avec le [portail Azure](../create-blueprint-portal.md#publish-a-blueprint) ou l’[API REST](../create-blueprint-rest-api.md#publish-a-blueprint).

## <a name="creating-and-editing-a-new-version-of-the-blueprint"></a>Création et modification d’une nouvelle version du blueprint

Une version **publiée** d’un blueprint ne peut pas être modifiée. Toutefois, une nouvelle version du blueprint peut être ajoutée au blueprint existant et modifiée en fonction des besoins. Apportez des changements à un blueprint existant. Quand les nouveaux changements sont enregistrés, le blueprint affiche **Changements non publiés**. Ces changements correspondent à une nouvelle version **Brouillon** du blueprint.

Vous modifiez un blueprint avec le [portail Azure](../create-blueprint-portal.md#edit-a-blueprint).

## <a name="publishing-a-new-version-of-the-blueprint"></a>Publication d’une nouvelle version du blueprint

Chaque version modifiée d’un blueprint doit être **publiée** avant de pouvoir être affectée. Quand des **changements non publiés** ont été apportés à un blueprint, mais n’ont pas encore été **publiés**, le bouton **Publier le blueprint** est disponible dans la page Modifier le blueprint. Si le bouton n’est pas visible, cela signifie que le blueprint a déjà été **publié** et qu’il ne comporte aucun **changement non publié**.

> [!NOTE]
> Un même blueprint peut avoir plusieurs versions **publiées** pouvant chacune être affectées à des abonnements.

Pour publier un blueprint avec des **changements non publiés**, utilisez les mêmes étapes que pour la publication d’un nouveau blueprint.

## <a name="deleting-a-specific-version-of-the-blueprint"></a>Suppression d’une version spécifique du blueprint

Chaque version d’un blueprint est un objet unique qui peut être **publié** individuellement. Par conséquent, chaque version d’un blueprint peut aussi être supprimée. La suppression d’une version d’un blueprint n’a aucun impact sur les autres versions de ce blueprint.

> [!NOTE]
> Il n’est pas possible de supprimer un blueprint ayant des affectations actives. Supprimez d’abord les affectations, puis supprimez la version du blueprint.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez **Définitions de blueprint** dans la page de gauche et utilisez les options de filtre pour trouver le blueprint dont vous souhaitez supprimer une version. Sélectionnez-le pour ouvrir la page de modification.

1. Sélectionnez l’onglet **Versions publiées** et recherchez la version que vous souhaitez supprimer.

1. Cliquez avec le bouton droit sur la version à supprimer, puis sélectionnez **Supprimer cette version**.

## <a name="deleting-the-blueprint"></a>Suppression du blueprint

Le blueprint de base peut également être supprimé. La suppression du blueprint de base entraîne la suppression de toutes les versions de ce blueprint, notamment celles dont l’état est **Brouillon** et **Publié**. Comme avec la suppression d’une version d’un blueprint, la suppression du blueprint de base ne supprime pas les affectations existantes des versions de blueprint.

> [!NOTE]
> Il n’est pas possible de supprimer un blueprint ayant des affectations actives. Supprimez d’abord les affectations, puis supprimez la version du blueprint.

Vous supprimez un blueprint avec le [portail Azure](../create-blueprint-portal.md#delete-a-blueprint) ou l’[API REST](../create-blueprint-rest-api.md#delete-a-blueprint).

## <a name="assignments"></a>Attributions

Le blueprint peut être attribué à un abonnement à plusieurs endroits du cycle de vie. Quand le mode d’une version du blueprint est **Publié**, cette version peut être affectée à un abonnement. Ce cycle de vie permet d’utiliser et d’attribuer activement des versions d’un blueprint pendant le développement d’une nouvelle version.

Lors de l’affectation des versions de blueprints, il est important de comprendre où elles sont affectées et avec quels paramètres elles ont été affectées. Les paramètres peuvent être statiques ou dynamiques. Pour plus d’informations, consultez [Paramètres statiques et dynamiques](./parameters.md).

### <a name="updating-assignments"></a>Mise à jour des affectations

Quand un blueprint est affecté, l’affectation peut être mise à jour. Plusieurs raisons peuvent motiver la mise à jour d’une affectation existante, notamment quand il s’agit de :

- Ajouter ou supprimer un [verrouillage de ressource](./resource-locking.md)
- Modifier la valeur de [paramètres dynamiques](./parameters.md#dynamic-parameters)
- Mettre à niveau l’affectation vers une version **publiée** plus récente du blueprint

Pour découvrir comment procéder, consultez [Mettre à jour des affectations existantes](../how-to/update-existing-assignments.md)

### <a name="unassigning-assignments"></a>Annulation de l’affectation des affectations

Si le blueprint n’est plus nécessaire, son attribution peut être annulée à partir du groupe d’administration ou de l’abonnement. Lors de la désaffectation d’un plan, voici ce qui se produit :

- Suppression du [verrouillage des ressources de blueprint](./resource-locking.md)
- Suppression de l’objet d’affectation de blueprint
- (Conditionnel) Si une **identité gérée affectée par le système** a été utilisée, elle est également supprimée

> [!NOTE]
> Toutes les ressources déployées par l’affectation du blueprint sont conservées, mais elles ne sont plus protégées par Azure Blueprints.

## <a name="next-steps"></a>Étapes suivantes

- Comprendre comment utiliser les [paramètres statiques et dynamiques](./parameters.md).
- Apprendre à personnaliser l’[ordre de séquencement des blueprints](./sequencing-order.md).
- Découvrir comment utiliser le [verrouillage de ressources de blueprint](./resource-locking.md).
- Découvrir comment [mettre à jour des affectations existantes](../how-to/update-existing-assignments.md).
- Résoudre les problèmes durant l’affectation d’un blueprint en suivant les étapes de [dépannage général](../troubleshoot/general.md).