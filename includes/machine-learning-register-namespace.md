---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 12/01/2020
ms.author: larryfr
ms.openlocfilehash: 6e9f349f2fd90fdb12d8d26ca904e504b75d9db2
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96445178"
---
* Lors de la création d’un espace de travail, vous pouvez soit créer automatiquement les services nécessaires à l’espace de travail, soit utiliser des services existants. Si vous voulez utiliser des __services existants d’un autre abonnement Azure__ que l’espace de travail, vous devez inscrire l’espace de noms Azure Machine Learning dans l’abonnement qui contient ces services. Par exemple, la création d’un espace de travail dans l’abonnement A qui utilise un compte de stockage de l’abonnement B, l’espace de noms Azure Machine Learning doit être inscrit dans l’abonnement B pour que vous puissiez utiliser le compte de stockage avec l’espace de travail.

    Le fournisseur de ressources pour Azure Machine Learning est __Microsoft.MachineLearningService__. Pour plus d’informations sur la façon de déterminer s’il est inscrit et de l’inscrire, consultez l’article [Fournisseurs et types de ressources Azure](../articles/azure-resource-manager/management/resource-providers-and-types.md).

    > [!IMPORTANT]
    > Cela s’applique uniquement aux ressources fournies lors de la création de l’espace de travail : comptes de stockage Azure, registre de conteneurs Azure, Azure Key Vault et Application Insights.