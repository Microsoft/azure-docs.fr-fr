---
title: Flux de travail et ressources du service Batch
description: Découvrez les fonctionnalités du service Batch et de son flux de travail de haut niveau du point de vue du développement.
ms.topic: conceptual
ms.date: 05/12/2020
ms.openlocfilehash: 76a0b140f3bea4b07a6de632abbcbc3fd26e582d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85965210"
---
# <a name="batch-service-workflow-and-resources"></a>Flux de travail et ressources du service Batch

Dans cette vue d’ensemble des principaux composants du service Azure Batch, nous abordons le flux de travail de haut niveau que les développeurs Batch peuvent utiliser pour créer des solutions de calcul parallèles à grande échelle, ainsi que les principales ressources de service utilisées.

Que vous développiez une application ou un service de calcul distribué émettant des appels [d’API REST](/rest/api/batchservice/) directs, ou que vous utilisiez un autre [Kit de développement logiciel (SDK) Batch](batch-apis-tools.md#batch-service-apis), vous utiliserez bon nombre des ressources et fonctionnalités présentées ici.

> [!TIP]
> Pour une présentation plus poussée du service Batch, consultez l’article [Notions de base d’Azure Batch](batch-technical-overview.md). Consultez également les dernières [mises à jour du service Batch](https://azure.microsoft.com/updates/?product=batch).

## <a name="basic-workflow"></a>Flux de travail de base

Le flux de travail de haut niveau suivant est caractéristique de la plupart des applications et services utilisant le service Batch pour le traitement des charges de travail parallèles :

1. Chargez les **fichiers de données** que vous souhaitez traiter vers un compte [Stockage Azure](../storage/index.yml). Le service Batch inclut la prise en charge intégrée de l’accès au stockage d’objets Blob Azure ; lorsque les tâches sont exécutées, celles-ci peuvent télécharger les fichiers en question vers les [nœuds de calcul](nodes-and-pools.md#nodes) .
2. Chargez le **fichiers d’application** que les tâches exécuteront. Ces fichiers peuvent être des fichiers binaires ou des scripts, accompagnés de leurs dépendances, et sont exécutés par les tâches dans vos travaux. Vos tâches peuvent télécharger ces fichiers à partir de votre compte Azure Storage ; vous pouvez également utiliser la fonctionnalité [packages d’application](nodes-and-pools.md#application-packages) de Batch pour le déploiement et la gestion des applications.
3. Créez un [pool](nodes-and-pools.md#pools) de nœuds de calcul. Lors de la création d’un pool, vous devez spécifier le nombre de nœuds de calcul du pool, leur taille ainsi que le système d’exploitation. Pour chaque exécution de tâche dans votre travail, cette dernière est exécutée sur l’un des nœuds du pool.
4. Créez un [travail](jobs-and-tasks.md#jobs). Un travail gère une collection de tâches. Vous associez chaque travail à un pool spécifique, dans lequel s’exécuteront les tâches du travail concerné.
5. Ajoutez des [tâches](jobs-and-tasks.md#tasks) au travail. Chaque tâche exécute l’application ou le script téléchargés afin de traiter les fichiers de données téléchargés à partir de votre compte de stockage. Lorsque chaque tâche est terminée, celle-ci peut télécharger sa sortie vers Azure Storage.
6. Surveillez la progression du travail et récupérez la sortie de la tâche à partir d’Azure Storage.

> [!NOTE]
> Un [compte Batch](accounts.md) est nécessaire pour utiliser le service Batch. La plupart des solutions Batch utilisent aussi un compte [Stockage Azure](../storage/index.yml), afin de stocker et récupérer un fichier.

## <a name="batch-service-resources"></a>Ressources du service Batch

Les rubriques suivantes présentent les ressources de Batch qui permettent l’exécution de vos scénarios de calcul distribué.

- [Comptes Batch et comptes de stockage](accounts.md)
- [Nœuds et pools](nodes-and-pools.md)
- [Travaux et tâches](jobs-and-tasks.md)
- [Fichiers et répertoires](files-and-directories.md)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [outils et API Batch](batch-apis-tools.md) disponibles pour créer des solutions Batch.
- Découvrez les bases du développement d’une application Batch à l’aide de la [bibliothèque Azure Batch pour .NET](quick-run-dotnet.md) ou [Python](quick-run-python.md). Ces démarrages rapides décrivent la création d’un exemple d’application qui utilise le service Batch pour exécuter une charge de travail sur plusieurs nœuds de calcul, et inclut l’utilisation d’Azure Storage pour la mise en attente et la récupération de fichiers de la charge de travail.
- Téléchargez et installez [Batch Explorer](https://azure.github.io/BatchExplorer/) pour l’utiliser lors du développement de vos solutions Batch. Utilisez Batch Explorer pour vous aider à créer, déboguer et analyser les applications de Azure Batch.
- Consultez les ressources de la Communauté, à savoir [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-batch), le [dépôt de la communauté Batch](https://github.com/Azure/Batch)et le [forum Azure Batch](/answers/topics/azure-batch.html).
