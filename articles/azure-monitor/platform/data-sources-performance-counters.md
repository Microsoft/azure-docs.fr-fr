---
title: Collecter des sources de données de performance Windows et Linux avec l’agent Log Analytics dans Azure Monitor
description: Azure Monitor collecte les compteurs de performances pour analyser les performances sur les agents Windows et Linux.  Cet article explique comment configurer la collecte des compteurs de performances sur les agents Windows et Linux, comment ils sont stockés dans l’espace de travail et comment les analyser dans le portail Azure.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/21/2020
ms.openlocfilehash: 533d4a83ea73b98e26a57febc077a607bcb25465
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532298"
---
# <a name="collect-windows-and-linux-performance-data-sources-with-log-analytics-agent"></a>Collecter des sources de données de performance Windows et Linux avec l’agent Log Analytics
Les compteurs de performances dans Windows et Linux fournissent des informations sur les performances des composants matériels, systèmes d’exploitation et applications.  Azure Monitor peut non seulement collecter les compteurs de performances des agents Log Analytics à intervalles réguliers pour effectuer une analyse en temps quasi réel, mais aussi agréger les données de performances pour réaliser des analyses à plus long terme et créer des rapports.

> [!IMPORTANT]
> Cet article traite de la collecte de données de performances avec l’[agent Log Analytics](log-analytics-agent.md), qui est un des agents utilisés par Azure Monitor. D’autres agents collectent des données différentes et sont configurés différemment. Pour obtenir la liste des agents disponibles et les données qu’ils peuvent collecter, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md).

![Compteurs de performance](media/data-sources-performance-counters/overview.png)

## <a name="configuring-performance-counters"></a>Configuration des compteurs de performances
Configurez les compteurs de performances à partir du [menu Données dans les paramètres avancés](agent-data-sources.md#configuring-data-sources) de l’espace de travail Log Analytics.

Lorsque vous configurez initialement des compteurs de performances Windows ou Linux pour un nouvel espace de travail, vous avez la possibilité de créer rapidement plusieurs compteurs communs.  Ils s’affichent avec une case à cocher en regard.  Vérifiez que les compteurs que vous voulez créer sont cochés, puis cliquez sur **Ajouter les compteurs de performances sélectionnés**.

Pour les compteurs de performances Windows, vous pouvez choisir une instance spécifique de chaque compteur de performances. Pour les compteurs de performances Linux, l’instance de chaque compteur choisi s’applique à tous les compteurs enfants du compteur parent. Le tableau suivant montre les instances courantes disponibles pour les compteurs de performances Linux et Windows.

| Nom de l’instance | Description |
| --- | --- |
| \_Total |Total de toutes les instances |
| \* |Toutes les instances |
| (/&#124;/var) |Correspond aux instances nommées : / ou /var |

### <a name="windows-performance-counters"></a>Compteurs de performances Windows

![Configurer des compteurs de performances Windows](media/data-sources-performance-counters/configure-windows.png)

Suivez cette procédure pour ajouter un nouveau compteur de performances Windows à collecter. Notez que les compteurs de performances Windows v2 ne sont pas pris en charge.

1. Tapez le nom du compteur dans la zone de texte, au format *objet(instance)\compteur*.  Lorsque vous commencez à taper, la liste des compteurs correspondants s’affiche.  Vous pouvez soit choisir un compteur dans cette liste, soit taper le nom de votre choix.  Vous pouvez également retourner toutes les instances d’un compteur particulier en spécifiant *objet\compteur*.  

    Lors de la collecte des compteurs de performances SQL Server à partir d’instances nommées, tous les compteurs des instances nommées commencent par *MSSQL$* qui est suivi du nom de l’instance.  Par exemple, pour collecter le compteur Taux d’accès au cache de journal pour toutes les bases de données à partir de l’objet de performance de base de données pour l’instance SQL nommée INST2, spécifiez `MSSQL$INST2:Databases(*)\Log Cache Hit Ratio`.

2. Cliquez sur **+** ou appuyez sur **Entrée** pour ajouter le compteur à la liste.
3. Lorsque vous ajoutez un compteur, il utilise la valeur par défaut de 10 secondes comme **Intervalle d’échantillonnage**.  Vous pouvez configurer jusqu’à 1 800 secondes (30 minutes) si vous souhaitez réduire l’espace de stockage requis pour les données de performances collectées.
4. Après avoir ajouté les compteurs souhaités, cliquez sur le bouton **Enregistrer** en haut de l’écran pour enregistrer la configuration.

### <a name="linux-performance-counters"></a>Compteurs de performances Linux

![Configurer des compteurs de performances Linux](media/data-sources-performance-counters/configure-linux-1.png)

Suivez cette procédure pour ajouter un nouveau compteur de performances Linux à collecter.

1. Tapez le nom du compteur dans la zone de texte, au format *objet(instance)\compteur*.  Lorsque vous commencez à taper, la liste des compteurs correspondants s’affiche.  Vous pouvez soit choisir un compteur dans cette liste, soit taper le nom de votre choix.  
1. Cliquez sur **+** , ou appuyez sur **Entrée** pour ajouter le compteur à la liste des autres compteurs de l’objet.
1. Tous les compteurs d’un objet utilisent le même **Intervalle d’échantillonnage**.  La valeur par défaut est 10 secondes.  Vous pouvez configurer jusqu’à 1 800 secondes (30 minutes) si vous souhaitez réduire l’espace de stockage requis pour les données de performances collectées.
1. Après avoir ajouté les compteurs souhaités, cliquez sur le bouton **Enregistrer** en haut de l’écran pour enregistrer la configuration.

#### <a name="configure-linux-performance-counters-in-configuration-file"></a>Configuration des compteurs de performances Linux dans le fichier de configuration
Au lieu de configurer les compteurs de performances Linux à l’aide du portail Azure, vous pouvez modifier les fichiers de configuration sur l’agent Linux.  Les mesures de performances à collecter sont contrôlées par la configuration du fichier **/etc/opt/microsoft/omsagent/\<workspace id\>/conf/omsagent.conf**.

Chaque objet, ou catégorie, de mesures de performances à collecter doit être défini dans le fichier de configuration comme un seul élément `<source>` . La syntaxe suit le modèle suivant.

```xml
<source>
    type oms_omi  
    object_name "Processor"
    instance_regex ".*"
    counter_name_regex ".*"
    interval 30s
</source>
```


Les paramètres de cet élément sont décrits dans le tableau suivant.

| Paramètres | Description |
|:--|:--|
| object\_name | Nom de l’objet pour la collecte. |
| instance\_regex |  *Expression régulière* qui définit les instances à collecter. La valeur `.*` spécifie toutes les instances. Pour ne collecter les mesures de processeur que de l’instance \_Total, vous pouvez spécifier `_Total`. Pour ne collecter les mesures de processeur que des instances crond ou sshd, vous pouvez spécifier `(crond\|sshd)`. |
| counter\_name\_regex | *Expression régulière* qui définit les compteurs (pour l’objet) à collecter. Pour collecter tous les compteurs de l’objet, spécifiez : `.*`. Pour ne collecter que les compteurs d’espace d’échange de l’objet mémoire, vous pouvez indiquer : `.+Swap.+` |
| interval | Fréquence de collecte des compteurs de l’objet. |


Le tableau suivant répertorie les objets et compteurs que vous pouvez indiquer dans le fichier de configuration.  Des compteurs supplémentaires sont disponibles pour certaines applications tel que décrit dans [Collecte des compteurs de performances pour les applications Linux dans Azure Monitor](data-sources-linux-applications.md).

| Nom d’objet | Nom de compteur |
|:--|:--|
| Logical Disk | % Free Inodes |
| Logical Disk | % Free Space |
| Logical Disk | % Used Inodes |
| Logical Disk | % Used Space |
| Logical Disk | Nb d’octets de lecture de disque/s |
| Logical Disk | Nb d’opérations de lectures de disque/s |
| Logical Disk | Disk Transfers/sec |
| Logical Disk | Nb d’octets d’écriture de disque/s |
| Logical Disk | Nb d’opération d’écriture de disque/s |
| Logical Disk | Free Megabytes |
| Logical Disk | Logical Disk Bytes/sec |
| Mémoire | % Available Memory |
| Mémoire | % Available Swap Space |
| Mémoire | % Used Memory |
| Mémoire | % Used Swap Space |
| Mémoire | Available MBytes Memory |
| Mémoire | Available MBytes Swap |
| Mémoire | Page Reads/sec |
| Mémoire | Page Writes/sec |
| Mémoire | Pages/sec |
| Mémoire | Used MBytes Swap Space |
| Mémoire | Used Memory MBytes |
| Réseau | Total Bytes Transmitted |
| Réseau | Total Bytes Received |
| Réseau | Total Bytes |
| Réseau | Total Packets Transmitted |
| Réseau | Total Packets Received |
| Réseau | Total Rx Errors |
| Réseau | Total Tx Errors |
| Réseau | Total Collisions |
| Physical Disk | Avg. Disk sec/Read |
| Physical Disk | Avg. Disk sec/Transfer |
| Physical Disk | Avg. Disk sec/Write |
| Physical Disk | Physical Disk Bytes/sec |
| Process | Pct Privileged Time |
| Process | Pct User Time |
| Process | Used Memory kBytes |
| Process | Virtual Shared Memory |
| Processeur | % DPC Time |
| Processeur | % Idle Time |
| Processeur | % Interrupt Time |
| Processeur | % IO Wait Time |
| Processeur | % Nice Time |
| Processeur | % Privileged Time |
| Processeur | % temps processeur |
| Processeur | % User Time |
| Système | Free Physical Memory |
| Système | Free Space in Paging Files |
| Système | Free Virtual Memory |
| Système | Processus |
| Système | Size Stored In Paging Files |
| Système | Uptime |
| Système | Utilisateurs |


La configuration par défaut des mesures de performances est la suivante.

```xml
<source>
    type oms_omi
    object_name "Physical Disk"
    instance_regex ".*"
    counter_name_regex ".*"
    interval 5m
</source>

<source>
    type oms_omi
    object_name "Logical Disk"
    instance_regex ".*"
    counter_name_regex ".*"
    interval 5m
</source>

<source>
    type oms_omi
    object_name "Processor"
    instance_regex ".*"
    counter_name_regex ".*"
    interval 30s
</source>

<source>
    type oms_omi
    object_name "Memory"
    instance_regex ".*"
    counter_name_regex ".*"
    interval 30s
</source>
```

## <a name="data-collection"></a>Collecte de données
Azure Monitor collecte tous les compteurs de performances spécifiés selon l’intervalle d’échantillonnage spécifié sur tous les agents où le compteur est installé.  Les données ne sont pas agrégées, et les données brutes sont disponibles dans toutes les vues de requête de journal pendant la durée spécifiée par votre espace de travail d’analytique des journaux d’activité.

## <a name="performance-record-properties"></a>Propriétés des enregistrements de performances
Les enregistrements de performances sont de type **Perf** et leurs propriétés sont décrites dans le tableau suivant.

| Propriété | Description |
|:--- |:--- |
| Computer |Ordinateur sur lequel l’événement a été collecté. |
| CounterName |Nom du compteur de performances. |
| CounterPath |Chemin complet du compteur au format \\\\\<Computer>\\objet(instance)\\compteur. |
| CounterValue |Valeur numérique du compteur. |
| InstanceName |Nom de l’instance de l’événement.  Vide si aucune instance. |
| ObjectName |Nom de l’objet de performance. |
| SourceSystem |Type d’agent sur lequel les données ont été collectées. <br><br>Ops Manager – Agent Windows, connexion directe ou SCOM <br> Linux – Tous les agents Linux  <br> AzureStorage – Diagnostics Azure |
| TimeGenerated |Date et heure d’échantillonnage des données. |

## <a name="sizing-estimates"></a>Tailles estimées
 La collecte d’un compteur toutes les 10 secondes correspond environ à 1 Mo par jour et par instance.  La formule suivante vous permet d’estimer l’espace de stockage requis pour un compteur particulier.

> 1 Mo × (nombre de compteurs) × (nombre d’agents) × (nombre d’instances)

## <a name="log-queries-with-performance-records"></a>Requêtes de journal avec des enregistrements de performances
Le tableau suivant fournit plusieurs exemples de requêtes qui extraient des enregistrements de performances.

| Requête | Description |
|:--- |:--- |
| Perf |Toutes les données de performances |
| Perf &#124; où l’ordinateur == « MyComputer » |Toutes les données de performances d’un ordinateur particulier |
| Perf &#124; où CounterName == « longueur de la file d’attente de disque actuelle » |Toutes les données de performances d’un compteur particulier |
| Perf &#124; où ObjectName == « Processeur », CounterName == « % du temps processeur » et InstanceName == « _Total » &#124; résumer AVGCPU = avg(CounterValue) par ordinateur |Utilisation moyenne du processeur entre tous les ordinateurs |
| Perf &#124; où CounterName == « % du temps processeur » &#124; résumer AggregatedValue = max(CounterValue) par ordinateur |Utilisation maximale du processeur entre tous les ordinateurs |
| Perf &#124; où ObjectName == « LogicalDisk », CounterName == « Longueur de la file d’attente de disque actuelle » et l’ordinateur == « MyComputerName » &#124; résumer AggregatedValue = avg(CounterValue) par InstanceName |Longueur actuelle moyenne de file d’attente du disque pour toutes les instances d’un ordinateur donné |
| Perf &#124; où CounterName == « Transferts disque/s » &#124; résumer AggregatedValue = centile(CounterValue, 95) par ordinateur |95e centile de transferts disque/s entre tous les ordinateurs |
| Perf &#124; où CounterName == « % du temps processeur » et InstanceName == « _Total » &#124; résumer AggregatedValue = avg(CounterValue) par emplacement (TimeGenerated, 1 h), ordinateur |Moyenne horaire d’utilisation du processeur sur tous les ordinateurs |
| Perf &#124; où l’ordinateur == « MyComputer », CounterName startswith_cs « % » et InstanceName == « _Total » &#124; résumer AggregatedValue = centile(CounterValue, 70) par emplacement (TimeGenerated, 1 h), CounterName | 70e centile horaire de chaque compteur de pourcentage pour un ordinateur particulier |
| Perf &#124; où CounterName == « % du temps processeur », InstanceName == « _Total » et l’ordinateur == « MyComputer » &#124; résumer [« min(CounterValue) »] = min(CounterValue), [« avg(CounterValue) »] = avg(CounterValue), [« percentile75(CounterValue) »] = centile (CounterValue, 75), [« max(CounterValue) »] = max(CounterValue) par emplacement (TimeGenerated, 1 h), ordinateur |Moyenne horaire, minimum, maximum et 75e centile d’utilisation du processeur pour un ordinateur spécifique |
| Perf &#124; où ObjectName == « MSSQL$ INST2 : bases de données » et InstanceName == « maître » | Toutes les données de performances de l’objet de performance de base de données pour la base de données MASTER à partir de l’instance de SQL Server nommée INST2.  




## <a name="next-steps"></a>Étapes suivantes
* [Collectez des compteurs de performances à partir d’applications Linux](data-sources-linux-applications.md), y compris Apache HTTP Server et MySQL.
* Découvrez les [requêtes dans les journaux](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions.  
* Exporter les données collectées vers [Power BI](powerbi.md) à des fins d’analyse et de visualisation.
