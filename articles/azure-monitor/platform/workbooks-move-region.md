---
title: Classeurs Azure Monitor – Déplacer des régions
description: Guide pratique pour déplacer un classeur vers une autre région
services: azure-monitor
author: gardnerjr
manager: acearun
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/12/2020
ms.author: jgardner
ms.openlocfilehash: d3b8bfbef544e754f684421daa847f1724435d53
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88875406"
---
# <a name="move-an-azure-workbook-to-another-region"></a>Déplacer un classeur Azure vers une autre région

Cet article explique comment déplacer des ressources de classeur Azure vers une autre région Azure. Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour déployer des fonctionnalités ou des services disponibles dans des régions spécifiques uniquement, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité.

## <a name="prerequisites"></a>Prérequis

* Vérifiez que les classeurs sont pris en charge dans la région cible.

* Ces instructions s’appliquent tant aux classeurs partagés (`microsoft.insights/workbooks`) qu’aux classeurs privés (`microsoft.insights/myworkbooks`) enregistrés dans Azure Monitor et sur la plupart des types de ressources.

  Toutefois, pour les classeurs spécifiquement liés au type de ressource Application Insights, ces classeurs sont stockés dans la région Azure où la ressource Application Insights est enregistrée.

  Ces classeurs ne peuvent pas être déplacés individuellement vers une autre région.

## <a name="move"></a>Déplacer

La façon la plus simple de déplacer un classeur Azure consiste à utiliser la commande « Enregistrer sous » de l’outil Classeurs de l’interface utilisateur du portail :

1. Ouvrez le classeur cible dans la visionneuse de classeur.
2. Utilisez le bouton de la barre d’outils « Modifier » pour passer en mode d’édition.
3. Utilisez le bouton de la barre d’outils « Enregistrer sous ».
4. Dans le formulaire d’enregistrement, choisissez un nom et la région souhaitée pour le classeur. Vérifiez que les autres champs pour l’abonnement, le groupe de ressources et le partage sont appropriés.

   > [!NOTE]
   > Il se peut que vous deviez utiliser un nouveau nom pour le classeur afin d’éviter d’avoir un nom en double.

5. Enregistrez. 

## <a name="verify"></a>Vérification

Utilisez l’interface utilisateur de parcours des classeurs Azure pour localiser le nouveau classeur. Vérifiez que l’emplacement est l’emplacement cible.

## <a name="clean-up"></a>Nettoyage

Une fois votre classeur créé dans la nouvelle région, supprimez le classeur d’origine dans la région précédente.
1. Ouvrez le classeur d’origine dans la visionneuse de classeur.
2. Utilisez le bouton de la barre d’outils « Modifier » pour passer en mode d’édition.
3. Dans la liste déroulante des outils d’édition (icône représentant un crayon), choisissez « Supprimer le classeur ».

Si vous avez renommé votre classeur pour l’importer dans une nouvelle région, vous pouvez lui réattribuer son nom précédent une fois le classeur d’origine supprimé à l’aide de l’élément « Renommer » dans la liste déroulante des outils de la barre d’outils Mode d’édition.

## <a name="next-steps"></a>Étapes suivantes

Vous devez déplacer un modèle de classeur plutôt qu’un classeur ? Découvrez comment [déplacer un modèle de classeur Azure vers une autre région](./workbook-templates-move-region.md).

Découvrez comment [déployer des classeurs et modèles de classeurs](./workbooks-automate.md) via des modèles ARM.
