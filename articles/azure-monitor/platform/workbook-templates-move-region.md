---
title: Modèles de classeur Azure Monitor – Déplacer des régions
description: Comment déplacer un modèle de classeur vers une autre région
services: azure-monitor
author: gardnerjr
manager: acearun
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: how-to
ms.custom: subject-moving-resources
ms.date: 08/12/2020
ms.author: jgardner
ms.openlocfilehash: f37550d914cf7efb0c75ed3dfa8854e1ec7be7e0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88875405"
---
# <a name="move-an-azure-workbook-template-to-another-region"></a>Déplacer un modèle de classeur Azure vers une autre région

Cet article décrit comment déplacer des ressources de modèle de classeur Azure vers une autre région Azure. Vous pouvez être amené à déplacer vos ressources vers une autre région pour diverses raisons. Par exemple, pour tirer parti d’une nouvelle région Azure, pour déployer des fonctionnalités ou des services disponibles dans des régions spécifiques uniquement, pour répondre à des exigences de stratégie et de gouvernance internes ou pour respecter des exigences de planification de la capacité.

Il n’existe actuellement aucune interface utilisateur de portail pour créer des ressources de modèle de classeur. Le seul moyen actuel de les créer est [via des déploiements de modèle Azure Resource Manager (modèle ARM)](./workbooks-automate.md). Par conséquent, le moyen le plus simple de déplacer un modèle consiste à réutiliser le modèle ARM précédent et à le mettre à jour pour le déployer dans la nouvelle région.

## <a name="prerequisites"></a>Prérequis

* Vérifiez que les modèles de classeur sont pris en charge dans la région cible.

## <a name="prepare"></a>Préparer

* Identifiez le modèle ARM précédemment utilisé pour le modèle de classeur.

## <a name="move"></a>Déplacer

1. Mettez à jour le modèle précédemment utilisé pour référencer la nouvelle région.

   > [!NOTE]
   > Il se peut que vous deviez utiliser un nouveau nom pour le modèle de classeur afin d’éviter les noms en double.

2. Déployez le modèle mis à jour via un déploiement de modèle ARM pour créer un modèle de classeur dans la région souhaitée.

## <a name="verify"></a>Vérification

Utilisez l’interface utilisateur de parcours des classeurs Azure pour localiser le nouveau modèle de classeur déployé. Vérifiez que l’emplacement est l’emplacement cible.

## <a name="clean-up"></a>Nettoyage

Une fois votre modèle de classeur créé dans la nouvelle région, supprimez le modèle de classeur d’origine dans la région précédente.
1. Recherchez le modèle de classeur dans l’interface utilisateur de parcours des classeurs Azure.
2. Sélectionnez le modèle de classeur à supprimer.
3. Sélectionnez la commande « Supprimer ».

Si vous avez renommé votre modèle de classeur pour l’importer dans une nouvelle région, vous pouvez lui réattribuer son nom précédent une fois l’élément d’origine supprimé à l’aide de la commande « Renommer » dans l’affichage des ressources de modèle de classeur Azure.

## <a name="next-steps"></a>Étapes suivantes

Vous devez déplacer un classeur à la place d’un modèle ? Découvrez comment [déplacer un classeur Azure vers une autre région](./workbooks-move-region.md).

