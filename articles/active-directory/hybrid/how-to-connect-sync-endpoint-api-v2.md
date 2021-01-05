---
title: Point de terminaison V2 pour la synchronisation d’Azure AD Connect | Microsoft Docs
description: Ce document traite des mises à jour de l’API des points de terminaison v2 pour la synchronisation Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/04/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 07acd71eb1a77592c82b80aa574d79b98c48f60e
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608297"
---
# <a name="azure-ad-connect-sync-v2-endpoint-api"></a>API de point de terminaison V2 pour la synchronisation d’Azure AD Connect 
Microsoft a déployé un nouveau point de terminaison (API) pour Azure AD Connect qui améliore les performances des opérations des services de synchronisation pour Azure Active Directory. En utilisant le nouveau point de terminaison v2, vous constaterez des gains de performances perceptibles lors de l’exportation et de l’importation vers Azure AD. Ce nouveau point de terminaison prend en charge les éléments suivants :
    
 - synchronisation des groupes avec jusqu’à 250 000 membres
 - gains de performances lors de l’exportation et de l’importation vers Azure AD
 
> [!NOTE]
> Actuellement, le nouveau point de terminaison n’a pas de limite de taille de groupe configurée pour les groupes Microsoft 365 réécrits. Cela peut avoir un effet sur vos latences Active Directory et du cycle de synchronisation. Il est recommandé d’augmenter la taille des groupes de façon incrémentielle.  

## <a name="pre-requisites"></a>Conditions préalables  
Pour utiliser le nouveau point de terminaison v2, vous devez utiliser [version Azure AD Connect 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) ou version ultérieure et suivre les étapes de déploiement fournies ci-dessous pour activer le point de terminaison v2 pour votre serveur Azure AD Connect.   

## <a name="deployment-guidance"></a>Conseils pour le déploiement 
Vous devez déployer la [version Azure AD Connect 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) ou version ultérieure pour utiliser le point de terminaison v2. Utilisez le lien fourni pour le téléchargement. 

Il est recommandé de suivre la méthode de [migration Swing](./how-to-upgrade-previous-version.md#swing-migration) pour déployer le nouveau point de terminaison dans votre environnement. Cela fournira un plan d’urgence clair en cas de nécessité d’une restauration importante. L’exemple suivant illustre la façon dont une migration Swing peut être utilisée dans ce scénario. Pour plus d’informations sur la méthode de déploiement de migration Swing, reportez-vous au lien fourni. 

### <a name="swing-migration-for-deploying-v2-endpoint"></a>Migration Swing pour le déploiement du point de terminaison v2
Les étapes suivantes vous guideront tout au long du déploiement du point de terminaison v2 à l’aide de la méthode Swing.

1. Déployez le point de terminaison v2 sur le serveur de la zone de transit actuel. Ce serveur sera connu sous le nom de **serveur v2** dans les étapes ci-dessous. Le serveur actif actuel continuera à traiter la charge de travail de production à l’aide du point de terminaison v1, qui sera appelé le **serveur v1** ci-dessous.
1. Vérifiez que le **serveur v2** traite toujours les importations comme prévu. À ce niveau, les groupes de grande taille ne seront pas approvisionnés vers Azure AD ou AD local, mais vous serez en mesure de vérifier que la mise à niveau n’a pas abouti à un autre impact inattendu sur le processus de synchronisation existant. 
2. Une fois la validation terminée, basculez le **serveur v2** pour qu’il soit le serveur actif et le **serveur v1** pour qu’il soit le serveur de mise en lots. À ce stade, les groupes volumineux qui doivent être synchronisés sont provisionnés dans Azure AD. De même, les grands groupes unifiés Microsoft 365 seront provisionnés dans AD si la réécriture des groupes est activée.
3. Vérifiez que le **serveur v2** exécute et traite correctement les groupes de grande taille. Vous pouvez choisir de rester à cette étape et d’analyser le processus de synchronisation pendant une période donnée.
  >[!NOTE]
  > Si vous devez revenir à votre configuration précédente, vous pouvez effectuer une migration Swing à partir du **serveur v2** vers le **serveur v1**. Étant donné que le point de terminaison v1 ne prend pas en charge les groupes avec plus de 50 000 membres, les grands groupes approvisionnés par Azure AD Connect, dans Azure AD ou AD local, seront supprimés par la suite. 
4. Une fois que vous êtes certain d’utiliser le point de terminaison v2, mettez à niveau le **serveur v1** pour commencer à utiliser le point de terminaison v2. 
 

## <a name="expectations-of-performance-impact"></a>Attentes en matière d’impact sur les performances  
Lorsque vous utilisez le point de terminaison v2, les gains de performances sont en fonction du nombre de groupes synchronisés, de la taille de ces groupes et de l’évolution de leur groupe (l’activité qui résulte de l’ajout et de la suppression d’utilisateurs en tant que membres du groupe). L’utilisation du nouveau point de terminaison, sans qu’il soit possible d’incrémenter le nombre, la taille ou l’évolution des groupes synchronisés, doit entraîner des délais plus courts pour l’exportation et l’importation vers Azure AD. 
 
Toutefois, les gains de performance peuvent être annulés par le traitement supplémentaire requis lors de la synchronisation de grands groupes. Vous pouvez augmenter la durée globale de synchronisation en ajoutant un trop grand nombre de groupes importants au processus de synchronisation.  

Pour mieux comprendre comment l’ajout de nouveaux groupes aura un impact sur les performances de synchronisation, il est recommandé de commencer par synchroniser uniquement quelques groupes de grande taille avec moins de 100 000 membres. Vous pouvez ensuite augmenter le nombre et la taille des groupes en en ajoutant d’autres à l’étendue, par le biais du filtrage de l’unité d’organisation, de l’attribut ou de la taille maximale du groupe. Les améliorations des performances seront réalisées sur les tâches d’exportation et d’importation pour le connecteur Azure AD et non sur le connecteur AD local. 

## <a name="deployment-step-by-step"></a>Déploiement étape par étape 
Les trois phases suivantes constituent un exemple détaillé de déploiement du nouveau point de terminaison v2.  Utilisez les phases comme lignes directrices pour votre déploiement.

### <a name="phase-1--install-and-validate-azure-ad-connect"></a>Phase 1 : installer et valider Azure AD Connect 
Il est recommandé d’effectuer les étapes d’installation ou de mise à niveau vers [Azure AD Connect version 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) ou ultérieure et de valider le processus de synchronisation avant de passer à la deuxième phase où vous allez activer le point de terminaison v2. Sur le serveur Azure AD Connect : 


1. [Facultatif] Effectuer une sauvegarde de bases données 
2. Installer ou mettre à niveau [Azure AD Connect version 1.5.30.0](https://www.microsoft.com/download/details.aspx?id=47594) ou ultérieure.
3. validation de l'installation 

### <a name="phase-2--enable-the-v2-endpoint"></a>Phase 2 : activer le point de terminaison v2 
L’étape suivante consiste à activer le point de terminaison v2. 

> [!NOTE]
> Une fois que vous avez activé le point de terminaison v2 pour votre serveur, vous serez en mesure d’améliorer les performances de votre charge de travail existante. Vous ne pourrez pas encore synchroniser des groupes avec plus de 50 000 membres. 

Pour basculer vers un point de terminaison v2, procédez comme suit : 

1. Ouvrez une invite PowerShell en tant qu’administrateur. 
2. Désactivez le planificateur de synchronisation après avoir vérifié qu’aucune opération de synchronisation n’est en cours d’exécution : 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $false`
 
3. Importez le nouveau module : 
 
 `Import-Module 'C:\Program Files\Microsoft Azure AD Sync\Extensions\AADConnector.psm1'` 
 
4.  Basculez vers le point de terminaison v2 :

 `Set-ADSyncAADConnectorExportApiVersion 2` 
 
 `Set-ADSyncAADConnectorImportApiVersion 2` 

 ![PowerShell](media/how-to-connect-sync-endpoint-api-v2/endpoint1.png)
 
Vous avez maintenant activé le point de terminaison v2 pour votre serveur. Prenez le temps de vérifier qu’il n’y a pas de résultats inattendus après avoir activé le point de terminaison v2 avant de passer à la phase suivante, où vous augmenterez la limite de taille du groupe. 
>[!NOTE]
>Les chemins d’accès de fichier/module peuvent utiliser une lettre de lecteur différente, en fonction du chemin d’installation fourni lors de l’installation d’Azure AD Connect. 


### <a name="phase-3--increase-the-group-membership-limit"></a>Phase 3 : augmenter la limite d’appartenance au groupe 
Une fois que vous avez vérifié que le service s’exécute sans résultats inattendus, vous pouvez passer à l’augmentation de la limite d’appartenance au groupe. Il est recommandé de commencer par augmenter la limite d’appartenance à une valeur légèrement supérieure, par ex. : 75 000 membres, pour voir la synchronisation des groupes plus importants avec Azure AD. Une fois que vous êtes satisfait des résultats, vous pouvez augmenter davantage la limite des membres.  

La limite maximale est de 250 000 membres par groupe. 

Les étapes suivantes peuvent être utilisées pour augmenter la limite d’appartenance :  

1. Ouvrez l’Éditeur de règles de synchronisation Azure AD 
2. Dans l’éditeur, choisissez **Sortant** pour la Direction. 
3. Cliquez sur la règle de synchronisation **Out to AAD - Group Join** 
4. Cliquez sur le bouton **Modifier**. ![Screenshot that shows the "View and manage your synchronization rules" with "Out to AAD - Group Join" selected.](media/how-to-connect-sync-endpoint-api-v2/endpoint2.png)

6. Cliquez sur le bouton **Oui** pour désactiver la règle par défaut et créer une copie modifiable.
 ![Screenshot that shows the "Edit Reserved Rule Confirmation" window with the "Yes" button selected.](media/how-to-connect-sync-endpoint-api-v2/endpoint3.png)

7. Dans la fenêtre contextuelle de la page **Description**, définissez la précédence sur une valeur disponible comprise entre 1 et 99. ![Screenshot that shows the "Edit outbound synchronization rule" window with "Precedence" highlighted.](media/how-to-connect-sync-endpoint-api-v2/endpoint4.png)

8. Dans la page **Transformations**, mettez à jour la valeur **Source** pour la transformation du **membre**, en remplaçant « 50 000 » par une valeur comprise entre 50 001 et 250 000. Ce remplacement augmentera la taille d’appartenance maximale des groupes synchronisés avec Azure AD. Nous vous suggérons de commencer avec un nombre de 100 000, afin de comprendre l’impact que la synchronisation de grands groupes aura sur les performances de synchronisation. 
 
 **Exemple** 
 
 `IIF((ValueCount("member")> 75000),Error("Maximum Group member count exceeded"),IgnoreThisFlow)` 
 
 ![Modifiez la règle de synchronisation](media/how-to-connect-sync-endpoint-api-v2/endpoint5.png)

9. Cliquez sur Enregistrer. 
10. Ouvrez l’invite administrateur PowerShell 
11. Réactivez le planificateur de synchronisation 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $true` 
 
>[!NOTE]
> Si Azure AD Connect Health n’est pas activé, modifiez les paramètres du journal des événements de l’application Windows pour archiver les journaux, au lieu de les remplacer. Les journaux peuvent être utilisés pour faciliter la résolution des problèmes. 

>[!NOTE]
> Après avoir activé le nouveau point de terminaison, vous pouvez voir d’autres erreurs d’exportation sur le connecteur AAD portant le nom « dn-attributes-failure ». Il y aura une entrée correspondante dans le journal des événements pour chaque erreur portant l’ID 6949. Les erreurs sont fournies à titre d’information et n’indiquent pas de problème avec votre installation, mais elles indiquent plutôt que le processus de synchronisation n’a pas pu ajouter certains membres à un groupe dans Azure AD, car l’objet membre lui-même n’a pas été synchronisé avec Azure AD. 

Le nouveau code de point de terminaison v2 gère certains types d’erreurs d’exportation légèrement différents de ceux du code v1.  Utilisez le point de terminaison v2 pour consulter plus de messages d’erreur diffusés à titre d’information. 

>[!NOTE]
> Lors de la mise à niveau d’Azure AD Connect, assurez-vous que les étapes de la phase 2 sont ré exécutées, car les modifications ne sont pas conservées au cours du processus de mise à niveau. 

Lors des augmentations ultérieures de la limite des membres du groupe dans la règle de synchronisation **Out to AAD – Group Join**, une synchronisation complète n’est pas nécessaire. Vous pouvez donc choisir de supprimer la synchronisation complète en exécutant la commande suivante dans PowerShell. 

 `Set-ADSyncSchedulerConnectorOverride -FullSyncRequired $false -ConnectorName "<AAD Connector Name>" `
 
>[!NOTE]
> Si vous avez des groupes unifiés Microsoft 365 qui comportent plus de 50 000 membres, les groupes sont lus dans Azure AD Connect et, si la réécriture des groupes est activée, ils sont écrits dans votre annuaire AD local. 

## <a name="rollback"></a>Restauration 
Si vous avez activé le point de terminaison v2 et que vous devez effectuer une restauration, procédez comme suit : 

1. Sur le serveur Azure AD Connect : a. [Facultatif] Effectuer une sauvegarde de bases données 
2. Ouvrir l’invite administrateur PowerShell :
3. Désactivez le planificateur de synchronisation après avoir vérifié qu’aucune opération de synchronisation n’est en cours d’exécution
 
 `Set-ADSyncScheduler -SyncCycleEnabled $false`

4. Basculez vers le point de terminaison v1 * 
 
 `Import-Module 'C:\Program Files\Microsoft Azure AD Sync\Extensions\AADConnector.psm1'` 

 `Set-ADSyncAADConnectorExportApiVersion 1`

 `Set-ADSyncAADConnectorImportApiVersion 1`
 
5. Ouvrez l’Éditeur de règles de synchronisation Azure AD 
6. Supprimez la copie modifiable de la règle de synchronisation **Out to AAD - Group Join** 
7. Activez la copie par défaut de la règle de synchronisation **Out to AAD - Group Join** 
8. Ouvrez une invite administrateur PowerShell 
9. Réactivez le planificateur de synchronisation 
 
 `Set-ADSyncScheduler -SyncCycleEnabled $true`
 
>[!NOTE]
> Lorsque vous basculez de nouveau du point de terminaison v2 vers v1, les groupes synchronisés avec plus de 50 000 membres seront supprimés après l’exécution d’une synchronisation complète, à la fois pour les groupes AD provisionnés dans Azure AD et les groupes unifiés Microsoft 365 provisionnés dans AD. 

## <a name="frequently-asked-questions"></a>Forum aux questions  
 
**Quand le nouveau point de terminaison deviendra-t-il le point par défaut pour les mises à niveau et les nouvelles installations ?**   
</br>Nous envisageons de publier une nouvelle version d’AADConnect qui sera disponible au téléchargement en janvier 2021. Cette version utilisera le point de terminaison v2 par défaut et permettra la synchronisation des groupes de plus de 50 000 membres, sans nécessiter de configuration supplémentaire. Cette version sera publiée par la suite dans le cadre de la mise à niveau automatique vers les serveurs éligibles.
 
## <a name="next-steps"></a>Étapes suivantes

* [Synchronisation Azure AD Connect : Comprendre et personnaliser la synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
