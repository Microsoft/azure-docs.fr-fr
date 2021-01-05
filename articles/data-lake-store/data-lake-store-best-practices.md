---
title: Bonnes pratiques d’utilisation d’Azure Data Lake Storage Gen1 | Microsoft Docs
description: Découvrez les bonnes pratiques pour l’ingestion des données, la sécurité des données et les performances liées à l’utilisation d’Azure Data Lake Storage Gen1 (anciennement connu sous le nom d’Azure Data Lake Store).
services: data-lake-store
documentationcenter: ''
author: sachinsbigdata
manager: mtillman
ms.service: data-lake-store
ms.devlang: na
ms.topic: article
ms.date: 06/27/2018
ms.author: sachins
ms.openlocfilehash: 9a5c5f9a4033b70a664071d6077a69f38c905093
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452229"
---
# <a name="best-practices-for-using-azure-data-lake-storage-gen1"></a>Bonnes pratiques d’utilisation d’Azure Data Lake Storage Gen1

[!INCLUDE [data-lake-storage-gen1-rename-note.md](../../includes/data-lake-storage-gen1-rename-note.md)]

Dans cet article, vous allez découvrir les meilleures pratiques et considérations concernant l’utilisation d’Azure Data Lake Storage Gen1. Cet article fournit des informations concernant la sécurité, les performances, la résilience et la surveillance pour Azure Data Lake Storage Gen1. Avant Azure Data Lake Storage Gen1, il était difficile de travailler avec de grandes quantités de données dans des services tels qu’Azure HDInsight. Il fallait partitionner les données sur plusieurs comptes de stockage d’objets blob afin de rendre possible, à cette échelle, le stockage pétaoctet et des performances optimales. Avec Azure Data Lake Storage Gen1, la plupart des limites inconditionnelles de taille et de performances ont été supprimées. Toutefois, il faut toujours tenir compte de certaines choses. Cet article vous en parle afin que vous profitiez des meilleures performances avec Azure Data Lake Storage Gen1.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Azure Data Lake Storage Gen1 offre des contrôles d’accès POSIX et une fonction d’audit détaillée pour les utilisateurs, les groupes et les principaux de service Azure Active Directory (Azure AD). Ces contrôles d’accès peuvent être définis pour des fichiers et dossiers existants. Ils peuvent aussi être utilisés pour créer des valeurs par défauts pouvant être appliquées à des nouveaux fichiers ou dossiers. Lorsque les autorisations sont définies pour des objets enfant et des dossiers existants, elles doivent être propagées de façon récursive sur chaque objet. Si le nombre de fichiers est important, la propagation des autorisations peut prendre du temps. Le temps nécessaire est de 30 à 50 objets traités par seconde. Prévoyez donc la structure du dossier et les groupes d’utilisateurs en conséquence. Sinon, vous pourriez rencontrer des délais et problèmes imprévus lorsque vous utilisez vos données.

Supposons que vous disposez d’un dossier qui contient 100 000 objets enfant. En admettant que la vitesse de propagation soit minimale, donc de 30 objets par seconde, la mise à jour de l’autorisation pour l’ensemble du dossier prendrait une heure. Pour plus d’informations sur la liste des contrôles d’accès de Azure Data Lake Storage Gen1, voir [Contrôle d’accès dans Azure Data Lake Storage Gen1](data-lake-store-access-control.md). Pour améliorer les performances pour assigner des listes de contrôles d’accès de façon récursive, vous pouvez utiliser l’outil en ligne de commande Azure Data Lake. Cet outil crée plusieurs threads et une logique de navigation récursive pour appliquer rapidement des listes de contrôles d’accès à des millions de fichiers. L’outil est disponible pour Linux et Windows, et toute la [documentation](https://github.com/Azure/data-lake-adlstool) et les [téléchargements](https://aka.ms/adlstool-download) pour cet outil se trouvent sur GitHub. Vous pouvez bénéficier des mêmes améliorations de performances grâce à vos propres outils écrits à l’aide des Kits de développement logiciel (SDK) Azure Data Lake Storage Gen1 [.NET](data-lake-store-data-operations-net-sdk.md) et [Java](data-lake-store-get-started-java-sdk.md).

### <a name="use-security-groups-versus-individual-users"></a>Utilisation de groupes de sécurité et utilisation d’utilisateurs individuels

Lorsque vous utilisez une quantité importante de données dans Data Lake Storage Gen1, il est fort probable qu’un principal de service est utilisé pour permettre à des services tels qu’Azure HDInsight d’utiliser ces données. Toutefois, dans certains cas, il se peut que les utilisateurs individuels aient aussi besoin d’accéder aux données. Dans ces cas-là, vous devez utiliser les [groupes de sécurité](data-lake-store-secure-data.md#create-security-groups-in-azure-active-directory) Azure Active Directory au lieu d’assigner des utilisateurs individuels à des dossiers et des fichiers.

Une fois que vous avez assigné les autorisations à un groupe de sécurité, ajouter ou supprimer des utilisateurs de ce groupe ne nécessite aucune mise à jour de Data Lake Storage Gen1. Ceci contribue également à garantir que vous ne dépassez pas la limite de [32 ACL d’accès et par défaut](../azure-resource-manager/management/azure-subscription-service-limits.md#data-lake-storage-limits) (ceci inclut les quatre ACL de type POSIX qui sont toujours associées à chaque fichier et dossier : [l’utilisateur propriétaire](data-lake-store-access-control.md#the-owning-user), [le groupe propriétaire](data-lake-store-access-control.md#the-owning-group), [le masque](data-lake-store-access-control.md#the-mask) et autres).

### <a name="security-for-groups"></a>Sécurité liée aux groupes

Comme mentionné, lorsque les utilisateurs ont besoin d’accéder à Data Lake Storage Gen1, il est préférable d’utiliser des groupes de sécurité Azure Active Directory. Parmi les groupes recommandés pour démarrer, on trouve **ReadOnlyUsers**, **WriteAccessUsers** et **FullAccessUsers** pour la racine du compte, et même d’autres pour des sous-dossiers clé. S’il y a d’autres groupes ou utilisateurs attendus qui pourraient être ajoutés plus tard, mais non identifiés à l’heure actuelle, vous devriez réfléchir à créer des groupes de sécurité test qui peuvent accéder à certains dossiers. L’utilisation d’un groupe de sécurité vous assure de ne pas avoir perdre trop de temps plus tard lors de l’assignation de nouvelles autorisations à des milliers de fichiers.

### <a name="security-for-service-principals"></a>Sécurité liée aux principaux de service

Les principaux de service Azure Active Directory sont en général utilisés par des services tels qu’Azure HDInsight pour accéder aux données dans Data Lake Storage Gen1. En fonction des conditions d’accès sur des charges de travail multiples, il pourrait être recommandé d’assurer la sécurité à l’intérieur et à l’extérieur de l’organisation. Pour de nombreux clients, un seul principal de service Azure Active Directory peut être adéquat, et ce dernier peut disposer de toutes les autorisations à la racine de Data Lake Storage Gen1. D’autres clients peuvent avoir besoin de plusieurs clusters avec différents principaux de service, où un cluster dispose de tous les accès aux données, et un autre de l’accès en lecture. Comme pour les groupes de sécurité, vous devriez alors réfléchir à faire un principal de service par scénario attendu (lecture, écriture, complet) quand un compte Data Lake Storage Gen1 est créé.

### <a name="enable-the-data-lake-storage-gen1-firewall-with-azure-service-access"></a>Activer le pare-feu Data Lake Storage Gen1 avec accès au service Azure

Data Lake Storage Gen1 prend en charge l’option d’activation d’un pare-feu et de limitation d’accès aux services Azure seulement, ce qui est recommandé pour un vecteur d’attaque réduit en cas d’intrusion extérieure. Le pare-feu peut être activé sur le compte Data Lake Storage Gen1 dans le portail Azure via les options **Pare-feu** > **Activer le pare-feu (ON)**  > **Autoriser l’accès aux services Azure**.

![Paramètres du pare-feu dans Data Lake Storage Gen1](./media/data-lake-store-best-practices/data-lake-store-firewall-setting.png "Paramètres du pare-feu dans Data Lake Storage Gen1")

Une fois le pare-feu activé, seuls les services Azure tels que HDInsight, Data Factory, Azure Synapse Analytics, etc., pourront accéder à Data Lake Storage Gen1. En raison de la traduction d’adresses réseau interne utilisée par Azure, le pare-feu Data Lake Storage Gen1 ne prend pas en charge la restriction de services spécifiques par adresse IP, et n’est prévu que pour des restrictions de points de terminaison hors d’Azure, comme les points de terminaison locaux.

## <a name="performance-and-scale-considerations"></a>Considérations sur les performances et l’échelle

L’une des fonctionnalités les plus puissantes de Data Lake Storage Gen1 est sa capacité à supprimer les limites inconditionnelles de sortie des données. La suppression des limites permet aux clients d’augmenter la quantité de données et des conditions de performances sans avoir à partitionner les données. L’optimisation des performances de Data Lake Storage Gen1 s’exécute le mieux lorsqu’elle dispose de parallélisme. Il s’agit de l’une des considérations les plus importantes.

### <a name="improve-throughput-with-parallelism"></a>Améliorer le débit avec le parallélisme

Donnez entre 8 et 12 threads par cœur pour obtenir un débit de lecture/écriture optimal. Ceci bloquera les lectures/écritures sur un seul thread, et plus de threads permettent une concurrence supérieure sur la machine virtuelle. Pour être sûr que les niveaux sont intègres et que le parallélisme peut être augmenté, veillez à surveiller l’utilisation du processeur de la machine virtuelle.

### <a name="avoid-small-file-sizes"></a>Éviter les fichiers de petite taille

Les services d’autorisations POSIX et d’audit dans Data Lake Storage Gen1 apportent un traitement qui devient apparent lorsque vous travaillez avec de nombreux fichiers de petite taille. Comme meilleure pratique, vous devez traiter vos données par lot dans des fichiers de taille plus importante et écrire des milliers voire des millions de fichiers de petite taille dans Data Lake Storage Gen1. Éviter des fichiers de petite taille peut apporter les avantages suivants :

* Réduction des vérifications d’authentification sur plusieurs fichiers
* Réduction du nombre de connexions à l’ouverture de fichiers
* Réplication/copie plus rapide
* Nombre de fichiers à traiter réduit lors de la mise à jour des autorisations POSIX Data Lake Storage Gen1

En fonction des services et des charges de travail qui utilisent les données, 256 Mo ou plus est une taille envisageable pour les fichiers. Si les tailles de fichiers ne peuvent être traitées par lot à leur arrivée dans Data Lake Storage Gen1, vous pouvez lancer une tâche de compactage pour combiner ces fichiers en des fichiers plus importants. Pour plus d’informations et de suggestions sur les tailles de fichiers et l’organisation des données dans Data Lake Storage Gen1, consultez [Structurer votre jeu de données](data-lake-store-performance-tuning-guidance.md#structure-your-data-set).

### <a name="large-file-sizes-and-potential-performance-impact"></a>Tailles de fichiers importantes et impact potentiel sur les performances

Bien que Data Lake Storage Gen1 prenne en charge des fichiers volumineux jusqu’à plusieurs pétaoctets, il n’est pas forcément idéal de dépasser 2 Go de moyenne pour des performances optimales, en fonction du traitement de lecture des données. Par exemple, lorsque vous utilisez **Distcp** pour copier des données entre des emplacements ou des comptes de stockage différent, les fichiers représentent le meilleur niveau de granularité utilisée pour déterminer les tâches de mappage. Ainsi, si vous copiez 10 fichiers d’1 To chacun, 10 mappeurs maximum sont alloués. Aussi, si vous disposez de nombreux fichiers avec des mappeurs assignés, ces derniers fonctionnent initialement parallèlement pour déplacer des fichiers volumineux. Toutefois, la tâche de ralentissement démarre et ne restent alors assignés que quelques mappeurs. Vous vous retrouvez alors coincé avec un seul mappeur assigné à un fichier volumineux. Microsoft a proposé des améliorations à Distcp pour régler ce problème dans les versions futures d’Hadoop.

Un autre exemple à prendre en compte est lors de l’utilisation d’Azure Data Lake Analytics avec Data Lake Storage Gen1. En fonction du traitement effectué par l’extracteur, les performances de certains fichiers qui ne peuvent être fractionnés (par exemple, XML, JSON) peuvent être affectées, lorsque ces fichiers font plus de 2 Go. Dans les cas où les fichiers peuvent être fractionnés par un extracteur (par exemple, CSV), des fichiers volumineux sont conseillés.

### <a name="capacity-plan-for-your-workload"></a>Plan de capacité pour votre charge de travail

Azure Data Lake Storage Gen1 supprime les limites E/S inconditionnelles placées sur les comptes de stockage d’objets blob. Toutefois, des limites conditionnelles demeurent et doivent être prises en compte. Les limites Entrée/Sortie par défaut correspondent aux besoins de la plupart des scénarios. S’il faut augmenter les limites de votre charge de travail, contactez le support Microsoft. De plus, regardez les limites à l’étape de démonstration afin que les limites E/S ne sont pas atteintes lors de la production. Si cela se produit, il faudra peut-être attendre une augmentation manuelle de la part de l’équipe d’ingénieurs Microsoft. Si la limite E/S est atteinte, Azure Data Lake Storage Gen1 retourne un code d’erreur 429, et un nouvel essai doit être idéalement fait avec une stratégie d’interruption exponentielle adaptée.

### <a name="optimize-writes-with-the-data-lake-storage-gen1-driver-buffer"></a>Optimiser « Écrit » avec le tampon du pilote Data Lake Storage Gen1

Pour optimiser les performances et diminuer les opérations d’E/S par seconde lors de l’écriture d’Hadoop dans Data Lake Storage Gen1, effectuez des opérations d’écriture aussi proches de la taille de la mémoire tampon du pilote Data Lake Storage Gen1. Essayez de ne pas dépasser la taille de la mémoire tampon avant de la vider, comme lors de la diffusion en continu avec Apache Storm ou la diffusion en continu de charges de travail avec Spark. Lorsque vous écrivez d’Hadoop/HDInsight dans Data Lake Storage Gen1, il est important de savoir que Data Lake Storage Gen1 possède un pilote avec une mémoire tampon de 4 Mo. Comme pour beaucoup de pilotes de système de fichiers, cette mémoire tampon peut être vidée manuellement avant d’atteindre 4 Mo. Dans le cas contraire, elle est immédiatement vidée dans le stockage si la prochaine écriture dépasse la taille maximum de la mémoire tampon. Lorsque c’est possible, vous devez éviter tout débordement ou sous-exécution importante de la mémoire tampon lors de la synchronisation ou effacement de stratégie par compte ou fenêtre de temps.

## <a name="resiliency-considerations"></a>Remarques relatives à la résilience

Lors de la création d’Architecture d’un système avec Data Lake Storage Gen1 ou n’importe quel service cloud, vous devez prendre en compte les prérequis de disponibilité et la façon de répondre à des interruptions potentielles du service. Un problème peut être localisé dans une instance spécifique ou même dans la région. Être préparé pour ces deux éventualités est important. Selon les SLA d’**objectif de temps de récupération** et d’**objectif de point de récupération** de votre charge de travail, vous choisirez une stratégie plus ou moins agressive pour la haute disponibilité et la récupération d’urgence.

### <a name="high-availability-and-disaster-recovery"></a>Haute disponibilité et récupération d’urgence

La haute disponibilité et la récupération d’urgence peuvent parfois être combinées, bien qu’elles aient chacune une stratégie légèrement différente, surtout lorsqu’il s’agit de données. Data Lake Storage Gen1 gère déjà 3 réplications sous le capot, pour se protéger contre des défaillances matérielles localisées. Toutefois, étant donné que la réplication entre régions n’est pas intégrée, vous devez la faire vous-même. Lorsque vous élaborez un plan pour la haute disponibilité, en cas d’interruption du service, la charge de travail doit accéder aux dernières données aussi vite que possible en passant à une instance répliquée séparément en local ou dans une nouvelle région.

Dans une stratégie de récupération d’urgence, pour se préparer à un évènement (improbable) d’une défaillance catastrophique d’une région, il est aussi important de disposer de données répliquées dans une région différente. Ces données peuvent être les mêmes que les données répliquées de haute disponibilité. Toutefois, vous devez aussi tenir compte des prérequis pour des cas extrêmes comme la corruption des données, où vous devrez créer des instantanés périodiques sur lesquels vous replier. En fonction de l’importance et de la taille des données, pensez à propager des instantanés delta à durée d’1-, 6- et 24 heures sur le marché local et/ou secondaire, selon les tolérances au risque.

En ce qui concernant la résilience des données avec Data Lake Storage Gen1, il est recommandé de géorépliquer vos données dans une région différente avec une fréquence qui répond aux prérequis de haute disponibilité et de récupération d’urgence, idéalement toutes les heures. Cette fréquence de réplication minimise les mouvements de données massifs qui peuvent avoir des besoins en débits en concurrence avec le système principal ainsi qu’un meilleur objectif de point de récupération. En outre, vous devez prendre en compte des méthodes pour que l’application utilisant Data Lake Storage Gen1 puisse automatiquement basculer sur le compte secondaire en surveillant des déclencheurs ou des durées de tentatives échouées, ou en envoyant une notification aux administrateurs pour une intervention manuelle. N’oubliez pas qu’il y a un compromis entre basculer et attendre qu’un service soit de nouveau en ligne. Si les données n’ont pas terminé la réplication, un basculement peut entraîner une perte des données, une inconsistance ou une fusion complexe des données.

Vous trouverez ci-dessous les trois options les plus recommandées pour orchestrer une réplication entre des comptes Data Lake Storage Gen1, et les différences clé entre chacune d’elles.

|  |Distcp  |Azure Data Factory  |AdlCopy  |
|---------|---------|---------|---------|
|**Limites de mise à l’échelle**     | Délimitée par des nœuds de travail        | Limitée par des unités de déplacement de données cloud max        | Délimitée par des unités Analytics        |
|**Prend en charge la copie des valeurs delta**     |   Oui      | Non         | Non         |
|**Orchestration intégrée**     |  NON (utiliser Oozie Airflow ou les tâches Cron)       | Oui        | NON (utiliser Azure Automation ou le Planificateur de tâches Windows)         |
|**Systèmes de fichiers pris en charge**     | ADL, HDFS, WASB, S3, GS, CFS        |Nombreux, voir [Connecteurs](../data-factory/connector-azure-blob-storage.md).         | ADL vers ADL, WASB vers ADL (même région uniquement)        |
|**Système d’exploitation pris en charge**     |N’importe quel système d’exploitation prenant en charge Hadoop         | N/A          | Windows 10         |

### <a name="use-distcp-for-data-movement-between-two-locations"></a>Utiliser Distcp pour déplacer des données entre deux emplacements

Diminutif de Distributed Copy, Distcp est un outil en ligne de commande Linux qui vient avec Hadoop et qui permet le déplacement de données distribuées entre deux emplacements. Les deux emplacements peuvent être Data Lake Storage Gen1, HDFS, WASB ou S3. Cet outil utilise des tâches MapReduce sur un cluster Hadoop (par exemple, HDInsight) pour effectuer un scale-out sur tous les nœuds. Distcp est considéré comme étant la façon la plus rapide de déplacer des données volumineuses sans appliances de compression de réseau spéciales. Cet outil fournit aussi une option pour ne mettre à jour que les valeurs delta entre deux emplacements, gère les essais automatiques et la mise à l’échelle dynamique de calcul. Cette approche est incroyablement efficace quand il est question de répliquer des choses telles que des tableaux Hive/Spark qui peuvent disposer de plusieurs fichiers volumineux dans un seul répertoire, et que vous ne voulez copier que les données modifiées. Pour ces raisons, Distcp est l’outil le plus recommandé pour la copie de données entre des magasins Big Data.

Les tâches de copie peuvent être déclenchées par des workflows Apache Oozie à l’ide de déclencheurs de fréquence ou de données, ainsi que des tâches Cron Linux. Pour les tâches de réplication intensives, il est recommandé d’ajouter un cluster Hadoop HDInsight qui peut être réglé et mis à l’échelle spécialement pour les tâches de copie. Ainsi, les tâches de copie n’interfèrent pas avec les tâches critiques. Si vous exécutez des réplications à une fréquence assez importante, le cluster peut même être arrêté entre chaque tâche. Si vous basculez vers une région secondaire, veillez à ce qu’un autre cluster soit aussi ajouté à cette région pour répliquer de nouvelles données vers le compte principal Data Lake Storage Gen1 lorsqu’il sera de nouveau en marche. Pour des exemples d’utilisation de Distcp, consultez [Utiliser Distcp pour copier des données entre des objets blob de Stockage Azure et Data Lake Storage Gen1](data-lake-store-copy-data-wasb-distcp.md).

### <a name="use-azure-data-factory-to-schedule-copy-jobs"></a>Utiliser Azure Data Factory pour planifier des tâches de copie

Azure Data Factory peut aussi être utilisé pour planifier des tâches de copie à l’aide d’une **Activité de copie**, et peut même être configuré sur une fréquence via l’**Assistant de copie**. N’oubliez pas qu’Azure Data Factory dispose d’une limite d’unités de déplacement de données cloud (DMU), et finit par atteindre la limite de débit/calcul pour des charges de travail de données volumineuses. En outre, Azure Data Factory n’offre actuellement pas de mises à jour des valeurs delta entre les comptes Data Lake Storage Gen1. Les dossiers comme les tableaux Hive nécessitent donc une copie complète pour être répliqués. Référez-vous au [guide de réglage de l’activité de copie](../data-factory/copy-activity-performance.md) pour en savoir plus sur la copie avec Data Factory.

### <a name="adlcopy"></a>AdlCopy

AdlCopy est un outil en ligne de commande Windows qui vous permet de copier des données entre deux comptes Data Lake Storage Gen1, dans une même région uniquement. Cet outil fournit une option autonome ou l’option d’utiliser un compte Azure Data Lake Analytics pour exécuter votre tâche de copie. Bien qu’il ait été créé à la base pour des copies à la demande et non pas pour une réplication robuste, il fournit une autre option pour copier des données distribuées entre des comptes Data Lake Storage Gen1 d’une même région. Pour des questions de fiabilité, il est recommandé d’utiliser l’option premium Data Lake Analytics pour des charges de travail de production. La version autonome peut renvoyer des réponses occupées et dispose d’une échelle et d’une surveillance limitées.

Comme Distcp, l’outil AdlCopy doit être orchestré par quelque chose comme Azure Automation ou le planificateur de tâche Windows. Comme avec Data Factory, AdlCopy ne prend pas en charge la copie des fichiers mis à jour uniquement, mais peut recopier et remplacer des fichiers existants. Pour plus d’informations et d’exemples sur l’utilisation d’AdlCopy, consultez [Copier des données de Stockage Blob Azure vers Data Lake Storage Gen1](data-lake-store-copy-data-azure-storage-blob.md).

## <a name="monitoring-considerations"></a>Surveillance - Éléments à prendre en compte

Data Lake Storage Gen1 fournit une fonction d’audit et des journaux de diagnostic détaillés. Il fournit certaines mesures de base dans le portail Azure dans le compte Data Lake Storage Gen1 et dans Azure Monitor. La disponibilité de Data Lake Storage Gen1 est affichée dans le portail Azure. Toutefois, cette mesure est actualisée toutes les sept minutes et ne peut être demandée via une API publique. Pour obtenir la disponibilité la plus à jour d’un compte Data Lake Storage Gen1, vous devez exécuter vos propres tests synthétiques pour valider la disponibilité. D’autres mesures telles que l’utilisation totale du stockage, les requêtes de lecture/écriture et d’entrée/sortie peuvent mettre jusqu’à 24 heures pour s’actualiser. Des mesures plus à jour doivent donc être calculées manuellement via les outils en ligne de commande Hadoop ou une agrégation de journaux d’information. La méthode la plus rapide pour obtenir l’utilisation de stockage la plus récente consiste à exécuter cette commande HDFS depuis un nœud de cluster Hadoop (par exemple, nœud principal) :

```console
hdfs dfs -du -s -h adl://<adlsg1_account_name>.azuredatalakestore.net:443/
```

### <a name="export-data-lake-storage-gen1-diagnostics"></a>Exporter des diagnostics Data Lake Storage Gen1

L’une des méthodes les plus rapides pour accéder aux journaux d’activité consultables à partir de Data Lake Storage Gen1 consiste à activer la copie des journaux d’activité de transaction vers **Log Analytics** sous le panneau **Diagnostics** du compte Data Lake Storage Gen1. Cela fournit une accès immédiat aux journaux d’activité entrants avec des filtres de temps et de contenus, ainsi que des options d’alerte (message électronique/webhook) qui se déclenchent toutes les 15 minutes. Pour plus d’instructions, consultez [Accès aux journaux de diagnostic d’Azure Data Lake Storage Gen1](data-lake-store-diagnostic-logs.md).

Pour plus d’alertes en temps réel et plus de contrôles sur le lieu d’arrivée des journaux d’activité, pensez à exporter les journaux d’activité vers Azure EventHub, où le contenu peut être analysé individuellement ou sur une fenêtre de temps pour soumettre des notifications en temps réel à une file d’attente. Une application différente telle qu’une [application logique](../connectors/connectors-create-api-azure-event-hubs.md) peut alors utiliser et communiquer les alertes au canal approprié, et soumettre des mesures pour surveiller des outils comme NewRelic, Datadog ou AppDynamics. Aussi, si vous utilisez un outil tiers comme ElasticSearch, vous pouvez exporter les journaux d’activité vers le stockage d’objets blob et utiliser le [plugin Azure Logstash](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob) pour utiliser les données dans votre pile ElasticSearch, Kibana et Logstash (ELK).

### <a name="turn-on-debug-level-logging-in-hdinsight"></a>Activer la journalisation de niveau de débogage dans HDInsight

Si l’envoi de journaux Azure Data Lake Storage Gen1 n’est pas activé, Azure HDInsight fournit aussi une manière d’activer la [journalisation côté client pour Data Lake Storage Gen1](data-lake-store-performance-tuning-mapreduce.md) via loh4j. Vous devez définir la propriété suivante dans **Ambari** > **YARN** > **Configurations** > **Configurations avancées yarn-log4j** :

`log4j.logger.com.microsoft.azure.datalake.store=DEBUG`

Une fois la propriété définie et les nœuds redémarrés, les diagnostics Data Lake Storage Gen1 sont écrits dans les journaux d’activité YARN sur les nœuds (/tmp/\<user\>/yarn.log) et les informations importantes comme les erreurs ou les limites (code d’erreur HTTP 429) peuvent être analysées. Ces mêmes informations peuvent aussi être supervisées dans les journaux Azure Monitor ou n’importe quel emplacement où sont envoyés les journaux dans le panneau [Diagnostics](data-lake-store-diagnostic-logs.md) du compte Data Lake Storage Gen1. Il est recommandé de disposer d’au moins un journal côté client activé ou d’utiliser l’option d’envoi de journaux avec Data Lake Storage Gen1 pour une visibilité opérationnelle et un débogage simplifié.

### <a name="run-synthetic-transactions"></a>Exécuter des transactions synthétiques

Actuellement, la mesure de disponibilité de service pour Data Lake Storage Gen1 dans le portail Azure dispose d’une fenêtre d’actualisation de 7 minutes. En outre, elle ne peut être interrogée avec une API exposée publiquement. Ainsi, il est recommandé de créer une application de base pour réaliser des transactions synthétiques dans Data Lake Storage Gen1 qui peuvent fournir une disponibilité à la minute. Comme exemple, vous pouvez créer une application WebJob, Logic App ou Azure Function App pour réaliser une action de lecture, création et mise à jour sur Data Lake Storage Gen1 et envoyer les résultats à votre solution de supervision. Les opérations peuvent être réalisées dans un dossier temporaire puis supprimées après le test, qui peut être exécuté toutes les 30 à 60 secondes, selon les prérequis.

## <a name="directory-layout-considerations"></a>Considérations relatives à la disposition du répertoire

Lorsque des données arrivent dans Data Lake Store, il est important d’avoir planifié la structure des données afin de pouvoir utiliser avec efficacité la sécurité, le partitionnement et le traitement. La plupart des suggestions suivantes peuvent être utilisées avec Azure Data Lake Storage Gen1, le stockage Blob ou HDFS. Chaque charge de travail dispose de prérequis différents concernant l’utilisation des données, mais vous trouverez ci-dessous des dispositions communes à prendre lorsque vous utilisez des scénarios IoT ou de traitement.

### <a name="iot-structure"></a>Structure IoT

Dans des charges de travail IoT peuvent se trouver un grand nombre de données arrivées dans le magasin de données, couvrant de nombreux produits, appareils, organisations et clients. Il est important de planifier à l’avance la disposition du répertoire pour des questions d’organisation, de sécurité et de traitement efficace des données pour des consommateurs de flux. Voici un modèle général de disposition à prendre en compte :

```console
{Region}/{SubjectMatter(s)}/{yyyy}/{mm}/{dd}/{hh}/
```

Par exemple, la télémétrie d’arrivée d’un moteur d’avion dans le Royaume-Uni ressemble à la structure suivante :

```console
UK/Planes/BA1293/Engine1/2017/08/11/12/
```

Il est important de placer la date à la fin de la structure du dossier. Si vous voulez verrouiller certaines régions ou certains thèmes à des utilisateurs/groupes, vous pouvez le faire facilement avec les autorisations POSIX. Sinon, s’il faut restreindre l’affichage des données du Royaume-Uni ou de certains avions à certains groupes de sécurité, avec la structure des données en face, une autorisation différente est nécessaire pour de nombreux dossiers sous chaque dossier heure. En outre, placer la structure de date en face augmente grandement le nombre de dossiers à mesure que le temps passe.

### <a name="batch-jobs-structure"></a>Structure de tâches de traitement par lots

Depuis un niveau supérieur, une approche communément usitée dans le traitement par lots est d’envoyer des données dans un dossier « in ». Ensuite, une fois que les données sont traitées, il faut placer les données obtenues dans un dossier « out » pour qu’elles soient utilisées par des processus en aval. Cette structure de répertoire est parfois visible avec des travaux qui nécessitent un traitement sur des fichiers individuels et qui ne nécessitent pas forcément de traitement parallèle massif sur des jeux de données volumineux. Tout comme la structure IoT recommandée plus haut, une bonne structure de répertoire dispose de dossiers de niveau parent pour des choses telles que la région et les thèmes (par exemple, organisation, produit/producteur). Cette structure facilite la sécurisation des données dans votre organisation et une meilleure gestion des données dans vos charges de travail. En outre, tenez compte de la date et l’heure dans la structure pour permettre une meilleure organisation, de meilleures recherches filtrées, une sécurité plus efficace et une automatisation du traitement. Le niveau granularité de la structure de date est déterminé par l’intervalle de chargement ou de traitement des données, par exemple à l’heure, quotidien, ou mensuel.

Parfois le traitement de fichiers échoue en raison de corruption des données ou de formats inattendus. Dans ces cas, la structure du répertoire peut profiter d’un dossier **/bad** où déplacer les fichiers pour une inspection plus en détail. La tâche de traitement par lots peut aussi gérer le rapport ou la notification de ces fichiers *bad* en vue d’une intervention manuelle. Considérez la structure de modèle suivante :

```console
{Region}/{SubjectMatter(s)}/In/{yyyy}/{mm}/{dd}/{hh}/
{Region}/{SubjectMatter(s)}/Out/{yyyy}/{mm}/{dd}/{hh}/
{Region}/{SubjectMatter(s)}/Bad/{yyyy}/{mm}/{dd}/{hh}/
```

Par exemple, une firme marketing qui reçoit quotidiennement des extraits de données de mises à jour client de la part de leurs clients en Amérique du Nord. Avant et après avoir été traité, il peut ressembler à l’extrait de code suivant :

```console
NA/Extracts/ACMEPaperCo/In/2017/08/14/updates_08142017.csv
NA/Extracts/ACMEPaperCo/Out/2017/08/14/processed_updates_08142017.csv
```

Dans le cas courant de traitement des données par lots dans des bases de données directement, telles que Hive ou des bases de données SQL traditionnelles, il n’y a pas besoin de dossiers **/in** ou **/out** car la sortie se trouve déjà dans un dossier différent pour le tableau Hive ou la base de données externe. Par exemple, des extraits quotidiens de la part de clients arrivent dans leurs dossiers respectifs, et l’orchestration par des outils comme Azure Data Factory, Apache Oozie ou Apache Airflow déclenche une tâche quotidienne Hive ou Spark pour traiter et écrire les données dans un tableau Hive.

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Contrôle d’accès dans Azure Data Lake Storage Gen1](data-lake-store-access-control.md)
* [Sécurité dans Azure Data Lake Storage Gen1](data-lake-store-security-overview.md)
* [Réglage d’Azure Data Lake Storage Gen1 pour les performances](data-lake-store-performance-tuning-guidance.md)
* [Recommandations en matière d’optimisation des performances pour l’utilisation de HDInsight Spark avec Azure Data Lake Storage Gen1](data-lake-store-performance-tuning-spark.md)
* [Recommandations en matière d’optimisation des performances pour l’utilisation de HDInsight Hive avec Azure Data Lake Storage Gen1](data-lake-store-performance-tuning-hive.md)
* [Créer des clusters HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
