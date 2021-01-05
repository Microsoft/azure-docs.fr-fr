---
title: Copier des applications et des données dans des nœuds de pool
description: Découvrez comment copier des applications et des données dans des nœuds de pool.
ms.topic: how-to
ms.date: 02/17/2020
ms.openlocfilehash: e21b8551fb62c4335910fd05bb9590eaf6f7e35a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85954891"
---
# <a name="copy-applications-and-data-to-pool-nodes"></a>Copier des applications et des données dans des nœuds de pool

Azure Batch prend en charge plusieurs façons d’obtenir des données et des applications sur des nœuds de calcul afin que les tâches puissent y accéder et les utiliser. Les données et les applications peuvent être nécessaires pour exécuter l’intégralité du travail et doivent donc être installées sur chaque nœud. Certaines peuvent être requises uniquement pour une tâche spécifique ou doivent être installées pour le travail, mais n’ont pas besoin d’être présentes sur chaque nœud. Batch dispose d’outils pour chacun de ces scénarios.

- **Fichiers de ressources de la tâche de démarrage du pool** : pour les applications ou les données qui doivent être installées sur chaque nœud du pool. Utilisez cette méthode avec un package d’application ou le regroupement de fichiers de ressources de la tâche de démarrage afin d’exécuter une commande d’installation.  

Exemples : 
- Utiliser la ligne de commande de la tâche de démarrage pour déplacer ou installer des applications

- Spécifiez une liste de fichiers ou de conteneurs spécifiques dans un compte de stockage Azure. Pour plus d’informations, consultez [Ajouter#ResourceFile dans la documentation REST](/rest/api/batchservice/pool/add#resourcefile).

- Chaque travail qui s’exécute sur le pool exécute MyApplication.exe qui doit d’abord être installé avec MyApplication.msi. Si vous utilisez ce mécanisme, vous devez définir la propriété **Attente de la réussite** de la tâche de démarrage sur **true**. Pour plus d’informations, consultez [Ajouter#StartTask dans la documentation REST](/rest/api/batchservice/pool/add#starttask).

- **Références du package d’application** sur le pool : pour les applications ou les données qui doivent être installées sur chaque nœud du pool. Il n’existe aucune commande d’installation associée à un package d’application, mais vous pouvez utiliser une tâche de démarrage pour exécuter une commande d’installation. Si votre application ne nécessite pas d’installation ou si elle comprend un grand nombre de fichiers, vous pouvez utiliser cette méthode. Les packages d’application sont particulièrement adaptés aux grandes quantités de fichiers, car ils combinent un grand nombre de références de fichiers en une petite charge utile. Si vous essayez d’inclure plus de 100 fichiers de ressources distincts dans une seule tâche, le service Batch peut rencontrer des limitations système internes pour une tâche unique. Utilisez également des packages d’application si vous avez des exigences rigoureuses en matière de gestion des versions, par exemple si vous avez plusieurs versions différentes de la même application et que vous devez choisir entre elles. Pour plus d’informations, consultez [Déployer des applications sur les nœuds avec des packages d’applications Batch](./batch-application-packages.md).

- **Fichiers de ressources de la tâche de préparation du travail** : pour les applications ou les données qui doivent être installées pour le travail à exécuter, mais qui n’ont pas besoin d’être installées sur l’ensemble du pool. Par exemple : si votre pool a de nombreux types différents de travaux et qu’un seul type de travail nécessite l’exécution de MyApplication.msi, il est logique de placer l’étape d’installation dans une tâche de préparation du travail. Pour plus d’informations sur les tâches de préparation de travail, consultez [Exécuter des tâches de préparation et de validation du travail sur les nœuds de calcul Batch](./batch-job-prep-release.md).

- **Fichiers de ressources de tâche** : quand une application ou des données sont pertinentes uniquement pour une tâche individuelle. Par exemple : vous avez cinq tâches, chacune traitant un fichier différent, puis écrivant la sortie dans le stockage blob.  Dans ce cas, le fichier d’entrée doit être spécifié dans le regroupement de **fichiers de ressources de la tâche**, car chaque tâche a son propre fichier d’entrée.

## <a name="determine-the-scope-required-of-a-file"></a>Déterminer l’étendue requise d’un fichier

Vous devez déterminer l’étendue d’un fichier : il s’agit du fichier requis pour un pool, un travail ou une tâche. Les fichiers dont l’étendue est limitée au pool doivent utiliser des packages d’application de pool ou une tâche de démarrage. Les fichiers dont l’étendue correspond au travail doivent utiliser une tâche de préparation du travail. Les applications sont un bon exemple de fichiers étendus au niveau du pool ou du travail. Les fichiers dont l’étendue est limitée à la tâche doivent utiliser des fichiers de ressources de tâche.

### <a name="other-ways-to-get-data-onto-batch-compute-nodes"></a>Autres méthodes pour récupérer des données sur des nœuds de calcul Batch

Il existe d’autres méthodes pour récupérer des données sur les nœuds de calcul Batch qui ne sont pas officiellement intégrés à l’API REST Batch. Étant donné que vous contrôlez les nœuds Azure Batch et que vous pouvez exécuter des exécutables personnalisés, vous pouvez extraire des données d’un nombre illimité de sources personnalisées tant que le nœud Batch dispose d’une connectivité à la cible et que vous disposez des informations d’identification de cette source sur le nœud Azure Batch. Voici quelques exemples courants :

- Téléchargement de données à partir de SQL
- Téléchargement de données à partir d’autres services web ou emplacements personnalisés
- Mappage d’un partage réseau

### <a name="azure-storage"></a>Stockage Azure

Le stockage blob a téléchargé des objectifs d’extensibilité. Les objectifs d’extensibilité des partages de fichiers du stockage Azure sont les mêmes que pour un blob unique. La taille aura un impact sur le nombre de nœuds et de pools dont vous avez besoin.

