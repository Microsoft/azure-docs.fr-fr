---
ms.openlocfilehash: c2b1e1d87600e83733eb8bdbf5bc6e2e63376cf4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993068"
---
## <a name="locate-a-cloud-spatial-anchor"></a>Localiser une ancre spatiale cloud

La capacité à localiser une ancre spatiale cloud enregistrée précédemment constitue l’un des principaux avantages d’Azure Spatial Anchors. Il existe plusieurs méthodes permettant de localiser une ancre spatiale cloud. Vous ne pouvez utiliser une stratégie que sur un seul observateur à la fois.
- Localisez les ancres à l’aide de leur identificateur.
- Localisez les ancres qui sont connectées à une ancre précédemment localisée. Pour plus d’informations sur les relations entre les ancres, [cliquez ici](../articles/spatial-anchors/concepts/anchor-relationships-way-finding.md).
- Localisez l’ancre à l’aide de l’option [Relocalisation grossière](../articles/spatial-anchors/concepts/coarse-reloc.md).

> [!NOTE]
> Chaque fois que vous localisez une ancre, les ancres spatiales Azure essaient d’utiliser les données d’environnement collectées pour compléter les informations visuelles sur l’ancre. Si vous rencontrez des difficultés pour localiser une ancre, il peut être utile de créer une ancre, puis de la localiser plusieurs fois à partir de différents angles et conditions d’éclairage.

Si vous recherchez des ancres spatiales cloud à l’aide de leur identificateur, vous devez stocker celui-ci dans le service back-end de votre application, et le rendre accessible à tous les appareils qui peuvent s’authentifier correctement auprès de lui. Pour obtenir un exemple, consultez [Tutoriel : Partager des ancres spatiales sur des appareils](../articles/spatial-anchors/tutorials/tutorial-share-anchors-across-devices.md).

Instanciez un objet `AnchorLocateCriteria`, définissez les identificateurs que vous recherchez, et invoquez la méthode `CreateWatcher` sur la session en fournissant votre `AnchorLocateCriteria`.