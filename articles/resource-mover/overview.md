---
title: Qu’est-ce qu’Azure Resource Mover ?
description: Découvrir Azure Resource Mover
author: rayne-wiselman
manager: evansma
ms.service: azure-migrate
ms.topic: overview
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 621c5e7b0061ccd76fd0109552107915b943511f
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/10/2020
ms.locfileid: "89653293"
---
# <a name="what-is-azure-resource-mover"></a>Qu’est-ce qu’Azure Resource Mover ?

Cet article donne une vue d’ensemble du service Azure Resource Mover. Resource Mover vous permet de déplacer des ressources Azure d’une région Azure à l’autre.

Vous pouvez déplacer des ressources dans d’autres régions Azure pour les raisons suivantes :

- **Aligner sur un lancement d’une région** : Déplacer des ressources vers une région Azure récemment introduite qui n’était pas disponible auparavant.
- Déplacer des ressources pour profiter des services ou fonctionnalités qui sont disponibles dans une région spécifique.
- **Répondre au développement de l’activité** : Déplacer des ressources vers une région en réponse à des changements de l’activité, comme des fusions ou des acquisitions.
- **Aligner pour cause de proximité** : Déplacer des ressources vers une région proche de votre entreprise.
- **Respecter les exigences relatives aux données** : Déplacer des ressources afin de respecter les exigences de résidence des données ou les besoins en matière de classification des données.
- **Respecter les exigences de déploiement** : Déplacer des ressources qui ont été déployées par erreur ou déplacez-les en réponse aux besoins de capacité.
- **Respecter la désaffectation** : Déplacer des ressources car une région est retirée.

> [!IMPORTANT]
> Azure Resource Mover est actuellement en préversion publique.

## <a name="why-use-resource-mover"></a>Pourquoi utiliser Azure Resource Mover ?

Resource Mover fournit :

- Un hub unique pour déplacer des ressources entre les régions.
- Une réduction de la complexité et du temps de déplacement. Tout ce dont vous avez besoin se trouve à un même emplacement.
- Une expérience simple et cohérente pour le déplacement des différents types de ressources Azure.
- Un moyen simple d’identifier les dépendances des ressources que vous souhaitez déplacer. Cela vous permet de déplacer les ressources associées ensemble, pour que tout fonctionne comme prévu dans la région cible après le déplacement.
- Le nettoyage automatique des ressources dans la région source, si vous souhaitez les supprimer après le déplacement.
- Test Vous pouvez essayer un déplacement, puis l’annuler si vous ne souhaitez pas effectuer un déplacement complet.

## <a name="move-across-regions"></a>Passer d’une région à l’autre

Pour déplacer des ressources entre les régions, vous sélectionnez celles que vous voulez déplacer. Resource Mover valide ces ressources, puis résout leurs dépendances vis-à-vis d’autres ressources. S’il existe des dépendances, vous avez deux possibilités :
- Déplacez les ressources dépendantes vers la région cible.
- Ne pas déplacer les ressources dépendantes, mais utiliser des ressources équivalentes dans la région cible à la place.

Une fois que toutes les dépendances sont résolues, Resource Mover vous guide tout au long d’un processus de déplacement simple.

1. Vous lancez un déplacement initial.
2. Après le déplacement initial, vous pouvez décider de valider et terminer le déplacement ou bien de l’abandonner.
3. Une fois le déplacement terminé, vous pouvez décider de supprimer les ressources situées à l’emplacement source.

Vous pouvez déplacer des ressources d’une région à l’autre dans le hub Resource Mover ou à partir d’un groupe de ressources. [En savoir plus](select-move-tool.md)

## <a name="what-resources-can-i-move-across-regions"></a>Quelles ressources puis-je déplacer d’une région à l’autre ?

À l’aide de Resource Mover, vous pouvez actuellement déplacer les ressources suivantes d’une région à l’autre :

- Machines virtuelles Azure et disques associés
- Cartes réseau
- Groupes à haute disponibilité 
- Réseaux virtuels Azure 
- Adresses IP publiques
- Groupes de sécurité réseau (NSG) :
- Équilibreurs de charge internes et publics 
- Bases de données Azure SQL et pools élastiques


## <a name="next-steps"></a>Étapes suivantes

[Découvrez-en plus](about-move-process.md) sur les composants de Resource Mover et le processus de déplacement.
