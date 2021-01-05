---
title: Gestion et détection des erreurs dans Azure Batch
description: En savoir plus sur la gestion des erreurs dans les workflows du service Batch du point de vue du développement.
ms.topic: article
ms.date: 05/15/2020
ms.openlocfilehash: 3bd460598dae08fa18415e1c9865249f3ca4c9c2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85964275"
---
# <a name="error-handling-and-detection-in-azure-batch"></a>Gestion et détection des erreurs dans Azure Batch

Parfois, il peut s’avérer nécessaire de gérer les échecs de tâche et d’application au sein de votre solution Batch. Cet article traite des types d’erreurs et explique comment les résoudre.

## <a name="error-codes"></a>Codes d’erreur

Les types d’erreurs généraux sont les suivants :

- Échecs de mise en réseau pour les requêtes qui n’ont jamais atteint Batch ou lorsque la réponse de Batch n’a pas atteint le client dans le temps imparti.
- Erreurs de serveur internes (réponse HTTP du code d’état 5xx standard).
- Erreurs liées à la limitation de bande passante, telles que les réponses HTTP du code d’état 429 ou 503 avec l’en-tête Retry-after.
- Erreurs 4xx telles que AlreadyExists et InvalidOperation. Cela signifie que la ressource n’est pas dans l’état correct pour la transition d’état.

Pour plus d’informations sur les codes d’erreur spécifiques, notamment les codes d’erreur pour l’API REST, le service Batch et la planification de travail ou de tâche, consultez [Codes d’état et d’erreur de Batch](/rest/api/batchservice/batch-status-and-error-codes).

## <a name="application-failures"></a>Échecs d’application

Pendant l’exécution, une application peut produire des diagnostics qui vous permettent de résoudre les problèmes. Comme décrit dans [Fichiers et répertoires](files-and-directories.md), le service Batch écrit des sorties et des sorties d’erreur standard dans les fichiers `stdout.txt` et `stderr.txt` du répertoire de tâche sur le nœud de calcul.

Vous pouvez utiliser le portail Azure ou l’un des Kits de développement logiciel (SDK) Batch pour télécharger ces fichiers. Par exemple, vous pouvez récupérer ces fichiers et d’autres à des fins de résolution des problèmes en utilisant [ComputeNode.GetNodeFile](/dotnet/api/microsoft.azure.batch.computenode) et [CloudTask.GetNodeFile](/dotnet/api/microsoft.azure.batch.cloudtask) dans la bibliothèque .NET Batch.

## <a name="task-errors"></a>Erreurs de tâche

Les erreurs de tâche se répartissent en plusieurs catégories.

### <a name="pre-processing-errors"></a>Erreurs de prétraitement

Si une tâche ne parvient pas à démarrer, une erreur de prétraitement est définie pour la tâche.  

Des erreurs de prétraitement peuvent se produire si des fichiers de ressources de la tâche ont été déplacés, si le compte de stockage n’est plus disponible ou si un autre problème ayant empêché la copie des fichiers sur le nœud est survenu.

### <a name="file-upload-errors"></a>Erreurs de chargement du fichier

Si le chargement des fichiers qui sont spécifiés pour une tâche échoue pour une raison quelconque, une erreur de chargement du fichier est définie pour la tâche.

Des erreurs de chargement de fichier peuvent se produire si la SAP permettant d’accéder au Stockage Azure n’est pas valide ou n’accorde pas d’autorisations d’écriture, si le compte de stockage n’est plus disponible ou si un autre problème ayant empêché la copie des fichiers sur le nœud est survenu.

### <a name="application-errors"></a>Erreurs d’application

Le processus spécifié par la ligne de commande de la tâche peut également échouer. Il est considéré comme ayant échoué lorsqu’un code de sortie différent de zéro est renvoyé par le processus exécuté par la tâche (voir *Codes de sortie de tâche* dans la section ci-après).

Pour les erreurs d’application, vous pouvez configurer Batch pour relancer automatiquement la tâche autant de fois que vous l’aurez spécifié.

### <a name="constraint-errors"></a>Erreurs de contrainte

Vous pouvez définir une contrainte qui spécifie la durée maximale d’exécution d’un travail ou d’une tâche, le *maxWallClockTime*. Cette opération peut se révéler utile pour terminer les tâches dont la progression échoue.

Lorsque la durée maximale est dépassée, la tâche est marquée comme *terminée*, mais le code de sortie est défini sur `0xC000013A` et le champ *schedulingError* est marqué comme `{ category:"ServerError", code="TaskEnded"}`.

## <a name="task-exit-codes"></a>Codes de sortie de tâche

Comme mentionné précédemment, une tâche est marquée comme ayant échoué par le service Batch si le processus exécuté par la tâche retourne un code de sortie différent de zéro. Lorsqu’une tâche exécute un processus, Batch remplit la propriété de code de sortie de la tâche avec le code de retour du processus.

Il est important de noter que le code de sortie d’une tâche n’est pas déterminé par le service Batch, mais par le processus proprement dit ou par le système d’exploitation sur lequel le processus est exécuté.

## <a name="task-failures-or-interruptions"></a>Échecs ou interruptions de tâche

Les tâches peuvent parfois échouer ou être interrompues. L’application de la tâche elle-même peut échouer, le nœud sur lequel elle s’exécute peut être redémarré, voire supprimé du pool pendant le redimensionnement de ce dernier (si la stratégie de désallocation du pool définie prévoit la suppression immédiate des nœuds, sans attendre la fin des tâches). Dans tous les cas, la tâche peut être automatiquement remise en file d’attente par Batch pour être exécutée sur un autre nœud.

Un problème intermittent peut également provoquer la non-réponse soudaine d’une tâche ou ralentir son exécution. Vous pouvez définir l’intervalle d’exécution maximal d’une tâche. Si l’intervalle d’exécution maximal est dépassé, le service Batch interrompt l’application de la tâche.

## <a name="connect-to-compute-nodes"></a>Connexion aux nœuds de calcul

Vous pouvez effectuer des actions supplémentaires de débogage et de résolution des problèmes en vous connectant à un nœud de calcul à distance. Vous pouvez utiliser le portail Azure pour télécharger un fichier RDP pour les nœuds Windows et obtenir des informations de connexion SSH pour les nœuds Linux. Vous pouvez également effectuer cette opération à l’aide des API Batch, par exemple [.NET Batch](/dotnet/api/microsoft.azure.batch.computenode) ou [Python Batch](batch-linux-nodes.md#connect-to-linux-nodes-using-ssh).

> [!IMPORTANT]
> Pour vous connecter à un nœud via RDP ou SSH, vous devez d’abord créer un utilisateur sur le nœud. Pour ce faire, vous pouvez utiliser le Portail Azure, [ajouter un compte d’utilisateur à un nœud](/rest/api/batchservice/computenode/adduser) en utilisant l’API REST Batch, appeler la méthode [ComputeNode.CreateComputeNodeUser](/dotnet/api/microsoft.azure.batch.computenode) dans .NET Batch ou appeler la méthode [add_user](batch-linux-nodes.md#connect-to-linux-nodes-using-ssh) dans le module Python de Batch.

Pour limiter ou désactiver l’accès RDP ou SSH aux nœuds de calcul, consultez [Configure or disable remote access to compute nodes in an Azure Batch pool](pool-endpoint-configuration.md) (Configurer ou désactiver l’accès distant aux nœuds de calcul dans un pool Azure Batch).

## <a name="troubleshoot-problem-nodes"></a>Résolution des problèmes liés aux nœuds

Quand certaines de vos tâches échouent, votre application cliente Batch ou un service peut examiner les métadonnées des tâches en échec pour identifier un nœud présentant un dysfonctionnement. Chaque nœud d’un pool se voit attribuer un ID unique et le nœud sur lequel s’exécute une tâche est inclus dans les métadonnées de la tâche. Une fois que vous avez identifié le nœud présentant un problème, vous pouvez effectuer les actions suivantes :

- **Redémarrer le nœud** ([REST](/rest/api/batchservice/computenode/reboot) | [.NET](/dotnet/api/microsoft.azure.batch.computenode.reboot))

    Le fait de redémarrer le nœud peut parfois résoudre des problèmes latents comme des processus bloqués ou défaillants. Si votre pool utilise une tâche de démarrage ou si votre travail utilise une tâche de préparation, ces deux éléments s’exécutent au redémarrage du nœud.
- **Réinitialiser le nœud** ([REST](/rest/api/batchservice/computenode/reimage) | [.NET](/dotnet/api/microsoft.azure.batch.computenode.reimage))

    Cette opération réinstalle le système d’exploitation sur le nœud. Comme avec le redémarrage d’un nœud, les tâches de démarrage et celles de préparation d’un travail sont relancées une fois le nœud réinitialisé.
- **Supprimer le nœud du pool** ([REST](/rest/api/batchservice/pool/removenodes) | [.NET](/dotnet/api/microsoft.azure.batch.pooloperations))

    Il est parfois nécessaire de supprimer entièrement le nœud à partir du pool.
- **Désactiver la planification des tâches sur le nœud** ([REST](/rest/api/batchservice/computenode/disablescheduling) | [.NET](/dotnet/api/microsoft.azure.batch.computenode.disablescheduling))

    Cette opération met le nœud hors connexion, de sorte qu’aucune tâche supplémentaire ne peut lui être affectée. Toutefois, le nœud est autorisé à poursuivre son exécution et à rester dans le pool. Cela vous permet de faire une recherche approfondie sur la cause des échecs sans perdre les données de la tâche en échec et sans que le nœud n’occasionne d’autres échecs de tâche supplémentaires. Par exemple, vous pouvez désactiver la planification des tâches sur le nœud, puis vous connecter à distance pour examiner les journaux des événements de ce nœud ou encore résoudre d’autres problèmes. Après avoir terminé votre recherche, vous pouvez remettre le nœud en ligne en activant la planification des tâches ([REST](/rest/api/batchservice/computenode/enablescheduling) | [.NET](/dotnet/api/microsoft.azure.batch.computenode.enablescheduling)) ou exécuter l’une des autres actions indiquées précédemment.

> [!IMPORTANT]
> À l’aide des actions décrites ci-dessus, vous pouvez spécifier la manière dont les tâches en cours d’exécution sur le nœud sont traitées lorsque vous effectuez l’action. Par exemple, lorsque vous désactivez la planification des tâches sur un nœud à l’aide de la bibliothèque client .NET Batch, vous pouvez spécifier une valeur d’énumération [DisableComputeNodeSchedulingOption](/dotnet/api/microsoft.azure.batch.common.disablecomputenodeschedulingoption) pour préciser s’il faut **terminer** les tâches en cours d’exécution, les **remettre en file d’attente** pour les planifier sur d’autres nœuds ou finaliser les tâches en cours avant d’exécuter l’action (**TaskCompletion**).

## <a name="retry-after-errors"></a>Nouvelle tentative après erreur

Les API du service Batch vous envoient une notification en cas d’erreur. Elles permettent toutes de faire une nouvelle tentative et elles incluent toutes un gestionnaire de nouvelles tentatives global à cet effet. Il est préférable d’utiliser ce mécanisme intégré.

Après une erreur, vous devez attendre un peu (quelques secondes entre deux nouvelles tentatives) avant de réessayer. Si vous réessayez trop souvent ou trop rapidement, le gestionnaire de nouvelles tentatives limite la bande passante.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [rechercher les erreurs de pool et de nœud](batch-pool-node-error-checking.md).
- Découvrez comment [rechercher les erreurs de travail et de tâche](batch-job-task-error-checking.md).
- Passez en revue la liste des [codes d’erreur et d’état de Batch](/rest/api/batchservice/batch-status-and-error-codes).
