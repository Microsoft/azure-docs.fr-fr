---
title: Glossaire de Monitor pour Windows Virtual Desktop (préversion) – Azure
description: Glossaire des termes et des concepts liés à Azure Monitor pour Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f13909d3835bdbd2931277a88244abfae3f80759
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465490"
---
# <a name="azure-monitor-for-windows-virtual-desktop-preview-glossary"></a>Glossaire d’Azure Monitor pour Windows Virtual Desktop (préversion)

>[!IMPORTANT]
>Azure Monitor pour Windows Virtual Desktop est en préversion publique. Cette préversion est fournie sans contrat de niveau de service, c’est pourquoi nous déconseillons son utilisation pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Dans cet article figurent la liste et une brève description des termes et concepts clés relatifs à Azure Monitor pour Windows Virtual Desktop (préversion).

## <a name="alerts"></a>Alertes

Toutes les alertes Azure Monitor actives configurées sur l’abonnement et classées avec la [gravité 1](#severity-1-alerts) s’affichent sur la page Vue d’ensemble. Pour savoir comment configurer des alertes, consultez [Réponse aux événements avec les alertes Azure Monitor](../azure-monitor/learn/tutorial-response.md).

## <a name="available-sessions"></a>Sessions disponibles

Nombre de sessions disponibles dans le pool d’hôtes. Pour le calculer, le service multiplie le nombre de machines virtuelles par le nombre maximal de sessions autorisées par machine virtuelle, puis soustrait le nombre total de sessions.

## <a name="connection-success"></a>Réussite de la connexion

Intégrité de la connexion. « Connexion réussie » signifie que la connexion a pu atteindre l’hôte, comme le confirme la pile sur cette machine virtuelle. Un échec de connexion indique qu’elle n’est pas parvenue à atteindre l’hôte.

## <a name="daily-active-users-dau"></a>Utilisateurs actifs par jour

Nombre total d’utilisateurs ayant commencé une session au cours des dernières 24 heures.

## <a name="daily-alerts"></a>Alertes quotidiennes

Nombre total [d’alertes de gravité 1](#severity-1-alerts) déclenchées au cours des dernières 24 heures.

## <a name="daily-connections-and-reconnections"></a>Connexions et reconnexions quotidiennes

Nombre total de connexions et de reconnexions commencées ou terminées au cours des dernières 24 heures.

## <a name="daily-connected-hours"></a>Heures connectées quotidiennes

Nombre total d’heures passées par les utilisateurs à se connecter à une session au cours des dernières 24 heures.

## <a name="diagnostics-and-errors"></a>Diagnostics et erreurs

Lorsqu’une erreur ou une alerte apparaît dans Azure Monitor pour Windows Virtual Desktop, elle est catégorisée selon trois éléments :

- Type d’activité : cette catégorie indique comment l’erreur est catégorisée par les diagnostics Windows Virtual Desktop. Les catégories sont les activités de gestion, les flux, les connexions, les inscriptions des hôtes, les erreurs et les points de contrôle. Pour plus d’informations sur ces catégories, consultez [Utilisation de Log Analytics pour la fonctionnalité de diagnostic](diagnostics-log-analytics.md).

- Genre : cette catégorie indique l’emplacement de l’erreur. 

     - Les erreurs signalées comme « service » ou « ServiceError = TRUE » se sont produites dans le service Windows Virtual Desktop.
     - Les erreurs marquées « deployment » ou « ServiceError = FALSE » sont survenues en dehors du service Windows Virtual Desktop.
     - Pour plus d’informations sur la balise ServiceError, consultez [Scénarios d’erreur courants](diagnostics-role-service.md#common-error-scenarios).

- Source : cette catégorie donne une description plus précise de l’endroit où l’erreur s’est produite.

     - Diagnostics : rôle de service responsable de l’activité de service de monitoring et de compte rendu permettant aux utilisateurs d’observer et de diagnostiquer les problèmes de déploiement.

     - RDBroker : rôle de service responsable de l’orchestration des activités de déploiement, de la gestion de l’état des objets, de la validation de l’authentification, etc.

     - RDGateway : rôle de service responsable de la gestion de la connectivité réseau entre les utilisateurs finaux et les machines virtuelles.

     - RDStack : composant logiciel installé sur les machines virtuelles pour leur permettre de communiquer avec le service Windows Virtual Desktop.

     - Client : logiciel s’exécutant sur l’ordinateur de l’utilisateur final qui fournit l’interface du service Windows Virtual Desktop. Il affiche la liste des ressources publiées et héberge la connexion Bureau à distance une fois que vous avez effectué une sélection.

Chaque problème ou erreur de diagnostic comprend un message expliquant la cause du problème. Pour plus d’informations sur la résolution des erreurs, consultez [Identification et diagnostic des problèmes liés à Windows Virtual Desktop](diagnostics-role-service.md).

## <a name="input-delay"></a>Délai d’entrée

Dans Azure Monitor pour Windows Virtual Desktop, délai d’entrée par compteur de performances de processus pour chaque session. Sur la page du niveau de performance de l’hôte, à <aka.ms/azmonwvdi>, ce compteur de performances est configuré pour envoyer un rapport au service toutes les 30 secondes. Ces intervalles de 30 secondes sont appelés « échantillons », et le rapport indique le pire scénario dans cette fenêtre. La valeur médiane et la valeur P95 reflètent la valeur médiane et le 95e percentile pour tous les exemples.

Sous **Délai d’entrée par hôte**, vous pouvez sélectionner une ligne d’hôte de session pour filtrer tous les autres visuels de la page sur cet hôte. Vous avez également la possibilité de sélectionner un nom de processus pour filtrer le graphe des délais d’entrée médians dans le temps.

Les délais sont placés dans les catégories suivantes :

- Bon : inférieur ou égal à 150 millisecondes
- Acceptable : entre 150 et 500 millisecondes
- Médiocre : entre 500 et 2 000 millisecondes (inférieur à 2 secondes)
- Mauvais : supérieur ou égal à 2 000 millisecondes (2 secondes)

Pour plus d’informations sur le fonctionnement du compteur de délai d’entrée, consultez [Compteurs de performances du délai d’entrée utilisateur](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

##  <a name="monthly-active-users-mau"></a>Utilisateurs actifs mensuels (MAU)

Nombre total d’utilisateurs ayant commencé une session au cours des 28 derniers jours. Si vous stockez des données pendant 30 jours maximum, vous risquez de constater des valeurs Utilisateurs actifs mensuels et Connexion moins élevées que prévu pendant les périodes sur lesquelles moins de 28 jours de données sont disponibles.

## <a name="performance-counters"></a>Compteurs de performance

Sur Windows et Linux, informations sur le niveau de performance des composants matériels, des systèmes d’exploitation et des applications.

Dans le tableau suivant figure la liste des compteurs de performances recommandés et des intervalles de temps appliqués par Azure Monitor pour Windows Virtual Desktop :

|Nom du compteur de performance|Intervalle de temps|
|---|---|
|Logical Disk(C:)\\Avg. Longueur de file d'attente de disque|30 secondes|
|Logical Disk(C:)\\Avg. Disk sec/Transfer|60 secondes|
|Logical Disk(C:)\\Current Disk Queue Length|30 secondes|
|Mémoire(\*)\\Mo disponibles|30 secondes|
|Memory(\*)\\Page Faults/sec|30 secondes|
|Memory(\*)\\Pages/sec|30 secondes|
|Memory(\*)\\% Committed Bytes in Use|30 secondes|
|PhysicalDisk(\*)\\Avg. Longueur de file d'attente de disque|30 secondes|
|PhysicalDisk(\*)\\Avg. Disk sec/Read|30 secondes|
|PhysicalDisk(\*)\\Avg. Disk sec/Transfer|30 secondes|
|PhysicalDisk(\*)\\Avg. Disk sec/Write|30 secondes|
|Process(\*)\\% Processor Time|20 secondes.|
|Process(\*)\\% User Time|30 secondes|
|Process(\*)\\Thread Count|30 secondes|
|Process(\*)\\ IO Write Operations/sec|30 secondes|
|Process(\*)\\ IO Read Operations/sec|30 secondes|
|Processor Information(_Total)\\% Processor Time|30 secondes|
|Terminal Services(\*)\\Active Sessions|60 secondes|
|Terminal Services(\*)\\Inactive Sessions|60 secondes|
|Terminal Services(\*)\\Total Sessions|60 secondes|
|\*User Input Delay per Process(\*)\\Max Input Delay|30 secondes|
|\*User Input Delay per Session(\*)\\Max Input Delay|30 secondes|
|RemoteFX Network(\*)\\Current TCP RTT|30 secondes|
|RemoteFX Network(\*)\\Current UDP Bandwidth|30 secondes|

Pour savoir comment lire les compteurs de performances, consultez [Configuration des compteurs de performances](../azure-monitor/platform/data-sources-performance-counters.md).

Pour plus d’informations sur les compteurs de performances de délai d’entrée, consultez [Compteurs de performances de délai d’entrée utilisateur](/windows-server/remote/remote-desktop-services/rds-rdsh-performance-counters/).

## <a name="potential-connectivity-issues"></a>Problèmes potentiels de connectivité

Hôtes, utilisateurs, ressources publiées et clients affichant un taux d’échec de connexion élevé. Une fois que vous avez choisi un filtre « Reporting selon », vous pouvez évaluer la gravité du problème en consultant les valeurs des colonnes suivantes :

- Tentatives (nombre de tentatives de connexion)
- Ressources (nombre d’applications ou de postes de travail publiés)
- Hôtes (nombre de machines virtuelles)
- Clients

Par exemple, si vous sélectionnez le filtre **Par utilisateur**, vous pouvez vérifier que les tentatives de connexion de chaque utilisateur s’affichent dans la colonne **Tentatives**.

Si vous remarquez qu’un problème de connexion s’étend sur plusieurs hôtes, utilisateurs, ressources ou clients, il est probable que le problème affecte l’ensemble du système. Dans le cas contraire, il s’agit d’un petit problème de priorité plus basse.

Il est également possible de sélectionner des entrées pour afficher des informations supplémentaires. Vous pouvez voir quels hôtes, quelles ressources et quelles versions du client sont impliqués dans le problème. L’affichage indique également les erreurs signalées lors des tentatives de connexion.

## <a name="round-trip-time-rtt"></a>Temps d’aller-retour

Estimation du temps d’aller-retour de la connexion entre l’emplacement de l’utilisateur final et la région Azure de la machine virtuelle. Pour voir quels emplacements présentent la meilleure latence, recherchez celui de votre choix dans [l’outil estimateur d’expérience Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/assessment/).

## <a name="session-history"></a>Historique de session

L’élément **Sessions** indique l’état de toutes les sessions, connectées ou non. Les **Sessions inactives** affichent uniquement les sessions déconnectées.

## <a name="severity-1-alerts"></a>Alertes de gravité 1

Éléments les plus urgents à prendre en charge immédiatement. Si vous ne résolvez pas ces problèmes, votre déploiement Windows Virtual Desktop risque de ne plus fonctionner.

## <a name="time-to-connect"></a>Temps pour se connecter

Délai entre le moment où un utilisateur commence sa session et celui où il est comptabilisé comme connecté au service. L’établissement de nouvelles connexions a tendance à prendre plus de temps que le rétablissement de connexions existantes.

## <a name="user-report"></a>Rapport utilisateur

La page Rapport utilisateur permet d’afficher l’historique des connexions et les informations de diagnostic d’un utilisateur spécifique. Chaque rapport utilisateur présente les modèles d’utilisation, les commentaires utilisateurs et les erreurs rencontrées par les utilisateurs lors de leurs sessions. La plupart des petits problèmes peuvent être résolus avec les commentaires des utilisateurs. Si vous devez aller plus loin, vous pouvez également filtrer les informations relatives à un ID de connexion ou à une période spécifique.

## <a name="users-per-core"></a>Utilisateurs par cœur

Nombre d’utilisateurs présents dans chaque noyau de machine virtuelle. Le suivi du nombre maximal d’utilisateurs par cœur au fil du temps peut permettre de déterminer si l’environnement s’exécute toujours avec un nombre élevé, faible ou fluctuant d’utilisateurs par cœur. Connaître le nombre d’utilisateurs actifs vous aidera à mettre efficacement l’environnement à l’échelle et à lui affecter les ressources nécessaires.

## <a name="windows-events"></a>Événements Windows

Les journaux des événements Windows sont des sources de données collectées par les agents Log Analytics sur des machines virtuelles Windows. Vous pouvez collecter des événements à partir de journaux standard comme Système et Application, ainsi que des journaux personnalisés créés par les applications que vous souhaitez surveiller.

Dans le tableau suivant figure la liste des événements Windows requis par Azure Monitor pour Windows Virtual Desktop :

|Nom d'événement|Type d'événement|
|---|---|
|Application|Erreur et avertissement|
|Microsoft-Windows-TerminalServices-RemoteConnectionManager/Admin|Erreur, Avertissement et Information|
|Microsoft-Windows-TerminalServices-LocalSessionManager/Operational|Erreur, Avertissement et Information|
|Système|Erreur et avertissement|
| Microsoft-FSLogix-Apps/Operational|Erreur, Avertissement et Information|
|Microsoft-FSLogix-Apps/Admin|Erreur, Avertissement et Information|

Pour plus d’informations sur les événements Windows, consultez [Propriétés des enregistrements des événements Windows](../azure-monitor/platform/data-sources-windows-events.md).

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec Azure Monitor pour Windows Virtual Desktop, consultez les articles suivants :

- [Monitoring d’un déploiement avec Azure Monitor pour Windows Virtual Desktop](azure-monitor.md)
- [Résolution des problèmes liés à Azure Monitor pour Windows Virtual Desktop](troubleshoot-azure-monitor.md)

Vous pouvez également configurer Azure Advisor pour savoir comment résoudre ou éviter les problèmes courants. Pour plus d’informations, consultez [Utilisation d’Azure Advisor avec Windows Virtual Desktop](azure-advisor.md).

Si vous avez besoin d’aide ou si vous avez des questions, consultez les ressources de la communauté :

- Posez des questions ou faites des suggestions à la communauté sur [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).
   
- Pour savoir comment laisser des commentaires, consultez [Vue d’ensemble de la résolution des problèmes, des commentaires et du support pour Windows Virtual Desktop](troubleshoot-set-up-overview.md#report-issues).

- Vous pouvez également laisser vos commentaires sur Windows Virtual Desktop sur le [Hub de commentaires Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app) ou notre [forum UserVoice](https://windowsvirtualdesktop.uservoice.com/forums/921118-general).
