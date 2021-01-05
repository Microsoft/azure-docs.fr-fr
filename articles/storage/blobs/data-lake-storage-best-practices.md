---
title: Bonnes pratiques d’utilisation d’Azure Data Lake Storage Gen2 | Microsoft Docs
description: Découvrez les bonnes pratiques pour l’ingestion des données, la sécurité des données et les performances liées à l’utilisation d’Azure Data Lake Storage Gen2 (anciennement Azure Data Lake Store).
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 12/06/2018
ms.author: normesta
ms.reviewer: sachins
ms.openlocfilehash: 0efcdfd1b14479edf84dc1892e7e1d9afabd5a81
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913553"
---
# <a name="best-practices-for-using-azure-data-lake-storage-gen2"></a>Bonnes pratiques d’utilisation d’Azure Data Lake Storage Gen2

Dans cet article, vous allez découvrir les bonnes pratiques et considérations concernant l’utilisation d’Azure Data Lake Storage Gen2. Cet article fournit des informations concernant la sécurité, les performances, la résilience et la supervision pour Azure Data Lake Storage Gen2. Avant Azure Data Lake Storage Gen2, il était difficile de travailler avec de grandes quantités de données dans des services comme Azure HDInsight. Il fallait partitionner les données sur plusieurs comptes de stockage d’objets blob afin de rendre possible, à cette échelle, le stockage pétaoctet et des performances optimales. Data Lake Storage Gen2 prend en charge des tailles de fichiers individuelles de 5 To maximum et la plupart des limites matérielles des performances ont été supprimées. Toutefois, il reste des points à prendre en compte que cet article aborde pour vous permettre d’obtenir les meilleures performances avec Azure Data Lake Storage Gen2.

## <a name="security-considerations"></a>Considérations relatives à la sécurité

Azure Data Lake Storage Gen2 offre des contrôles d’accès POSIX pour les utilisateurs, les groupes et les principaux de service Azure Active Directory (Azure AD). Ces contrôles d’accès peuvent être définis pour des fichiers et des répertoires existants. Ils peuvent aussi être utilisés pour créer des autorisation par défauts pouvant être appliquées automatiquement à de nouveaux fichiers ou répertoires. Pour plus d’informations sur les listes de contrôle d’accès (ACL) Azure Data Lake Storage Gen2, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](./data-lake-storage-access-control.md).

### <a name="use-security-groups-versus-individual-users"></a>Utilisation de groupes de sécurité et utilisation d’utilisateurs individuels

Quand vous utilisez une quantité importante de données dans Data Lake Storage Gen2, un principal de service est probablement utilisé pour permettre à des services comme Azure HDInsight d’utiliser ces données. Toutefois, dans certains cas, il se peut que les utilisateurs individuels aient aussi besoin d’accéder aux données. Dans tous les cas, nous vous conseillons vivement d’utiliser des [groupes de sécurité](../common/storage-auth-aad.md) Azure Active Directory au lieu d’assigner des utilisateurs individuels à des répertoires et des fichiers.

Une fois que vous avez assigné les autorisations à un groupe de sécurité, ajouter ou supprimer des utilisateurs de ce groupe ne nécessite aucune mise à jour de Data Lake Storage Gen2. Cela vous aide également à ne pas dépasser le nombre maximal d’entrées de contrôle d’accès par liste ACL. Actuellement, cette limite est de 32 (y compris les quatre listes ACL de style POSIX qui sont toujours associées à chaque fichier et répertoire : l’utilisateur propriétaire, le groupe propriétaire, le masque et autres). Chaque répertoire peut comporter deux types de listes ACL, la liste ACL d’accès et la liste ACL par défaut, pour un total de 64 entrées de contrôle d’accès. Pour plus d’informations sur ces listes ACL, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

### <a name="security-for-groups"></a>Sécurité liée aux groupes

Quand vos utilisateurs ou vous-même avez besoin d’accéder aux données dans un compte de stockage avec espace de noms hiérarchique activé, il est préférable d’utiliser des groupes de sécurité Azure Active Directory. Parmi les groupes recommandés pour démarrer, citons **ReadOnlyUsers**, **WriteAccessUsers** et **FullAccessUsers** pour la racine du conteneur, et même d’autres pour des sous-répertoires clés. S’il y a d’autres groupes ou utilisateurs attendus qui pourraient être ajoutés plus tard, mais non identifiés à l’heure actuelle, vous devriez réfléchir à créer des groupes de sécurité test qui peuvent accéder à certains dossiers. L’utilisation d’un groupe de sécurité vous évite de perdre du temps plus tard lors de l’assignation de nouvelles autorisations à des milliers de fichiers.

### <a name="security-for-service-principals"></a>Sécurité liée aux principaux de service

Les principaux de service Azure Active Directory sont en général utilisés par des services tels qu’Azure Databricks pour accéder aux données dans Data Lake Storage Gen2. Pour de nombreux clients, un seul principal de service Azure Active Directory peut suffire, et ce dernier peut disposer de toutes les autorisations à la racine du conteneur Data Lake Storage Gen2. D’autres clients peuvent avoir besoin de plusieurs clusters avec différents principaux de service, où un cluster dispose de tous les accès aux données, et un autre de l’accès en lecture. 

### <a name="enable-the-data-lake-storage-gen2-firewall-with-azure-service-access"></a>Activer le pare-feu Data Lake Storage Gen2 avec accès au service Azure

Data Lake Storage Gen2 prend en charge l’option d’activation d’un pare-feu et de limitation de l’accès uniquement aux services Azure, ce qui est recommandé pour limiter le vecteur d’attaques extérieures. Le pare-feu peut être activé sur un compte de stockage dans le portail Azure par le biais des options **Pare-feu** > **Activer le pare-feu (ON)**  > **Autoriser l’accès aux services Azure**.

Pour accéder à votre compte de stockage depuis Azure Databricks, déployez Azure Databricks sur votre réseau virtuel, puis ajoutez ce réseau virtuel à votre pare-feu. Consultez [Configurer des pare-feu Stockage Azure et des réseaux virtuels](../common/storage-network-security.md).

## <a name="resiliency-considerations"></a>Remarques relatives à la résilience

Lors de la conception de l’architecture d’un système avec Data Lake Storage Gen2 ou n’importe quel service cloud, vous devez prendre en compte les exigences de disponibilité et la façon de répondre à des interruptions potentielles du service. Un problème peut être localisé dans une instance spécifique ou même dans la région. Être préparé pour ces deux éventualités est important. En fonction des contrats SLA d’objectif de temps de récupération et d’objectif de point de récupération de votre charge de travail, vous choisirez une stratégie plus ou moins agressive pour la haute disponibilité et la reprise d’activité après sinistre.

### <a name="high-availability-and-disaster-recovery"></a>Haute disponibilité et récupération d’urgence

La haute disponibilité et la récupération d’urgence peuvent parfois être combinées, bien qu’elles aient chacune une stratégie légèrement différente, surtout lorsqu’il s’agit de données. Data Lake Storage Gen2 gère déjà trois réplications en arrière-plan pour se protéger contre des défaillances matérielles localisées. De plus, d’autres options de réplication, comme le stockage ZRS ou GZRS, améliorent la haute disponibilité, tandis que GRS & RA-GRS améliorent la reprise d’activité. Lorsque vous élaborez un plan pour la haute disponibilité, en cas d’interruption du service, la charge de travail doit accéder aux dernières données aussi vite que possible en passant à une instance répliquée séparément en local ou dans une nouvelle région.

Dans une stratégie de reprise d’activité, pour se préparer à une improbable défaillance catastrophique d’une région, il est aussi important de disposer de données répliquées dans une autre région à l’aide de la réplication GRS ou RA-GRS. Vous devez aussi tenir compte de vos exigences pour des cas extrêmes comme l’altération des données, où vous voudrez peut-être créer régulièrement des instantanés sur lesquels vous replier. En fonction de l’importance et de la taille des données, pensez à prendre des instantanés delta à intervalles de 1, 6 et 24 heures, en fonction des tolérances au risque.

Pour la résilience des données avec Data Lake Storage Gen2, il est recommandé de géorépliquer vos données par le biais de GRS ou RA-GRS conformément à vos exigences en matière de haute disponibilité et de reprise d’activité après sinistre. De plus, vous devez prendre en compte des méthodes pour que l’application utilisant Data Lake Storage Gen2 puisse basculer automatiquement vers la région secondaire en supervisant des déclencheurs ou les durées des échecs de tentatives, ou en envoyant une notification aux administrateurs pour une intervention manuelle. N’oubliez pas qu’il y a un compromis entre basculer et attendre qu’un service soit de nouveau en ligne.

### <a name="use-distcp-for-data-movement-between-two-locations"></a>Utiliser Distcp pour déplacer des données entre deux emplacements

Diminutif de Distributed Copy, DistCp est un outil en ligne de commande Linux fourni avec Hadoop et qui permet le déplacement de données distribuées entre deux emplacements. Les deux emplacements peuvent être Data Lake Storage Gen2, HDFS ou S3. Cet outil utilise des tâches MapReduce sur un cluster Hadoop (par exemple, HDInsight) pour effectuer un scale-out sur tous les nœuds. Distcp est considéré comme étant la façon la plus rapide de déplacer des données volumineuses sans appliances de compression de réseau spéciales. Cet outil fournit aussi une option pour ne mettre à jour que les valeurs delta entre deux emplacements, gère les essais automatiques et la mise à l’échelle dynamique de calcul. Cette approche est incroyablement efficace quand il est question de répliquer des choses telles que des tableaux Hive/Spark qui peuvent disposer de plusieurs fichiers volumineux dans un seul répertoire, et que vous ne voulez copier que les données modifiées. Pour ces raisons, Distcp est l’outil le plus recommandé pour la copie de données entre des magasins Big Data.

Les tâches de copie peuvent être déclenchées par des workflows Apache Oozie à l’ide de déclencheurs de fréquence ou de données, ainsi que des tâches Cron Linux. Pour les tâches de réplication intensives, il est recommandé d’ajouter un cluster Hadoop HDInsight qui peut être réglé et mis à l’échelle spécialement pour les tâches de copie. Ainsi, les tâches de copie n’interfèrent pas avec les tâches critiques. Si vous exécutez des réplications à une fréquence assez importante, le cluster peut même être arrêté entre chaque tâche. Si vous basculez vers une région secondaire, veillez à ce qu’un autre cluster soit aussi ajouté à cette région pour répliquer de nouvelles données sur le compte principal Data Lake Storage Gen2 quand il sera de nouveau opérationnel. Pour obtenir des exemples d’utilisation de DistCp, consultez [Utiliser DistCp pour copier des données entre des objets blob de Stockage Azure et Data Lake Storage Gen2](../blobs/data-lake-storage-use-distcp.md).

### <a name="use-azure-data-factory-to-schedule-copy-jobs"></a>Utiliser Azure Data Factory pour planifier des tâches de copie

Vous pouvez aussi utiliser Azure Data Factory pour planifier des tâches de copie à l’aide d’une Activité de copie, et même le configurer pour une exécution selon une fréquence spécifique par le biais de l’Assistant de copie. N’oubliez pas qu’Azure Data Factory dispose d’une limite d’unités de déplacement de données cloud (DMU), et finit par atteindre la limite de débit/calcul pour des charges de travail de données volumineuses. Azure Data Factory n’offre pas non plus de mises à jour des valeurs delta entre les comptes Data Lake Storage Gen2 pour l’instant. Les répertoires comme les tableaux Hive nécessitent donc une copie complète pour être répliqués. Pour plus d’informations sur la copie avec Data Factory, consultez l’[article sur la fabrique de données](../../data-factory/load-azure-data-lake-storage-gen2.md).

## <a name="monitoring-considerations"></a>Surveillance - Éléments à prendre en compte

Data Lake Storage Gen2 fournit des métriques dans le portail Azure sous le compte Data Lake Storage Gen2 et dans Azure Monitor. La disponibilité de Data Lake Storage Gen2 est affichée dans le portail Azure. Pour obtenir la disponibilité la plus à jour d’un compte Data Lake Storage Gen2, vous devez exécuter vos propres tests synthétiques afin de valider la disponibilité. D’autres métriques telles que l’utilisation totale du stockage, les requêtes de lecture/écriture et les entrées/sorties peuvent être exploitées grâce à la supervision des applications, et peuvent également déclencher des alertes quand des seuils (par exemple la latence moyenne ou le nombre d’erreurs par minute) sont dépassés.

## <a name="directory-layout-considerations"></a>Considérations relatives à la disposition du répertoire

Lorsque des données arrivent dans Data Lake Store, il est important d’avoir planifié la structure des données afin de pouvoir utiliser avec efficacité la sécurité, le partitionnement et le traitement. La plupart des recommandations suivantes sont applicables à toutes les charges de travail de Big Data. Chaque charge de travail dispose de prérequis différents concernant l’utilisation des données, mais vous trouverez ci-dessous des dispositions communes à prendre lorsque vous utilisez des scénarios IoT ou de traitement.

### <a name="iot-structure"></a>Structure IoT

Dans des charges de travail IoT peuvent se trouver un grand nombre de données arrivées dans le magasin de données, couvrant de nombreux produits, appareils, organisations et clients. Il est important de planifier à l’avance la disposition du répertoire pour des questions d’organisation, de sécurité et de traitement efficace des données pour des consommateurs de flux. Voici un modèle général de disposition à prendre en compte :

*{Region}/{SubjectMatter(s)}/{yyyy}/{mm}/{dd}/{hh}/*

Par exemple, la télémétrie d’arrivée d’un moteur d’avion dans le Royaume-Uni ressemble à la structure suivante :

*UK/Planes/BA1293/Engine1/2017/08/11/12/*

Il est important de placer la date à la fin de la structure de répertoires. Si vous voulez verrouiller certaines régions ou certains thèmes à des utilisateurs/groupes, vous pouvez le faire facilement avec les autorisations POSIX. Sinon, s’il fallait restreindre l’affichage des données du Royaume-Uni ou de certains avions à certains groupes de sécurité, avec la structure de date au début, une autorisation différente serait nécessaire pour de nombreux répertoires sous chaque répertoire heure. De plus, placer la structure de date au début augmente grandement le nombre de répertoires à mesure que le temps passe.

### <a name="batch-jobs-structure"></a>Structure de tâches de traitement par lots

À un niveau global, une approche communément adoptée dans le traitement par lots consiste à envoyer les données dans un répertoire « in ». Ensuite, une fois les données traitées, il faut placer les données obtenues dans un répertoire « out » pour qu’elles soient utilisées par des processus en aval. Cette structure de répertoire est parfois visible avec des travaux qui nécessitent un traitement sur des fichiers individuels et qui ne nécessitent pas forcément de traitement parallèle massif sur des jeux de données volumineux. Tout comme la structure IoT recommandée plus haut, une bonne structure de répertoires dispose de répertoires de niveau parent pour des choses telles que la région et les thèmes (par exemple organisation, produit/producteur). Cette structure facilite la sécurisation des données dans votre organisation et une meilleure gestion des données dans vos charges de travail. En outre, tenez compte de la date et l’heure dans la structure pour permettre une meilleure organisation, de meilleures recherches filtrées, une sécurité plus efficace et une automatisation du traitement. Le niveau granularité de la structure de date est déterminé par l’intervalle de chargement ou de traitement des données, par exemple à l’heure, quotidien, ou mensuel.

Parfois le traitement de fichiers échoue en raison de corruption des données ou de formats inattendus. Dans ces cas, la structure du répertoire peut profiter d’un dossier **/bad** où déplacer les fichiers pour une inspection plus en détail. La tâche de traitement par lots peut aussi gérer le rapport ou la notification de ces fichiers *bad* en vue d’une intervention manuelle. Considérez la structure de modèle suivante :

*{Region}/{SubjectMatter(s)}/In/{yyyy}/{mm}/{dd}/{hh}/* \
*{Region}/{SubjectMatter(s)}/Out/{yyyy}/{mm}/{dd}/{hh}/* \
*{Region}/{SubjectMatter(s)}/Bad/{yyyy}/{mm}/{dd}/{hh}/*

Par exemple, une firme marketing qui reçoit quotidiennement des extraits de données de mises à jour client de la part de leurs clients en Amérique du Nord. Avant et après avoir été traité, il peut ressembler à l’extrait de code suivant :

*NA/Extracts/ACMEPaperCo/In/2017/08/14/updates_08142017.csv*\
*NA/Extracts/ACMEPaperCo/Out/2017/08/14/processed_updates_08142017.csv*

Dans le cas courant de traitement des données par lots dans des bases de données directement, telles que Hive ou des bases de données SQL traditionnelles, il n’y a pas besoin de dossiers **/in** ou **/out** car la sortie se trouve déjà dans un dossier différent pour le tableau Hive ou la base de données externe. Par exemple, des extraits quotidiens de la part de clients arrivent dans leurs dossiers respectifs, et l’orchestration par des outils comme Azure Data Factory, Apache Oozie ou Apache Airflow déclenche une tâche quotidienne Hive ou Spark pour traiter et écrire les données dans un tableau Hive.