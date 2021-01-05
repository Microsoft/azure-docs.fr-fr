---
title: Conserver la sortie des travaux et des tâches dans un magasin de données
description: Découvrez les différentes possibilités pour conserver les données de sortie des tâches et des travaux Batch. Vous pouvez conserver les données dans le Stockage Azure ou un autre magasin de données.
ms.topic: how-to
ms.date: 11/14/2018
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4ebe0b6d57225eff9f3f1251d5e491c95e9b7ffc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85965108"
---
# <a name="persist-job-and-task-output"></a>Conserver les résultats des tâches et des travaux

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

Voici quelques exemples courants de sortie de tâche :

- Fichiers créés au moment où la tâche traite des données d’entrée.
- Fichiers journaux associés à l’exécution des tâches.

Cet article décrit les différentes options de conservation d’une sortie de tâche.

## <a name="options-for-persisting-output"></a>Options de conservation de sortie

Selon votre scénario, vous pouvez adopter différentes approches pour conserver une sortie de tâche :

- [Utiliser l’API du service Batch](batch-task-output-files.md).  
- [Utiliser la bibliothèque Batch File Conventions pour .NET](batch-task-output-file-conventions.md).  
- Implémenter les conventions standard de nommage des fichiers Batch dans votre application.
- Implémenter une solution de déplacement de fichiers personnalisée.

Les sections suivantes décrivent brièvement chaque approche ainsi que les considérations de conception générale pour conserver la sortie.

### <a name="use-the-batch-service-api"></a>Utiliser l’API du service Batch

Le service Batch permet de spécifier des fichiers de sortie dans le Stockage Azure pour les données de tâches quand vous [ajoutez une tâche à un travail](/rest/api/batchservice/add-a-task-to-a-job) ou [ajoutez une collection de tâches à un travail](/rest/api/batchservice/add-a-collection-of-tasks-to-a-job).

Pour plus d’informations sur la conservation de la sortie des tâches avec l’API du service Batch, consultez [Conserver les données des tâches dans le Stockage Azure à l’aide de l’API du service Batch](batch-task-output-files.md).

### <a name="use-the-batch-file-conventions-library-for-net"></a>Utiliser la bibliothèque Conventions de nommage des fichiers Batch pour .NET

Batch définit un ensemble facultatif de conventions pour nommer les fichiers de sortie de tâche dans le Stockage Azure. Les conventions standard de nommage des fichiers Batch sont décrites dans ce [document](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files#conventions). Les conventions standard de nommage des fichiers déterminent les noms du conteneur et du chemin d’objet blob de destination dans le Stockage Azure pour un fichier de sortie donné en fonction des noms du travail et de la tâche.

C’est à vous qu’il appartient d’appliquer ou non les conventions standard de nommage des fichiers pour vos fichiers de données de sortie. Vous pouvez aussi nommer le conteneur et l’objet blob de destination comme vous le souhaitez. Si vous utilisez cette norme pour nommer les fichiers de sortie, vos fichiers de sortie sont visibles dans le [portail Azure][portal].

Les développeurs qui élaborent des solutions Batch avec C# et .NET peuvent utiliser la [bibliothèque Conventions de nommage des fichiers pour .NET][nuget_package] pour conserver les données des tâches dans un compte de Stockage Azure, conformément aux [conventions standard de nommage des fichiers Batch](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files#conventions). La bibliothèque Conventions de nommage des fichiers gère le déplacement des fichiers de sortie vers le Stockage Azure, ainsi que le nommage des conteneurs et objets blob de destination d’une façon bien connue.

Pour plus d’informations sur la conservation de la sortie des tâches avec la bibliothèque File Conventions pour .NET, consultez [Conserver les données des travaux et des tâches dans Stockage Azure avec la bibliothèque File Conventions Batch pour .NET](batch-task-output-file-conventions.md).

### <a name="implement-the-batch-file-conventions-standard"></a>Implémenter les conventions standard de nommage des fichiers Batch

Si vous utilisez un autre langage que .NET, vous pouvez implémenter les [conventions standard de nommage des fichiers](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files#conventions) dans votre propre application.

Vous pouvez souhaiter implémenter les conventions standard de nommage des fichiers par vous-même si vous voulez un schéma de nommage qui a fait ses preuves ou si vous voulez afficher la sortie des tâches dans le portail Azure.

### <a name="implement-a-custom-file-movement-solution"></a>Implémenter une solution de déplacement de fichiers personnalisée

Vous pouvez aussi implémenter votre propre solution de déplacement de fichiers complète. Suivez cette approche dans les cas suivants :

- Vous voulez conserver les données des tâches dans un magasin de données différent du Stockage Azure. Pour charger les fichiers dans un magasin de données comme Azure SQL ou Azure DataLake, vous pouvez créer un script personnalisé ou un exécutable à charger à cet emplacement. Vous pouvez ensuite l’appeler sur la ligne de commande après avoir exécuté votre exécutable principal. Par exemple, sur un nœud Windows, vous pouvez appeler ces deux commandes : `doMyWork.exe && uploadMyFilesToSql.exe`
- Vous voulez procéder à la création de points de contrôle ou effectuer un chargement anticipé des résultats initiaux.
- Vous voulez conserver un contrôle précis de la gestion des erreurs. Par exemple, vous pouvez souhaiter implémenter votre propre solution si vous voulez utiliser des actions de dépendance de tâches pour effectuer certaines actions de chargement en fonction de codes de sortie de tâche spécifiques. Pour plus d’informations sur les actions de dépendance de tâches, consultez [Créer des dépendances de tâches pour exécuter des tâches qui dépendent d’autres tâches](batch-task-dependencies.md).

## <a name="design-considerations-for-persisting-output"></a>Considérations de conception pour la conservation des sorties

Au moment de concevoir votre solution Batch, tenez compte des points suivants s’agissant des sorties des travaux et des tâches.

- **Durée de vie de nœud de calcul** : les nœuds de calcul sont souvent temporaires, notamment dans les pools à mise à l’échelle automatique. La sortie d’une tâche qui s’exécute sur un nœud est disponible uniquement pendant que le nœud existe, et seulement pendant la période de rétention des fichiers que vous avez définie pour la tâche. Si une tâche génère une sortie qui peut être nécessaire à la fin de la tâche, celle-ci doit charger ses fichiers de sortie dans un magasin durable tels que le Stockage Azure.

- **Stockage de sortie** : Stockage Azure est recommandé comme magasin de données pour la sortie des tâches, mais vous pouvez utiliser n’importe quel stockage durable. L’écriture de la sortie des tâches dans le Stockage Azure est intégrée dans l’API du service Batch. Si vous utilisez une autre forme de stockage durable, vous devez écrire la logique d’application pour conserver vous-même la sortie des tâches.

- **Récupération de sortie** : vous pouvez récupérer la sortie des tâches directement à partir des nœuds de calcul de votre pool ou depuis Stockage Azure ou un autre magasin de données si vous avez conservé la sortie des tâches. Pour récupérer la sortie d’une tâche directement à partir d’un nœud de calcul, vous avez besoin du nom de fichier et de son emplacement de sortie sur le nœud. Si vous conservez la sortie des tâches dans le Stockage Azure, vous avez besoin du chemin complet des fichiers dans le Stockage Azure pour télécharger les fichiers de sortie à l’aide du kit SDK Stockage Azure.

- **Affichage de sortie** : lorsque vous accédez à une tâche Batch dans le portail Azure et sélectionnez **Fichiers sur le nœud**, tous les fichiers associés à la tâche s’affichent, et pas seulement les fichiers de sortie qui vous intéressent. De nouveau, les fichiers hébergés sur les nœuds de calcul sont disponibles uniquement lorsque le nœud existe, et seulement pendant la durée de rétention des fichiers que vous avez définie pour la tâche. Pour afficher la sortie des tâches que vous avez conservée dans le Stockage Azure, vous pouvez utiliser le portail Azure ou une application cliente du Stockage Azure telle que l’[Explorateur Stockage Azure][storage_explorer]. Pour afficher des données de sortie dans le Stockage Azure avec le portail ou un autre outil, vous devez connaître l’emplacement du fichier correspondant et y accéder directement.

## <a name="next-steps"></a>Étapes suivantes

- Explorez l’utilisation des nouvelles fonctionnalités de l’API du service Batch pour conserver les données des tâches dans [Conserver les données des tâches dans le Stockage Azure à l’aide de l’API du service Batch](batch-task-output-files.md).
- Découvrez comment utiliser la bibliothèque File Conventions Batch pour .NET dans [Conserver les données des travaux et des tâches dans Stockage Azure avec la bibliothèque File Conventions Batch pour .NET](batch-task-output-file-conventions.md).
- Consultez l’exemple de projet [PersistOutputs][github_persistoutputs] sur GitHub, qui montre comment utiliser la bibliothèque Client Batch pour .NET et la bibliothèque File Conventions pour .NET pour conserver la sortie des tâches dans un stockage durable.

[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.com
[storage_explorer]: https://storageexplorer.com/
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs 
