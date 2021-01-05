---
title: Différences par rapport à Azure Data Factory
description: Découvrez comment les fonctionnalités d’intégration de données d’Azure Synapse Analytics diffèrent de celles d’Azure Data Factory
services: synapse-analytics
author: kromerm
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 12/10/2020
ms.author: makromer
ms.reviewer: jrasnick
ms.openlocfilehash: 8818d4db489cef8203ae515c18c61e215d577033
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387613"
---
# <a name="data-integration-in-azure-synapse-analytics-versus-azure-data-factory"></a>Intégration de données dans Azure Synapse Analytics par rapport à Azure Data Factory

Dans Azure Synapse Analytics, les fonctionnalités d’intégration de données telles que des pipelines et flux de données Synapse sont basées sur celles d’Azure Data Factory. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Data Factory](../../data-factory/introduction.md).


## <a name="available-features-in-adf--azure-synapse-analytics"></a>Fonctionnalités disponibles dans ADF & Azure Synapse Analytics

Vérifiez la disponibilité des fonctionnalités dans le tableau ci-dessous :

| Catégorie                 | Fonctionnalité    |  Azure Data Factory  | Azure Synapse Analytics |
| ------------------------ | ---------- | :------------------: | :---------------------: |
| **Runtime d’intégration**  | Utilisation de SSIS et de SSIS Integration Runtime | ✓ | ✗ |
|                          | Prise en charge de runtime d’intégration inter-régions (flux de données) | ✓ | ✗ |
|                          | Partage de runtime d’intégration | ✓<br><small>*La passerelle peut-elle être partagée entre plusieurs fabriques de données ?* | ✗ |
|                          | Durée de vie | ✓ | ✗ |
| **Activités des pipelines** | Activité de package SSIS | ✓ | ✗ |
|                          | Prise en charge de l’activité de Power Query | ✓ | ✓ |
| **Galerie de modèles et Centre de connaissances** | Modèles de solution | ✓<br><small>*Galerie de modèles Azure Data Factory* | ✓<br><small>*Centre de connaissances de l’espace de travail Synapse* |
| **Intégration du dépôt GIT** | Intégration de GIT | ✓ | ✓ |
| **Surveillance**           | Surveillance des travaux Spark pour Data Flow | ✗ | ✓<br><small>*Tirer parti des pools Synapse Spark* |
|                          | Intégration avec Azure Monitor | ✓ | ✗ |

> [!Note]
> La **Durée de vie** est un paramètre d’Azure Integration Runtime qui permet au cluster Spark de *rester chaud* pendant un certain temps après l’exécution d’un flux de données.
>


## <a name="next-steps"></a>Étapes suivantes

Commencez à utiliser l’intégration de données dans votre espace de travail Synapse en découvrant comment [ingérer des données dans un compte Azure Data Lake Storage Gen2](data-integration-data-lake.md).
