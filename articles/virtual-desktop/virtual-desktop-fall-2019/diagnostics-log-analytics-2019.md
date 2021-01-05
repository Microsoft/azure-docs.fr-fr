---
title: Analytique des journaux de diagnostic Windows Virtual Desktop (classique) – Azure
description: Comment utiliser l’analytique des journaux d’activité avec la fonctionnalité de diagnostic de Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 257656a29733972672570648d4317b7847fedff1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88997674"
---
# <a name="use-log-analytics-for-the-diagnostics-feature-in-windows-virtual-desktop-classic"></a>Utiliser Log Analytics pour la fonctionnalité de diagnostic dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../diagnostics-log-analytics.md).

Windows Virtual Desktop offre une fonctionnalité de diagnostic qui permet à l’administrateur d’identifier les problèmes via une seule interface. Cette fonctionnalité consigne les informations de diagnostic chaque fois qu'une personne à qui un rôle Windows Virtual Desktop a été attribué utilise le service. Chaque journal contient des informations sur le rôle Windows Virtual Desktop impliqué dans l'activité, les éventuels messages d'erreur qui apparaissent pendant la session, les informations sur le locataire et les informations sur l'utilisateur. La fonctionnalité de diagnostic crée des journaux d'activité pour les actions des utilisateurs et des administrateurs. Chaque journal d'activités se divise en trois grandes catégories :

- Activités d’ajout de flux : lorsqu’un utilisateur essaie de se connecter à son flux via des applications Bureau à distance Microsoft.
- Activités de connexion : lorsqu’un utilisateur essaie de se connecter à un ordinateur de bureau ou à RemoteApp via des applications Bureau à distance Microsoft.
- Activités de gestion : lorsqu’un administrateur effectue des opérations de gestion sur le système, telles que la création de pools d’hôte, l’attribution d’utilisateurs à des groupes d’applications et la création d’attributions de rôles.

Les connexions qui n’atteignent pas Windows Virtual Desktop ne figureront pas dans les résultats de diagnostic, car le service de rôle de diagnostics fait partie de Windows Virtual Desktop. Des problèmes de connexion à Windows Virtual Desktop peuvent survenir lorsque l’utilisateur rencontre des problèmes de connectivité au réseau.

## <a name="why-you-should-use-log-analytics"></a>Pourquoi vous devriez utiliser Log Analytics

Nous vous recommandons d'utiliser Log Analytics pour analyser les données de diagnostic dans le client Azure qui vont au-delà du dépannage par un utilisateur unique. Étant donné que vous pouvez extraire des compteurs de performances des machines virtuelles dans Log Analytics, vous disposez d'un outil unique pour recueillir des informations en vue de votre déploiement.

## <a name="before-you-get-started"></a>Avant de commencer

Avant de pouvoir utiliser Log Analytics avec la fonctionnalité de diagnostic, vous devrez [créer un espace de travail](../../azure-monitor/learn/quick-collect-windows-computer.md#create-a-workspace).

Après avoir créé votre espace de travail, suivez les instructions de la section [Connecter des ordinateurs Windows à Azure Monitor](../../azure-monitor/platform/log-analytics-agent.md#workspace-id-and-key) pour obtenir les informations suivantes :

- L’ID de l’espace de travail
- La clé primaire de votre espace de travail

Ces informations vous seront nécessaires plus loin dans le processus de configuration.

## <a name="push-diagnostics-data-to-your-workspace"></a>Envoyer (push) les données de diagnostic à votre espace de travail

Vous pouvez envoyer (push) les données de diagnostic de votre locataire Windows Virtual Desktop à Log Analytics pour votre espace de travail. Vous pouvez configurer cette fonctionnalité dès la création de votre locataire en liant votre espace de travail à votre locataire, ou vous pouvez la configurer plus tard avec un locataire existant.

Pour lier votre locataire à votre espace de travail Log Analytics pendant que vous configurez votre nouveau locataire, exécutez l’applet de commande suivante pour vous connecter à Windows Virtual Desktop avec votre compte utilisateur TenantCreator :

```powershell
Add-RdsAccount -DeploymentUrl https://rdbroker.wvd.microsoft.com
```

Si vous voulez lier un locataire existant plutôt qu'un nouveau locataire, exécutez cette applet de commande :

```powershell
Set-RdsTenant -Name <TenantName> -AzureSubscriptionId <SubscriptionID> -LogAnalyticsWorkspaceId <String> -LogAnalyticsPrimaryKey <String>
```

Vous aurez besoin d'exécuter ces applets de commande pour chaque locataire que vous voulez lier à Log Analytics.

>[!NOTE]
>Si vous ne voulez pas lier l'espace de travail Log Analytics lorsque vous créez un locataire, exécutez plutôt l’applet de commande `New-RdsTenant`.

## <a name="cadence-for-sending-diagnostic-events"></a>Cadence d’envoi des événements de diagnostic

Les événements de diagnostic sont envoyés à Log Analytics lorsqu’ils sont terminés.

## <a name="example-queries"></a>Exemples de requêtes

Les exemples de requêtes suivants montrent comment la fonctionnalité de diagnostic génère un rapport pour les activités les plus fréquentes dans votre système :

Ce premier exemple montre les activités de connexion initiées par des utilisateurs disposant de clients Bureau à distance pris en charge :

```powershell
WVDActivityV1_CL

| where Type_s == "Connection"

| join kind=leftouter (

    WVDErrorV1_CL

    | summarize Errors = makelist(pack('Time', Time_t, 'Code', ErrorCode_s , 'CodeSymbolic', ErrorCodeSymbolic_s, 'Message', ErrorMessage_s, 'ReportedBy', ReportedBy_s , 'Internal', ErrorInternal_s )) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g 

| join  kind=leftouter (

    WVDCheckpointV1_CL

    | summarize Checkpoints = makelist(pack('Time', Time_t, 'ReportedBy', ReportedBy_s, 'Name', Name_s, 'Parameters', Parameters_s) ) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g

|project-away ActivityId_g, ActivityId_g1
```

L'exemple de requête suivant montre les activités de gestion par les administrateurs sur les locataires :

```powershell
WVDActivityV1_CL

| where Type_s == "Management"

| join kind=leftouter (

    WVDErrorV1_CL

    | summarize Errors = makelist(pack('Time', Time_t, 'Code', ErrorCode_s , 'CodeSymbolic', ErrorCodeSymbolic_s, 'Message', ErrorMessage_s, 'ReportedBy', ReportedBy_s , 'Internal', ErrorInternal_s )) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g 

| join  kind=leftouter (

    WVDCheckpointV1_CL

    | summarize Checkpoints = makelist(pack('Time', Time_t, 'ReportedBy', ReportedBy_s, 'Name', Name_s, 'Parameters', Parameters_s) ) by ActivityId_g

    ) on $left.Id_g  == $right.ActivityId_g

|project-away ActivityId_g, ActivityId_g1
```

## <a name="stop-sending-data-to-log-analytics"></a>Arrêter d'envoyer des données à Log Analytics

Pour arrêter l'envoi des données d'un locataire existant à Log Analytics, exécutez l’applet de commande suivante et définissez des chaînes vides :

```powershell
Set-RdsTenant -Name <TenantName> -AzureSubscriptionId <SubscriptionID> -LogAnalyticsWorkspaceId <String> -LogAnalyticsPrimaryKey <String>
```

Vous devrez exécuter cette applet de commande pour chaque locataire pour lequel vous voulez arrêter l’envoi des données.

## <a name="next-steps"></a>Étapes suivantes

Pour examiner les scénarios d’erreur courants que la fonctionnalité de diagnostic peut identifier pour vous, consulter [Identifier et diagnostiquer les problèmes](diagnostics-role-service-2019.md#common-error-scenarios).
