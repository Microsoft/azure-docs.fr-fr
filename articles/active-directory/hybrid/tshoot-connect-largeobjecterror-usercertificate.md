---
title: 'Azure AD Connect : erreurs LargeObject provoquées par l’attribut userCertificate | Microsoft Docs'
description: Cette rubrique propose une procédure de correction des erreurs LargeObject provoquées par l’attribut userCertificate.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 146ad5b3-74d9-4a83-b9e8-0973a19828d9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: d33b419e0f24201d661ad0f5f1373022ea6e9e9f
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861746"
---
# <a name="azure-ad-connect-sync-handling-largeobject-errors-caused-by-usercertificate-attribute"></a>Synchronisation d’Azure AD Connect : gérer les erreurs LargeObject provoquées par l’attribut userCertificate

Azure AD impose une limite maximale de **15** valeurs de certificat sur l’attribut **userCertificate**. Si Azure AD Connect exporte un objet comportant plus de 15 valeurs vers Azure AD, Azure AD renvoie une erreur **LargeObject** avec le message :

>*« L’objet approvisionné est trop volumineux. Réduisez le nombre de valeurs d’attribut sur cet objet. L’opération sera retentée lors du prochain cycle de synchronisation... »*

L’erreur LargeObject peut provenir d’autres attributs AD. Pour vérifier qu’elle est bien causée par l’attribut userCertificate, vous devez la confronter à l’objet dans le service AD local ou dans la [Recherche métaverse de Synchronization Service Manager](./how-to-connect-sync-service-manager-ui-mvsearch.md).

Pour obtenir la liste des objets de votre client comportant des erreurs LargeObject, appliquez l’une des méthodes suivantes :

 * Si votre client gère la synchronisation avec Azure AD Connect Health, vous pouvez vous référer au [rapport d’erreurs de synchronisation](./how-to-connect-health-sync.md) fourni.
 
 * Le courrier électronique de notification des erreurs de synchronisation d’annuaires qui est envoyé à la fin de chaque cycle de synchronisation contient la liste des objets comportant des erreurs LargeObject. 
 * [L’onglet opérations de Synchronization Service Manager](./how-to-connect-sync-service-manager-ui-operations.md) affiche la liste des objets comportant des erreurs LargeObject si vous cliquez sur la dernière opération Exporter vers Azure AD.
 
## <a name="mitigation-options"></a>Options de correction
Tant que l’erreur LargeObject n’est pas résolue, les autres modifications d’attribut apportées à l’objet ne peuvent pas être exportées vers Azure AD. Pour résoudre l’erreur, vous pouvez envisager les options suivantes :

 * Mettez à niveau Azure AD Connect vers la build 1.1.524.0 ou ultérieure. Dans la build 1.1.524.0 d’Azure AD Connect, les règles de synchronisation out-of-box ont été mises à jour de sorte que les attributs userCertificate et userSMIMECertificate ne soient pas exportés s’ils contiennent plus 15 valeurs. Pour plus d’informations sur la procédure de mise à niveau d’Azure AD Connect, consultez l’article [Azure AD Connect : effectuer une mise à niveau vers la dernière version](./how-to-upgrade-previous-version.md).

 * Implémenter une **règle de synchronisation sortante** dans Azure AD Connect qui exporte une **valeur Null au lieu des valeurs réelles des objets possédant plus de 15 valeurs de certificat**. Cette option est adaptée si vous n’avez pas besoin d’exporter vers Azure AD les valeurs de certificat des objets en possédant plus de 15. Pour plus d’informations sur l’implémentation de cette règle de synchronisation, reportez-vous à la section suivante, [Implémenter des règles de synchronisation pour limiter l’exportation de l’attribut userCertificate](#implementing-sync-rule-to-limit-export-of-usercertificate-attribute).

 * Réduire le nombre de valeurs de certificat sur l’objet AD local (15 au plus) en supprimant les valeurs qui ne sont plus utilisées par votre organisation. Cette option est adaptée si la multiplication des attributs est provoquée par des certificats expirés ou non utilisés. Vous pouvez utiliser le [script PowerShell disponible ici](https://gallery.technet.microsoft.com/Remove-Expired-Certificates-0517e34f) pour trouver, sauvegarder et supprimer les certificats arrivés à expiration dans votre service AD local. Nous vous recommandons, avant de supprimer les certificats, de consulter les administrateurs de l’infrastructure de clé publique de votre organisation.

 * Configurer Azure AD Connect pour exclure l’attribut userCertificate de l’exportation vers Azure AD. En règle générale, nous déconseillons cette option, car l’attribut peut être utilisé par Microsoft Online Services pour certains scénarios, En particulier :
    * L’attribut userCertificate de l’objet utilisateur est utilisé par les clients Outlook et Exchange Online pour le chiffrement et la signature des messages. Pour en savoir plus sur cette fonctionnalité, reportez-vous à l’article [S/MIME pour le chiffrement et la signature des messages](/microsoft-365/security/office-365-security/s-mime-for-message-signing-and-encryption).

    * L’attribut userCertificate sur l’objet ordinateur est utilisé par Azure AD pour autoriser les appareils Windows 10 locaux et joints à un domaine à se connecter à Azure AD. Pour en savoir plus sur cette fonctionnalité, reportez-vous à l’article [Connecter des appareils joints à un domaine à Azure AD pour les expériences Windows 10](../devices/hybrid-azuread-join-plan.md).

## <a name="implementing-sync-rule-to-limit-export-of-usercertificate-attribute"></a>Implémenter la règle de synchronisation pour limiter l’exportation de l’attribut userCertificate
Pour résoudre l’erreur LargeObject causée par l’attribut userCertificate, vous pouvez implémenter une règle de synchronisation sortante dans Azure AD Connect qui exporte une **valeur Null au lieu des valeurs réelles des objets possédant plus de 15 valeurs de certificat**. Cette section décrit la procédure nécessaire pour implémenter la règle de synchronisation pour les objets **User**. Les étapes peuvent être adaptées pour les objets **Contact** et **Computer**.

> [!IMPORTANT]
> Le fait d’exporter la valeur Null supprime les valeurs de certificat précédemment exportées vers Azure AD.

La procédure peut se résumer ainsi :

1. Désactiver le planificateur de synchronisation et vérifier qu'aucune synchronisation n’est en cours.
3. Rechercher la règle de synchronisation sortante existante pour l’attribut userCertificate.
4. Créer la règle de synchronisation sortante qui s’impose.
5. Vérifier la nouvelle règle de synchronisation sur un objet existant comportant l’erreur LargeObject.
6. Appliquer la nouvelle règle de synchronisation aux objets restants avec l’erreur LargeObject.
7. Vérifier l’absence de modification inattendue en attente d’exportation vers Azure AD.
8. Exporter les modifications vers Azure AD.
9. Réactiver le planificateur de synchronisation.

### <a name="step-1-disable-sync-scheduler-and-verify-there-is-no-synchronization-in-progress"></a>Étape 1. Désactiver le planificateur de synchronisation et vérifier qu'aucune synchronisation n’est en cours
Veillez à ce qu'aucune synchronisation ne se produise au cours de l’implémentation d’une nouvelle règle de synchronisation, afin de ne pas exporter des modifications inattendues vers Azure AD. Pour désactiver le planificateur de synchronisation intégré :
1. Lancez une session PowerShell sur le serveur Azure AD Connect.

2. Désactivez la synchronisation planifiée en exécutant l’applet de commande : `Set-ADSyncScheduler -SyncCycleEnabled $false`

> [!Note]
> Les étapes précédentes ne s’appliquent qu’aux versions récentes (1.1.xxx.x) d’Azure AD Connect avec le planificateur intégré. Si vous utilisez des versions plus anciennes (1.0.xxx.x) d’Azure AD Connect, qui utilisent le planificateur de tâches Windows, ou votre propre planificateur personnalisé (peu courant) pour déclencher une synchronisation périodique, vous devez les désactiver en conséquence.

1. Lancez **Synchronization Service Manager** en accédant au menu DÉMARRER → Service de synchronisation.

1. Accédez à l’onglet **Opérations** et confirmez qu’aucune opération n’est à l’état *« En cours ».*

### <a name="step-2-find-the-existing-outbound-sync-rule-for-usercertificate-attribute"></a>Étape 2. Rechercher la règle de synchronisation sortante existante pour l’attribut userCertificate
Il existe normalement une règle de synchronisation activée et configurée pour exporter l’attribut userCertificate des objets User vers Azure AD. Localisez cette règle de synchronisation pour déterminer sa configuration de **filtre d’étendue** et de **priorité** :

1. Lancez **Synchronization Rules Editor** dans le menu DÉMARRER → Éditeur de règles de synchronisation.

2. Configurez les filtres de recherche en appliquant les valeurs suivantes :

    | Attribut | Valeur |
    | --- | --- |
    | Sens |**Sortante** |
    | Type d'objet MV |**Person** |
    | Connecteur |*nom de votre connecteur Azure AD* |
    | Type d’objet de connecteur |**user** |
    | Attribut MV |**userCertificate** |

3. Si vous utilisez des règles de synchronisation OOB (« out-of-box ») vers le connecteur Azure AD pour exporter l’attribut userCertificate des objets User, revenez à la règle *« Out to AAD – User ExchangeOnline »* .
4. Notez la valeur de **priorité** de cette règle de synchronisation.
5. Sélectionnez la règle de synchronisation et cliquez sur **Modifier**.
6. Dans la boîte de dialogue contextuelle *« Modifier la confirmation de règle réservée »* , cliquez sur **Non**. (Ne vous inquiétez pas, nous n’allons pas apporter de modifications à cette règle de synchronisation).
7. Dans l’écran de modification, sélectionnez l’onglet **Filtre d’étendue**.
8. Notez la configuration du filtre d’étendue. Si vous utilisez la règle de synchronisation OOB, il doit y avoir exactement **un groupe de filtres d’étendue comportant deux clauses**, notamment :

    | Attribut | Opérateur | Valeur |
    | --- | --- | --- |
    | sourceObjectType | EQUAL | Utilisateur |
    | cloudMastered | NOTEQUAL | True |

### <a name="step-3-create-the-outbound-sync-rule-required"></a>Étape 3. Créer la règle de synchronisation sortante nécessaire
La nouvelle règle de synchronisation doit avoir le même **filtre d’étendue** et une **priorité plus élevée** que la règle de synchronisation existante. Cela garantit que la nouvelle règle de synchronisation s’applique au même jeu d’objets que la règle de synchronisation existante, et la remplace pour l’attribut userCertificate. Pour créer la règle de synchronisation :
1. Dans l’éditeur de règles de synchronisation, cliquez sur le bouton **Ajouter une nouvelle règle**.
2. Sous **l’onglet Description**, fournissez la configuration suivante :

    | Attribut | Valeur | Détails |
    | --- | --- | --- |
    | Nom | *Donnez-lui un nom* | Ex. *« Out to AAD – Custom override for userCertificate »* |
    | Description | *Fournissez une description* | Ex. *« Si l’attribut userCertificate comporte plus de 15 valeurs, exporter NULL »* |
    | Système connecté | *Sélectionnez le connecteur Azure AD* |
    | Type d’objet système connecté | **user** | |
    | Type d’objet métaverse | **person** | |
    | Type de lien | **Join** | |
    | Priorité | *Choisissez un nombre compris entre 1 et 99* | Le nombre sélectionné ne doit pas être utilisé par une règle de synchronisation existante ; il doit avoir une valeur inférieure à celle de la règle de synchronisation existante (et, par conséquent, une priorité plus élevée). |

3. Accédez à l’onglet **Filtre d’étendue** et implémentez le filtre d’étendue utilisé par la règle de synchronisation existante.
4. Ignorez l’onglet **Règles de jointure**.
5. Accédez à l’onglet **Transformations** pour ajouter une nouvelle transformation selon la configuration suivante :

    | Attribut | Valeur |
    | --- | --- |
    | Type de flux |**Expression** |
    | Attribut cible |**userCertificate** |
    | Attribut source |*Utilisez l’expression suivante* : `IIF(IsNullOrEmpty([userCertificate]), NULL, IIF((Count([userCertificate])> 15),AuthoritativeNull,[userCertificate]))` |
    
6. Cliquez sur le bouton **Ajouter** pour créer la règle de synchronisation.

### <a name="step-4-verify-the-new-sync-rule-on-an-existing-object-with-largeobject-error"></a>Étape 4. Vérifier la nouvelle règle de synchronisation sur un objet existant comportant l’erreur LargeObject
Il s’agit de vérifier que la règle de synchronisation créée fonctionne correctement sur un objet AD existant comportant une erreur LargeObject, avant de l’appliquer à d’autres objets :
1. Accédez à l’onglet **Opérations** dans Synchronization Service Manager.
2. Sélectionnez l’opération Exporter vers Azure AD la plus récente, puis cliquez sur l’un des objets comportant une erreur LargeObject.
3.  Dans l’écran contextuel de propriétés de l’objet CS (Connector Space), cliquez sur le bouton **Aperçu**.
4. Dans l’écran contextuel Aperçu, sélectionnez **Synchronisation complète** et cliquez sur **Valider l’aperçu**.
5. Fermez l’écran Aperçu et l’écran Propriétés de l’objet CS.
6. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
7. Cliquez avec le bouton droit sur le connecteur **Azure AD** et sélectionnez **Exécuter...**
8. Dans la fenêtre contextuelle Exécuter le connecteur, sélectionnez l’étape **Exporter** et cliquez sur **OK**.
9. Attendez la fin de l’exportation vers Azure AD et vérifiez qu'il n’y a plus d’erreurs LargeObject sur cet objet.

### <a name="step-5-apply-the-new-sync-rule-to-remaining-objects-with-largeobject-error"></a>Étape 5. Appliquer la nouvelle règle de synchronisation aux objets restants avec l’erreur LargeObject
Une fois la règle de synchronisation ajoutée, vous devez lancer une étape de synchronisation complète sur le connecteur AD :
1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
2. Cliquez avec le bouton droit sur le connecteur **AD** et sélectionnez **Exécuter...**
3. Dans la fenêtre contextuelle Exécuter le connecteur, sélectionnez l’étape **Synchronisation complète** et cliquez sur **OK**.
4. Attendez la fin de l’étape de synchronisation complète.
5. Répétez les étapes ci-dessus pour les autres connecteurs AD si vous en avez plusieurs. En règle générale, plusieurs connecteurs sont requis si vous avez plusieurs annuaires locaux.

### <a name="step-6-verify-there-are-no-unexpected-changes-waiting-to-be-exported-to-azure-ad"></a>Étape 6. Vérifier l’absence de modification inattendue en attente d’exportation vers Azure AD
1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
2. Cliquez avec le bouton droit sur le connecteur **Azure AD** et sélectionnez **Rechercher dans l’espace connecteur**.
3. Dans la fenêtre contextuelle Rechercher dans l’espace connecteur :
    1. Définissez l’étendue sur **En attente d’exportation**.
    2. Cochez les trois cases : **Ajouter**, **Modifier** et **Supprimer**.
    3. Cliquez sur le bouton **Rechercher** pour renvoyer tous les objets comportant des modifications en attente d’exportation vers Azure AD.
    4. Vérifiez l’absence de modification inattendue. Pour examiner les modifications d’un objet donné, double-cliquez sur l’objet.

### <a name="step-7-export-the-changes-to-azure-ad"></a>Étape 7. Exporter les modifications vers Azure AD
Pour exporter les modifications vers Azure AD :
1. Accédez à l’onglet **Connecteurs** dans Synchronization Service Manager.
2. Cliquez avec le bouton droit sur le connecteur **Azure AD** et sélectionnez **Exécuter...**
4. Dans la fenêtre contextuelle Exécuter le connecteur, sélectionnez l’étape **Exporter** et cliquez sur **OK**.
5. Attendez la fin de l’exportation vers Azure AD et vérifiez qu'il n’y a plus d’erreurs LargeObject.

### <a name="step-8-re-enable-sync-scheduler"></a>Étape 8 : Réactiver le planificateur de synchronisation
Maintenant que le problème est résolu, réactivez le planificateur de synchronisation intégré :
1. Lancez une session PowerShell.
2. Réactivez la synchronisation planifiée en exécutant l’applet de commande : `Set-ADSyncScheduler -SyncCycleEnabled $true`

> [!Note]
> Les étapes précédentes ne s’appliquent qu’aux versions récentes (1.1.xxx.x) d’Azure AD Connect avec le planificateur intégré. Si vous utilisez des versions plus anciennes (1.0.xxx.x) d’Azure AD Connect, qui utilisent le planificateur de tâches Windows, ou votre propre planificateur personnalisé (peu courant) pour déclencher une synchronisation périodique, vous devez les désactiver en conséquence.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
