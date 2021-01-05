---
title: Rechercher les erreurs des pools et des nœuds
description: Cet article décrit les opérations d'arrière-plan qui peuvent se produire, ainsi que les erreurs à rechercher et comment les éviter lors de la création de pools et de nœuds.
author: mscurrell
ms.author: markscu
ms.date: 08/23/2019
ms.topic: how-to
ms.openlocfilehash: 519b357e4e5fde30221f7dc804bb848ecec9704c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85979915"
---
# <a name="check-for-pool-and-node-errors"></a>Rechercher les erreurs des pools et des nœuds

Lorsque vous créez et gérez des pools Azure Batch, certaines opérations se produisent immédiatement. Toutefois, certaines opérations sont asynchrones et s’exécutent en arrière-plan, ce qui prend plusieurs minutes.

La détection des échecs des opérations qui ont lieu immédiatement est simple, car les échecs éventuels sont retournés immédiatement par l’API, l’interface CLI ou l’interface utilisateur.

Cet article traite des opérations d’arrière-plan qui peuvent survenir pour les groupes et les nœuds de groupe. Il indique comment vous pouvez détecter et éviter les échecs.

## <a name="pool-errors"></a>Erreurs de pool

### <a name="resize-timeout-or-failure"></a>Dépassement du délai d’attente ou échec du redimensionnement

Lors de la création d’un pool ou du redimensionnement d’un pool existant, vous spécifiez le nombre de nœuds cible.  L’opération de création ou de redimensionnement se termine immédiatement, mais l’allocation réelle des nouveaux nœuds ou la suppression de nœuds existants peut prendre plusieurs minutes.  Vous spécifiez le délai d’expiration du redimensionnement dans l’API [create](/rest/api/batchservice/pool/add) ou l’API [resize](/rest/api/batchservice/pool/resize). Si Batch ne peut pas obtenir le nombre cible de nœuds pendant le délai d'expiration du redimensionnement, le pool passe à l’état stable et signale les erreurs de redimensionnement.

La propriété [ResizeError](/rest/api/batchservice/pool/get#resizeerror) de l’évaluation la plus récente répertorie toutes les erreurs survenues.

Les causes courantes des erreurs de redimensionnement sont notamment les suivantes :

- Le délai d’expiration du redimensionnement est trop court
  - Dans la plupart des cas, le délai d’attente par défaut de 15 minutes est suffisamment long pour que les nœuds de pool puissent être affectés ou supprimés.
  - Si vous allouez un grand nombre de nœuds, nous vous recommandons de régler le délai d’expiration du redimensionnement à 30 minutes. Par exemple, lorsque vous redimensionnez plus de 1 000 nœuds à partir d’une image Place de marché Microsoft Azure, ou plus de 300 nœuds à partir d’une image de machine virtuelle personnalisée.
- Quota de cœurs insuffisant
  - Un compte Batch est limité dans le nombre de cœurs qu’il peut allouer dans tous les pools. Batch arrête l’allocation de nœuds une fois que le quota a été atteint. Vous [ pouvez augmenter](./batch-quota-limit.md) le quota de cœurs pour que Batch puisse allouer plus de nœuds.
- Nombre insuffisant d’adresses IP du sous-réseau quand un [pool est dans un réseau virtuel](./batch-virtual-network.md)
  - Un sous-réseau de réseau virtuel doit avoir suffisamment d’adresses IP non affectées à allouer à chaque nœud du pool demandés. Sinon, les nœuds ne peuvent pas être créés.
- Ressources insuffisantes quand un [pool est dans un réseau virtuel](./batch-virtual-network.md)
  - Vous pouvez créer des ressources comme des équilibreurs de charge, des adresses IP publiques et des groupes de sécurité réseau dans le même abonnement que le compte Batch. Vérifiez que les quotas de l’abonnement sont suffisants pour ces ressources.
- Vastes pools avec images de machines virtuelles personnalisées
  - L’allocation de vastes pools utilisant des images de machine virtuelle personnalisées peut prendre plus de temps et des dépassements du délai d’expiration peuvent se produire.  Pour obtenir des recommandations sur les limites et la configuration, voir [Créer un pool avec Shared Image Gallery](batch-sig-images.md).

### <a name="automatic-scaling-failures"></a>Échecs de mise à l’échelle automatique

Vous pouvez également configurer Azure Batch pour mettre automatiquement à l’échelle le nombre de nœuds dans un pool. Vous définissez les paramètres de la [formule de mise à l’échelle automatique d’un pool](./batch-automatic-scaling.md). Le service Batch utilise la formule pour évaluer régulièrement le nombre de nœuds dans le pool et définir un nouveau nombre de cibles. Les types de problèmes suivants peuvent se produire :

- L’évaluation de la mise à l’échelle automatique échoue.
- L’opération de redimensionnement qui en résulte échoue et dépasse le délai d’expiration.
- Un problème portant sur la formule de mise à l’échelle automatique produit des valeurs cibles de nœud incorrectes. Le redimensionnement fonctionne ou arrive à expiration.

Vous pouvez obtenir des informations sur la dernière évaluation automatique de mise à l’échelle à l’aide de la propriété [autoScaleRun](/rest/api/batchservice/pool/get#autoscalerun). Cette propriété rapporte le temps d’évaluation, les valeurs et les résultats, ainsi que les éventuelles erreurs de performance.

Un [événement de fin de redimensionnement de pool](./batch-pool-resize-complete-event.md) capture les informations sur toutes les évaluations.

### <a name="delete"></a>DELETE

Lorsque vous supprimez un pool qui contient des nœuds, le premier Batch supprime les nœuds. Il supprime ensuite l’objet pool lui-même. Quelques minutes peuvent être nécessaires pour supprimer les nœuds du pool.

Batch définit l’[état du pool](/rest/api/batchservice/pool/get#poolstate) sur **En cours de suppression** pendant le processus de suppression. L’application appelante peut détecter si la suppression du pool prend trop de temps en utilisant les propriétés **state** et **stateTransitionTime**.

## <a name="pool-compute-node-errors"></a>Erreurs des nœuds de calcul du pool

Même lorsque Batch alloue des nœuds d’un pool, différents problèmes peuvent les endommager ou les rendre incapables d’exécuter des tâches. Ces nœuds entraînent toujours des frais. Il est donc important de détecter les problèmes afin d’éviter de payer pour des nœuds qui ne peuvent pas être utilisés. En plus des erreurs de nœud courantes, il est utile de connaître l’[état actuel de la tâche](/rest/api/batchservice/job/get#jobstate) en vue de la résolution des problèmes.

### <a name="start-task-failures"></a>Échecs de la tâche de démarrage

Vous souhaiterez peut-être indiquer une [tâche de démarrage](/rest/api/batchservice/pool/add#starttask) facultative pour un pool. Comme avec n’importe quelle tâche, vous pouvez utiliser une ligne de commande et des fichiers de ressources à télécharger à partir du stockage. La tâche de démarrage est exécutée pour chaque nœud après son lancement. La propriété **waitForSuccess** indique si Batch attend que la tâche de démarrage se termine avec succès avant de planifier des tâches vers un nœud.

Que se passe-t-il si vous avez configuré le nœud de sorte qu’il attende l’achèvement réussi de la tâche de démarrage mais que la tâche de démarrage échoue ? Dans ce cas, le nœud ne sera pas utilisable, mais il entraînera toujours des frais.

Vous pouvez détecter les échecs des tâches de démarrage via les propriétés [result](/rest/api/batchservice/computenode/get#taskexecutionresult) et [failureInfo](/rest/api/batchservice/computenode/get#taskfailureinformation) de la propriété du nœud [startTaskInfo](/rest/api/batchservice/computenode/get#starttaskinformation) du plus haut niveau.

Une tâche de démarrage qui a échoué fait passer [l’état](/rest/api/batchservice/computenode/get#computenodestate) du nœud à **starttaskfailed** si **waitForSuccess** a été défini sur **true**.

Comme avec n’importe quelle tâche, plusieurs causes d’échec de la tâche de démarrage sont possibles.  Pour résoudre les problèmes, examinez les fichiers journaux stdout, stderr et tous les fichiers journaux spécifiques aux tâches.

Les tâches de démarrage doivent être réentrantes, car il est possible que la tâche de démarrage soit exécutée plusieurs fois sur le même nœud ; la tâche de démarrage est exécutée lors de la réinitialisation ou du redémarrage d’un nœud. Dans de rares cas, une tâche de démarrage est exécutée après un événement provoquant un redémarrage du nœud, où l’un des systèmes d’exploitation ou des disques éphémères a été réinitialisé alors que l’autre ne l’a pas été. Étant donné que les tâches de démarrage de Batch (comme toutes les tâches Batch) sont exécutées à partir du disque éphémère, ce n’est normalement pas un problème. Cependant, dans certains cas, où la tâche de démarrage installe une application sur le disque du système d'exploitation et conserve les autres données sur le disque éphémère, des problèmes liés à l’absence de synchronisation peuvent survenir. Protégez votre application en conséquence si vous utilisez les deux disques.

### <a name="application-package-download-failure"></a>Échec du téléchargement des packages d’application

Vous pouvez spécifier un ou plusieurs packages d’application pour un pool. Batch télécharge les fichiers de package spécifiés vers chaque nœud et décompresse les fichiers après le démarrage du nœud, mais avant que les tâches ne soient planifiées. Il est courant d’utiliser une ligne de commande de tâche de démarrage conjointement avec des packages d’application. Par exemple, pour copier des fichiers vers un emplacement différent ou exécuter le programme d’installation.

La propriété [errors](/rest/api/batchservice/computenode/get#computenodeerror) du nœud rapporte un échec du téléchargement et de décompression d’un package d’application ; l’état du nœud est défini sur **inutilisable**.

### <a name="container-download-failure"></a>Échec du téléchargement du conteneur

Vous pouvez spécifier une ou plusieurs références de conteneur sur un pool. Batch télécharge les conteneurs spécifiés pour chaque nœud. La propriété [erreurs](/rest/api/batchservice/computenode/get#computenodeerror) du nœud signale l’échec de téléchargement d’un conteneur et définit l’état du nœud sur **inutilisable**.

### <a name="node-in-unusable-state"></a>Nœud dans un état inutilisable

Azure Batch peut définir [l’état du nœud](/rest/api/batchservice/computenode/get#computenodestate) sur **inutilisable** pour de nombreuses raisons. Quand l’état du nœud est défini sur **inutilisable**, il n’est pas possible de planifier des tâches pour le nœud, mais il continue d’entraîner la facturation de frais.

Lorsque des nœuds sont en état **inutilisable** sans [erreurs](/rest/api/batchservice/computenode/get#computenodeerror), cela signifie que Batch n’est pas en mesure de communiquer avec la machine virtuelle. Dans ce cas, Batch tente toujours de récupérer la machine virtuelle. Batch ne tente pas automatiquement de récupérer les machines virtuelles qui n’ont pas réussi à installer les packages d’applications ou les conteneurs, même si leur état est **inutilisable**.

Si Batch peut déterminer la cause de l’erreur, Batch la signale à l’aide de la propriété [errors](/rest/api/batchservice/computenode/get#computenodeerror).

Voici des exemples supplémentaires de causes aboutissant à des nœuds **inutilisables** :

- Une image de machine virtuelle personnalisée n’est pas valide. Par exemple, il s’agit d’une image qui n’est pas correctement préparée.

- Une machine est déplacée en raison d’une défaillance de l’infrastructure ou d’une mise à niveau de bas niveau. Batch récupère le nœud.

- Une image de machine virtuelle a été déployée sur un matériel qui ne la prend pas en charge. Par exemple, si vous essayez d’exécuter une image HPC CentOS sur une machine virtuelle [Standard_D1_v2](../virtual-machines/dv2-dsv2-series.md).

- Les machines virtuelles sont dans un [réseau virtuel Azure](batch-virtual-network.md), et le trafic a été bloqué pour les ports principaux.

- Les machines virtuelles se trouvent dans un réseau virtuel, mais le trafic sortant vers le stockage Azure est bloqué.

- Les machines virtuelles se trouvent dans un réseau virtuel avec une configuration DNS du client et le serveur DNS ne peut pas résoudre le stockage Azure.

### <a name="node-agent-log-files"></a>Fichiers de journaux de l’agent du nœud

Si vous devez contacter le support technique concernant un problème de nœud de pool, le processus de l’agent Batch qui s’exécute sur chaque nœud du pool peut vous fournir des fichiers journaux très utiles. Les fichiers journaux pour un nœud peuvent être chargés via le portail Azure, Batch Explorer ou une [API](/rest/api/batchservice/computenode/uploadbatchservicelogs). Il est utile de charger et d’enregistrer les fichiers journaux. Ensuite, vous pouvez supprimer le nœud ou le pool pour maîtriser le coût des nœuds en cours d’exécution.

### <a name="node-disk-full"></a>Disque de nœud plein

Le lecteur temporaire d’une machine virtuelle de nœud de pool est utilisé par Batch pour les fichiers de travail, les fichiers de tâches et les fichiers partagés.

- Fichiers de packages d’applications
- Fichiers de ressources de tâche
- Fichiers spécifiques à l’application téléchargés vers l’un des dossiers Batch
- Fichiers stdout et stderr pour chaque exécution d’application de tâche
- Fichiers de sortie spécifiques à l’application

Certains de ces fichiers sont écrits une seule fois lors de la création de nœuds de pool, tels que les packages d’applications de pools ou les fichiers de ressources de tâche de démarrage de pool. Même s’ils sont écrits une seule fois lors de la création du nœud, ces fichiers peuvent remplir le lecteur temporaire s’ils sont trop volumineux.

D’autres fichiers sont écrits pour chaque tâche exécutée sur un nœud, par exemple stdout et stderr. Si un grand nombre de tâches s’exécutent sur le même nœud et/ou que les fichiers de tâches sont trop volumineux, ils peuvent remplir le lecteur temporaire.

La taille du lecteur temporaire dépend de la taille de la machine virtuelle. L’un des points à prendre en compte lors du choix de la taille d’une machine virtuelle consiste à s’assurer que le lecteur temporaire dispose de suffisamment d’espace.

- Dans le portail Azure, lors de l’ajout d’un pool, la liste complète des tailles de machines virtuelles peut être affichée et il existe une colonne « Taille du disque de ressources ».
- Les articles décrivant toutes les tailles de machines virtuelles, par exemple [Tailles de machines virtuelles optimisées pour le calcul](../virtual-machines/sizes-compute.md), ont des tables avec une colonne « Stockage temporaire ».

Concernant les fichiers écrits par chaque tâche, une durée de rétention déterminant la durée pendant laquelle ils seront conservés avant d’être automatiquement nettoyés peut respectivement être spécifiée. La durée de rétention peut être raccourcie pour réduire les exigences de stockage.


Si le disque temporaire manque (ou est sur le point de manquer) d’espace, le nœud passera à l’état [Inutilisable](/rest/api/batchservice/computenode/get#computenodestate) et une erreur de nœud indiquera que le disque est plein.

### <a name="what-to-do-when-a-disk-is-full"></a>Procédure à suivre lorsqu’un disque est plein

Déterminez la raison pour laquelle le disque est plein : Si vous ne savez pas ce qui occupe de l’espace sur le nœud, nous vous recommandons d’y accéder à distance et d’examiner manuellement à quoi cet espace est utilisé. Vous pouvez également recourir à [l’API Batch Lister les fichiers](/rest/api/batchservice/file/listfromcomputenode) pour examiner les fichiers des dossiers gérés par Batch (par exemple, les sorties de tâches). Il faut savoir que cette API ne répertorie que les fichiers présents dans les répertoires gérés par Batch ; si vos tâches ont créé des fichiers ailleurs, ceux-ci n’apparaîtront pas.

Vérifiez que toutes les données dont vous avez besoin ont été récupérées à partir du nœud ou chargées dans un magasin durable. L’atténuation du problème de disque plein implique nécessairement la suppression de données pour libérer de l’espace.

### <a name="recovering-the-node"></a>Récupération du nœud

1. Si votre pool est un pool [C.loudServiceConfiguration](/rest/api/batchservice/pool/add#cloudserviceconfiguration), vous pouvez recréer l’image du nœud avec [l’API Batch Recréation d’image](/rest/api/batchservice/computenode/reimage), qui nettoie la totalité du disque. La recréation d’image n’est à l’heure actuelle pas prise en charge pour les pools [VirtualMachineConfiguration](/rest/api/batchservice/pool/add#virtualmachineconfiguration).

2. Si votre pool est un pool [VirtualMachineConfiguration](/rest/api/batchservice/pool/add#virtualmachineconfiguration), vous pouvez supprimer le nœud du pool avec [l’API Suppression de nœuds](/rest/api/batchservice/pool/removenodes), puis agrandir le pool pour remplacer le mauvais nœud par un nouveau.

3.  Supprimez les anciennes tâches et les anciens travaux terminés dont les données se trouvent toujours sur les nœuds. Pour savoir quelles données sont concernées, vous pouvez regarder dans la [collection RecentTasks](/rest/api/batchservice/computenode/get#taskinformation) sur le nœud, ou examiner les [fichiers sur le nœud](/rest/api/batchservice/file/listfromcomputenode). La suppression du travail entraîne la suppression de toutes les tâches qu’il comporte, qui elle entraîne la suppression des données des répertoires de tâches du nœud, ce qui libère de l’espace. Une fois que vous avez libéré suffisamment d’espace, redémarrez le nœud ; il devrait repasser de l’état « Inutilisable » à l’état « Inactif ».

## <a name="next-steps"></a>Étapes suivantes

Vérifiez que vous avez paramétré votre application pour implémenter le contrôle complet des erreurs, en particulier pour les opérations asynchrones. Il peut être essentiel de rapidement détecter et diagnostiquer les problèmes.
