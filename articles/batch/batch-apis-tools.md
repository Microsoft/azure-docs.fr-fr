---
title: API et outils pour développeurs
description: Découvrez les API et les outils disponibles pour développer des solutions avec le service Azure Batch.
ms.topic: conceptual
ms.date: 05/22/2020
ms.custom: seodec18
ms.openlocfilehash: 502eb08631223215933b75dca882c12c02d17bd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89146400"
---
# <a name="overview-of-batch-apis-and-tools"></a>Vue d’ensemble des outils et API Batch

Le traitement des charges de travail parallèles avec Azure Batch est généralement effectué par programmation avec l’une des API Batch. Votre application cliente ou le service peut utiliser les API Batch pour communiquer avec le service Batch. Avec les API Batch, vous pouvez créer et gérer les pools de nœuds de calcul, des machines virtuelles ou services de cloud. Vous pouvez ensuite planifier des tâches à exécuter sur ces nœuds.

Vous pouvez efficacement traiter des charges de travail à grande échelle pour votre organisation, ou fournir un service frontal à vos clients afin qu’ils puissent exécuter des travaux et des tâches, à la demande ou selon un calendrier, sur un nœud, des centaines de nœuds, voire des milliers. Vous pouvez également utiliser Azure Batch dans le cadre d’un plus grand workflow géré par des outils comme [Azure Data Factory](../data-factory/transform-data-using-dotnet-custom-activity.md?toc=%2fazure%2fbatch%2ftoc.json).

> [!TIP]
> Pour en savoir plus sur les fonctionnalités et le workflow utilisés dans Azure Batch, consultez [Flux de travail et ressources du service Batch](batch-service-workflow-features.md).

## <a name="azure-accounts-for-batch-development"></a>Comptes Azure pour le développement de Batch

Lorsque vous développez des solutions Batch, vous devez utiliser les comptes suivants de votre abonnement Azure :

- **Compte Batch** : les ressources Azure Batch, notamment les pools, les nœuds de calcul, les travaux et les tâches, sont associées à un [compte Batch](accounts.md) Azure. Lorsque votre application effectue une requête auprès du service Batch, il authentifie la requête en utilisant le nom du compte Azure Batch, l’URL du compte et une clé d’accès ou bien un jeton de Azure Active Directory. Vous pouvez [créer un compte Batch](batch-account-create-portal.md) dans le Portail Azure ou par programme.
- **Compte de stockage** : Batch inclut la prise en charge intégrée de l’utilisation des fichiers dans [Stockage Azure](../storage/index.yml). Presque tous les scénarios du service Batch font appel au stockage Blob Azure pour la préparation des programmes exécutés par vos tâches et les données qu’ils traitent ainsi que pour le stockage des données de sortie qu’elles génèrent. Chaque compte Batch est généralement associé à un compte de stockage correspondant.

## <a name="service-level-and-management-level-apis"></a>API au niveau de service et au niveau de gestion

Azure Batch comprend deux ensembles d’API : l’un pour le niveau de service et l’autre pour le niveau de gestion. L’attribution de noms est souvent similaire, mais ils retournent des résultats différents.

Seules les actions des API de gestion sont consignées dans le journal d’activité. Les API de niveau de service ignorent la couche de gestion des ressources Azure (management.azure.com), et elles ne sont pas journalisées.

Par exemple, [l’API de service Batch pour supprimer un pool](/rest/api/batchservice/pool/delete) est ciblée directement sur le compte Batch : `DELETE {batchUrl}/pools/{poolId}`

Tandis que [l’API de gestion Batch pour supprimer un pool](/rest/api/batchmanagement/pool/delete) est ciblée sur la couche management.azure.com : `DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Batch/batchAccounts/{accountName}/pools/{poolName}`

## <a name="batch-service-apis"></a>API du service Batch

Vos applications et services peuvent émettre des appels directs de l’API REST ou utiliser une ou plusieurs des bibliothèques clientes suivantes pour exécuter et gérer vos charges de travail Azure Batch.

| API | Informations de référence sur l'API | Téléchargement | Didacticiel | Exemples de code | En savoir plus |
| --- | --- | --- | --- | --- | --- |
| **Batch REST** |[API Azure REST - Docs](/rest/api/batchservice/) |N/A |- |- | [Versions prises en charge](/rest/api/batchservice/batch-service-rest-api-versioning) |
| **Batch .NET** |[SDK Azure pour .NET - Docs](/dotnet/api/overview/azure/batch) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Batch/) |[Didacticiel](tutorial-parallel-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) | [Notes de publication](https://aka.ms/batch-net-dataplane-changelog) |
| **Python Batch** |[SDK Azure pour Python - Docs](/python/api/overview/azure/batch/client) |[PyPI](https://pypi.org/project/azure-batch/) |[Didacticiel](tutorial-parallel-python.md)|[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Python/Batch) | [Lisez-moi](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/batch/azure-batch/README.md) |
| **Batch Node.js** |[SDK Azure pour JavaScript - Docs](/javascript/api/overview/azure/batch/client) |[npm](https://www.npmjs.com/package/azure-batch) |[Didacticiel](batch-nodejs-get-started.md) |- | [Lisez-moi](https://github.com/Azure/azure-sdk-for-node/tree/master/lib/services/batch) |
| **Java Batch** |[SDK Azure pour Java - Docs](/java/api/overview/azure/batch) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/Java) | [Lisez-moi](https://github.com/Azure/azure-batch-sdk-for-java)|

## <a name="batch-management-apis"></a>API de Batch Management

Les API Azure Resource Manager pour le service Batch fournissent un accès par programme aux comptes Batch. À l’aide de ces API, vous pouvez gérer par programme les comptes Batch, les quotas, les packages d’application et d’autres ressources via le fournisseur Microsoft.Batch.  

| API | Informations de référence sur l'API | Téléchargement | Didacticiel | Exemples de code |
| --- | --- | --- | --- | --- |
| **REST Batch Management** |[API Azure REST - Docs](/rest/api/batchmanagement/) |- |- |[GitHub](https://github.com/Azure-Samples/batch-dotnet-manage-batch-accounts) |
| **Gestion de lots .NET** |[SDK Azure pour .NET - Docs](/dotnet/api/overview/azure/batch/management) |[NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/) | [Didacticiel](batch-management-dotnet.md) |[GitHub](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp) |
| **Python Batch Management** |[SDK Azure pour Python - Docs](/python/api/overview/azure/batch/management) |[PyPI](https://pypi.org/project/azure-mgmt-batch/) |- |- |
| **Node.js Batch Management** |[SDK Azure pour JavaScript - Docs](/javascript/api/overview/azure/batch/management) |[npm](https://www.npmjs.com/package/azure-arm-batch) |- |- | 
| **Java Batch Management** |[SDK Azure pour Java - Docs](/java/api/overview/azure/batch/management) |[Maven](https://search.maven.org/search?q=a:azure-batch) |- |- |

## <a name="batch-command-line-tools"></a>Outils en ligne de commande Batch

Ces outils en ligne de commande fournissent les mêmes fonctionnalités que les API des services Batch et Batch Management : 

- [Applets de commande PowerShell pour Batch](/powershell/module/az.batch/) : les applets de commande Azure Batch contenues dans le module [Azure PowerShell](/powershell/azure/) vous permettent de gérer les ressources du service Batch avec PowerShell.
- [Azure CLI](/cli/azure) : Azure CLI est un ensemble d’outils multiplateforme qui fournit des commandes shell permettant d’interagir avec de nombreux services Azure, notamment les services Batch et Batch Management. Consultez la page [Gérer les ressources Batch avec Azure CLI](batch-cli-get-started.md) pour plus d’informations sur l’utilisation d’Azure CLI avec Batch.

## <a name="other-tools-for-application-development"></a>Autres outils pour le développement d’applications

Ces outils supplémentaires peuvent être utiles pour générer et déboguer vos applications et services Batch.

- [Portail Azure](https://portal.azure.com/): vous pouvez créer, superviser et supprimer des pools, des travaux et des tâches Batch dans le portail Azure. Vous pouvez consulter les informations d’état de ces éléments et d’autres ressources pendant que vous exécutez vos travaux, et même télécharger des fichiers depuis les nœuds de calcul de vos pools. Par exemple, vous pouvez télécharger une tâche ayant échoué `stderr.txt` lors de la résolution des problèmes. Vous pouvez également télécharger des fichiers du Bureau à distance (RDP), à utiliser pour vous connecter aux nœuds de calcul.
- [Azure Batch Explorer](https://azure.github.io/BatchExplorer/) : Batch Explorer (anciennement BatchLabs) est un outil client autonome, gratuit et doté de nombreuses fonctionnalités, qui vous aide à créer, déboguer et analyser des applications Azure Batch. Téléchargez un [package d’installation](https://azure.github.io/BatchExplorer/) pour Mac, Linux ou Windows.
- [Azure Batch Shipyard](https://github.com/Azure/batch-shipyard) : Batch Shipyard est un outil qui permet de provisionner, exécuter et superviser le traitement par lots basé sur des conteneurs et les charges de travail HPC sur Azure Batch.
- [Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/) : bien qu’il ne s’agisse pas à proprement parler d’un outil Azure Batch, l’Explorateur Stockage est l’un de vos meilleurs alliés pendant le développement et le débogage de vos solutions Batch.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour en savoir plus sur la journalisation des événements à partir de votre application Batch, consultez [Métriques, alertes et journaux d’activité Batch pour l’évaluation de diagnostic et la supervision](batch-diagnostics.md).
- Pour obtenir des informations de référence sur les événements déclenchés par le service Batch, voir [Batch Analytics](batch-analytics.md).
- Pour plus d’informations sur les variables d’environnement des nœuds de calcul, consultez [Variables d’environnement runtime Azure Batch](batch-compute-node-environment-variables.md).

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
- [Prise en main de la bibliothèque Azure Batch pour .NET](tutorial-parallel-dotnet.md) pour apprendre à utiliser C# et la bibliothèque .NET Batch pour exécuter une charge de travail simple à l’aide d’un flux de travail Batch courant. Une [version Python](tutorial-parallel-python.md) et un [didacticiel Node.js](batch-nodejs-get-started.md) sont également disponibles.
- Téléchargez les [exemples de code sur GitHub](https://github.com/Azure-Samples/azure-batch-samples) pour voir comment C# et Python peuvent interagir avec Batch afin de planifier et traiter des exemples de charge de travail.
