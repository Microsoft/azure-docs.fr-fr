---
title: 'Synchronisation d’Azure AD Connect : Planificateur | Microsoft Docs'
description: Cette rubrique décrit la fonctionnalité de planificateur intégré dans Azure AD Connect Sync.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 6b1a598f-89c0-4244-9b20-f4aaad5233cf
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/01/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: ad7b0039602add7f4cd3cdd300bd829c4f148a79
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90084734"
---
# <a name="azure-ad-connect-sync-scheduler"></a>Synchronisation d’Azure AD Connect : Scheduler
Cette rubrique décrit le planificateur intégré dans Azure AD Connect Sync (moteur de synchronisation).

Cette fonctionnalité a été introduite avec la version 1.1.105.0 (publiée en février 2016).

## <a name="overview"></a>Vue d’ensemble
La synchronisation Azure AD Connect synchronise les modifications dans votre répertoire local à l’aide d’un planificateur. Il existe deux processus de planificateur, l’un pour la synchronisation de mot de passe et l’autre pour la synchronisation d’attribut/d’objet, ainsi que des tâches de maintenance. Cette rubrique couvre ce dernier.

Dans les versions antérieures, le planificateur utilisé pour les objets et attributs était externe au moteur de synchronisation. Il utilisait le Planificateur de tâches Windows ou un service Windows distinct pour déclencher le processus de synchronisation. Le planificateur de la version 1.1 est intégré au moteur de synchronisation et permet de personnaliser certains aspects. La nouvelle valeur de la fréquence de synchronisation par défaut est de 30 minutes.

Le planificateur supervise deux tâches :

* **Cycle de synchronisation**. Processus d’importation, de synchronisation et d’exportation des modifications.
* **Tâches de maintenance**. Renouvelle les clés et les certificats pour la réinitialisation de mot de passe et le service DRS (Device Registration Service). Vide les anciennes entrées dans le journal des opérations.

Le planificateur est toujours en cours d’exécution, mais il peut être configuré pour exécuter uniquement une tâche, ou bien aucune tâche. Par exemple, si vous avez besoin d’avoir votre propre processus de cycle de synchronisation, vous pouvez désactiver cette tâche dans le planificateur, mais toujours exécuter la tâche de maintenance.

>[!IMPORTANT]
>Par défaut, un cycle de synchronisation est lancé toutes les 30 minutes. Si vous avez changé le cycle de synchronisation, vous devez vous assurer qu’un cycle de synchronisation est lancé au moins une fois tous les 7 jours. 
>
>* Une nouvelle synchronisation delta est requise dans les 7 jours suivant la dernière synchronisation delta.
>* Une synchronisation delta (après une synchronisation complète) doit être effectuée dans les 7 jours suivant la dernière synchronisation complète.
>
>À défaut, vous risquez de rencontrer des problèmes de synchronisation, qui ne pourront être résolus qu’en effectuant une synchronisation complète. Cela s’applique également aux serveurs en préproduction.

## <a name="scheduler-configuration"></a>Configuration du planificateur
Pour afficher vos paramètres de configuration en cours, accédez à PowerShell et exécutez `Get-ADSyncScheduler`. Vous obtenez un écran semblable à celui-ci :

![GetSyncScheduler](./media/how-to-connect-sync-feature-scheduler/getsynccyclesettings2016.png)

Si le message **La commande de synchronisation ou l’applet de commande n’est pas disponible** apparaît lorsque vous exécutez cette applet de commande, le module PowerShell n'est pas chargé. Ce problème peut se produire si vous exécutez Azure AD Connect sur un contrôleur de domaine ou sur un serveur avec des niveaux de restriction PowerShell plus élevés que les paramètres par défaut. Si vous recevez cette erreur, exécutez `Import-Module ADSync` pour libérer l’applet de commande.

* **AllowedSyncCycleInterval**. L’intervalle de temps le plus court entre les cycles de synchronisation autorisé par Azure AD. Vous ne pouvez pas synchroniser plus fréquemment que ne le spécifie ce paramètre tout en maintenant la prise en charge.
* **CurrentlyEffectiveSyncCycleInterval**. Planificateur actuellement en vigueur. La valeur est identique à celle de CustomizedSyncInterval (si ce paramètre est défini) si la fréquence n’est pas supérieure à AllowedSyncInterval. Si vous utilisez une version antérieure à 1.1.281 et que vous modifiez CustomizedSyncCycleInterval, cette modification prendra effet après le prochain cycle de synchronisation. À partir de la version 1.1.281, la modification prend effet immédiatement.
* **CustomizedSyncCycleInterval**. Si vous souhaitez que le planificateur s’exécute à une fréquence autre que celle par défaut de 30 minutes, vous devez configurer ce paramètre. Dans l’image ci-dessus, le planificateur a été défini pour s’exécuter toutes les heures. Si vous choisissez une valeur inférieure à AllowedSyncInterval, ce dernier sera utilisé.
* **NextSyncCyclePolicyType**. Delta ou Initial. Indique si la prochaine exécution doit uniquement traiter les modifications delta du processus ou si elle doit effectuer une importation et une synchronisation intégrales, ce qui aura pour effet de retraiter les nouvelles règles ou les règles modifiées.
* **NextSyncCycleStartTimeInUTC**. Heure à laquelle le planificateur démarrera le prochain cycle de synchronisation.
* **PurgeRunHistoryInterval**. Durée pendant laquelle les journaux d’activité des opérations doivent être conservés. Ces journaux d’activité peuvent être consultés dans Synchronization Service Manager. La valeur de conservation des journaux d’activité par défaut est de 7 jours.
* **SyncCycleEnabled**. Indique si le planificateur exécute les processus d’importation, de synchronisation et d’exportation dans le cadre de son fonctionnement.
* **MaintenanceEnabled**. Indique si le processus de maintenance est activé. Met à jour les certificats/clés et vide le journal des opérations.
* **StagingModeEnabled**. Indique si le [mode intermédiaire](how-to-connect-sync-staging-server.md) est activé. Si ce paramètre est activé, les exportations ne sont plus exécutées. Cependant, l’importation et la synchronisation sont toujours exécutées.
* **SchedulerSuspended**. Défini par Connect pendant une mise à niveau pour bloquer temporairement le l’exécution du planificateur.

Vous pouvez modifier certains de ces paramètres avec `Set-ADSyncScheduler`. Les paramètres suivants peuvent être modifiés :

* CustomizedSyncCycleInterval
* NextSyncCyclePolicyType
* PurgeRunHistoryInterval
* SyncCycleEnabled
* MaintenanceEnabled

Dans les versions antérieures d’Azure AD Connect, **isStagingModeEnabled** était exposé dans Set-ADSyncScheduler. Elle **n’est pas prise en charge** pour la définition de cette propriété. La propriété **SchedulerSuspended** doit uniquement être modifiée par Connect. Elle **n’est pas prise en charge** pour la définition avec PowerShell directement.

La configuration du planificateur est stockée dans Azure AD. Si vous avez un serveur intermédiaire, toute modification sur le serveur principal a également un effet sur le serveur intermédiaire (à l’exception de IsStagingModeEnabled).

### <a name="customizedsynccycleinterval"></a>CustomizedSyncCycleInterval
Syntaxe : `Set-ADSyncScheduler -CustomizedSyncCycleInterval d.HH:mm:ss`  
d - jours, HH - heures, mm - minutes, ss - secondes

Exemple : `Set-ADSyncScheduler -CustomizedSyncCycleInterval 03:00:00`  
Modifie le planificateur pour qu’il s’exécute toutes les 3 heures.

Exemple : `Set-ADSyncScheduler -CustomizedSyncCycleInterval 1.0:0:0`  
Modifie le planificateur pour qu’il s’exécute tous les jours.

### <a name="disable-the-scheduler"></a>Désactivation du planificateur  
Si vous devez apporter des modifications de configuration, vous devez désactiver le planificateur. Par exemple, lorsque vous [configurez le filtrage](how-to-connect-sync-configure-filtering.md) ou [apportez des modifications aux règles de synchronisation](how-to-connect-sync-change-the-configuration.md).

Exécutez `Set-ADSyncScheduler -SyncCycleEnabled $false` pour désactiver le planificateur.

![Désactivation du planificateur](./media/how-to-connect-sync-feature-scheduler/schedulerdisable.png)

Une fois vos modifications apportées, n’oubliez pas d’activer de nouveau le planificateur en exécutant `Set-ADSyncScheduler -SyncCycleEnabled $true`.

## <a name="start-the-scheduler"></a>Démarrer le planificateur
Par défaut, le planificateur s’exécute toutes les 30 minutes. Dans certains cas, vous souhaiterez peut-être exécuter un cycle de synchronisation entre les cycles planifiés ou vous devrez exécuter un autre type de synchronisation.

### <a name="delta-sync-cycle"></a>Cycle de synchronisation delta
Un cycle de synchronisation delta comprend les étapes suivantes :


- Importation delta sur tous les connecteurs
- Synchronisation delta sur tous les connecteurs
- Exportation sur tous les connecteurs

### <a name="full-sync-cycle"></a>Cycle de synchronisation complète
Un cycle de synchronisation complète comprend les étapes suivantes :

- Importation complète sur tous les connecteurs
- Synchronisation complète sur tous les connecteurs
- Exportation sur tous les connecteurs

Cela peut indiquer une modification urgente à synchroniser immédiatement, nécessitant l’exécution manuelle d’un cycle. 

Si nécessaire, vous pouvez exécuter manuellement un cycle de synchronisation en exécutant `Start-ADSyncSyncCycle -PolicyType Delta` à partir de PowerShell.

Pour initier un cycle de synchronisation complète, exécutez `Start-ADSyncSyncCycle -PolicyType Initial` à partir d’une invite PowerShell.   

L'exécution d'un cycle de synchronisation complète peut prendre beaucoup de temps. Reportez-vous à la section suivante pour savoir comment optimiser ce processus.

### <a name="sync-steps-required-for-different-configuration-changes"></a>Étapes de synchronisation requises en fonction des modifications apportées à la configuration
Selon les modifications apportées à la configuration, différentes étapes de synchronisation sont nécessaires pour veiller à ce que les modifications soient correctement appliquées à tous les objets.

- Ajout de plusieurs objets ou attributs à importer à partir d'un répertoire source (en ajoutant/modifiant des règles de synchronisation)
    - Une importation complète est requise sur le connecteur pour ce répertoire source
- Modifications apportées aux règles de synchronisation
    - Une synchronisation complète est requise sur le connecteur pour les règles de synchronisation modifiées
- Le [filtrage](how-to-connect-sync-configure-filtering.md) étant modifié, un nombre différent d’objets doit être inclus
    - Une importation complète est requise sur le connecteur pour chaque connecteur AD, SAUF si vous utilisez un filtrage basé sur les attributs déjà importés dans le moteur de synchronisation

### <a name="customizing-a-sync-cycle-run-the-right-mix-of-delta-and-full-sync-steps"></a>La personnalisation d'un cycle de synchronisation exécute la bonne combinaison d'étapes de synchronisation delta et de synchronisation complète
Pour éviter d'exécuter un cycle de synchronisation complète, vous pouvez indiquer à certains connecteurs d'exécuter une étape Complète à l'aide des cmdlets suivantes.

`Set-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid> -FullImportRequired $true`

`Set-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid> -FullSyncRequired $true`

`Get-ADSyncSchedulerConnectorOverride -Connector <ConnectorGuid>` 

Exemple :  Si vous avez apporté des modifications aux règles de synchronisation du connecteur « Forêt AD A » et qu'il n'est pas nécessaire d'importer de nouveaux attributs, utilisez les cmdlets suivantes pour exécuter un cycle de synchronisation delta ayant également suivi une étape Synchronisation complète pour ce connecteur.

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullSyncRequired $true`

`Start-ADSyncSyncCycle -PolicyType Delta`

Exemple :  Si vous avez apporté des modifications aux règles de synchronisation du connecteur « Forêt AD A » et qu'il est désormais nécessaire d'importer un nouvel attribut, utilisez les cmdlets suivantes pour exécuter un cycle de synchronisation delta ayant également suivi une étape Importation complète, Synchronisation complète pour ce connecteur.

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullImportRequired $true`

`Set-ADSyncSchedulerConnectorOverride -ConnectorName “AD Forest A” -FullSyncRequired $true`

`Start-ADSyncSyncCycle -PolicyType Delta`


## <a name="stop-the-scheduler"></a>Arrêter le planificateur
Si le planificateur exécute un cycle de synchronisation, vous devrez l’arrêter. Par exemple, si vous démarrez l’Assistant d’installation et que vous obtenez l’erreur suivante :

![Capture d’écran montrant le message d’erreur « Impossible de modifier la configuration ».](./media/how-to-connect-sync-feature-scheduler/synccyclerunningerror.png)

Lorsqu’un cycle de synchronisation est en cours d’exécution, vous ne pouvez pas modifier la configuration. Vous pouvez attendre jusqu’à ce que le planificateur ait terminé le processus, mais vous pouvez également l’arrêter afin d’implémenter immédiatement vos modifications. L’arrêt du cycle en cours n’est pas dangereux, et les modifications en attente seront traitées à la prochaine exécution.

1. Commencez par indiquer au planificateur d’arrêter son cycle en cours à l’aide de l’applet de commande PowerShell `Stop-ADSyncSyncCycle`.
2. Si vous utilisez une version antérieure à 1.1.281, l’arrêt du planificateur n’empêche pas le connecteur actuel de terminer sa tâche en cours. Pour forcer le connecteur à s’arrêter, prenez les mesures suivantes : 

   ![Capture d’écran montrant Synchronization Service Manager avec l’option Connecteurs sélectionnée et un connecteur en cours d’exécution mis en évidence avec l’action Arrêter sélectionnée.](./media/how-to-connect-sync-feature-scheduler/stopaconnector.png)

   * Démarrez le **Service de synchronisation** depuis le menu Démarrer. Accédez à **Connecteurs**, mettez en surbrillance le connecteur avec l’état **En cours d’exécution** et sélectionnez **Arrêter** dans les actions.

Le planificateur est toujours actif et redémarrera à la prochaine occasion.

## <a name="custom-scheduler"></a>Planificateur personnalisé
Les applets de commande décrites dans cette section sont uniquement disponibles dans le build [1.1.130.0](reference-connect-version-history.md) et versions ultérieures.

Si le planificateur intégré ne satisfait pas à vos exigences, vous pouvez planifier les connecteurs à l'aide de PowerShell.

### <a name="invoke-adsyncrunprofile"></a>Invoke-ADSyncRunProfile
Vous pouvez démarrer un profil pour un connecteur de cette façon :

```
Invoke-ADSyncRunProfile -ConnectorName "name of connector" -RunProfileName "name of profile"
```

Les noms à utiliser pour [Noms de connecteur](how-to-connect-sync-service-manager-ui-connectors.md) et [Exécuter les noms de profil](how-to-connect-sync-service-manager-ui-connectors.md#configure-run-profiles) se trouvent dans [l’interface utilisateur Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md).

![Appeler le profil d’exécution](./media/how-to-connect-sync-feature-scheduler/invokerunprofile.png)  

L’applet de commande `Invoke-ADSyncRunProfile` est synchrone, c’est-à-dire qu’elle ne retournera pas de contrôle avant que le connecteur ait terminé l’opération, avec succès ou avec une erreur.

Lorsque vous planifiez vos connecteurs, il est recommandé de le faire dans l'ordre suivant :

1. (Intégral/Delta) Importation à partir de répertoires locaux, tels qu'Active Directory
2. (Intégral/Delta) Importation à partir d'Azure AD
3. (Intégral/Delta) Synchronisation à partir de répertoires locaux, tels qu'Active Directory
4. (Intégral/Delta) Synchronisation à partir d'Azure AD
5. Exportation vers Azure AD
6. Exportation vers des répertoires locaux, tels qu'Active Directory

Il s’agit de l’ordre dans lequel le planificateur intégré exécute les connecteurs.

### <a name="get-adsyncconnectorrunstatus"></a>Get-ADSyncConnectorRunStatus
Vous pouvez également surveiller le moteur de synchronisation pour voir s'il est occupé ou inactif. Cette applet de commande renvoie un résultat vide si le moteur de synchronisation est inactif et qu’il n’exécute pas un connecteur. Si un connecteur est en cours d’exécution, il retourne le nom du connecteur.

```
Get-ADSyncConnectorRunStatus
```

![État d’exécution du connecteur](./media/how-to-connect-sync-feature-scheduler/getconnectorrunstatus.png)  
Dans l'illustration ci-dessus, la première ligne est dans un état où le moteur de synchronisation est inactif. La deuxième ligne lorsque le Connecteur Azure AD est en cours d'exécution.

## <a name="scheduler-and-installation-wizard"></a>Planificateur et Assistant d’installation
Si vous démarrez l’Assistant d’installation, le planificateur est temporairement interrompu. Ce comportement repose sur l’hypothèse que vous apportez des modifications de configuration et le fait que ces paramètres ne peuvent pas être appliqués si le moteur de synchronisation s’exécute activement. Pour cette raison, ne laissez pas l’Assistant d’installation ouvert, car il empêche le moteur de synchronisation d’effectuer toutes les actions de synchronisation.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la configuration de la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md) .

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
