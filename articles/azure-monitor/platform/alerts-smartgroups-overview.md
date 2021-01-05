---
title: Groupes intelligents
description: Les groupes intelligents sont des regroupements d’alertes qui vous aident à limiter leur nombre.
ms.topic: conceptual
ms.subservice: alerts
ms.date: 05/15/2018
ms.openlocfilehash: 743bd1a674c034cd6a0350f959289ac3ecb568de
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186607"
---
# <a name="smart-groups"></a>Groupes intelligents

En matière de gestion des alertes, il est souvent difficile de distinguer les alertes qui sont véritablement importantes au milieu du bruit de fond. Les groupes intelligents visent à résoudre ce problème.  

Les groupes intelligents sont automatiquement créés en utilisant des algorithmes de Machine Learning pour combiner des alertes associées qui correspondent à un même problème.  Quand une alerte est créée, l’algorithme l’ajoute à un groupe intelligent ou à un groupe intelligent, nouveau ou existant, en fonction d’informations telles que des modèles historiques, la similarité des propriétés ou la similarité de la structure. Par exemple, si le pourcentage d’UC augmente simultanément sur plusieurs machines virtuelles d’un abonnement, de nombreuses alertes individuelles sont créées. Avec les groupes intelligents, si ces alertes se sont déjà produites de la même manière par le passé, elles seront regroupées au sein d’un groupe intelligent, impliquant que la cause racine est peut-être commune. De plus, cela signifie que pour certaines alertes de dépannage, non seulement les groupes intelligents permettent de réduire le bruit de fond en gérant les alertes associées en tant qu’unité agrégée, mais ils proposent une potentielle cause racine commune à ces alertes.

L’algorithme ne prend actuellement en compte que les alertes du même service d’analyse au sein d’un abonnement. Grâce à cette consolidation, les groupes intelligents peuvent réduire jusqu’à 99 % du bruit d’alerte. Vous pouvez connaître la raison pour laquelle des alertes ont été incluses dans un groupe dans la page de détails du groupe intelligent.

Vous pouvez afficher les détails des groupes intelligents et en définir l’état comme vous le pouvez pour des alertes. Chaque alerte est un membre d’un seul groupe intelligent. 

## <a name="smart-group-state"></a>État du groupe intelligent

L’état du groupe intelligent est un concept similaire à l’état de l’alerte, ce qui vous permet de gérer le processus de résolution au niveau d’un groupe intelligent. Comme pour l’état d’une alerte, lorsqu’un groupe intelligent est créé, il dispose de l’état **Nouveau**, qui peut être modifié sur **Reconnu** ou **Fermé**.

Les états de groupe intelligent suivants sont pris en charge.

| State | Description |
|:---|:---|
| Nouveau | Le problème vient d’être détecté et n’a pas encore été examiné. |
| Reconnu | Un administrateur a révisé le groupe intelligent et a commencé à y travailler. |
| Fermés | Le problème a été résolu. Après qu’un groupe intelligent a été fermé, vous pouvez le rouvrir en modifiant son état. |

[Découvrez comment modifier l’état de votre groupe intelligent.](./alerts-managing-alert-states.md?toc=%2fazure%2fazure-monitor%2ftoc.json)

> [!NOTE]
>  La modification de l'état d'un groupe intelligent ne modifie pas l'état des alertes de membre individuelles.

## <a name="smart-group-details-page"></a>Page de détails du groupe intelligent

La page Détails du groupe intelligent s’affiche lorsque vous sélectionnez un groupe intelligent. Elle fournit des détails sur le groupe intelligent, dont le raisonnement utilisé pour créer le groupe, et vous permet d’en modifier l’état.
 
![Détails du groupe intelligent](media/alerts-smartgroups-overview/smart-group-detail.png)


La page Détails du groupe intelligent comprend les sections suivantes.

| Section | Description |
|:---|:---|
| Alertes | Répertorie les alertes individuelles qui sont incluses dans le groupe intelligent. Sélectionnez une alerte pour ouvrir sa page de détails. |
| Historique | Répertorie chaque action effectuée par le groupe intelligent, et toutes les modifications qui y sont apportées. Il se limite actuellement aux changements d’état et aux changements d’appartenance d’alerte. |

## <a name="smart-group-taxonomy"></a>Taxonomie du groupe intelligent

Le nom d’un groupe intelligent correspond au nom de sa première alerte. Vous ne pouvez pas créer ou renommer un groupe intelligent.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les groupes intelligents](./alerts-managing-smart-groups.md?toc=%2fazure%2fazure-monitor%2ftoc.json)
- [Modifier les états des alertes et des groupes intelligents](./alerts-managing-alert-states.md?toc=%2fazure%2fazure-monitor%2ftoc.json)