---
title: Analytique des journaux d'activité Windows Virtual Desktop - Azure
description: Comment utiliser l’analytique des journaux d’activité avec la fonctionnalité de diagnostic de Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 05/27/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 5d8b696b175c4ef841eef1a51f1d357d1781cba7
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018288"
---
# <a name="use-log-analytics-for-the-diagnostics-feature"></a>Utiliser Log Analytics pour la fonctionnalité de diagnostic

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/diagnostics-log-analytics-2019.md).

Windows Virtual Desktop utilise [Azure Monitor](../azure-monitor/overview.md) pour la supervision et les alertes, comme de nombreux autres services Azure. Les administrateurs peuvent ainsi identifier les problèmes par le biais d’une interface unique. Le service crée des journaux d’activité pour les actions des utilisateurs et des administrateurs. Chaque journal d’activité se divise en les catégories suivantes :

- Activités de gestion :
    - Effectue le suivi de la réussite des tentatives de modification des objets Windows Virtual Desktop à l’aide d’API ou de PowerShell. Par exemple, une personne peut-elle créer un pool d’hôtes avec PowerShell ?
- Flux :
    - Les utilisateurs parviennent-ils à s’abonner aux espaces de travail ?
    - Les utilisateurs voient-ils toutes les ressources publiées dans le client Bureau à distance ?
- Connexions :
    - Moment auquel les utilisateurs lancent et finalisent les connexions au service.
- Inscription de l’hôte :
    - L’hôte de session a-t-il été correctement inscrit auprès du service lors de la connexion ?
- Erreurs :
    - Les utilisateurs rencontrent-ils des problèmes avec des activités spécifiques ? Cette fonctionnalité peut générer une table qui effectue le suivi des données d’activité tant que les informations sont jointes aux activités.
- Points de contrôle :
    - Étapes spécifiques dans la durée de vie d’une activité qui ont été atteintes. Par exemple, au cours d’une session, un utilisateur a fait l’objet d’un équilibrage de charge sur un hôte particulier, puis a été connecté pendant une connexion, et ainsi de suite.

Les connexions qui n’atteignent pas Windows Virtual Desktop ne figureront pas dans les résultats de diagnostic, car le service de rôle de diagnostics fait partie de Windows Virtual Desktop. Des problèmes de connexion à Windows Virtual Desktop peuvent survenir lorsque l’utilisateur rencontre des problèmes de connectivité au réseau.

Azure Monitor vous permet d’analyser les données Windows Virtual Desktop et de consulter les compteurs de performances des machines virtuelles, le tout au sein du même outil. Cet article met l’accent sur l’activation des diagnostics pour votre environnement Windows Virtual Desktop.

>[!NOTE]
>Pour savoir comment superviser vos machines virtuelles dans Azure, consultez [Supervision de machines virtuelles Azure avec Azure Monitor](../azure-monitor/insights/monitor-vm-azure.md). Veillez également à [passer en revue les seuils des compteurs de performances](../virtual-desktop/virtual-desktop-fall-2019/deploy-diagnostics.md#windows-performance-counter-thresholds) pour mieux comprendre votre expérience utilisateur sur l’hôte de session.

## <a name="before-you-get-started"></a>Avant de commencer

Avant de pouvoir utiliser Log Analytics, vous devez créer un espace de travail. Pour ce faire, suivez les instructions de l’un des deux articles suivants :

- Si vous préférez utiliser le portail Azure, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../azure-monitor/learn/quick-create-workspace.md).
- Si vous préférez PowerShell, consultez [Créer un espace de travail Log Analytics avec PowerShell](../azure-monitor/platform/powershell-workspace-configuration.md).

Après avoir créé votre espace de travail, suivez les instructions de la section [Connecter des ordinateurs Windows à Azure Monitor](../azure-monitor/platform/log-analytics-agent.md#workspace-id-and-key) pour obtenir les informations suivantes :

- L’ID de l’espace de travail
- La clé primaire de votre espace de travail

Ces informations vous seront nécessaires plus loin dans le processus de configuration.

Veillez à passer en revue la gestion des autorisations pour Azure Monitor afin d’activer l’accès aux données pour les personnes qui supervisent et gèrent votre environnement Windows Virtual Desktop. Pour plus d’informations, consultez [Prise en main des rôles, des autorisations et de la sécurité dans Azure Monitor](../azure-monitor/platform/roles-permissions-security.md).

## <a name="push-diagnostics-data-to-your-workspace"></a>Envoyer (push) les données de diagnostic à votre espace de travail

Vous pouvez envoyer (push) les données de diagnostic de vos objets Windows Virtual Desktop à Log Analytics pour votre espace de travail. Vous pouvez configurer cette fonctionnalité immédiatement au moment où vous créez vos objets.

Pour configurer Log Analytics pour un nouvel objet :

1. Connectez-vous au portail Azure et accédez à **Windows Virtual Desktop**.

2. Accédez à l’objet (par exemple, un pool d’hôtes, un groupe d’applications ou un espace de travail) pour lequel vous souhaitez capturer des journaux et des événements.

3. Sélectionnez **Paramètres de diagnostic** dans le menu sur le côté gauche de l’écran.

4. Sélectionnez **Ajouter un paramètre de diagnostic** dans le menu sur le côté gauche de l’écran.

    Les options affichées dans la page Paramètres de diagnostic varient selon le type d’objet que vous modifiez.

    Par exemple, quand vous activez les diagnostics pour un groupe d’applications, vous voyez les options permettant de configurer les points de contrôle, les erreurs et la gestion. Dans le cas des espaces de travail, ces catégories configurent un flux pour suivre le moment auquel les utilisateurs s’abonnent à la liste des applications. Pour en savoir plus sur les paramètres de diagnostic, consultez [Créer un paramètre de diagnostic pour collecter les journaux et les métriques des ressources dans Azure](../azure-monitor/platform/diagnostic-settings.md).

     >[!IMPORTANT]
     >N’oubliez pas d’activer les diagnostics pour chaque objet Azure Resource Manager que vous souhaitez superviser. Les données sont disponibles pour les activités une fois les diagnostics activés. La première configuration peut prendre quelques heures.

5. Entrez un nom pour la configuration de vos paramètres, puis sélectionnez **Envoyer à Log Analytics**. Le nom que vous utilisez ne doit pas contenir d’espaces et doit être conforme aux [conventions de nommage Azure](../azure-resource-manager/management/resource-name-rules.md). Dans le cadre des journaux, vous pouvez sélectionner toutes les options que vous souhaitez ajouter à votre analytique des journaux, telles que les points de contrôle, les erreurs, la gestion, etc.

6. Sélectionnez **Enregistrer**.

>[!NOTE]
>Log Analytics vous donne la possibilité d’effectuer un streaming des données vers [Event Hubs](../event-hubs/event-hubs-about.md) ou de les archiver dans un compte de stockage. Pour en savoir plus sur cette fonctionnalité, consultez [Envoyer en streaming des données de supervision Azure vers un hub d’événements](../azure-monitor/platform/stream-monitoring-data-event-hubs.md) et [Archiver des journaux de ressource Azure dans un compte de stockage](../azure-monitor/platform/resource-logs.md#send-to-azure-storage).

## <a name="how-to-access-log-analytics"></a>Comment accéder à Log Analytics

Vous pouvez accéder aux espaces de travail Log Analytics sur le portail Azure ou sur Azure Monitor.

### <a name="access-log-analytics-on-a-log-analytics-workspace"></a>Accéder à Log Analytics sur un espace de travail Log Analytics

1. Connectez-vous au portail Azure.

2. Recherchez **Espace de travail Log Analytics**.

3. Sous Services, sélectionnez **Espaces de travail Log Analytics**.

4. Dans la liste, sélectionnez l’espace de travail que vous avez configuré pour votre objet Windows Virtual Desktop.

5. Une fois dans votre espace de travail, sélectionnez **Journaux**. Vous pouvez filtrer votre liste de menu avec la fonction **Rechercher**.

### <a name="access-log-analytics-on-azure-monitor"></a>Accéder à Log Analytics sur Azure Monitor

1. Se connecter au portail Azure

2. Recherchez et sélectionnez **Surveiller**.

3. Sélectionnez **Journaux d’activité**.

4. Suivez les instructions de la page de journalisation pour définir l’étendue de votre requête.

5. Vous êtes prêt à interroger les diagnostics. Toutes les tables de diagnostics ont un préfixe « WVD ».

>[!NOTE]
>Pour plus d’informations sur les tables stockées dans les journaux d’activité d’Azure Monitor, consultez la [documentation de référence sur les données d’Azure Monitor](/azure/azure-monitor/reference/). Toutes les tables relatives à Windows Virtual Desktop sont étiquetées « WVD ».

## <a name="cadence-for-sending-diagnostic-events"></a>Cadence d’envoi des événements de diagnostic

Les événements de diagnostic sont envoyés à Log Analytics lorsqu’ils sont terminés.

Les rapports Log Analytics ne portent que sur les états intermédiaires suivants des activités de connexion :

- Démarré : lorsqu’un utilisateur sélectionne et se connecte à une application ou à un ordinateur de bureau dans le client Bureau à distance.
- Connecté : lorsque l’utilisateur se connecte avec succès à la machine virtuelle sur laquelle l’application ou le bureau est hébergé.
- Terminé : lorsque l’utilisateur ou le serveur déconnecte la session dans laquelle l’activité a eu lieu.

## <a name="example-queries"></a>Exemples de requêtes

Accédez à des exemples de requêtes par le biais de l’interface utilisateur d’Azure Monitor Log Analytics :
1. Accédez à votre espace de travail Log Analytics, puis sélectionnez **Journaux**. L’interface utilisateur de l’exemple de requête s’affiche automatiquement.
1. Changez le filtre en **Catégorie**.
1. Sélectionnez **Windows Virtual Desktop** pour examiner les requêtes disponibles.
1. Sélectionnez **Exécuter** pour exécuter la requête sélectionnée.

Pour plus d’informations sur l’interface de l’exemple de requête, consultez [Requêtes enregistrées dans Azure Monitor Log Analytics](../azure-monitor/log-query/example-queries.md).

La liste de requêtes suivante vous permet de consulter les informations ou les problèmes de connexion pour un seul utilisateur. Vous pouvez exécuter ces requêtes dans l’[éditeur de requête Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md#write-a-query). Pour chaque requête, remplacez `userupn` par l’UPN de l’utilisateur que vous souhaitez rechercher.


Pour rechercher toutes les connexions d’un seul utilisateur :

```kusto
WVDConnections
|where UserName == "userupn"
|take 100
|sort by TimeGenerated asc, CorrelationId
```


Pour déterminer le nombre de fois qu’un utilisateur s’est connecté par jour :

```kusto
WVDConnections
|where UserName == "userupn"
|take 100
|sort by TimeGenerated asc, CorrelationId
|summarize dcount(CorrelationId) by bin(TimeGenerated, 1d)
```

Pour rechercher la durée de session par utilisateur :

```kusto
let Events = WVDConnections | where UserName == "userupn" ;
Events
| where State == "Connected"
| project CorrelationId , UserName, ResourceAlias , StartTime=TimeGenerated
| join (Events
| where State == "Completed"
| project EndTime=TimeGenerated, CorrelationId)
on CorrelationId
| project Duration = EndTime - StartTime, ResourceAlias
| sort by Duration asc
```

Pour rechercher les erreurs liées à un utilisateur spécifique :

```kusto
WVDErrors
| where UserName == "userupn"
|take 100
```

Pour déterminer si une erreur spécifique s’est produite pour d’autres utilisateurs :

```kusto
WVDErrors
| where CodeSymbolic =="ErrorSymbolicCode"
| summarize count(UserName) by CodeSymbolic
```


>[!NOTE]
>- Quand un utilisateur ouvre le Bureau complet, l’utilisation de son application dans la session n’est pas suivie sous la forme de points de contrôle dans la table WVDCheckpoints.
>- La colonne ResourcesAlias de la table WVDConnections indique si un utilisateur s’est connecté à un bureau complet ou à une application publiée. La colonne affiche uniquement la première application ouverte pendant la connexion. Toutes les applications publiées que l’utilisateur ouvre sont suivies dans WVDCheckpoints.
>- La table WVDErrors affiche les erreurs de gestion, les problèmes d’inscription de l’hôte et d’autres problèmes qui se produisent lorsque l’utilisateur s’abonne à une liste d’applications ou de bureaux.
>- WVDErrors vous permet d’identifier les problèmes qui peuvent être résolus par des tâches d’administration. La valeur sur ServiceError indique toujours « false » pour ces types de problèmes. Si ServiceError a pour valeur « true », vous devez faire remonter le problème à Microsoft. Vérifiez que vous fournissez le CorrelationID pour les erreurs que vous escaladez.

## <a name="next-steps"></a>Étapes suivantes

Pour examiner les scénarios d’erreur courants que la fonctionnalité de diagnostic peut identifier pour vous, consulter [Identifier et diagnostiquer les problèmes](diagnostics-role-service.md#common-error-scenarios).