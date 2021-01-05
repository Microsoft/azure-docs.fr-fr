---
title: Surveiller Azure File Sync | Microsoft Docs
description: Découvrez comment surveiller votre déploiement Azure File Sync à l’aide d’Azure Monitor, de Storage Sync Service et de Windows Server.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/28/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 59c489fac8bf02263cc51833675af414d5de6a52
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686002"
---
# <a name="monitor-azure-file-sync"></a>Superviser Azure File Sync

Utilisez Azure File Sync pour centraliser les partages de fichiers de votre organisation dans Azure Files tout en conservant la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Azure File Sync transforme Windows Server en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible dans Windows Server pour accéder à vos données localement, notamment SMB, NFS et FTPS. Vous pouvez avoir autant de caches que nécessaire dans le monde entier.

Cet article explique comment surveiller votre déploiement Azure File Sync à l’aide d’Azure Monitor, de Storage Sync Service et de Windows Server.

Les scénarios abordés dans ce guide sont les suivants : 
- Afficher les métriques d’Azure File Sync dans Azure Monitor.
- Créer des alertes dans Azure Monitor pour vous informer de manière proactive en cas de conditions critiques.
- Affichez l’intégrité de votre déploiement Azure File Sync à l’aide du Portail Azure.
- Comment utiliser les journaux des événements et les compteurs de performance sur vos serveurs Windows pour superviser l’intégrité de votre déploiement Azure File Sync. 

## <a name="azure-monitor"></a>Azure Monitor

Utilisez [Azure Monitor](../../azure-monitor/overview.md) pour afficher les métriques et configurer des alertes pour la synchronisation, la hiérarchisation cloud et la connectivité du serveur.  

### <a name="metrics"></a>Mesures

Les métriques pour Azure File Sync sont activées par défaut et sont envoyées à Azure Monitor toutes les 15 minutes.

**Comment afficher les métriques d’Azure File Sync dans Azure Monitor**
1. Rendez-vous sur votre service de synchronisation **Storage Sync Service** dans le **portail Azure** et cliquez sur **Métriques**.
2. Cliquez dans la liste déroulante **Métrique** et sélectionnez la mesure que vous souhaitez visualiser.

![Capture d’écran de métriques d’Azure File Sync](media/storage-sync-files-troubleshoot/file-sync-metrics.png)

Les métriques suivantes pour Azure File Sync sont disponibles dans Azure Monitor :

| Nom de métrique | Description |
|-|-|
| Octets synchronisés | Taille des données transférées (chargement et téléchargement).<br><br>Unité : Octets<br>Type d’agrégation : SUM<br>Dimensions applicables : Nom du point de terminaison de serveur, sens de la synchronisation, nom du groupe de synchronisation |
| Rappel de hiérarchisation cloud | Taille des données rappelées.<br><br>**Remarque** : cette métrique sera supprimée ultérieurement. Utilisez la métrique de taille de rappel de la hiérarchisation cloud pour surveiller la taille des données rappelées.<br><br>Unité : Octets<br>Type d’agrégation : SUM<br>Dimension applicable : Nom du serveur |
| Taille de rappel de la hiérarchisation cloud | Taille des données rappelées.<br><br>Unité : Octets<br>Type d’agrégation : SUM<br>Dimension applicable : Nom du serveur, nom du groupe de synchronisation |
| Taille de rappel de hiérarchisation cloud par application | Taille des données rappelées par application.<br><br>Unité : Octets<br>Type d’agrégation : SUM<br>Dimension applicable : Nom de l’application, nom du serveur, nom du groupe de synchronisation |
| Débit de rappel de la hiérarchisation cloud | Taille de débit de rappel des données.<br><br>Unité : Octets<br>Type d’agrégation : SUM<br>Dimension applicable : Nom du serveur, nom du groupe de synchronisation |
| Fichiers ne se synchronisant pas | Nombre de fichiers qui ne peuvent pas être synchronisés.<br><br>Unité : Count<br>Type d’agrégation : SUM<br>Dimensions applicables : Nom du point de terminaison de serveur, sens de la synchronisation, nom du groupe de synchronisation |
| Fichiers synchronisés | Nombre de fichiers transférés (chargement et téléchargement).<br><br>Unité : Count<br>Type d’agrégation : SUM<br>Dimensions applicables : Nom du point de terminaison de serveur, sens de la synchronisation, nom du groupe de synchronisation |
| État du serveur en ligne | Nombre de pulsations reçues du serveur.<br><br>Unité : Count<br>Type d’agrégation : Maximale<br>Dimension applicable : Nom du serveur |
| Résultat de session de synchronisation | Résultat de session de synchronisation (1 = session de synchronisation réussie ; 0 = session de synchronisation ayant échoué)<br><br>Unité : Count<br>Types d’agrégation : Maximale<br>Dimensions applicables : Nom du point de terminaison de serveur, sens de la synchronisation, nom du groupe de synchronisation |

### <a name="alerts"></a>Alertes

Les alertes vous avertissent de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Pour en savoir plus sur la configuration des alertes dans Azure Monitor, consultez [Vue d’ensemble des alertes dans Microsoft Azure](../../azure-monitor/platform/alerts-overview.md).

**Guide pratique pour créer des alertes pour Azure File Sync**

1. Rendez-vous sur votre service de synchronisation **Storage Sync Service** dans le **Portail Azure**. 
2. Cliquez sur **Alertes** dans la section Surveillance, puis cliquez sur **+ Nouvelle règle d’alerte**.
3. Cliquez sur **Sélectionnez une condition** et fournissez les informations suivantes pour l’alerte : 
    - **Mesure**
    - **Nom de la dimension**
    - **Logique d'alerte**
4. Cliquez sur **Sélectionner un groupe d’actions**, puis ajoutez un groupe d’actions (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
5. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).
6. Cliquez sur **Créer une règle d’alerte** pour créer l’alerte.  

Le tableau suivant répertorie quelques exemples de scénarios destinés à la surveillance ainsi que la métrique appropriée à utiliser pour l’alerte :

| Scénario | Mesure à utiliser pour l’alerte |
|-|-|
| L’intégrité du point de terminaison de serveur affiche une erreur dans le portail | Résultat de session de synchronisation |
| La synchronisation des fichiers avec le serveur ou le point de terminaison cloud échoue | Fichiers ne se synchronisant pas |
| Le serveur inscrit ne parvient pas à communiquer avec le service de synchronisation de stockage | État du serveur en ligne |
| La taille de rappel de la hiérarchisation cloud a dépassé 500 Gio en une journée  | Taille de rappel de la hiérarchisation cloud |

Pour obtenir des instructions sur la création d’alertes dans le cadre de ces scénarios, consultez la section [Exemples d’alerte](#alert-examples).

## <a name="storage-sync-service"></a>Service de synchronisation du stockage

Pour visualiser l’intégrité de votre déploiement Azure File Sync dans le **Portail Azure**, accédez à **Storage Sync Service** ; les informations suivantes sont alors disponibles :

- Intégrité du serveur inscrit
- Intégrité du point de terminaison de serveur
    - Fichiers ne se synchronisant pas
    - Activité de synchronisation
    - Efficacité de la hiérarchisation cloud
    - Fichiers non hiérarchisés
    - Erreurs de rappel
- Mesures

### <a name="registered-server-health"></a>Intégrité du serveur inscrit

Pour afficher l’**intégrité du serveur inscrit** dans le portail, accédez à la section **Serveurs inscrits** du **Service de synchronisation de stockage**.

![Capture d’écran de l’intégrité des serveurs inscrits](media/storage-sync-files-troubleshoot/file-sync-registered-servers.png)

- Si l'état du **serveur inscrit** est défini sur **En ligne**, cela signifie que le serveur communique avec le service.
- Si l’état du **serveur inscrit** est **Apparaître hors connexion**, ce problème peut se produire si le processus de supervision de la synchronisation du stockage (AzureStorageSyncMonitor.exe) ne s’exécute pas ou que le serveur ne peut pas accéder au service Azure File Sync. Pour plus d’instructions, consultez la [documentation sur le dépannage](./storage-sync-files-troubleshoot.md?tabs=portal1%252cazure-portal#server-endpoint-noactivity).

### <a name="server-endpoint-health"></a>Intégrité du point de terminaison de serveur

Pour afficher l’intégrité d’un **point de terminaison de serveur**  dans le portail, accédez à la section **Groupes de synchronisation** du **service de synchronisation de stockage**, puis sélectionnez un **groupe de synchronisation**.

![Capture d’écran de l’intégrité du point de terminaison de serveur](media/storage-sync-files-troubleshoot/file-sync-server-endpoint-health.png)

- L’**intégrité du point de terminaison de serveur** et l’**activité de synchronisation** dans le portail sont basées sur les événements de synchronisation qui sont enregistrés dans le journal d’événements de télémétrie sur le serveur (ID 9102 et 9302). Si une session de synchronisation échoue à cause d’une erreur transitoire, telle qu’une erreur annulée, le point de terminaison de serveur apparaît toujours comme **sain** sur le portail tant que la session de synchronisation progresse (les fichiers sont appliqués). L’événement ID 9302 est l’événement de progression de la synchronisation et l’événement ID 9102 est consigné une fois la session de synchronisation terminée.  Pour plus d'informations, consultez [Intégrité de la synchronisation](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#broken-sync) et [Progression de la synchronisation](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#how-do-i-monitor-the-progress-of-a-current-sync-session). Si l’intégrité du point de terminaison de serveur affiche une **Erreur** ou **Aucune activité**, consultez la [documentation sur la résolution des problèmes](./storage-sync-files-troubleshoot.md?tabs=portal1%252cazure-portal#common-sync-errors) pour obtenir de l’aide.
- Le nombre de fichiers **non synchronisés** dans le portail est basé sur l’ID d’événement 9121 qui est consigné dans le journal des événements de télémétrie sur le serveur. Cet événement est journalisé pour chaque erreur par élément une fois la session de synchronisation terminée. Pour résoudre ce type d’erreur, consultez [Comment puis-je voir s’il existe des fichiers ou dossiers qui ne sont pas synchronisés ?](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing).
- Pour visualiser l’**efficacité de la hiérarchisation cloud** dans le portail, accédez à la section **Propriétés du point de terminaison de serveur**, puis à la section **Hiérarchisation cloud**. Les données fournies pour l’efficacité de la hiérarchisation cloud sont basées sur l’événement ID 9071 qui est consigné dans le journal des événements de télémétrie sur le serveur. Pour plus d’informations, consultez [Vue d’ensemble de la hiérarchisation cloud](./storage-sync-cloud-tiering.md).
- Pour visualiser les **fichiers non hiérarchisés** et les **erreurs de rappel** dans le portail, accédez à la section **Propriétés du point de terminaison de serveur**, puis à la section **Hiérarchisation cloud**. Les **fichiers non hiérarchisés** sont basés sur l’ID d’événement 9003 qui est consigné dans le journal des événements de télémétrie sur le serveur et les **erreurs de rappel** sont basées sur l’ID d’événement 9006. Pour examiner les fichiers qui ne peuvent pas être hiérarchisés ou rappelés, consultez [Résoudre les problèmes de hiérarchisation de fichiers](./storage-sync-files-troubleshoot.md?tabs=portal1%252cazure-portal#how-to-troubleshoot-files-that-fail-to-tier) et [Résoudre les problèmes de hiérarchisation de fichiers](./storage-sync-files-troubleshoot.md?tabs=portal1%252cazure-portal#how-to-troubleshoot-files-that-fail-to-be-recalled).

### <a name="metric-charts"></a>Graphiques de métrique

- Les graphiques de métrique suivants sont visibles sur le portail du service de synchronisation de stockage :

  | Nom de métrique | Description | Nom du panneau |
  |-|-|-|
  | Octets synchronisés | Taille des données transférées (chargement et téléchargement) | Groupe de synchronisation, point de terminaison du serveur |
  | Rappel de hiérarchisation cloud | taille des données rappelées ; | Serveurs inscrits |
  | Fichiers ne se synchronisant pas | Nombre de fichiers qui ne peuvent pas être synchronisés | Point de terminaison de serveur |
  | Fichiers synchronisés | Nombre de fichiers transférés (chargement et téléchargement) | Groupe de synchronisation, point de terminaison du serveur |
  | État du serveur en ligne | Nombre de pulsations reçues du serveur | Serveurs inscrits |

- Pour en savoir plus, consultez [Azure Monitor](#azure-monitor).

  > [!Note]  
  > Un intervalle de temps de 24 heures s'applique aux graphiques du portail du service de synchronisation de stockage. Pour afficher d'autres intervalles de temps ou dimensions, utilisez Azure Monitor.

## <a name="windows-server"></a>Windows Server

Sur le **serveur Windows** où l’agent Azure File Sync est installé, vous pouvez visualiser l’intégrité des points de terminaison du serveur en utilisant les **journaux d’événements** et les **compteurs de performance**.

### <a name="event-logs"></a>Journaux d’événements

Utilisez le journal des événements de télémétrie sur le serveur pour surveiller l’intégrité de hiérarchisation cloud, de synchronisation et de serveur inscrit. Le journal des événements de télémétrie se trouve dans l'observateur d'événements, sous *Applications and Services\Microsoft\FileSync\Agent*.

Intégrité de la synchronisation

- L’ID d’événement 9102 est enregistré une fois que la session de terminaison se termine. Utilisez cet événement pour déterminer si les sessions de synchronisation aboutissent (**HResult = 0**) et s'il existe des erreurs de synchronisation par élément (**PerItemErrorCount**). Pour plus d'informations, consultez la documentation [Intégrité de la synchronisation](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#broken-sync) et [Erreurs par élément](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing).

  > [!Note]  
  > Parfois, les sessions de synchronisation échouent totalement ou présentent un champ PerItemErrorCount différent de zéro. Pourtant, elles progressent toujours et la synchronisation de certains fichiers aboutit. Ceci est visible dans les champs de type Applied, comme AppliedFileCount, AppliedDirCount, AppliedTombstoneCount et AppliedSizeBytes. Ces champs indiquent dans quelle mesure la session a abouti. Si vous voyez plusieurs sessions de synchronisation consécutives échouer tout en ayant un nombre Applied croissant, laissez à la synchronisation le temps de réessayer avant d'ouvrir un ticket de support.

- L’ID d’événement 9121 est journalisé pour chaque erreur par élément une fois la session de synchronisation terminée. Utilisez cet événement pour déterminer le nombre de fichiers qui ne se synchronisent pas avec cette erreur (**PersistentCount** et **TransientCount**). Pour savoir rechercher les erreurs persistantes par élément, consultez [Comment puis-je voir s’il existe des fichiers ou dossiers qui ne sont pas synchronisés ?](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing).

- L’ID d’événement 9302 est enregistré toutes les 5 à 10 minutes s’il existe une session de synchronisation active. Utilisez cet événement pour déterminer le nombre d’éléments qui doivent être synchronisés (**TotalItemCount**), le nombre d’éléments qui ont été synchronisés jusqu’à présent (**AppliedItemCount**) et le nombre d’éléments dont la synchronisation a échoué en raison d’une erreur par élément (**PerItemErrorCount**). Si la synchronisation ne progresse pas (**AppliedItemCount=0**), la session de synchronisation finira par échouer, et un ID d’événement 9102 sera journalisé avec l’erreur. Pour plus d'informations, consultez la [documentation consacrée à la progression de la synchronisation](./storage-sync-files-troubleshoot.md?tabs=server%252cazure-portal#how-do-i-monitor-the-progress-of-a-current-sync-session).

Intégrité du serveur inscrit

- L’ID d’événement 9301 est enregistré toutes les 30 secondes quand un serveur interroge le service pour des travaux. Si GetNextJob se termine avec **status = 0**, cela signifie que le serveur est en mesure de communiquer avec le service. Si GetNextJob se termine avec une erreur, consultez la [documentation de dépannage](./storage-sync-files-troubleshoot.md?tabs=portal1%252cazure-portal#server-endpoint-noactivity) pour obtenir des conseils.

Intégrité de la hiérarchisation cloud

- Pour surveiller l'activité de hiérarchisation sur un serveur, utilisez les ID d'événement 9003, 9016 et 9029 dans le journal des événements de télémétrie. Celui-ci se trouve dans l'observateur d'événements, sous *Applications and Services\Microsoft\FileSync\Agent*.

  - L’ID d’événement 9003 fournit la distribution des erreurs de distribution pour un point de terminaison de serveur. Par exemple : le nombre total d'erreurs et le code d'erreur. Un événement est enregistré par code d'erreur.
  - L’ID d’événement 9016 fournit des résultats de dédoublement pour un volume. Par exemple : le pourcentage d'espace libre, le nombre de fichiers dupliqués au cours de la session et le nombre d'échecs de duplication de fichiers.
  - L’ID d’événement 9029 fournit des informations sur les sessions de duplication d’un point de terminaison de serveur. Par exemple : le nombre de fichiers tentés au cours de la session, le nombre de fichiers hiérarchisés au cours de la session et le nombre de fichiers déjà hiérarchisés.
  
- Pour surveiller l'activité de rappel sur un serveur, utilisez les ID d'événement 9005, 9006, 9009, 9059 et 9071 dans le journal des événements de télémétrie. Celui-ci se trouve dans l'observateur d'événements, sous *Applications and Services\Microsoft\FileSync\Agent*.

  - L’ID d’événement 9005 fournit une fiabilité de rappel pour un point de terminaison de serveur. Par exemple : le nombre de total de fichiers uniques consultés et le nombre total de fichiers uniques dont l'accès a échoué.
  - L’ID d’événement 9006 fournit la distribution des erreurs de rappel pour un point de terminaison de serveur. Par exemple : le nombre total de demandes ayant échoué et le code d'erreur. Un événement est enregistré par code d'erreur.
  - L’ID d’événement 9009 fournit des informations sur les sessions de rappel d’un point de terminaison de serveur. Par exemple : DurationSeconds, CountFilesRecallSucceeded et CountFilesRecallFailed.
  - L’ID d’événement 9059 fournit la distribution des rappels d’application pour un point de terminaison de serveur. Par exemple : ShareId, Application Name et TotalEgressNetworkBytes.
  - L’ID d’événement 9071 fournit l’efficacité de la hiérarchisation cloud pour un point de terminaison de serveur. Par exemple : TotalDistinctFileCountCacheHit, TotalDistinctFileCountCacheMiss, TotalCacheHitBytes et TotalCacheMissBytes.

### <a name="performance-counters"></a>Compteurs de performance

Utilisez les compteurs de performances d’Azure File Sync sur le serveur pour surveiller l’activité de synchronisation.

Pour afficher les compteurs de performances Azure File Sync sur le serveur, ouvrez l'Analyseur de performances (Perfmon.exe). Les compteurs se trouvent sous les objets **AFS Bytes Transferred** et **AFS Sync Operations**.

Les compteurs de performances suivants pour Azure File Sync sont disponibles dans l’Analyseur de performances :

| Objet de performances\Nom de compteur | Description |
|-|-|
| AFS Bytes Transferred (Octets transférés AFS)\Downloaded Bytes/sec (Octets téléchargés/s) | Nombre d’octets téléchargés par seconde. |
| AFS Bytes Transferred (Octets transférés AFS)\Uploaded Bytes/sec (Octets chargés/s) | Nombre d’octets chargés par seconde. |
| AFS Bytes Transferred (Octets transférés AFS)\Total Bytes/sec (Total des octets/s) | Nombre total d’octets par seconde (chargement et téléchargement). |
| AFS Sync Operations (Opérations de synchronisation AFS)\Downloaded Sync Files/sec (Fichiers de synchronisation téléchargés/s) | Nombre de fichiers téléchargés par seconde. |
| AFS Sync Operations (Opérations de synchronisation AFS)\Uploaded Sync Files/sec (Fichiers de synchronisation chargés/s) | Nombre de fichiers chargés par seconde. |
| AFS Sync Operations (Opérations de synchronisation AFS)\Total Sync File Operations/sec (Total des opérations de fichier de synchronisation/s) | Nombre total de fichiers synchronisés (chargement et téléchargement). |

## <a name="alert-examples"></a>Exemples d’alerte
Cette section propose des exemples d’alertes pour Azure File Sync.

  > [!Note]  
  > Si vous créez une alerte et que celle-ci est trop bruyante, ajustez la valeur de seuil et la logique d’alerte.
  
### <a name="how-to-create-an-alert-if-the-server-endpoint-health-shows-an-error-in-the-portal"></a>Comment créer une alerte si l’intégrité du point de terminaison de serveur affiche une erreur dans le portail

1. Dans le **Portail Azure**, accédez au **Service de synchronisation de stockage** respectif. 
2. Accédez à la section **Supervision** et cliquez sur **Alertes**. 
3. Cliquez sur **+ Nouvelle règle d’alerte** pour créer une nouvelle règle d’alerte. 
4. Configurez la condition en cliquant sur **Sélectionner la condition**.
5. Dans le panneau **Configurer la logique du signal**, cliquez sur **Résultat de session de synchronisation** sous le nom du signal.  
6. Sélectionnez la configuration de dimension suivante : 
    - Nom de la dimension : **Nom de point de terminaison de serveur**  
    - Opérateur : **=** 
    - Valeurs de dimension : **Toutes les valeurs actuelles et futures**  
7. Accédez à **Logique d’alerte** et procédez comme suit : 
    - Seuil défini sur **Statique** 
    - Opérateur : **Inférieur à** 
    - Type d’agrégation : **Maximum**  
    - Valeur de seuil : **1** 
    - Évaluées sur la base de : Granularité de l’agrégation = **24 heures** | Fréquence d’évaluation = **toutes les heures** 
    - Cliquez sur **Terminé.** 
8. Cliquez sur **Sélectionner un groupe d’actions** pour ajouter un groupe d’actions (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
9. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).
10. Cliquez sur **Créer une règle d'alerte**. 

### <a name="how-to-create-an-alert-if-files-are-failing-to-sync-to-a-server-or-cloud-endpoint"></a>Comment créer une alerte en cas d’échec de la synchronisation des fichiers sur un point de terminaison de serveur ou cloud

1. Dans le **Portail Azure**, accédez au **Service de synchronisation de stockage** respectif. 
2. Accédez à la section **Supervision** et cliquez sur **Alertes**. 
3. Cliquez sur **+ Nouvelle règle d’alerte** pour créer une nouvelle règle d’alerte. 
4. Configurez la condition en cliquant sur **Sélectionner la condition**.
5. Dans le panneau **Configurer la logique du signal**, cliquez sur **Fichiers ne se synchronisant pas** sous le nom du signal.  
6. Sélectionnez la configuration de dimension suivante : 
     - Nom de la dimension : **Nom de point de terminaison de serveur**  
     - Opérateur : **=** 
     - Valeurs de dimension : **Toutes les valeurs actuelles et futures**  
7. Accédez à **Logique d’alerte** et procédez comme suit : 
     - Seuil défini sur **Statique** 
     - Opérateur : **Supérieur à** 
     - Type d’agrégation : **Moyenne**  
     - Valeur de seuil : **100** 
     - Évaluées sur la base de : Granularité de l’agrégation = **5 minutes** | Fréquence d’évaluation = **toutes les 5 minutes** 
     - Cliquez sur **Terminé.** 
8. Cliquez sur **Sélectionner un groupe d’actions** pour ajouter un groupe d’actions (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
9. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).
10. Cliquez sur **Créer une règle d'alerte**. 

### <a name="how-to-create-an-alert-if-a-registered-server-is-failing-to-communicate-with-the-storage-sync-service"></a>Comment créer une alerte si un serveur inscrit ne parvient pas à communiquer avec le service de synchronisation de stockage

1. Dans le **Portail Azure**, accédez au **Service de synchronisation de stockage** respectif. 
2. Accédez à la section **Supervision** et cliquez sur **Alertes**. 
3. Cliquez sur **+ Nouvelle règle d’alerte** pour créer une nouvelle règle d’alerte. 
4. Configurez la condition en cliquant sur **Sélectionner la condition**.
5. Dans le panneau **Configurer la logique du signal**, cliquez sur **État du serveur en ligne** sous le nom du signal.  
6. Sélectionnez la configuration de dimension suivante : 
     - Nom de la dimension : **Nom du serveur**  
     - Opérateur : **=** 
     - Valeurs de dimension : **Toutes les valeurs actuelles et futures**  
7. Accédez à **Logique d’alerte** et procédez comme suit : 
     - Seuil défini sur **Statique** 
     - Opérateur : **Inférieur à** 
     - Type d’agrégation : **Maximum**  
     - Valeur de seuil (en octets) : **1** 
     - Évaluées sur la base de : Granularité de l’agrégation = **1 heure** | Fréquence d’évaluation = **toutes les 30 minutes** 
        - Notez que les métriques sont envoyées à Azure Monitor toutes les 15 à 20 minutes. Ne définissez pas la **Fréquence d’évaluation** sur une valeur inférieure à 30 minutes (générera des alertes erronées).
     - Cliquez sur **Terminé.** 
8. Cliquez sur **Sélectionner un groupe d’actions** pour ajouter un groupe d’actions (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
9. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).
10. Cliquez sur **Créer une règle d'alerte**. 

### <a name="how-to-create-an-alert-if-the-cloud-tiering-recall-size-has-exceeded-500gib-in-a-day"></a>Comment créer une alerte si la taille de rappel de la hiérarchisation cloud a dépassé 500 Gio en une journée

1. Dans le **Portail Azure**, accédez au **Service de synchronisation de stockage** respectif. 
2. Accédez à la section **Supervision** et cliquez sur **Alertes**. 
3. Cliquez sur **+ Nouvelle règle d’alerte** pour créer une nouvelle règle d’alerte. 
4. Configurez la condition en cliquant sur **Sélectionner la condition**.
5. Dans le panneau **Configurer la logique du signal**, cliquez sur **Taille de rappel de la hiérarchisation cloud** sous le nom du signal.  
6. Sélectionnez la configuration de dimension suivante : 
     - Nom de la dimension : **Nom du serveur**  
     - Opérateur : **=** 
     - Valeurs de dimension : **Toutes les valeurs actuelles et futures**  
7. Accédez à **Logique d’alerte** et procédez comme suit : 
     - Seuil défini sur **Statique** 
     - Opérateur : **Supérieur à** 
     - Type d’agrégation : **Total**  
     - Valeur de seuil (en octets) : **67108864000** 
     - Évaluées sur la base de : Granularité de l’agrégation = **24 heures** | Fréquence d’évaluation = **toutes les heures** 
    - Cliquez sur **Terminé.** 
8. Cliquez sur **Sélectionner un groupe d’actions** pour ajouter un groupe d’actions (e-mail, SMS, etc.) à l’alerte, soit en sélectionnant un groupe d’actions existant, soit en créant un nouveau groupe d’actions.
9. Renseignez les **détails de l’alerte** (**Nom de règle d’alerte**, **Description** et **Gravité** par exemple).
10. Cliquez sur **Créer une règle d'alerte**. 

## <a name="next-steps"></a>Étapes suivantes
- [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Prenez en compte les paramètres de pare-feu et de proxy](storage-sync-files-firewall-and-proxy.md)
- [Déployer Azure File Sync](storage-sync-files-deployment-guide.md)
- [Résoudre les problèmes d’Azure File Sync](storage-sync-files-troubleshoot.md)
- [Forum Aux Questions Azure Files](storage-files-faq.md)
