---
title: 'Azure AD Connect : Effectuer une mise à niveau depuis une version précédente | Microsoft Docs'
description: Explique les différentes méthodes de mise à niveau vers la dernière version d’Azure Active Directory Connect, y compris la mise à niveau sur place et la migration « swing ».
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 31f084d8-2b89-478c-9079-76cf92e6618f
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 04/08/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: b78d3cab17b0cc4085c824cf35d4c6037f0e2af5
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91319858"
---
# <a name="azure-ad-connect-upgrade-from-a-previous-version-to-the-latest"></a>Azure AD Connect : Effectuer une mise à niveau depuis une version précédente vers la dernière
Cette rubrique décrit les différentes méthodes que vous pouvez utiliser pour mettre à niveau votre installation Azure Active Directory (Azure AD) Connect vers la dernière version. Nous vous recommandons d’utiliser systématiquement les dernières versions d’Azure AD Connect. Vous pouvez également suivre les étapes décrites dans la section [Migration « swing »](#swing-migration) lorsque vous appliquez une modification importante à la configuration.

>[!NOTE]
> Il est actuellement pris en charge pour la mise à niveau de n’importe quelle version d’Azure AD Connect vers la version actuelle. Les mises à niveau en place de DirSync ou ADSync ne sont pas prises en charge et une migration swing est nécessaire.  Si vous souhaitez effectuer une mise à niveau à partir de DirSync, consultez [Effectuer une mise à niveau à partir de l’outil de synchronisation Azure AD (DirSync)](how-to-dirsync-upgrade-get-started.md) ou la section [Migration « swing »](#swing-migration).  </br>Dans la pratique, les clients qui utilisent des versions extrêmement anciennes peuvent rencontrer des problèmes qui ne sont pas directement liés à Azure AD Connect. Les serveurs qui ont été en production pendant plusieurs années, ont généralement eu plusieurs correctifs qui leur ont été appliqués et qui ne peuvent pas tous être comptabilisés.  En général, les clients qui n’ont pas effectué de mise à niveau au bout de 12 à 18 mois devraient plutôt envisager une mise à niveau rapide, car c’est l’option la plus prudente et la moins risquée.

Si vous souhaitez effectuer une mise à niveau à partir de DirSync, consultez plutôt [Effectuer une mise à niveau à partir de l’outil de synchronisation Azure AD (DirSync)](how-to-dirsync-upgrade-get-started.md) .

Il existe plusieurs stratégies que vous pouvez utiliser pour mettre à niveau Azure AD Connect.

| Méthode | Description |
| --- | --- |
| [Mise à niveau automatique](how-to-connect-install-automatic-upgrade.md) |Pour les clients avec une installation rapide, il s’agit de la méthode la plus simple. |
| [Mise à niveau sur place](#in-place-upgrade) |Si vous avez un seul serveur, vous pouvez mettre à niveau l’installation sur place sur le même serveur. |
| [Migration « swing »](#swing-migration) |Avec deux serveurs, vous pouvez en préparer un avec la nouvelle version ou configuration, et changer de serveur actif quand vous êtes prêt. |

Pour plus d’informations sur les autorisations, reportez-vous aux [autorisations requises pour une mise à niveau](reference-connect-accounts-permissions.md#upgrade).

> [!NOTE]
> Une fois que vous avez activé votre nouveau serveur Azure AD Connect pour démarrer la synchronisation des modifications avec Azure AD, vous ne devez pas revenir à l’utilisation de DirSync ou Azure AD Sync. La rétrogradation à partir d’Azure AD Connect vers des clients hérités, notamment DirSync et Azure AD Sync, n’est pas prise en charge et peut entraîner des problèmes tels que la perte de données dans Azure AD.

## <a name="in-place-upgrade"></a>Mise à niveau sur place
Une mise à niveau sur place fonctionne à partir d’Azure AD Sync ou d’Azure AD Connect, Elle ne fonctionne pas pour la migration à partir de DirSync ou pour une solution avec Forefront Identity Manager (FIM) + connecteur Azure AD.

Nous vous recommandons cette méthode si vous avez un seul serveur et moins de 100 000 objets environ. Après la mise à niveau, une importation et une synchronisation complètes se produisent si des modifications ont été apportées aux règles de synchronisation out-of-box, pour que la nouvelle configuration s’applique à tous les objets existants dans le système. Cette opération peut prendre plusieurs heures selon le nombre d’objets figurant dans l’étendue du moteur de synchronisation. La synchronisation différentielle normale planifiée (par défaut toutes les 30 minutes) est suspendue, mais la synchronisation des mots de passe continue. Vous pouvez envisager une mise à niveau sur place pendant le week-end. S’il n’y a aucune modification de la configuration par défaut avec la nouvelle version d’Azure AD Connect, une importation/synchronisation différentielle normale démarre à la place.  
![Mise à niveau sur place](./media/how-to-upgrade-previous-version/inplaceupgrade.png)

Si vous avez apporté des modifications aux règles de synchronisation par défaut, celles-ci retrouvent leur configuration par défaut au moment de la mise à niveau. Pour conserver votre configuration d’une mise à niveau à l’autre, vérifiez que les modifications sont effectuées conformément aux [meilleures pratiques pour modifier la configuration par défaut](how-to-connect-sync-best-practices-changing-default-configuration.md).

Pendant la mise à niveau sur place, des modifications introduites peuvent nécessiter l’exécution d’activités de synchronisation spécifiques (notamment les étapes d’importation complète et de synchronisation complète) une fois la mise à niveau terminée. Pour différer ces activités, consultez la section [Comment différer la synchronisation complète après la mise à niveau](#how-to-defer-full-synchronization-after-upgrade).

Si vous utilisez Azure AD Connect avec un connecteur non standard (par exemple, le connecteur LDAP générique ou le connecteur SQL générique), vous devez actualiser la configuration du connecteur en question dans [Synchronization Service Manager](./how-to-connect-sync-service-manager-ui-connectors.md) après la mise à niveau sur place. Pour savoir comment actualiser la configuration du connecteur, reportez-vous à la section Résolution des problèmes de l’article [Historique de publication des versions du connecteur](/microsoft-identity-manager/reference/microsoft-identity-manager-2016-connector-version-history#troubleshooting). Si vous n’actualisez pas la configuration, les étapes d’importation et d’exportation ne fonctionneront pas correctement pour le connecteur. Vous recevrez le message d’erreur suivant dans le journal des événements de l’application : *« Assembly version in AAD Connector configuration ("X.X.XXX.X") is earlier than the actual version ("X.X.XXX.X") of "C:\Program Files\Microsoft Azure AD Sync\Extensions\Microsoft.IAM.Connector.GenericLdap.dll". »* (La version de l’assembly dans la configuration du connecteur AAD (« X.X.XXX.X ») est antérieure à la version réelle (« X.X.XXX.X ») de « C:\Program Files\Microsoft Azure AD Sync\Extensions\Microsoft.IAM.Connector.GenericLdap.dll ».).

## <a name="swing-migration"></a>Migration « swing »
Si vous avez un déploiement complexe ou un nombre d’objets élevé, il peut être difficile d’effectuer une mise à niveau sur place sur le système réel. Pour certains clients, ce processus peut prendre plusieurs jours pendant lesquels aucune modification différentielle n’est traitée. Vous pouvez également utiliser cette méthode lorsque vous envisagez d’apporter des modifications importantes à votre configuration et que vous souhaitez les tester avant de les envoyer dans le cloud.

La méthode recommandée pour ces scénarios consiste à utiliser une migration « swing ». Vous devez avoir (au moins) deux serveurs : un serveur actif et un serveur intermédiaire. Le serveur actif (représenté par des lignes bleues continues dans l’image ci-dessous) est responsable de la charge de production active. Le serveur intermédiaire (représenté par des lignes violettes en pointillé) est préparé avec la nouvelle version ou configuration. Lorsqu’il est prêt, ce serveur devient actif. Le serveur actif précédent, sur lequel est désormais installée l’ancienne version ou configuration, devient le serveur intermédiaire et est mis à niveau.

Les deux serveurs peuvent utiliser différentes versions. Par exemple, le serveur actif que vous projetez de désactiver peut utiliser Azure AD Sync et le nouveau serveur intermédiaire peut utiliser Azure AD Connect. Si vous utilisez la migration « swing » pour développer une nouvelle configuration, il est judicieux d’installer les mêmes versions sur les deux serveurs.  
![Serveur de test](./media/how-to-upgrade-previous-version/stagingserver1.png)

> [!NOTE]
> Certains clients préfèrent avoir trois ou quatre serveurs pour ce scénario. Pendant la mise à niveau du serveur intermédiaire, vous n’avez pas de serveur de sauvegarde pour la [récupération d’urgence](how-to-connect-sync-staging-server.md#disaster-recovery). Avec trois ou quatre serveurs, vous pouvez préparer un ensemble de serveurs principaux/de secours disposant de la nouvelle version. Ainsi, un serveur intermédiaire est toujours prêt à prendre le relais.

Ces étapes fonctionnent également pour la migration à partir d’Azure AD Sync ou d’une solution avec FIM + connecteur Azure AD. En revanche, elles ne fonctionnent pas pour DirSync, mais on trouve cette même méthode de migration « swing » (également appelée déploiement parallèle) pour DirSync dans [Azure AD Connect : Mise à niveau à partir de DirSync](how-to-dirsync-upgrade-get-started.md).

### <a name="use-a-swing-migration-to-upgrade"></a>Utiliser une migration « swing » pour effectuer la mise à niveau
1. Si vous utilisez Azure AD Connect sur les deux serveurs et que vous prévoyez uniquement de modifier la configuration, assurez-vous que votre serveur actif et votre serveur intermédiaire utilisent tous deux la même version. Il sera plus facile de comparer ensuite les différences. Si vous effectuez une mise à niveau à partir d’Azure AD Sync, ces serveurs auront des versions différentes. Si vous effectuez une mise à niveau à partir d’une ancienne version d’Azure AD Connect, il est judicieux, mais pas obligatoire, de commencer avec les deux serveurs qui utilisent la même version.
2. Si vous avez effectué une configuration personnalisée et que votre serveur intermédiaire ne l’a pas, suivez les étapes décrites dans [Déplacer une configuration personnalisée depuis le serveur actif vers le serveur intermédiaire](#move-a-custom-configuration-from-the-active-server-to-the-staging-server).
3. Si vous mettez à niveau depuis une version antérieure d’Azure AD Connect, mettez à niveau le serveur intermédiaire vers la dernière version. Si vous migrez à partir d’Azure AD Sync, installez Azure AD Connect sur votre serveur intermédiaire.
4. Laissez le moteur de synchronisation effectuer une importation et une synchronisation complètes sur votre serveur intermédiaire.
5. Vérifiez que la nouvelle configuration n’a pas entraîné de modifications inattendues à l’aide de la procédure indiquée sous « Vérifier » dans [Vérifiez la configuration d’un serveur](how-to-connect-sync-staging-server.md#verify-the-configuration-of-a-server). Si quelque chose n’est pas comme prévu, corrigez le problème, exécutez l’importation et la synchronisation et vérifiez les données jusqu’à ce qu’elles vous conviennent en suivant les étapes.
6. Convertissez le serveur de test en serveur actif. C’est la dernière étape, « Changer de serveur actif », de la procédure [Vérifiez la configuration d’un serveur](how-to-connect-sync-staging-server.md#verify-the-configuration-of-a-server).
7. Si vous mettez à niveau Azure AD Connect, mettez à niveau le serveur actuellement en mode intermédiaire vers la dernière version. Suivez la même procédure pour mettre à niveau les données et la configuration. Si vous avez effectué une mise à niveau à partir d’Azure AD Sync, vous pouvez maintenant désactiver et retirer votre ancien serveur.

### <a name="move-a-custom-configuration-from-the-active-server-to-the-staging-server"></a>Déplacer une configuration personnalisée depuis le serveur actif vers le serveur intermédiaire
Si vous avez apporté des modifications à la configuration du serveur actif, vous devez vérifier que les mêmes modifications sont appliquées au serveur intermédiaire. Pour faciliter ce déplacement, vous pouvez utiliser la [documentation de configuration d’Azure AD Connect](https://github.com/Microsoft/AADConnectConfigDocumenter).

Vous pouvez migrer les règles de synchronisation personnalisées que vous avez créées à l’aide de PowerShell. Vous devez appliquer d’autres modifications de la même manière sur les deux systèmes, et vous ne pouvez pas migrer les modifications. La [documentation de configuration](https://github.com/Microsoft/AADConnectConfigDocumenter) peut vous aider à comparer les deux systèmes pour vérifier qu’ils sont identiques. L’outil peut également vous aider à automatiser les étapes indiquées dans cette section.

Vous devez configurer ce qui suit de la même façon sur les deux serveurs :

* Connexion aux mêmes forêts
* Tout filtrage de domaine et d’unité organisationnelle
* Fonctionnalités facultatives identiques, telles que la synchronisation de mot de passe et la réécriture du mot de passe

**Migrer les règles de synchronisation personnalisées**  
Pour déplacer des règles de synchronisation personnalisées, procédez comme suit :

1. Ouvrez l’ **Éditeur de règles de synchronisation** sur votre serveur actif.
2. Sélectionnez une règle personnalisée. Cliquez sur **Exporter**. Une fenêtre Bloc-notes apparaît. Enregistrez le fichier temporaire avec une extension PS1. Le fichier devient un script PowerShell. Copiez le fichier PS1 sur le serveur intermédiaire.  
   ![Exportation des règles de synchronisation](./media/how-to-upgrade-previous-version/exportrule.png)
3. Le GUID du connecteur est différent sur le serveur intermédiaire et doit être modifié. Pour obtenir le GUID, démarrez **l’Éditeur de règles de synchronisation**, sélectionnez une des règles par défaut représentant le même système connecté, puis cliquez sur **Exporter**. Remplacez le GUID dans votre fichier PS1 par le GUID se trouvant sur le serveur de test.
4. Dans une invite de PowerShell, exécutez le fichier PS1. Cette action crée la règle de synchronisation personnalisée sur le serveur intermédiaire.
5. Répétez cette procédure pour toutes vos règles personnalisées.

## <a name="how-to-defer-full-synchronization-after-upgrade"></a>Comment différer la synchronisation complète après la mise à niveau
Pendant la mise à niveau sur place, des modifications introduites peuvent nécessiter l’exécution d’activités de synchronisation spécifiques (notamment les étapes d’importation complète et de synchronisation complète). Par exemple, les modifications du schéma de connecteur nécessitent l’exécution de l’étape **Importation complète**, et les modifications des règles de synchronisation par défaut nécessitent l’exécution de l’étape **Synchronisation complète** sur les connecteurs affectés. Pendant la mise à niveau, Azure AD Connect détermine les activités de synchronisation requises et les enregistre en tant qu’*actions prioritaires*. Dans le cycle de synchronisation suivant, le planificateur de synchronisation récupère ces actions prioritaires et les exécute. Dès qu’une action prioritaire a été exécutée avec succès, elle est supprimée.

Il peut arriver que vous ne souhaitiez pas que ces actions prioritaires aient lieu immédiatement après la mise à niveau. Par exemple, vous avez de nombreux objets synchronisés et vous souhaitez que ces étapes de synchronisation aient lieu après les heures de bureau. Pour supprimer ces actions prioritaires :

1. Pendant la mise à niveau, **désactivez** l’option **Démarrez le processus de synchronisation une fois la configuration terminée**. Cela désactive le planificateur de synchronisation et empêche l’exécution automatique du cycle de synchronisation avant la suppression des actions prioritaires.

   ![Capture d’écran mettant en évidence l’option Démarrer le processus de synchronisation une fois la configuration terminée que vous devez désactiver.](./media/how-to-upgrade-previous-version/disablefullsync01.png)

2. Une fois la mise à niveau terminée, exécutez l’applet de commande suivante pour connaître les actions prioritaires qui ont été ajoutées : `Get-ADSyncSchedulerConnectorOverride | fl`

   >[!NOTE]
   > Les actions prioritaires sont propres au connecteur. Dans l’exemple suivant, les étapes d’importation complète et de synchronisation complète ont été ajoutées au connecteur Active Directory et au connecteur Azure AD locaux.

   ![DisableFullSyncAfterUpgrade](./media/how-to-upgrade-previous-version/disablefullsync02.png)

3. Prenez note des actions prioritaires existantes qui ont été ajoutées.
   
4. Pour supprimer les actions prioritaires pour l’importation complète et la synchronisation complète sur un connecteur arbitraire, exécutez l’applet de commande suivante : `Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier <Guid-of-ConnectorIdentifier> -FullImportRequired $false -FullSyncRequired $false`

   Pour supprimer les actions prioritaires sur tous les connecteurs, exécutez le script PowerShell suivant :

   ```
   foreach ($connectorOverride in Get-ADSyncSchedulerConnectorOverride)
   {
       Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier $connectorOverride.ConnectorIdentifier.Guid -FullSyncRequired $false -FullImportRequired $false
   }
   ```

5. Pour redémarrer le planificateur, exécutez l’applet de commande suivante : `Set-ADSyncScheduler -SyncCycleEnabled $true`

   >[!IMPORTANT]
   > N’oubliez pas d’exécuter les étapes de synchronisation requises dès que vous le pourrez. Vous pouvez exécuter ces étapes manuellement à l’aide de Synchronization Service Manager ou rajouter les actions prioritaires à l’aide de l’applet de commande Set-ADSyncSchedulerConnectorOverride.

Pour ajouter les actions prioritaires pour l’importation complète et la synchronisation complète sur un connecteur arbitraire, exécutez l’applet de commande suivante : `Set-ADSyncSchedulerConnectorOverride -ConnectorIdentifier <Guid> -FullImportRequired $true -FullSyncRequired $true`

## <a name="troubleshooting"></a>Dépannage
La section suivante contient des informations et des solutions de dépannage que vous pouvez utiliser si vous rencontrez des difficultés lors de la mise à niveau d’Azure AD Connect.

### <a name="azure-active-directory-connector-missing-error-during-azure-ad-connect-upgrade"></a>Erreur de connecteur Azure Active Directory manquant au cours de la mise à niveau d’Azure AD Connect

Lorsque vous mettez à niveau Azure AD Connect à partir d’une version antérieure, vous risquez de rencontrer l’erreur suivante au début de la mise à niveau 

![Error](./media/how-to-upgrade-previous-version/error1.png)

Cette erreur se produit, car le connecteur Azure Active Directory avec l’identificateur b891884f-051e-4a83-95af-2544101c9083 n’existe pas dans la configuration d’Azure AD Connect actuelle. Pour le vérifier, ouvrez une fenêtre PowerShell et exécutez l’applet de commande `Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083`

```
PS C:\> Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083
Get-ADSyncConnector : Operation failed because the specified MA could not be found.
At line:1 char:1
+ Get-ADSyncConnector -Identifier b891884f-051e-4a83-95af-2544101c9083
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ReadError: (Microsoft.Ident...ConnectorCmdlet:GetADSyncConnectorCmdlet) [Get-ADSyncConne
   ctor], ConnectorNotFoundException
    + FullyQualifiedErrorId : Operation failed because the specified MA could not be found.,Microsoft.IdentityManageme
   nt.PowerShell.Cmdlet.GetADSyncConnectorCmdlet

```

L’applet de commande PowerShell signale l’erreur selon laquelle **le MA spécifié est introuvable**.

Le fait que la configuration d’Azure AD Connect actuelle ne soit pas prise en charge pour la mise à niveau en est la raison. 

Si vous voulez installer une nouvelle version d’Azure AD Connect : fermez l’Assistant Azure AD Connect, désinstallez la version Azure AD Connect existante et procédez à une nouvelle installation avec une version plus récente.



## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur [l’intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).