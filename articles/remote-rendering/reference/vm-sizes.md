---
title: Tailles des serveurs
description: Décrit les tailles de serveur distinctes qui peuvent être allouées.
author: florianborn71
ms.author: flborn
ms.date: 05/28/2020
ms.topic: reference
ms.custom: devx-track-csharp
ms.openlocfilehash: b37aabb39e19fa5ec53d2b006a7cbc1793adad72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90988040"
---
# <a name="server-sizes"></a>Tailles des serveurs

Le service Azure Remote Rendering est disponible dans deux configurations de serveur : `Standard` et `Premium`.

## <a name="polygon-limits"></a>Limites de polygones

Le service Remote Rendering avec un serveur de taille `Standard` a une taille de scène maximale de 20 millions de polygones. Le service Remote Rendering de taille `Premium` n’impose pas de maximum inconditionnel, mais les performances peuvent de dégrader si votre contenu dépasse les capacités de rendu du service.

Quand le convertisseur sur un serveur de taille « standard » atteint cette limitation, il bascule le rendu vers un damier d’arrière-plan :

![La capture d’écran présente une grille de carrés noirs et blancs avec un menu Outils.](media/checkerboard.png)

## <a name="specify-the-server-size"></a>Spécifier la taille du serveur

Le type souhaité de configuration de serveur doit être spécifié au moment de l’initialisation de la session de rendu. Il ne peut pas être modifié pendant une session en cours d’exécution. Les exemples de code suivants illustrent l’emplacement où la taille de serveur doit être spécifiée :

```cs
async void CreateRenderingSession(AzureFrontend frontend)
{
    RenderingSessionCreationParams sessionCreationParams = new RenderingSessionCreationParams();
    sessionCreationParams.Size = RenderingSessionVmSize.Standard; // or  RenderingSessionVmSize.Premium

    AzureSession session = await frontend.CreateNewRenderingSessionAsync(sessionCreationParams).AsTask();
}
```

```cpp
void CreateRenderingSession(ApiHandle<AzureFrontend> frontend)
{
    RenderingSessionCreationParams sessionCreationParams;
    sessionCreationParams.Size = RenderingSessionVmSize::Standard; // or  RenderingSessionVmSize::Premium

    if (auto createSessionAsync = frontend->CreateNewRenderingSessionAsync(sessionCreationParams))
    {
        // ...
    }
}
```

Pour les [exemples de scripts PowerShell](../samples/powershell-example-scripts.md), la taille de serveur souhaitée doit être spécifiée dans le fichier `arrconfig.json` :

```json
{
  "accountSettings": {
    ...
  },
  "renderingSessionSettings": {
    "vmSize": "<standard or premium>",
    ...
  },
```

### <a name="how-the-renderer-evaluates-the-number-of-polygons"></a>Comment le convertisseur évalue-t-il le nombre de polygones ?

Le nombre de polygones pris en compte pour le test de limitation est le nombre de polygones qui sont réellement transmis au convertisseur. Cette géométrie correspond généralement à la somme de tous les modèles instanciés, mais il existe également des exceptions. La géométrie suivante n’est **pas incluse** :
* Instances de modèle chargées situées entièrement à l’extérieur du tronc d’affichage
* Modèles ou parties de modèle basculés en invisible, à l’aide du [composant de remplacement d’état hiérarchique](../overview/features/override-hierarchical-state.md)

Par conséquent, il est possible d’écrire une application qui cible la taille `standard` et qui charge plusieurs modèles avec un nombre de polygones proche de la limite pour chaque modèle unique. Quand l’application n’affiche qu’un seul modèle à la fois, le damier n’est pas déclenché.

### <a name="how-to-determine-the-number-of-polygons"></a>Comment déterminer le nombre de polygones ?

Il existe deux façons de déterminer le nombre de polygones d’un modèle ou d’une scène qui contribuent à la limite budgétaire de la taille de serveur `standard` :
* Du côté conversion du modèle, récupérez le [fichier json de sortie de conversion](../how-tos/conversion/get-information.md) et vérifiez l’entrée `numFaces` dans la [section *inputStatistics*](../how-tos/conversion/get-information.md#the-inputstatistics-section).
* Si votre application traite du contenu dynamique, le nombre de polygones rendus peut être interrogé de manière dynamique pendant l’exécution. Utilisez une [requête d’évaluation des performances](../overview/features/performance-queries.md#performance-assessment-queries) et recherchez le membre `polygonsRendered` dans le struct `FrameStatistics`. Le champ `polygonsRendered` est défini sur `bad` lorsque le convertisseur atteint la limite du polygone. Le damier d’arrière-plan apparaît toujours en fondu avec un certain délai, afin de garantir qu’une action utilisateur puisse être effectuée après cette requête asynchrone. L’action utilisateur peut par exemple masquer ou supprimer des instances de modèle.

## <a name="pricing"></a>Tarifs

Pour obtenir une description détaillée des tarifs pour chaque type de configuration, consultez la page [Tarification Remote Rendering](https://azure.microsoft.com/pricing/details/remote-rendering).

## <a name="next-steps"></a>Étapes suivantes
* [Exemples de scripts PowerShell](../samples/powershell-example-scripts.md)
* [Conversion de modèle](../how-tos/conversion/model-conversion.md)

