---
title: Travaux et tâches dans Azure Batch
description: En savoir plus sur les travaux et les tâches et sur leur utilisation dans un workflow Azure Batch du point de vue du développeur.
ms.topic: conceptual
ms.date: 11/23/2020
ms.openlocfilehash: e1ca721ec7527d9d042c129c22cf0266e57c32e9
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95808591"
---
# <a name="jobs-and-tasks-in-azure-batch"></a>Travaux et tâches dans Azure Batch

Dans Azure Batch, une *tâche* représente une unité de calcul. Un *travail* est une collection de ces tâches. Voir ci-dessous pour plus d’informations sur les travaux et les tâches et sur leur utilisation dans un workflow Azure Batch.

## <a name="jobs"></a>travaux

Un travail est une collection de tâches. Il gère la façon dont le calcul est effectué par ses tâches sur les nœuds de calcul d’un pool.

Un travail spécifie le [pool](nodes-and-pools.md#pools) sur lequel l’opération doit être exécutée. Vous pouvez créer un nouveau pool pour chaque tâche, ou utiliser un pool pour plusieurs travaux. Vous pouvez créer un pool pour chaque travail associé à une [planification de travail](#scheduled-jobs), ou un pool unique pour tous les travaux associés à une planification de travail.

### <a name="job-priority"></a>priorité de travail

Vous pouvez attribuer une priorité facultative aux travaux que vous créez. Le service Batch utilise la valeur de priorité du travail pour déterminer l’ordre de planification (pour toutes les tâches au sein du travail) dans chaque pool.

Pour mettre à jour la priorité d’un travail, appelez l’opération [Mettre à jour les propriétés d’un travail](/rest/api/batchservice/job/update) (REST Batch) ou modifiez [CloudJob.Priority](/dotnet/api/microsoft.azure.batch.cloudjob) (.NET Batch). Les valeurs de priorité sont comprises entre –1000 (priorité la plus basse) et 1000 (priorité la plus élevée).

Dans un même pool, les travaux de priorité supérieure sont prioritaires en termes de planification sur les travaux de priorité inférieure. Les tâches des travaux de priorité inférieure qui sont déjà en cours d’exécution ne sont pas devancées par les tâches d’un travail de priorité plus élevée. Des travaux avec le même niveau de priorité ont la même chance d’être planifiés et l’ordre d’exécution des tâches n’est pas défini.

Un travail avec une valeur de priorité élevée en cours d’exécution dans un pool n’affecte pas la planification des travaux en cours d’exécution dans un pool distinct ou dans un autre compte Batch. La priorité des travaux ne s’applique pas aux [pools automatiques](nodes-and-pools.md#autopools), qui sont créés lors de l’envoi du travail.

### <a name="job-constraints"></a>Contraintes de travail

Vous pouvez utiliser des contraintes de travail pour spécifier certaines limites pour vos travaux :

- Vous pouvez définir une **durée maximale** pour les travaux ; si la durée d’exécution du travail est supérieure à la durée maximale spécifiée, le travail et toutes les tâches qui lui sont associées seront terminés.
- Vous pouvez spécifier le **nombre maximal de nouvelles tentatives de tâche** en tant que contrainte et indiquer notamment si une tâche doit toujours faire ou non l’objet d’une nouvelle tentative. Une nouvelle tentative de tâche signifie que, si la tâche échoue, elle est replacée dans la file d’attente pour s’exécuter à nouveau.

### <a name="job-manager-tasks-and-automatic-termination"></a>Tâches du gestionnaire de travaux et arrêt automatique

Votre application cliente peut ajouter des tâches à un travail. Vous pouvez également spécifier une [tâche du gestionnaire de travaux](#job-manager-task). Une tâche de ce type contient les informations nécessaires à la création des tâches requises pour un travail et elle s’exécute sur l’un des nœuds de calcul du pool. La tâche du gestionnaire de travaux est gérée spécifiquement par Batch : elle est mise en file d’attente dès que le travail est créé et elle est redémarrée en cas d’échec. Une tâche du gestionnaire de travaux est requise pour les travaux créés dans le cadre d’une [planification de travail](#scheduled-jobs), car il s’agit du seul moyen de définir les tâches avant que le travail soit instancié.

Par défaut, l’état des tâches est maintenu comme actif lorsque toutes les tâches du travail sont terminées. Vous pouvez modifier cette configuration pour terminer automatiquement le travail lorsque toutes les tâches afférentes ont été accomplies. Définissez la propriété **onAllTasksComplete** du travail ([OnAllTasksComplete](/dotnet/api/microsoft.azure.batch.cloudjob) dans .NET Batch) sur `terminatejob`*` pour terminer automatiquement le travail lorsque l’état de toutes les tâches afférentes est défini comme terminé.

Le service Batch considère les travaux ne présentant *aucune* tâche comme des travaux dont toutes les tâches sont terminées. C’est la raison pour laquelle cette option est généralement utilisée avec une [tâche de gestionnaire de travaux](#job-manager-task). Si vous souhaitez utiliser l’arrêt automatique d’un travail sans utiliser de gestionnaire de travaux, définissez initialement la propriété **onAllTasksComplete** d’un nouveau travail sur `noaction`, puis sur `terminatejob`*` seulement une fois que vous avez terminé d’ajouter des tâches au travail.

### <a name="scheduled-jobs"></a>Scheduled jobs

Les [planifications de travaux](/rest/api/batchservice/jobschedule) vous permettent de créer des travaux périodiques au sein du service Batch. Une planification de travail indique le moment où exécuter des tâches et les spécifications des travaux à exécuter. Vous pouvez spécifier la durée de la planification (durée proprement dite et date d’entrée en vigueur de la planification) ainsi que la fréquence de création des travaux au cours de la période de planification.

## <a name="tasks"></a>Tâches

Une tâche est une unité de calcul associée à un travail. Elle s’exécute sur un nœud. Les tâches sont affectées à un nœud afin d’être exécutées ou sont mises en file d’attente jusqu’à ce qu’un nœud soit disponible. Une tâche exécute un ou plusieurs programmes ou scripts sur un nœud de calcul afin d’effectuer le travail nécessaire.

Lorsque vous créez une tâche, vous pouvez spécifier les éléments suivants :

- **Ligne de commande** de la tâche. Il s’agit de la ligne de commande qui exécute votre application ou script sur le nœud de calcul.

    Il est important de noter que la ligne de commande ne s’exécute pas sous un interpréteur de commandes. Elle ne peut donc pas tirer parti en mode natif des fonctionnalités de l’interpréteur de commandes, comme l’expansion de [variables d’environnement](#environment-settings-for-tasks) (qui inclut le `PATH`). Pour tirer parti de ces fonctionnalités, vous devez appeler l’interpréteur de commandes dans la ligne de commande, par exemple en lançant `cmd.exe` sur les nœuds Windows ou `/bin/sh` sur Linux :

    `cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

    `/bin/sh -c MyTaskApplication $MY_ENV_VAR`

    Si vos tâches doivent exécuter une application ou un script hors du `PATH` du nœud ou référencer des variables d’environnement, appelez l’interpréteur de commandes explicitement dans la ligne de commande de la tâche.
- **Fichiers de ressources** contenant les données à traiter. Ces fichiers sont automatiquement copiés sur le nœud à partir du stockage Blob dans un compte Stockage Azure avant l’exécution de la ligne de commande de la tâche. Pour plus d’informations, consultez [Tâche de démarrage](#start-task) et [Fichiers et répertoires](files-and-directories.md).
- Les **variables d’environnement** requises par l’application. Pour plus d’informations, consultez [Paramètres d’environnement des tâches](#environment-settings-for-tasks).
- Les **contraintes** sous lesquelles la tâche doit s’exécuter. Les contraintes comprennent notamment la durée maximale pendant laquelle la tâche est autorisée à s’exécuter, le nombre maximal de nouvelles tentatives en cas d’échec de la tâche, ainsi que la durée maximale de conservation des fichiers dans le répertoire de travail de la tâche.
- **Packages d’applications** à déployer sur le nœud de calcul sur lequel l’exécution de la tâche est planifiée. [Application packages](batch-application-packages.md) permettent le déploiement simplifié et le contrôle de version des applications exécutées par vos tâches. Les packages d’applications au niveau des tâches sont particulièrement utiles dans les environnements de pool partagé, où différentes tâches sont exécutées sur un même pool et le pool n’est pas supprimé lorsqu’un travail est terminé. Si votre travail présente moins de tâches que le pool ne contient de nœuds, les packages d’applications au niveau des tâches peuvent réduire le transfert de données, votre application n’étant déployée que sur les nœuds exécutant les tâches.
- Une référence d’**image de conteneur** dans Docker Hub ou bien un registre privé et des paramètres supplémentaires pour créer un conteneur Docker dans lequel la tâche s’exécute sur le nœud. Vous spécifiez uniquement ces informations si le pool est configuré avec une configuration de conteneur.

> [!NOTE]
> La durée de vie maximale d’une tâche, entre le moment où elle est ajoutée au travail et la fin de son exécution, est de 180 jours. Les tâches terminées sont conservées pendant sept jours. Les données de tâches non terminées pendant la durée de vie maximale ne sont pas accessibles.

Outre les tâches que vous pouvez définir pour effectuer des calculs sur un nœud, plusieurs tâches spéciales sont également fournies par le service Batch :

- [Tâche de démarrage](#start-task)
- [Tâche du gestionnaire de travaux](#job-manager-task)
- [Tâches de préparation et lancement](#job-preparation-and-release-tasks)
- [Tâche multi-instance](#multi-instance-task)
- [Dépendances de la tâche](#task-dependencies)

### <a name="start-task"></a>Tâche de démarrage

En associant une tâche de démarrage à un pool, vous pouvez préparer l’environnement d’exploitation de ses nœuds. Par exemple, vous pouvez effectuer des opérations comme l’installation des applications que vos tâches exécuteront ou le démarrage des processus d’arrière-plan. La tâche de démarrage s’exécute chaque fois qu’un nœud démarre, aussi longtemps qu’il reste dans le pool. Cela inclut le moment où le nœud est ajouté au pool pour la première fois et celui où il est redémarré ou réinitialisé.

Le principal avantage de la tâche de démarrage est qu’elle peut contenir toutes les informations nécessaires pour configurer un nœud de calcul et installer les applications requises pour l’exécution de la tâche. L’augmentation du nombre de nœuds dans un pool revient donc tout simplement à spécifier le nouveau nombre de nœuds cible. La tâche de démarrage fournit les informations nécessaires au service Batch pour configurer les nouveaux nœuds et les préparer à accepter des tâches.

Comme dans le cas de n’importe quelle tâche Azure Batch, vous pouvez spécifier une liste de fichiers de ressources dans [Stockage Azure](../storage/index.yml), en plus d’une ligne de commande à exécuter. Le service Batch commence par copier les fichiers de ressources sur le nœud à partir de Stockage Azure, puis exécute la ligne de commande. Pour une tâche de démarrage de pool, la liste des fichiers contient généralement l’application de la tâche et ses dépendances.

Toutefois, la tâche de démarrage peut également inclure des données de référence qui doivent être utilisées par toutes les tâches exécutées sur le nœud de calcul. Par exemple, la ligne de commande d’une tâche de démarrage peut effectuer une opération `robocopy` afin de copier les fichiers d’application (qui ont été spécifiés comme fichiers de ressources et téléchargés sur le nœud) à partir du [répertoire de travail](files-and-directories.md) de la tâche de démarrage vers le dossier **partagé**, puis exécuter un MSI ou `setup.exe`.

Il est généralement préférable pour le service Batch d’attendre que la tâche de démarrage soit terminée avant de considérer que le nœud est prêt à recevoir des tâches, mais vous pouvez configurer ce paramètre.

Si une tâche de démarrage échoue sur un nœud du pool, l’état du nœud est mis à jour pour refléter l’échec et aucune tâche n’est affectée au nœud. Une tâche de démarrage peut échouer en cas de problème de copie des fichiers de ressources depuis le stockage, ou si le processus exécuté par sa ligne de commande retourne un code de sortie différent de zéro.

Si vous ajoutez ou mettez à jour la tâche de démarrage pour un pool existant, vous devez redémarrer ses nœuds de calcul pour que cette tâche soit appliquée aux nœuds.

>[!NOTE]
> Batch limite la taille totale d’une tâche de démarrage, ce qui inclut les fichiers de ressources et les variables d’environnement. Si vous avez besoin de réduire la taille d’une tâche de démarrage, vous pouvez utiliser une des deux approches :
>
> 1. Utilisez des packages d’application pour distribuer des applications ou des données sur chaque nœud de votre pool  Batch. Pour en savoir plus sur les packages d’applications, consultez [Déployer des applications sur les nœuds avec des packages d’applications Batch](batch-application-packages.md).
> 2. Vous pouvez créer manuellement une archive au format ZIP contenant vos fichiers d’applications. Chargez l’archive ZIP dans le stockage Azure, en tant que blob. Spécifiez cette archive sous la forme d’un fichier de ressources pour votre tâche de démarrage. Avant d’exécuter la ligne de commande pour votre tâche de démarrage, décompressez l’archive à partir de la ligne de commande. 
>
>    Pour ce faire, vous pouvez utiliser l’outil d’archivage de votre choix. Vous devez inclure l’outil qui vous permet de décompresser l’archive en tant que fichier de ressources pour la tâche de démarrage.

### <a name="job-manager-task"></a>Tâche du gestionnaire de travaux

Vous utilisez généralement une tâche du gestionnaire de travaux pour contrôler ou surveiller l’exécution des travaux. Par exemple, les tâches du gestionnaire de travaux sont souvent utilisées pour créer et soumettre les tâches d’un travail, identifier les autres tâches à exécuter et déterminer le moment où le travail est terminé.

Toutefois, une tâche du gestionnaire de travaux n’est pas limitée à ces activités. C’est une tâche à part entière qui peut effectuer toutes les actions requises pour le travail. Par exemple, une tâche du gestionnaire de travaux peut télécharger un fichier spécifié en tant que paramètre, en analyser le contenu et soumettre des tâches supplémentaires en fonction de ce contenu.

Une tâche du gestionnaire de travaux est lancée avant toutes les autres tâches. Elle fournit les fonctionnalités suivantes :

- Elle est automatiquement soumise en tant que tâche par le service Batch lorsque le travail est créé.
- Elle est planifiée pour s’exécuter avant les autres tâches d’un projet.
- Le nœud associé est le dernier à être supprimé d’un pool lorsque la taille de ce dernier diminue.
- Son arrêt peut être lié à l'arrêt de toutes les tâches du travail.
- Une tâche de gestionnaire de travaux reçoit la priorité la plus élevée lorsqu’elle doit être redémarrée. Si aucun nœud inactif n’est disponible, le service Batch peut mettre fin à une tâche en cours d’exécution dans le pool pour libérer de l’espace pour l’exécution de la tâche du gestionnaire de travaux.
- Une tâche du gestionnaire de travaux associée à un travail n’a pas la priorité sur les tâches d’autres travaux. Parmi les travaux, seules les priorités au niveau du travail sont observées.

### <a name="job-preparation-and-release-tasks"></a>Tâches de préparation et lancement

Batch propose des tâches de préparation des travaux pour la configuration préliminaire de l’exécution des travaux et des tâches de validation des travaux pour la maintenance ou le nettoyage postérieurs aux travaux.

une tâche de préparation de travail s’exécute sur tous les nœuds de calcul qui sont planifiés pour exécuter des tâches, avant l’exécution des autres tâches du travail. Par exemple, vous pouvez utiliser une tâche de préparation du travail pour copier les données qui sont partagées par toutes les tâches, mais qui sont propres au travail.

quand un travail est terminé, une tâche de validation de travail s’exécute sur chaque nœud du pool ayant exécuté au moins une tâche. Par exemple, une tâche de validation du travail peut supprimer les données copiées par la tâche de préparation du travail ou elle peut compresser et charger des données de journaux de diagnostic.

Les tâches de préparation et de validation de travail vous permettent de spécifier une ligne de commande à exécuter lorsque la tâche est appelée. Elles offrent des fonctionnalités telles que le téléchargement de fichiers, une exécution élevée, des variables d’environnement personnalisées, une durée d’exécution maximale, le nombre de tentatives et la durée de rétention des fichiers.

Pour plus d’informations sur les tâches de préparation de travail et de validation, consultez [Exécuter les tâches de préparation de travail et de validation sur les nœuds de calcul Azure Batch](batch-job-prep-release.md).

### <a name="multi-instance-task"></a>Tâche multi-instance

Une [tâche multi-instance](batch-mpi.md) est une tâche configurée pour s’exécuter simultanément sur plusieurs nœuds de calcul. Avec des tâches multi-instances, vous pouvez permettre des scénarios de calcul haute performance qui requièrent un groupe de nœuds de calcul alloués ensemble pour traiter une seule et même charge de travail, telle que l’interface de passage de messages (MPI).

Pour une présentation détaillée de l’exécution des travaux MPI dans Batch à l’aide de la bibliothèque .NET Batch, consultez l’article [Utiliser les tâches multi-instances pour exécuter des applications MPI (Message Passing Interface) dans Azure Batch](batch-mpi.md).

### <a name="task-dependencies"></a>Dépendances de la tâche

La [dépendance entre tâches](batch-task-dependencies.md), comme son nom l’indique, vous permet de préciser que l’exécution d’une tâche dépend de l’achèvement préalable d’autres tâches. Cette fonctionnalité prend en charge des situations dans lesquelles une tâche « en aval » consomme la sortie d’une tâche « en amont », ou lorsqu’une tâche en amont effectue une initialisation requise par une tâche en aval.

Pour utiliser cette fonctionnalité, vous devez d’abord [activer les dépendances de tâche](batch-task-dependencies.md#enable-task-dependencies
) sur le travail Batch. Ensuite, pour chaque tâche qui dépend d’une autre (ou de plusieurs autres), vous devez spécifier les tâches dont elle dépend.

Avec l’interdépendance des tâches, vous pouvez configurer des scénarios tels que :

- *taskB* dépend de *taskA* (l’exécution de *taskB* ne commence pas tant que celle de *taskA* n’est pas terminée).
- *taskC* dépend de *taskA* et de *taskB*.
- *taskD* dépend d’une plage de tâches, notamment des tâches *1* à *10* avant de pouvoir s’exécuter.

Pour plus d’informations, consultez l’article [Dépendances de tâches dans Azure Batch](batch-task-dependencies.md) et l’exemple de code [TaskDependencies](https://github.com/Azure-Samples/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies) dans le référentiel GitHub [azure-batch-samples](https://github.com/Azure-Samples/azure-batch-samples).

### <a name="environment-settings-for-tasks"></a>Paramètres d’environnement des tâches

Chaque tâche exécutée par le service Batch a accès aux variables d’environnement définies sur les nœuds de calcul. Cela inclut les variables d’environnement définies par le service Batch ([service-defined](./batch-compute-node-environment-variables.md)) et les variables d’environnement personnalisées que vous pouvez définir pour vos tâches. Les applications et les scripts que vos tâches exécutent sur les nœuds ont accès à ces variables d’environnement pendant l’exécution.

Vous pouvez définir des variables d’environnement personnalisées au niveau de la tâche ou du travail en remplissant la propriété de *paramètres d’environnement* pour ces entités. Pour plus d’informations, consultez l’opération [Ajouter une tâche à un travail](/rest/api/batchservice/task/add?) (API REST Batch) ou les propriétés [CloudTask.EnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudtask) et [CloudJob.CommonEnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudjob) dans .NET Batch.

L’application cliente ou le service peuvent obtenir des variables d’environnement d’une tâche, à la fois définies par le service et personnalisées, à l’aide de l’opération [Obtenir des informations sur une tâche](/rest/api/batchservice/task/get) (REST Batch) ou en accédant à la propriété [CloudTask.EnvironmentSettings](/dotnet/api/microsoft.azure.batch.cloudtask) (.NET Batch). Les processus qui s’exécutent sur un nœud de calcul peuvent accéder à ces dernières ainsi qu’à d’autres variables d’environnement sur le nœud, par exemple à l’aide de la syntaxe classique `%VARIABLE_NAME%` (Windows) ou la syntaxe `$VARIABLE_NAME` (Linux).

Vous trouverez la liste complète des variables d’environnement définies par le service dans l’article [Compute node environment variables](batch-compute-node-environment-variables.md) (Variables d’environnement de nœud de calcul).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [fichiers et répertoires](files-and-directories.md).
