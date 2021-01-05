---
title: Surveiller la protection par mot de passe Azure AD localement
description: Découvrez comment surveiller la protection par mot de passe Azure AD dans un environnement Active Directory Domain Services local et en consulter les journaux d’activité
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: edc246a414401c4c1c0248787eda0381fcd63037
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741760"
---
# <a name="monitor-and-review-logs-for-on-premises-azure-ad-password-protection-environments"></a>Surveiller les environnements de protection par mot de passe Azure AD locaux et en consulter les journaux d’activité

Après le déploiement de la protection par mot de passe Azure AD, la supervision et la génération de rapports sont des tâches essentielles. Cet article détaillé vous aide à comprendre plusieurs techniques de supervision, notamment en vous expliquant où chaque service journalise les informations et en vous montrant comment créer des rapports sur l’utilisation de la protection par mot de passe Azure AD.

La surveillance et la création de rapports sont effectuées soit par des messages du journal des événements, soit en exécutant des cmdlets PowerShell. L'agent DC et les services proxy consignent tous deux les messages du journal des événements. Toutes les cmdlets PowerShell décrites ci-dessous sont uniquement disponibles sur le serveur proxy (voir le module AzureADPasswordProtection PowerShell). Le logiciel de l'agent DC n'installe pas de module PowerShell.

## <a name="dc-agent-event-logging"></a>Journalisation des événements de l’agent de contrôleur de domaine (DC)

Sur chaque contrôleur de domaine, le logiciel du service d’agent DC écrit les résultats de chacune des opérations de validation de mot de passe (et un autre état) dans un journal d’événements local :

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Admin`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Operational`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Trace`

Le journal d’administration de l’agent DC est la principale source d’informations pour superviser le comportement du logiciel.

Notez que le journal des traces est désactivé par défaut.

Les événements enregistrés par les différents composants de l’agent DC appartiennent aux plages suivantes :

|Composant |Plage d’ID d’événement|
| --- | --- |
|DLL de filtre de mot de passe de l’agent DC| 10000-19999|
|Processus d’hébergement du service d’agent DC| 20000-29999|
|Logique de validation de stratégie du service d’agent DC| 30000-39999|

## <a name="dc-agent-admin-event-log"></a>Journal des événements d’administration de l’agent DC

### <a name="password-validation-outcome-events"></a>Événements des résultats d’opérations de validation de mot de passe

Sur chaque contrôleur de domaine, le logiciel du service d’agent DC écrit les résultats de chacune des opérations de validation de mot de passe dans le journal des événements d’administration de l’agent DC.

Pour une opération de validation de mot de passe réussie, un événement est généralement consigné à partir de la DLL de filtre de mot de passe de l’agent DC. Pour une opération de validation de mot de passe qui a échoué, deux événements sont généralement consignés, un à partir du service d’agent DC et un à partir de la DLL de filtre de mot de passe de l’agent DC.

Les événements discrets pour capturer ces situations sont enregistrés en fonction des facteurs suivants :

* Si un mot de passe donné est défini ou modifié.
* Indique si la validation de mot de passe a réussi ou échoué.
* Si la validation a échoué à cause de la stratégie globale de Microsoft, de la stratégie de l’organisation ou de ces stratégies combinées.
* Si mode d’audit seul est actuellement activé ou désactivé pour la stratégie de mot de passe en vigueur.

Les principaux événements relatifs à la validation de mot de passe sont les suivants :

| Événement |Modification de mot de passe |Définition de mot de passe|
| --- | :---: | :---: |
|Réussite |10014 |10015|
|Échec (à cause de la stratégie de mot de passe client)| 10016, 30002| 10017, 30003|
|Échec (à cause de la stratégie de mot de passe Microsoft)| 10016, 30004| 10017, 30005|
|Échec (à cause des stratégies de mot de passe combinées Microsoft et client)| 10016, 30026| 10017, 30027|
|Échec (à cause du nom d'utilisateur)| 10016, 30021| 10017, 30022|
|Réussite d’audit seul (aurait échoué avec la stratégie de mot de passe client)| 10024, 30008| 10025, 30007|
|Réussite d’audit seul (aurait échoué avec la stratégie de mot de passe Microsoft)| 10024, 30010| 10025, 30009|
|Réussite d’audit seul (aurait échoué avec les stratégies de mot de passe combinées Microsoft et client)| 10024, 30028| 10025, 30029|
|Réussite - Mode Audit uniquement (aurait échoué à cause du nom d'utilisateur)| 10016, 30024| 10017, 30023|

Les exemples du tableau ci-dessus qui mentionnent des « stratégies combinées » font référence à des situations où un mot de passe utilisateur comporte au moins un jeton provenant à la fois de la liste des mots de passe interdits de Microsoft et de la liste des mots de passe interdits du client.

Les exemples du tableau ci-dessus qui font référence au « nom d'utilisateur » concernent les situations dans lesquelles le mot de passe d'un utilisateur contient le nom de son compte et/ou l'un de ses noms conviviaux. Dans les deux cas, le mot de passe de l'utilisateur est rejeté si la stratégie est définie sur Appliquer, ou accepté si la stratégie est en mode Audit.

Quand deux événements sont consignés ensemble, ils sont explicitement associés par le même CorrelationId.

### <a name="password-validation-summary-reporting-via-powershell"></a>Récapitulatif des opérations de validation de mot de passe avec PowerShell

L’applet de commande `Get-AzureADPasswordProtectionSummaryReport` permet d’obtenir un récapitulatif des opérations de validation de mot de passe. Voici un exemple de sortie de cette cmdlet :

```powershell
Get-AzureADPasswordProtectionSummaryReport -DomainController bplrootdc2
DomainController                : bplrootdc2
PasswordChangesValidated        : 6677
PasswordSetsValidated           : 9
PasswordChangesRejected         : 10868
PasswordSetsRejected            : 34
PasswordChangeAuditOnlyFailures : 213
PasswordSetAuditOnlyFailures    : 3
PasswordChangeErrors            : 0
PasswordSetErrors               : 1
```

L’étendue du rapport de la cmdlet peut être influencée à l’aide d’un des paramètres -Forest, -Domain ou -DomainController. Ne pas spécifier un paramètre équivaut à –Forest.

L’applet de commande `Get-AzureADPasswordProtectionSummaryReport` demande le journal des événements d’administration de l’agent DC, puis compte le nombre total d’événements dans chaque catégorie de résultats affichés. Le tableau suivant indique à quel ID d’événement correspond chaque résultat :

|Propriété Get-AzureADPasswordProtectionSummaryReport |ID d’événement correspondant|
| :---: | :---: |
|PasswordChangesValidated |10014|
|PasswordSetsValidated |10015|
|PasswordChangesRejected |10016|
|PasswordSetsRejected |10017|
|PasswordChangeAuditOnlyFailures |10024|
|PasswordSetAuditOnlyFailures |10025|
|PasswordChangeErrors |10012|
|PasswordSetErrors |10013|

Notez que l’applet de commande `Get-AzureADPasswordProtectionSummaryReport` est fournie sous forme de script PowerShell et qu’elle peut si nécessaire être référencée directement à l’emplacement suivant :

`%ProgramFiles%\WindowsPowerShell\Modules\AzureADPasswordProtection\Get-AzureADPasswordProtectionSummaryReport.ps1`

> [!NOTE]
> Cette cmdlet fonctionne en ouvrant une session PowerShell sur chaque contrôleur de domaine. Pour réussir, le support de la session à distance PowerShell doit être activé sur chaque contrôleur de domaine et le client doit disposer de privilèges suffisants. Pour plus d’informations sur les exigences de session à distance PowerShell, exécutez « Get-Help about_Remote_Troubleshooting» dans une fenêtre PowerShell.

> [!NOTE]
> Cette cmdlet fonctionne en interrogeant à distance le journal des événements administrateur de chaque agent de service DC. Si les journaux des événements contiennent un grand nombre d’événements, la cmdlet peut prendre beaucoup de temps à se faire. En outre, les requêtes de réseau en bloc de grands jeux de données peuvent affecter les performances du contrôleur de domaine. Par conséquent, cette cmdlet doit être utilisée avec précaution dans les environnements de production.

### <a name="sample-event-log-message-for-event-id-10014-successful-password-change"></a>Exemple de message de journal des événements pour l’ID d’événement 10014 (mot de passe changé)

```text
The changed password for the specified user was validated as compliant with the current Azure password policy.

 UserName: BPL_02885102771
 FullName:
```

### <a name="sample-event-log-message-for-event-id-10017-and-30003-failed-password-set"></a>Exemple de message de journal des événements pour les ID d’événement 10017 et 30003 (mot de passe non défini)

10017 :

```text
The reset password for the specified user was rejected because it did not comply with the current Azure password policy. Please see the correlated event log message for more details.

 UserName: BPL_03283841185
 FullName:
```

30003 :

```text
The reset password for the specified user was rejected because it matched at least one of the tokens present in the per-tenant banned password list of the current Azure password policy.

 UserName: BPL_03283841185
 FullName:
```

### <a name="sample-event-log-message-for-event-id-30001-password-accepted-due-to-no-policy-available"></a>Exemple de message de journal des événements pour l’ID d’événement 30001 (mot de passe accepté, aucune stratégie disponible)

```text
The password for the specified user was accepted because an Azure password policy is not available yet

UserName: SomeUser
FullName: Some User

This condition may be caused by one or more of the following reasons:%n

1. The forest has not yet been registered with Azure.

   Resolution steps: an administrator must register the forest using the Register-AzureADPasswordProtectionForest cmdlet.

2. An Azure AD password protection Proxy is not yet available on at least one machine in the current forest.

   Resolution steps: an administrator must install and register a proxy using the Register-AzureADPasswordProtectionProxy cmdlet.

3. This DC does not have network connectivity to any Azure AD password protection Proxy instances.

   Resolution steps: ensure network connectivity exists to at least one Azure AD password protection Proxy instance.

4. This DC does not have connectivity to other domain controllers in the domain.

   Resolution steps: ensure network connectivity exists to the domain.
```

### <a name="sample-event-log-message-for-event-id-30006-new-policy-being-enforced"></a>Exemple de message de journal des événements pour l’ID d’événement 30006 (nouvelle stratégie appliquée)

```text
The service is now enforcing the following Azure password policy.

 Enabled: 1
 AuditOnly: 1
 Global policy date: ‎2018‎-‎05‎-‎15T00:00:00.000000000Z
 Tenant policy date: ‎2018‎-‎06‎-‎10T20:15:24.432457600Z
 Enforce tenant policy: 1
```

### <a name="sample-event-log-message-for-event-id-30019-azure-ad-password-protection-is-disabled"></a>Exemple de message de journal des événements pour l’ID d’événement 30019 (protection par mot de passe Azure AD désactivée)

```text
The most recently obtained Azure password policy was configured to be disabled. All passwords submitted for validation from this point on will automatically be considered compliant with no processing performed.

No further events will be logged until the policy is changed.%n

```

## <a name="dc-agent-operational-log"></a>Journal des opérations de l’agent DC

Le service d’agent DC enregistre également les événements opérationnels dans le journal suivant :

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Operational`

## <a name="dc-agent-trace-log"></a>Journal de suivi de l’agent DC

Le service d’agent DC peut également enregistrer les événements de suivi au niveau du débogage détaillé dans le journal suivant :

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\DCAgent\Trace`

Le journal de trace est désactivé par défaut.

> [!WARNING]
> Lorsqu’il est activé, le journal de suivi reçoit un volume élevé d’événements et peut influer sur les performances du contrôleur de domaine. Par conséquent, ce journal amélioré doit uniquement être activé lorsqu’un problème nécessite un examen plus approfondi, puis seulement pendant une courte période.

## <a name="dc-agent-text-logging"></a>Journalisation de texte de l’agent DC

Le service d’agent DC peut être configuré pour écrire dans un journal de texte en définissant la valeur de Registre suivante :

```text
HKLM\System\CurrentControlSet\Services\AzureADPasswordProtectionDCAgent\Parameters!EnableTextLogging = 1 (REG_DWORD value)
```

Le journal texte est désactivé par défaut. Un redémarrage du service d’agent DC est nécessaire pour que les modifications de cette valeur entrent en vigueur. Lorsqu’il est activé, le service d’agent DC, il écrit dans un fichier journal situé dans :

`%ProgramFiles%\Azure AD Password Protection DC Agent\Logs`

> [!TIP]
> Le journal texte reçoit les mêmes entrées de niveau débogage que celles qui peuvent être consignées dans le journal de trace, mais sont format est généralement plus facile à consulter et à analyser.

> [!WARNING]
> Lorsqu’il est activé, ce journal reçoit un volume élevé d’événements et peut influer sur les performances du contrôleur de domaine. Par conséquent, ce journal amélioré doit uniquement être activé lorsqu’un problème nécessite un examen plus approfondi, puis seulement pendant une courte période.

## <a name="dc-agent-performance-monitoring"></a>Supervision des performances de l’agent DC

Le logiciel de service d’agent DC installe un objet de compteur de performances appelé **protection de mot de passe Azure AD**. Les compteurs de performances suivants sont actuellement disponibles :

|Nom de compteur de performances | Description|
| --- | --- |
|Mots de passe traités |Ce compteur affiche le nombre total de mots de passe traités (acceptés ou rejetés) depuis le dernier redémarrage.|
|Mots de passe acceptés |Ce compteur affiche le nombre total de mots de passe acceptés depuis le dernier redémarrage.|
|Mots de passe rejetés |Ce compteur affiche le nombre total de mots de passe rejetés depuis le dernier redémarrage.|
|Requêtes de filtre de mot de passe en cours |Ce compteur affiche le nombre de requêtes de filtre de mot de passe en cours d’exécution.|
|Pic de requêtes de filtre de mot de passe |Ce compteur affiche le nombre maximal de requêtes de filtre de mot de passe simultanées depuis le dernier redémarrage.|
|Erreurs de requête de filtre de mot de passe |Ce compteur affiche le nombre total de requêtes de filtre de mot de passe qui ont échoué en raison d’une erreur depuis le dernier redémarrage. Des erreurs peuvent se produire quand le service d’agent DC de protection par mot de passe Azure AD n’est pas exécuté.|
|Requêtes de filtre de mot de passe/seconde |Ce compteur affiche la vitesse à laquelle les mots de passe sont traités.|
|Temps de traitement des requêtes de filtre de mot de passe |Ce compteur affiche le temps moyen nécessaire pour traiter une requête de filtre de mot de passe.|
|Temps maximal de traitement des requêtes de filtre de mot de passe |Ce compteur affiche le temps de traitement maximal de requêtes de filtre de mot de passe depuis le dernier redémarrage.|
|Mots de passe acceptés en raison du mode d’audit |Ce compteur affiche le nombre total de mots de passe qui aurait normalement été rejetés, mais qui ont été acceptés, car la stratégie de mot de passe a été configurée en mode d’audit (depuis le dernier redémarrage).|

## <a name="dc-agent-discovery"></a>Détection d’agent DC

La cmdlet `Get-AzureADPasswordProtectionDCAgent` peut être utilisée pour afficher des informations de base sur les divers agents DC en cours d’exécution dans un domaine ou une forêt. Ces informations sont extraites des objets serviceConnectionPoint inscrits par les services d’agent DC en cours d’exécution.

Voici un exemple de sortie de cette cmdlet :

```powershell
Get-AzureADPasswordProtectionDCAgent
ServerFQDN            : bplChildDC2.bplchild.bplRootDomain.com
Domain                : bplchild.bplRootDomain.com
Forest                : bplRootDomain.com
PasswordPolicyDateUTC : 2/16/2018 8:35:01 AM
HeartbeatUTC          : 2/16/2018 8:35:02 AM
```

Les différentes propriétés sont mises à jour par chaque service de l’agent DC environ toutes les heures. Les données sont toujours soumises à la latence de la réplication Active Directory.

L’étendue de la requête de la cmdlet peut être influencée à l’aide des paramètres -Forest ou -Domain.

Si la valeur HeartbeatUTC est périmée, cela peut indiquer que l’agent DC de la protection par mot de passe Azure AD sur ce contrôleur de domaine n’est pas actuellement exécuté ou qu’il a été désinstallé, ou encore que la machine a été rétrogradée et n’a plus le rôle de contrôleur de domaine.

Si la valeur PasswordPolicyDateUTC est périmée, cela peut indiquer que l’agent DC de la protection par mot de passe Azure AD sur cette machine ne fonctionne pas correctement.

## <a name="dc-agent-newer-version-available"></a>Version plus récente de l’agent DC disponible

Le service de l’agent DC enregistre un événement d’avertissement 30034 dans le journal des opérations lors de la détection d’une version plus récente du logiciel de l’agent DC, par exemple :

```text
An update for Azure AD Password Protection DC Agent is available.

If autoupgrade is enabled, this message may be ignored.

If autoupgrade is disabled, refer to the following link for the latest version available:

https://aka.ms/AzureADPasswordProtectionAgentSoftwareVersions

Current version: 1.2.116.0
```

L’événement ci-dessus ne spécifie pas la version du logiciel le plus récent. Vous devez accéder au lien dans le message d’événement pour obtenir ces informations.

> [!NOTE]
> Malgré les références à « AutoUpgrade » dans le message d’événement ci-dessus, le logiciel de l’agent DC ne prend pas en charge cette fonctionnalité pour le moment.

## <a name="proxy-service-event-logging"></a>Journalisation des événements du service Proxy

Le service Proxy émet un jeu minimal d’événements pour les journaux d’événements suivants :

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Admin`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Operational`

`\Applications and Services Logs\Microsoft\AzureADPasswordProtection\ProxyService\Trace`

Notez que le journal des traces est désactivé par défaut.

> [!WARNING]
> Lorsqu’il est activé, ce journal de suivi reçoit un volume élevé d’événements et peut influer sur les performances de l’hôte proxy. Par conséquent, ce journal doit uniquement être activé lorsqu’un problème nécessite un examen plus approfondi, puis seulement pendant une courte période.

Les événements sont enregistrés par les différents composants du service Proxy à l’aide des plages suivantes :

|Composant |Plage d’ID d’événement|
| --- | --- |
|Processus d’hébergement du service Proxy| 10000-19999|
|Logique métier principale du service proxy| 20000-29999|
|Applets de commande PowerShell| 30000-39999|

## <a name="proxy-service-text-logging"></a>Journalisation de type texte du service Proxy

Le service Proxy peut être configuré pour écrire dans un journal de texte en définissant la valeur de Registre suivante :

HKLM\System\CurrentControlSet\Services\AzureADPasswordProtectionProxy\Parameters!EnableTextLogging = 1 (REG_DWORD value)

Le journal texte est désactivé par défaut. Un redémarrage du service Proxy est nécessaire pour que les modifications de cette valeur entrent en vigueur. Lorsqu’il est activé, le service Proxy, il écrit dans un fichier journal situé dans :

`%ProgramFiles%\Azure AD Password Protection Proxy\Logs`

> [!TIP]
> Le journal texte reçoit les mêmes entrées de niveau débogage que celles qui peuvent être consignées dans le journal de trace, mais sont format est généralement plus facile à consulter et à analyser.

> [!WARNING]
> Quand il est activé, ce journal reçoit un volume élevé d’événements, ce qui peut impacter les performances de la machine. Par conséquent, ce journal amélioré doit uniquement être activé lorsqu’un problème nécessite un examen plus approfondi, puis seulement pendant une courte période.

## <a name="powershell-cmdlet-logging"></a>Journalisation des applets de commande PowerShell

Les applets de commande PowerShell qui connaissent un changement d’état (par exemple, Register-AzureADPasswordProtectionProxy) consignent normalement un événement de résultat dans le journal des opérations.

De plus, la plupart des applets de commande PowerShell de la protection par mot de passe Azure AD consignent un événement dans un journal texte sous :

`%ProgramFiles%\Azure AD Password Protection Proxy\Logs`

En cas d’erreur du cmdlet et si solution et/ou la cause n’est pas évidente, ces journaux d’activité texte peuvent servir de référence.

## <a name="proxy-discovery"></a>Détection de services Proxy

L’applet de commande `Get-AzureADPasswordProtectionProxy` permet d’afficher des informations de base sur les différents services Proxy de protection par mot de passe Azure AD qui sont actuellement exécutés dans un domaine ou une forêt. Ces informations sont extraites des objets serviceConnectionPoint inscrits par les services Proxy en cours d’exécution.

Voici un exemple de sortie de cette cmdlet :

```powershell
Get-AzureADPasswordProtectionProxy
ServerFQDN            : bplProxy.bplchild2.bplRootDomain.com
Domain                : bplchild2.bplRootDomain.com
Forest                : bplRootDomain.com
HeartbeatUTC          : 12/25/2018 6:35:02 AM
```

Les différentes propriétés sont mises à jour par chaque service Proxy environ toutes les heures. Les données sont toujours soumises à la latence de la réplication Active Directory.

L’étendue de la requête de la cmdlet peut être influencée à l’aide des paramètres -Forest ou -Domain.

Si la valeur HeartbeatUTC est périmée, cela peut indiquer que le service Proxy de la protection par mot de passe Azure AD sur cette machine n’est pas actuellement exécuté ou qu’il a été désinstallé.

## <a name="proxy-agent-newer-version-available"></a>Version plus récente de l’agent proxy disponible

Le service proxy enregistre un événement d’avertissement 20002 dans le journal des opérations lors de la détection d’une version plus récente du logiciel proxy, par exemple :

```text
An update for Azure AD Password Protection Proxy is available.

If autoupgrade is enabled, this message may be ignored.

If autoupgrade is disabled, refer to the following link for the latest version available:

https://aka.ms/AzureADPasswordProtectionAgentSoftwareVersions

Current version: 1.2.116.0
.
```

L’événement ci-dessus ne spécifie pas la version du logiciel le plus récent. Vous devez accéder au lien dans le message d’événement pour obtenir ces informations.

Cet événement est émis même si l’agent proxy est configuré avec la mise à jour automatique activée.

## <a name="next-steps"></a>Étapes suivantes

[Résoudre les problèmes de la protection par mot de passe Azure AD](howto-password-ban-bad-on-premises-troubleshoot.md)

Pour plus d’informations sur les listes de mots de passe interdits globales et personnalisées, consultez l’article [Éliminer les mots de passe interdits](concept-password-ban-bad.md).
