---
title: Nœuds et pools dans Azure Batch
description: En savoir plus sur les nœuds de calcul et les pools et leur utilisation dans un flux de travail Azure Batch du point de vue du développeur.
ms.topic: conceptual
ms.date: 11/20/2020
ms.openlocfilehash: 880a956a2d839483c59578afad1b62146799578a
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95243067"
---
# <a name="nodes-and-pools-in-azure-batch"></a>Nœuds et pools dans Azure Batch

Dans un workflow Azure Batch, un *nœud de calcul* (ou *nœud*) est une machine virtuelle qui traite une partie de la charge de travail de votre application. Un *pool* est une collection regroupant ces nœuds, sur lesquels votre application s’exécute. Cet article explique plus en détail les nœuds et les pools, ainsi que les éléments à prendre en compte lors de leur création et de leur utilisation dans un flux de travail Azure Batch.

## <a name="nodes"></a>Nœuds

Un nœud est une machine virtuelle Azure ou une machine virtuelle de service cloud dédiée au traitement d’une partie de la charge de travail de votre application. La taille d’un nœud détermine le nombre de cœurs du processeur, la capacité de mémoire et la taille du système de fichiers local qui lui est allouée.

Vous pouvez créer des pools de nœuds Windows ou Linux à l’aide des services Azure Cloud Services, des images provenant de la [Place de marché de machines virtuelles Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/compute?filters=virtual-machine-images&page=1) ou des images personnalisées par vos soins.

Les nœuds peuvent exécuter tout fichier exécutable ou script pris en charge par l’environnement de système d’exploitation du nœud. Les exécutables ou les scripts incluent \*.exe, \*.cmd et \*.bat et les scripts PowerShell (pour Window)s, les fichiers binaires, ainsi que les scripts shell et Python (pour Linux).

Tous les nœuds de calcul Batch incluent également les éléments suivants :

- Une [structure de dossiers](files-and-directories.md) standard et des [variables d’environnement](jobs-and-tasks.md) associées pouvant être référencées par les tâches.
- **pare-feu** configurés pour le contrôle de l’accès.
- [Accès à distance](error-handling.md#connect-to-compute-nodes) aux nœuds Windows (Remote Desktop Protocol, RDP) et Linux (Secure Shell, SSH) (sauf si vous [créez votre pool avec l’accès à distance désactivé](pool-endpoint-configuration.md)).

Par défaut, les nœuds peuvent communiquer entre eux, mais ils ne peuvent pas communiquer avec les machines virtuelles qui ne font pas partie du même pool. Pour permettre aux nœuds de communiquer en toute sécurité avec d’autres machines virtuelles, ou avec un réseau local, vous pouvez provisionner le pool [dans un sous-réseau d’un réseau virtuel Azure (VNet)](batch-virtual-network.md). Dans ce cas, vos nœuds sont accessibles via des adresses IP publiques. Ces adresses IP publiques sont créées par Batch et peuvent changer au cours de la durée de vie du pool. Vous pouvez également [créer un pool avec des adresses IP publiques statiques](create-pool-public-ip.md) que vous contrôlez, ce qui garantit qu’elles ne changeront pas de manière inattendue.

## <a name="pools"></a>Pools

Un pool est une collection de nœuds sur lesquels votre application s’exécute.

Les pools Azure Batch sont créés en parallèle à la plate-forme de calcul Azure principale. Ils permettent d’installer des applications, de répartir les données, et d’effectuer des allocations à grande échelle, des analyses de fonctionnement et des ajustements flexibles du nombre de nœuds de calcul au sein d’un pool ([mise à l’échelle](#automatic-scaling-policy)).

Chaque nœud ajouté à un pool se voit attribuer un nom unique et l’adresse IP. Lorsqu’un nœud est supprimé d’un pool, toutes les modifications apportées au système d’exploitation ou aux fichiers sont perdues, et son nom et son adresse IP sont libérés pour une utilisation ultérieure. Lorsqu’un nœud quitte un pool, sa durée de vie est terminée.

Un pool ne peut être utilisé que par le compte Batch dans lequel il a été créé. Un compte Batch peut créer plusieurs pools pour répondre aux besoins en ressources des applications qu’il exécute.

Vous pouvez créer le pool manuellement ou il peut être créé [automatiquement par le service Batch](#autopools) lorsque vous spécifiez le travail à accomplir. Lorsque vous créez un pool, vous pouvez spécifier les attributs suivants :

- [Système d’exploitation et version du nœud](#operating-system-and-version)
- [Type des nœuds et nombre cible de nœuds](#node-type-and-target)
- [Taille du nœud](#node-size)
- [Stratégie de mise à l’échelle automatique](#automatic-scaling-policy)
- [Stratégie de planification de tâches](#task-scheduling-policy)
- [État des communications](#communication-status)
- [Tâches de départ](#start-tasks)
- [Packages d’applications](#application-packages)
- [Configuration du pare-feu et du réseau virtuel (VNet)](#virtual-network-vnet-and-firewall-configuration)
- [Durée de vie](#pool-and-compute-node-lifetime)

> [!IMPORTANT]
> Les comptes Batch ont un quota par défaut qui limite le nombre de cœurs dans un compte Batch. Le nombre de cœurs correspond au nombre de nœuds de calcul. Pour en savoir plus sur les quotas par défaut et obtenir des instructions sur [l’augmentation d’un quota](batch-quota-limit.md#increase-a-quota), consultez l’article [Quotas et limites pour le service Azure Batch](batch-quota-limit.md). Si votre pool n’a pas obtenu son nombre cible de nœuds, cela peut être dû au quota principal.

## <a name="operating-system-and-version"></a>Système d’exploitation et version

Lorsque vous créez un pool Batch, vous pouvez spécifier la configuration de machines virtuelles Azure et le type de système d’exploitation que vous souhaitez exécuter sur chaque nœud de calcul dans le pool.

## <a name="configurations"></a>Configurations

Deux types de configurations de pool sont disponibles dans Batch.

### <a name="virtual-machine-configuration"></a>Configuration de la machine virtuelle

La **configuration de machines virtuelles** indique que le pool est composé de machines virtuelles Azure. Ces machines virtuelles peuvent être créées à partir d’images Linux ou Windows.

L’[agent de nœud de Batch](https://github.com/Azure/Batch/blob/master/changelogs/nodeagent/CHANGELOG.md) est un programme qui s’exécute sur chaque nœud dans le pool et fournit l’interface de commande et de contrôle entre le nœud et le service Batch. Il existe différentes implémentations de l’agent de nœud pour différents systèmes d’exploitation, connues sous le nom de références. Lorsque vous créez un pool basé sur la configuration de machine virtuelle, vous devez spécifier la taille des nœuds, mais aussi la source et les images utilisées pour les créer, la **référence d’image de machine virtuelle** et la **référence SKU de l’agent du nœud** à installer sur les nœuds. Pour plus d’informations sur la spécification des propriétés de pool, voir [Configurer des nœuds de calcul Linux dans des pools Azure Batch](batch-linux-nodes.md). Vous pouvez éventuellement attacher un ou plusieurs disques de données vides au pool de machines virtuelles créé à partir d’images de la Place de Marché, ou inclure des disques de données dans des images personnalisées utilisées pour créer les machines virtuelles. Quand vous incluez des disques de données, vous devez monter et formater les disques à partir d’une machine virtuelle pour les utiliser.

### <a name="cloud-services-configuration"></a>Configuration des Services cloud

La **configuration des Services cloud** indique que le pool est composé de nœuds Azure Cloud Services. Les Services cloud fournissent uniquement des nœuds de calcul Windows.

Les systèmes d’exploitation disponibles pour les pools Configuration des Services cloud sont répertoriés dans la [matrice de compatibilité entre les versions du système d’exploitation invités Azure et les kits de développement logiciel (SDK)](../cloud-services/cloud-services-guestos-update-matrix.md), et les tailles de nœuds de calcul disponibles sont répertoriés dans [Tailles pour Cloud Services](../cloud-services/cloud-services-sizes-specs.md). Lorsque vous créez un pool qui contient des nœuds Microsoft Azure Cloud Services, vous spécifiez la taille du nœud et sa *famille de systèmes d’exploitation* (laquelle détermine les versions de .NET installées avec le système d’exploitation concerné). Les Services cloud sont déployés plus rapidement sur Azure que les machines virtuelles exécutant Windows. Si vous souhaitez créer des nœuds de calcul Windows, vous pouvez constater que Cloud Services propose un délai de déploiement moins important.

Comme avec les rôles de travail dans Cloud Services, vous pouvez spécifier une *Version du système d’exploitation* (pour plus d’informations sur les rôles de travail, consultez l’article [Vue d’ensemble de Services cloud](../cloud-services/cloud-services-choose-me.md)). Nous vous recommandons de spécifier `Latest (*)` comme *version du système d’exploitation* afin que les nœuds soient automatiquement mis à niveau et qu’aucun travail supplémentaire ne soit requis pour gérer ces nouvelles versions. La principale raison de sélectionner une version de système d’exploitation spécifique est d’assurer la compatibilité des applications, ce qui permet aux tests de compatibilité descendante d’être réalisés avant d’autoriser la mise à jour de la version. Une fois validée, la *version du système d’exploitation* du pool peut être mise à jour et la nouvelle image du système d’exploitation peut être installée. Toutes les tâches en cours d’exécution sont interrompues et remises en file d’attente.

### <a name="node-agent-skus"></a>Références SKU de l’agent de nœud

Lorsque vous créez un pool, vous devez sélectionner la valeur **nodeAgentSkuId** appropriée, selon le système d’exploitation de l’image de base de votre VHD. Vous pouvez obtenir un mappage des ID de référence SKU d’agents de nœud disponibles sur leurs références d’image de système d’exploitation en appelant l’opération permettant de [lister les références SKU d’agent de nœud prises en charge](/rest/api/batchservice/list-supported-node-agent-skus).

### <a name="custom-images-for-virtual-machine-pools"></a>Images personnalisées pour les pools de machines virtuelles

Pour savoir comment créer un pool d’images personnalisées, voir [Utiliser Shared Image Gallery pour créer un pool personnalisé](batch-sig-images.md).

Vous pouvez également créer un pool personnalisé de machines virtuelles à l’aide d’une ressource d’[image managée](batch-custom-images.md). Pour plus d’informations sur la préparation d’images Linux personnalisées à partir de machines virtuelles Azure, consultez [Guide pratique pour créer une image d’une machine virtuelle ou d’un VHD](../virtual-machines/linux/capture-image.md). Pour plus d’informations sur la préparation d’images Windows personnalisées à partir de machines virtuelles Azure, consultez la section [Créer une image managée d’une machine virtuelle généralisée dans Azure](../virtual-machines/windows/capture-image-resource.md).

### <a name="container-support-in-virtual-machine-pools"></a>Prise en charge du conteneur dans les pools de machines virtuelles

Lorsque vous créez un pool de configuration de machines virtuelles à l’aide des API de lot, vous pouvez configurer le pool pour exécuter des tâches dans des conteneurs Docker. À l’heure actuelle, vous devez créer le pool à l’aide d’une image qui prend en charge les conteneurs Docker. Utilisez Windows Server 2016 Datacenter avec l’image Containers provenant de la Place de Marché Azure ou fournissez une image de machine virtuelle personnalisée qui inclut Docker Community Edition ou Enterprise Edition et tous les pilotes requis. Les paramètres du pool doivent inclure une [configuration du conteneur](/rest/api/batchservice/pool/add) copiant des images de conteneur sur les machines virtuelles une fois le pool créé. Les tâches qui s’exécutent sur le pool peuvent ensuite référencer les images et les options d’exécution du conteneur.

Pour plus d’informations, consultez [Exécutez des applications de conteneur Docker sur Azure Batch](batch-docker-container-workloads.md).

## <a name="node-type-and-target"></a>Cible et type de nœud

Lorsque vous créez un pool, vous pouvez spécifier les types de nœuds de calcul souhaités et le nombre cible pour chacun. Les deux types de nœuds sont les suivants :

- **Nœuds dédiés.** Les nœuds de calcul dédiés sont réservés à vos charges de travail. Ils sont plus chers que les nœuds à faible priorité, mais sont assurés de ne jamais être reportés.
- **Nœuds à faible priorité.** Les nœuds à faible priorité tirent profit de la capacité excédentaire dans Azure pour exécuter vos charges de travail par lots. Le coût horaire des nœuds à faible priorité est moins élevé que celui des nœuds dédiés. Par ailleurs, ces nœuds activent des charges de travail nécessitant une importante puissance de calcul. Pour plus d’informations, consultez [Utiliser des machines virtuelles à faible priorité avec Batch](batch-low-pri-vms.md).

Les nœuds à faible priorité peuvent être reportés lorsque Microsoft Azure n’a pas suffisamment de capacité excédentaire. Si un nœud est reporté lors de l’exécution de tâches, celles-ci sont remises dans la file d’attente et exécutées à nouveau dès qu’un nœud de calcul redevient disponible. Les nœuds à faible priorité sont intéressants pour les charges de travail pour lesquelles l’heure d’achèvement est flexible et le travail est réparti entre plusieurs nœuds. Avant de décider d’utiliser des nœuds à faible priorité pour votre scénario, vérifiez que le nombre de tâches risquant d’être perdues en raison de la préemption sera peu élevé, et que ces tâches seront faciles à recréer, le cas échéant.

Un même pool peut contenir des nœuds de calcul dédiés et à faible priorité. Chaque type de nœud a son propre paramètre de cible, pour lequel vous pouvez spécifier le nombre de nœuds souhaité.

Le nombre de nœuds de calcul est appelé *cible* car, dans certaines situations, votre pool ne pourra peut-être pas atteindre le nombre de nœuds souhaité. Par exemple, un pool ne pourra peut-être pas atteindre la cible s’il atteint d’abord le [quota de base](batch-quota-limit.md) de votre compte Batch ou si vous avez appliqué au pool une formule de mise à l’échelle automatique qui limite le nombre maximal de nœuds.

Pour obtenir les informations de tarification des nœuds dédiés et à faible priorité, voir [Tarification du service Batch](https://azure.microsoft.com/pricing/details/batch/).

## <a name="node-size"></a>Taille du nœud

Lorsque vous créez un pool Azure Batch, vous avez le choix entre quasiment toutes les tailles et les familles de machine virtuelle disponibles dans Azure. Azure offre une plage de tailles de machines virtuelles pour différentes charges de travail, y compris des tailles de machines virtuelles spécialisées [HPC](../virtual-machines/sizes-hpc.md) ou [compatibles GPU](../virtual-machines/sizes-gpu.md). 

Pour plus d’informations, consultez [Choisir une taille de machine virtuelle pour des nœuds de calcul dans un pool Azure Batch](batch-pool-vm-sizes.md).

## <a name="automatic-scaling-policy"></a>Stratégie de mise à l’échelle automatique

Pour les charges de travail dynamiques, vous pouvez appliquer une stratégie de mise à l’échelle automatique à un pool. Le service Batch évalue régulièrement la pertinence de votre formule et ajuste de manière dynamique le nombre de nœuds d’un pool, en fonction de la charge de travail actuelle et de l’utilisation des ressources de votre scénario de calcul. Cela vous permet de réduire le coût global d’exécution de votre application en utilisant uniquement les ressources dont vous avez besoin et en libérant les autres.

Pour activer la mise à l’échelle automatique, écrivez une [formule de mise à l’échelle automatique](batch-automatic-scaling.md#autoscale-formulas) et associez-la à un pool. Le service Batch utilise la formule pour déterminer le nombre cible de nœuds dans le pool pour le prochain intervalle de mise à l’échelle (intervalle que vous pouvez configurer). Vous pouvez spécifier les paramètres de mise à l’échelle automatique pour un pool lorsque vous le créez ou activez la mise à l’échelle sur un pool ultérieurement. Vous pouvez également mettre à jour les paramètres de mise à l’échelle sur un pool compatible avec la mise à l’échelle.

Par exemple, il se peut qu’un travail exige que vous envoyiez un grand nombre de tâches devant être exécutées. Vous pouvez attribuer au pool une formule de mise à l’échelle qui règle le nombre de nœuds du pool en fonction du nombre actuel de tâches en file d’attente et du degré d’achèvement des tâches dans le travail. Le service Batch évalue régulièrement la formule et redimensionne le pool en fonction de la charge de travail et des autres paramètres de votre formule. Le service ajoute des nœuds selon les besoins lorsqu’il existe un grand nombre de tâches en file d’attente, et supprime des nœuds lorsqu’aucune tâche n’est en file d’attente ni en cours d’exécution.

Une formule de mise à l’échelle peut être basée sur les mesures suivantes :

- **mesures temporelles** sont basées sur les statistiques collectées toutes les cinq minutes dans le nombre d’heures spécifié.
- **mesures de ressources** sont basées sur l’utilisation du processeur, de la bande passante et de la mémoire, et sur le nombre de nœuds.
- Les **mesures de tâches** sont basées sur l’état de la tâche, tel que *Active* (en file d’attente), *En cours d’exécution* ou *Terminée*.

Lorsque la mise à l’échelle automatique diminue le nombre de nœuds de calcul dans un pool, vous devez réfléchir aux méthodes pour gérer les tâches s’exécutant au moment de cette opération de réduction. Pour ce faire, Batch fournit une [*option de désallocation de nœud*](/rest/api/batchservice/pool/removenodes#computenodedeallocationoption) que vous pouvez inclure dans vos formules. Par exemple, vous pouvez spécifier d’arrêter immédiatement les tâches en cours, puis les remettre en file d’attente pour une exécution sur un autre nœud, ou les autoriser à terminer avant la suppression du nœud du pool. Notez que la définition de l’option de désallocation de sur en `taskcompletion` ou `retaineddata` empêche les opérations de redimensionnement de pool jusqu’à ce que toutes les tâches soient terminées ou que toutes les périodes de rétention des tâches aient expiré, respectivement.

Pour plus d’informations sur la mise à l’échelle automatique d’une application, consultez la section [Mettre automatiquement à l’échelle les nœuds de calcul dans un pool Azure Batch](batch-automatic-scaling.md).

> [!TIP]
> Pour optimiser l’utilisation des ressources de calcul, définissez à zéro le nombre cible de nœuds à la fin d’un travail tout en autorisant les tâches en cours à s’achever.

## <a name="task-scheduling-policy"></a>Stratégie de planification des tâches

L’option de configuration [Nombre maximal de tâches par nœud](batch-parallel-node-tasks.md) détermine le nombre maximal de tâches qui peuvent être exécutées en parallèle sur chaque nœud de calcul au sein du pool.

La configuration par défaut prévoit de n’exécuter qu’une seule tâche à la fois sur un nœud. Toutefois, dans certains scénarios, il est préférable d’exécuter plusieurs tâches simultanément sur un nœud. Consultez [l’exemple de scénario](batch-parallel-node-tasks.md#example-scenario) de l’article sur les [tâches de nœud simultanées](batch-parallel-node-tasks.md) pour découvrir comment tirer parti de plusieurs tâches par nœud.

Vous pouvez également spécifier un *type de remplissage*. Il détermine si l’option Batch répartit les tâches uniformément sur tous les nœuds d’un pool, ou remplit chaque nœud avec le nombre maximal de tâches avant d’affecter des tâches à un autre nœud.

## <a name="communication-status"></a>État des communications

Dans la plupart des scénarios, les tâches fonctionnent indépendamment et n’ont pas besoin de communiquer les unes avec les autres. Cependant, il existe des applications dans lesquelles les tâches doivent communiquer, par exemple les [scénarios impliquant des applications MPI](batch-mpi.md).

Vous pouvez configurer un pool de sorte qu’il autorise la **communication entre les nœuds**, afin de permettre aux nœuds du pool de communiquer au moment de l’exécution. Lorsque la communication entre les nœuds est activée, les nœuds des pools Configuration de Cloud Services peuvent communiquer entre eux sur les ports supérieurs à 1100, et les pools Configuration de la machine virtuelle ne limitent pas le trafic sur les ports.

L’activation de la communication entre les nœuds affecte également le placement des nœuds dans des clusters et peut limiter le nombre maximal de nœuds dans un pool en raison des restrictions de déploiement. Si votre application ne nécessite pas de communication entre les nœuds, le service Batch peut éventuellement allouer au pool un grand nombre de nœuds issus de différents centres de données et clusters pour accroître la puissance de traitement parallèle.

## <a name="start-tasks"></a>Tâches de départ

Le cas échéant, vous pouvez ajouter une [tâche de démarrage](jobs-and-tasks.md#start-task), qui s’exécutera sur chacun des nœuds rejoignant le pool, ainsi qu’à chaque redémarrage ou création d’image d’un nœud. La tâche de démarrage est particulièrement utile pour la préparation des nœuds de calcul à l’exécution de tâches, comme l’installation des applications que vos tâches exécutent sur les nœuds de calcul.

## <a name="application-packages"></a>packages d’application

Vous pouvez spécifier des packages d’applications à déployer sur les nœuds de calcul dans le pool. Les packages d’applications permettent un déploiement simplifié et le contrôle de version des applications exécutées par vos tâches. Les packages d’applications que vous spécifiez pour un pool sont installés sur chaque nœud qui rejoint le pool, et à chaque fois qu’un nœud est redémarré ou réinitialisé.

Pour plus d’informations sur l’utilisation de packages d’applications pour déployer vos applications sur vos nœuds Batch, consultez [Déployer des applications sur les nœuds avec des packages d’applications Batch](batch-application-packages.md).

## <a name="virtual-network-vnet-and-firewall-configuration"></a>Configuration du pare-feu et du réseau virtuel (VNet)

Lorsque vous configurez un pool de nœuds de calcul dans Batch, vous pouvez associer le pool au sous-réseau d’un [réseau virtuel (VNet)](../virtual-network/virtual-networks-overview.md) Azure. Pour utiliser un réseau virtuel Azure, l’API du client Batch doit utiliser l’authentification Azure Active Directory (AD). La prise en charge d’Azure Batch pour Azure AD est documentée dans [Authentifier les solutions de service Batch avec Active Directory](batch-aad-auth.md).

### <a name="vnet-requirements"></a>Configuration requise du réseau virtuel

[!INCLUDE [batch-virtual-network-ports](../../includes/batch-virtual-network-ports.md)]

Pour plus d’informations sur la configuration d’un pool Batch dans un réseau virtuel, consultez [Créer un pool de machines virtuelles avec votre réseau virtuel](batch-virtual-network.md).

> [!TIP]
> Pour vous assurer que les adresses IP publiques utilisées pour accéder aux nœuds ne changent pas, vous pouvez [créer un pool avec des adresses IP publiques spécifiées que vous contrôlez](create-pool-public-ip.md).

## <a name="pool-and-compute-node-lifetime"></a>Durée de vie de nœud de pool et de calcul

Lorsque vous créez votre solution Azure Batch, vous devez indiquer de quelle manière et à quel moment les pools doivent être créés, ainsi que la durée pendant laquelle les nœuds de calcul de ces pools restent disponibles.

D’un côté, vous pouvez créer un pool pour chaque travail que vous soumettez et supprimer le pool dès la fin de l’exécution des tâches associées. Cela permet d’optimiser l’utilisation, car les nœuds sont seulement alloués lorsqu’ils sont requis, et sont arrêtés dès qu’ils sont inactifs. Cela signifie que le travail doit attendre que les nœuds soient alloués, mais il est important de noter que l’exécution des tâches est planifiée dès que les nœuds sont individuellement alloués et que la tâche de démarrage est terminée. Batch n’attend *pas* que tous les nœuds d’un pool soient disponibles avant d’affecter des tâches aux nœuds. Cela garantit l’utilisation maximale de tous les nœuds disponibles.

À l’opposé, si la priorité absolue consiste à démarrer immédiatement des travaux, vous pouvez créer un pool avant l’heure et mettre ses nœuds à disposition avant la soumission des travaux. Dans ce scénario, les tâches peuvent démarrer immédiatement, mais il se peut que les nœuds restent inactifs en attendant que les tâches soient affectées.

Une approche combinée est généralement utilisée pour la gestion d’une charge variable, mais continue. Vous pouvez disposer d’un pool dans lequel plusieurs travaux sont envoyés, et mettre à l’échelle le nombre de nœuds en fonction de la charge du travail. Vous pouvez procéder en réaction, en fonction de la charge actuelle, ou en amont, si la charge peut être prédite. Pour plus d’informations, voir [Stratégie de mise à l’échelle automatique](#automatic-scaling-policy).

## <a name="autopools"></a>Pools automatiques

Un [pool automatique](/rest/api/batchservice/job/add#autopoolspecification) est un pool créé par le service Batch lors de l’envoi d’un travail, au lieu d’être créé avant les travaux qui s’exécuteront dans le pool. Le service Batch gère la durée de vie d’un pool automatique en fonction des caractéristiques que vous spécifiez. La plupart du temps, ces pools sont également configurés pour être supprimés automatiquement une fois que leurs travaux sont terminés.

## <a name="security-with-certificates"></a>Sécurité avec certificats

En principe, vous devez utiliser des certificats lorsque vous chiffrez ou déchiffrez des informations sensibles pour les tâches, comme la clé d’un [compte Stockage Azure](accounts.md#azure-storage-accounts). Pour ce faire, vous pouvez installer des certificats sur les nœuds. Les secrets chiffrés sont transmis aux tâches dans les paramètres de ligne de commande ou incorporés dans l’une des ressources et les certificats installés peuvent être utilisés pour les déchiffrer.

Pour ajouter un certificat à un compte Batch, vous utilisez l’opération [Ajouter le certificat](/rest/api/batchservice/certificate/add) (REST Batch) ou la méthode [CertificateOperations.CreateCertificate](/dotnet/api/microsoft.azure.batch.certificateoperations) (.NET Batch). Vous pouvez ensuite associer le certificat à un pool existant ou nouveau.

Lorsqu’un certificat est associé à un pool, le service Batch installe le certificat sur chaque nœud du pool. Le service Batch installe les certificats appropriés au démarrage du nœud, avant de lancer une tâche quelconque (notamment les [tâches de démarrage](jobs-and-tasks.md#start-task) et celles du [gestionnaire de travaux](jobs-and-tasks.md#job-manager-task)).

Si vous ajoutez des certificats à un pool existant, vous devez redémarrer ses nœuds de calcul afin que ces certificats soient appliqués aux nœuds.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [travaux et la tâches](jobs-and-tasks.md).
