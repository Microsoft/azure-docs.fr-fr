---
title: Résoudre les problèmes de synchronisation du hachage de mot de passe avec Azure AD Connect Sync | Microsoft Docs
description: Cet article donne des informations sur la résolution des problèmes de synchronisation du hachage de mot de passe.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 03/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 77271679306b0fbde10c748afc7535f3ad3d0945
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317563"
---
# <a name="troubleshoot-password-hash-synchronization-with-azure-ad-connect-sync"></a>Résoudre les problèmes de synchronisation du hachage de mot de passe avec Azure AD Connect Sync

Cette rubrique explique comment résoudre les problèmes de synchronisation du hachage de mot de passe. Si les mots de passe ne se synchronisent pas comme prévu, il peut s’agir d’un sous-ensemble d’utilisateurs ou de tous les utilisateurs.

Pour un déploiement d’Azure Active Directory (Azure AD) Connect version 1.1.614.0 ou ultérieure, utilisez la tâche de dépannage de l’Assistant pour résoudre les problèmes de synchronisation du hachage de mot de passe :

* Si vous rencontrez un problème qui bloque la synchronisation de tous les mots de passe, consultez la section [Aucun mot de passe n’est synchronisé : résoudre les problèmes à l’aide de la tâche de résolution des problèmes](#no-passwords-are-synchronized-troubleshoot-by-using-the-troubleshooting-task).

* Si vous rencontrez des problèmes avec certains objets, consultez la section [Un objet ne synchronise pas les mots de passe : résoudre les problèmes à l’aide de la tâche de résolution des problèmes](#one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-troubleshooting-task).

Pour un déploiement de la version 1.1.524.0 ou ultérieure, il existe une cmdlet de diagnostic qui permet de résoudre les problèmes de synchronisation du hachage de mot de passe :

* Si vous rencontrez un problème qui bloque la synchronisation de tous les mots de passe, consultez la section [Aucun mot de passe n’est synchronisé : résoudre les problèmes à l’aide de l’applet de commande de diagnostic](#no-passwords-are-synchronized-troubleshoot-by-using-the-diagnostic-cmdlet).

* Si vous avez un problème avec des objets spécifiques, consultez la section [Un objet ne synchronise pas les mots de passe : résoudre les problèmes à l’aide de l’applet de commande de diagnostic](#one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-diagnostic-cmdlet).

Pour les versions antérieures de déploiement Azure AD Connect :

* Si vous rencontrez un problème qui bloque la synchronisation de tous les mots de passe, consultez la section [Aucun mot de passe n’est synchronisé : étapes de dépannage manuel](#no-passwords-are-synchronized-manual-troubleshooting-steps).

* Si vous avez un problème avec des objets spécifiques, consultez la section [Un objet ne synchronise pas les mots de passe : étapes de dépannage manuel](#one-object-is-not-synchronizing-passwords-manual-troubleshooting-steps).



## <a name="no-passwords-are-synchronized-troubleshoot-by-using-the-troubleshooting-task"></a>Aucun mot de passe n’est synchronisé : résoudre les problèmes à l’aide de la tâche de résolution des problèmes

Vous pouvez utiliser la tâche de résolution des problèmes pour déterminer la raison pour laquelle aucun mot de passe n’est synchronisé.

> [!NOTE]
> La tâche de résolution des problèmes est uniquement disponible pour Azure AD Connect version 1.1.614.0 ou ultérieure.

### <a name="run-the-troubleshooting-task"></a>Exécuter la tâche de résolution des problèmes

Pour résoudre les problèmes bloquant la synchronisation de tous les mots de passe

1. Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option **Exécuter en tant qu’administrateur**.

2. Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.

3. Lancez l’Assistant Azure AD Connect.

4. Accédez à la page **Tâches supplémentaires**, sélectionnez **Résoudre les problèmes**, puis cliquez sur **Suivant**.

5. Dans la page de résolution des problèmes, cliquez sur **Lancer** pour ouvrir le menu de dépannage de PowerShell.

6. Dans le menu principal, sélectionnez **Résoudre les problèmes de synchronisation du hachage de mot de passe**.

7. Dans le sous-menu, sélectionnez **La synchronisation du hachage de mot de passe ne fonctionne pas du tout**.

### <a name="understand-the-results-of-the-troubleshooting-task"></a>Comprendre les résultats de la tâche de résolution des problèmes

La tâche de résolution des problèmes effectue les vérifications suivantes :

* Elle vérifie que la fonctionnalité de synchronisation du hachage de mot de passe est activée pour votre client Azure AD.

* Elle vérifie que le serveur Azure AD Connect n’est pas en mode intermédiaire.

* Pour chaque connecteur Active Directory local existant (qui correspond à une forêt Active Directory existante) :

   * Elle vérifie que la fonctionnalité de synchronisation du hachage de mot de passe est activée.
   
   * Elle recherche les événements de pulsation de synchronisation de hachage de mot de passe dans les Journaux des événements de l’application Windows.

   * Pour chaque domaine Active Directory sous le connecteur Active Directory local :

      * Elle vérifie que le domaine est accessible à partir du serveur Azure AD Connect.

      * Elle vérifie que les comptes Active Directory Domain Services (AD DS) utilisés par le connecteur Active Directory local possèdent le nom d’utilisateur, le mot de passe et les autorisations requis pour la synchronisation du hachage de mot de passe.

L’image suivante illustre les résultats de l’applet de commande pour une topologie Active Directory locale à domaine unique :

![Sortie de diagnostic pour la synchronisation du hachage de mot de passe](./media/tshoot-connect-password-hash-synchronization/phsglobalgeneral.png)

Le reste de cette section décrit les résultats qui sont retournés par la tâche et les problèmes correspondants.

#### <a name="password-hash-synchronization-feature-isnt-enabled"></a>La fonctionnalité de synchronisation du hachage de mot de passe n’est pas activée

Si vous n’avez pas encore activé la synchronisation du hachage de mot de passe avec l’Assistant Azure AD Connect, l’erreur suivante est retournée :

![La synchronisation du hachage de mot de passe n’est pas activée](./media/tshoot-connect-password-hash-synchronization/phsglobaldisabled.png)

#### <a name="azure-ad-connect-server-is-in-staging-mode"></a>Le serveur Azure AD Connect est en mode intermédiaire

Si le serveur Azure AD Connect est en mode intermédiaire, la synchronisation du hachage de mot de passe est temporairement désactivée et l’erreur suivante est retournée :

![Le serveur Azure AD Connect est en mode intermédiaire](./media/tshoot-connect-password-hash-synchronization/phsglobalstaging.png)

#### <a name="no-password-hash-synchronization-heartbeat-events"></a>Aucun événement de pulsation de synchronisation du hachage de mot de passe

Chaque connecteur Active Directory local a son propre canal de synchronisation du hachage de mot de passe. Quand le canal de synchronisation du hachage de mot de passe est établi et qu’il n’y a aucun changement de mot de passe à synchroniser, un événement de pulsation (EventId 654) est généré toutes les 30 minutes dans le Journal des événements de l’application Windows. Pour chaque connecteur Active Directory local, l’applet de commande recherche les événements de pulsation d’inventaire correspondants au cours des trois dernières heures. Si aucun événement de pulsation n’est trouvé, l’erreur suivante est retournée :

![Aucun événement de pulsation de synchronisation du hachage de mot de passe](./media/tshoot-connect-password-hash-synchronization/phsglobalnoheartbeat.png)

#### <a name="ad-ds-account-does-not-have-correct-permissions"></a>Le compte AD DS n’a pas les autorisations appropriées

Si le compte AD DS utilisé par le connecteur Active Directory local pour synchroniser les hachages de mot de passe n’a pas les autorisations appropriées, l’erreur suivante est retournée :

![Capture d’écran montrant l’erreur renvoyée quand le compte AD DS a un nom d’utilisateur ou un mot de passe incorrect.](./media/tshoot-connect-password-hash-synchronization/phsglobalaccountincorrectpermission.png)

#### <a name="incorrect-ad-ds-account-username-or-password"></a>Nom d’utilisateur ou mot de passe du compte AD DS incorrect

Si le compte AD DS utilisé par le connecteur Active Directory local pour synchroniser les hachages de mot de passe a un nom d’utilisateur ou un mot de passe incorrect, l’erreur suivante est retournée :

![Informations d’identification incorrectes](./media/tshoot-connect-password-hash-synchronization/phsglobalaccountincorrectcredential.png)



## <a name="one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-troubleshooting-task"></a>Un objet ne synchronise pas les mots de passe : résoudre les problèmes à l’aide de la tâche de résolution des problèmes

Vous pouvez utiliser la tâche de résolution des problèmes pour déterminer la raison pour laquelle un objet ne synchronise pas les mots de passe.

> [!NOTE]
> La tâche de résolution des problèmes est uniquement disponible pour Azure AD Connect version 1.1.614.0 ou ultérieure.

### <a name="run-the-diagnostics-cmdlet"></a>Exécuter l’applet de commande de diagnostic

Pour résoudre les problèmes liés à un objet utilisateur :

1. Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option **Exécuter en tant qu’administrateur**.

2. Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.

3. Lancez l’Assistant Azure AD Connect.

4. Accédez à la page **Tâches supplémentaires**, sélectionnez **Résoudre les problèmes**, puis cliquez sur **Suivant**.

5. Dans la page de résolution des problèmes, cliquez sur **Lancer** pour ouvrir le menu de dépannage de PowerShell.

6. Dans le menu principal, sélectionnez **Résoudre les problèmes de synchronisation du hachage de mot de passe**.

7. Dans le sous-menu, sélectionnez **Password is not synchronized for a specific user account** (Le mot de passe n’est pas synchronisé pour un compte utilisateur).

### <a name="understand-the-results-of-the-troubleshooting-task"></a>Comprendre les résultats de la tâche de résolution des problèmes

La tâche de résolution des problèmes effectue les vérifications suivantes :

* Elle examine l’état de l’objet Active Directory dans l’espace de connecteur Active Directory, dans le métaverse et dans l’espace de connecteur Azure AD.

* Elle vérifie qu’il existe des règles de synchronisation pour lesquelles la synchronisation du hachage de mot de passe est activée et appliquée à l’objet Active Directory.

* Elle tente de récupérer et d’afficher les résultats de la dernière tentative de synchronisation de mot de passe pour l’objet.

Le diagramme suivant illustre les résultats de la cmdlet lors de la résolution des problèmes de synchronisation du hachage de mot de passe pour un seul objet :

![Sortie de diagnostic pour la synchronisation du hachage de mot de passe – objet unique](./media/tshoot-connect-password-hash-synchronization/phssingleobjectgeneral.png)

Le reste de cette section décrit les résultats spécifiques qui sont retournés par l’applet de commande et les problèmes correspondants.

#### <a name="the-active-directory-object-isnt-exported-to-azure-ad"></a>L’objet Active Directory n’est pas exporté vers Azure AD

La synchronisation du hachage de mot de passe de ce compte Active Directory local échoue, car il n’existe aucun objet correspondant dans le client Azure AD. L'erreur suivante est retournée :

![Objet Azure Active Directory manquant](./media/tshoot-connect-password-hash-synchronization/phssingleobjectnotexported.png)

#### <a name="user-has-a-temporary-password"></a>L’utilisateur a un mot de passe temporaire

Actuellement, Azure AD Connect ne prend pas en charge la synchronisation des mots de passe temporaires avec Azure AD. Un mot de passe est considéré comme temporaire si l’option **L’utilisateur doit changer le mot de passe à la prochaine ouverture de session** est définie pour l’utilisateur Active Directory local. L'erreur suivante est retournée :

![Le mot de passe temporaire n’est pas exporté](./media/tshoot-connect-password-hash-synchronization/phssingleobjecttemporarypassword.png)

#### <a name="results-of-last-attempt-to-synchronize-password-arent-available"></a>Les résultats de la dernière tentative de synchronisation de mot de passe ne sont pas disponibles

Par défaut, Azure AD Connect stocke les résultats des tentatives de synchronisation du hachage de mot de passe pendant sept jours. Si aucun résultat n’est disponible pour l’objet Active Directory sélectionné, l’avertissement suivant est retourné :

![Sortie de diagnostic pour un seul objet - aucun historique de synchronisation de mot de passe](./media/tshoot-connect-password-hash-synchronization/phssingleobjectnohistory.png)



## <a name="no-passwords-are-synchronized-troubleshoot-by-using-the-diagnostic-cmdlet"></a>Aucun mot de passe n’est synchronisé : résoudre les problèmes à l’aide de l’applet de commande de diagnostic

Vous pouvez utiliser l’applet de commande `Invoke-ADSyncDiagnostics` pour déterminer la raison pour laquelle aucun mot de passe n’est synchronisé.

> [!NOTE]
> L’applet de commande `Invoke-ADSyncDiagnostics` est disponible uniquement pour Azure AD Connect version 1.1.524.0 ou ultérieure.

### <a name="run-the-diagnostics-cmdlet"></a>Exécuter l’applet de commande de diagnostic

Pour résoudre les problèmes bloquant la synchronisation de tous les mots de passe

1. Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option **Exécuter en tant qu’administrateur**.

2. Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.

3. Exécutez `Import-Module ADSyncDiagnostics`.

4. Exécutez `Invoke-ADSyncDiagnostics -PasswordSync`.



## <a name="one-object-is-not-synchronizing-passwords-troubleshoot-by-using-the-diagnostic-cmdlet"></a>Un objet ne synchronise pas les mots de passe : résoudre les problèmes à l’aide de l’applet de commande de diagnostic

Vous pouvez utiliser l’applet de commande `Invoke-ADSyncDiagnostics` pour déterminer pourquoi un objet ne synchronise pas les mots de passe.

> [!NOTE]
> L’applet de commande `Invoke-ADSyncDiagnostics` est disponible uniquement pour Azure AD Connect version 1.1.524.0 ou ultérieure.

### <a name="run-the-diagnostics-cmdlet"></a>Exécuter l’applet de commande de diagnostic

Pour résoudre les problèmes liés à l’absence de synchronisation des mots de passe pour un utilisateur :

1. Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option **Exécuter en tant qu’administrateur**.

2. Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.

3. Exécutez `Import-Module ADSyncDiagnostics`.

4. Exécutez l’applet de commande suivante :

   ```
   Invoke-ADSyncDiagnostics -PasswordSync -ADConnectorName <Name-of-AD-Connector> -DistinguishedName <DistinguishedName-of-AD-object>
   ```

   Par exemple :

   ```powershell
   Invoke-ADSyncDiagnostics -PasswordSync -ADConnectorName "contoso.com" -DistinguishedName "CN=TestUserCN=Users,DC=contoso,DC=com"
   ```



## <a name="no-passwords-are-synchronized-manual-troubleshooting-steps"></a>Aucun mot de passe n’est synchronisé : étapes de dépannage manuel

Effectuez les étapes suivantes pour déterminer la raison pour laquelle aucun mot de passe n’est synchronisé :

1. Le serveur Connect est-il en [mode intermédiaire](how-to-connect-sync-staging-server.md) ? Un serveur en mode intermédiaire ne synchronise pas les mots de passe.

2. Exécutez le script dans la section [Obtenir l’état des paramètres de synchronisation de mot de passe](#get-the-status-of-password-sync-settings). Cela vous donne une vue d’ensemble de la configuration de la synchronisation de mot de passe.  

    ![Sortie de script PowerShell à partir des paramètres de synchronisation de mot de passe](./media/tshoot-connect-password-hash-synchronization/psverifyconfig.png)  

3. Si cette fonction n’est pas activée dans Azure AD ou si l’état du canal de synchronisation n’est pas activé, exécutez l’Assistant Installation d’Azure AD Connect. Sélectionnez **Personnaliser les options de synchronisation** et désélectionnez la synchronisation de mot de passe. Cette modification désactive temporairement la fonction. Réexécutez l’Assistant, puis réactivez la synchronisation de mot de passe. Réexécutez le script pour vérifier que la configuration est correcte.

4. Recherchez des erreurs dans le journal des événements. Recherchez les événements suivants, qui indiquent un problème :
    * Source : « Synchronisation d’annuaires » ID : 0, 611, 652, 655 Si vous voyez ces événements, vous avez un problème de connectivité. Le message du journal des événements contient des informations sur la forêt où vous avez un problème. Pour plus d’informations, consultez [Problème de connectivité](#connectivity problem).

5. Si vous ne voyez aucune pulsation ou que rien d’autre n’a fonctionné, exécutez [Déclencher une synchronisation complète de tous les mots de passe](#trigger-a-full-sync-of-all-passwords). Exécutez le script une seule fois.

6. Consultez la section Dépanner un objet qui bloque la synchronisation des mots de passe.

### <a name="connectivity-problems"></a>Problèmes de connectivité

Avez-vous une connectivité avec Azure AD ?

Le compte dispose-t-il des autorisations requises pour lire les hachages de mot de passe dans tous les domaines ? Si vous avez installé Connect à l’aide des paramètres Express, les autorisations doivent déjà être correctes. 

Si vous avez utilisé une installation personnalisée, définissez les autorisations manuellement en procédant comme suit :
    
1. Pour trouver le compte utilisé par le connecteur Active Directory, démarrez **Synchronization Service Manager**. 
 
2. Accédez à **Connecteurs** et recherchez la forêt locale Active Directory que vous dépannez. 
 
3. Sélectionnez le connecteur et cliquez sur **Propriétés**. 
 
4. Accédez à **Se connecter à la forêt Active Directory**.  
    
    ![Compte utilisé par le connecteur Active Directory](./media/tshoot-connect-password-hash-synchronization/connectoraccount.png)  
    Notez le nom d’utilisateur et le domaine où se trouve le compte.
    
5. Démarrez **Utilisateurs et ordinateurs Active Directory**, puis vérifiez que le compte que vous avez trouvé précédemment dispose des autorisations suivantes à la racine de tous les domaines de votre forêt :
    * Répliquer les changements d’annuaires
    * Répliquer les changements d’annuaire Tout

6. Les contrôleurs de domaine sont-ils accessibles pour Azure AD Connect ? Si le serveur Connect ne peut pas se connecter à tous les contrôleurs de domaine, vous devez configurer **Utiliser uniquement le contrôleur de domaine préféré**.  
    
    ![Contrôleur de domaine utilisé par le connecteur Active Directory](./media/tshoot-connect-password-hash-synchronization/preferreddc.png)  

7. Revenez à **Synchronization Service Manager** et **Configurer une partition d’annuaire**. 
 
8. Sélectionnez votre domaine dans **Sélectionner les partitions d’annuaire**, cochez la case **Utiliser uniquement les contrôleurs de domaine préférés**, puis cliquez sur **Configurer**. 

9. Dans la liste, entrez les contrôleurs de domaine que Connect doit utiliser pour la synchronisation de mot de passe. La même liste est également utilisée pour importer et exporter. Effectuez ces étapes pour tous vos domaines.

> [!NOTE]
> Pour appliquer ces modifications, redémarrez le service **Microsoft Azure AD Sync** (ADSync).

10. Si le script indique qu’il n’y a aucune pulsation, exécutez le script [Déclencher une synchronisation complète de tous les mots de passe](#trigger-a-full-sync-of-all-passwords).

## <a name="one-object-is-not-synchronizing-passwords-manual-troubleshooting-steps"></a>Un objet ne synchronise pas les mots de passe : étapes de dépannage manuel

Vous pouvez résoudre facilement les problèmes de synchronisation du hachage de mot de passe en consultant l’état d’un objet.

1. Dans **Utilisateurs et ordinateurs Active Directory**, recherchez l’utilisateur, puis vérifiez que la case **L’utilisateur doit changer le mot de passe à la prochaine ouverture de session** est décochée.  

    ![Mots de passe productifs Active Directory](./media/tshoot-connect-password-hash-synchronization/adprodpassword.png)  

    Si elle est cochée, demandez à l’utilisateur de se connecter et de changer son mot de passe. Les mots de passe temporaires ne sont pas synchronisés avec Azure AD.

2. Si le mot de passe semble correct dans Active Directory, suivez l’utilisateur dans le moteur de synchronisation. En suivant l’utilisateur de l’annuaire Active Directory local vers Azure AD, vous pouvez voir si un message d’erreur descriptif apparaît sur l’objet.

    a. Démarrez [Synchronization Service Manager](how-to-connect-sync-service-manager-ui.md).

    b. Cliquez sur **Connecteurs**.

    c. Sélectionnez le **Connecteur Active Directory** où se trouve l’utilisateur.

    d. Sélectionnez **Search Connector Space**(Rechercher l’espace de connecteur).

    e. Dans la zone **Étendue**, sélectionnez **DN ou ancre**, puis entrez le nom unique complet de l’utilisateur que vous dépannez.

    ![Rechercher un utilisateur dans l’espace de connecteur avec le nom unique](./media/tshoot-connect-password-hash-synchronization/searchcs.png)  

    f. Localisez l’utilisateur que vous recherchez et cliquez sur **Propriétés** pour voir tous ses attributs. Si l’utilisateur n’est pas dans les résultats de la recherche, vérifiez vos [règles de filtrage](how-to-connect-sync-configure-filtering.md) et assurez-vous que vous exécutez [Appliquer et vérifier les modifications](how-to-connect-sync-configure-filtering.md#apply-and-verify-changes) pour que l’utilisateur apparaisse dans Connect.

    g. Pour afficher les détails de synchronisation de mot de passe de l’objet pour la semaine écoulée, cliquez sur **Journal**.  

    ![Détails d’un journal d’objet](./media/tshoot-connect-password-hash-synchronization/csobjectlog.png)  

    Si le journal de l’objet est vide, cela signifie qu’Azure AD Connect n’a pas pu lire le hachage de mot de passe à partir d’Active Directory. Continuez la résolution des problèmes avec Erreurs de connectivité. Si vous voyez une valeur autre que **Réussite**, consultez le tableau dans [Journal de synchronisation de mot de passe](#password-sync-log).

    h. Sélectionnez l’onglet **Lignage** et vérifiez qu’au moins une règle de synchronisation dans la colonne **PasswordSync** est **True**. Dans la configuration par défaut, le nom de la règle de synchronisation est **In from AD - User AccountEnabled**.  

    ![Informations de lignage sur un utilisateur](./media/tshoot-connect-password-hash-synchronization/cspasswordsync.png)  

    i. Cliquez sur **Propriétés de l’objet métaverse** pour afficher une liste d’attributs de l’utilisateur.  

    ![Capture d’écran montrant la liste des attributs utilisateur pour les propriétés d’objet métaverse.](./media/tshoot-connect-password-hash-synchronization/mvpasswordsync.png)  

    Vérifiez qu’aucun attribut **cloudFiltered** n’est présent. Assurez-vous que les attributs de domaine (domainFQDN et domainNetBios) ont les valeurs attendues.

    j. Cliquez sur l’onglet **Connecteurs**. Vérifiez que les connecteurs à Active Directory local et à Azure AD sont visibles.

    ![Informations de métaverse](./media/tshoot-connect-password-hash-synchronization/mvconnectors.png)  

    k. Sélectionnez la ligne qui représente Azure AD, cliquez sur **Propriétés**, puis sur l’onglet **Lignage**. L’objet d’espace connecteur doit avoir une règle sortante dans la colonne **PasswordSync** définie sur **True**. Dans la configuration par défaut, le nom de la règle de synchronisation est **Out to AAD - User Join**.  

    ![Boîte de dialogue Propriétés de l’objet espace connecteur](./media/tshoot-connect-password-hash-synchronization/cspasswordsync2.png)  

### <a name="password-sync-log"></a>Journal de synchronisation de mot de passe

La colonne Statut peut avoir les valeurs suivantes :

| Statut | Description |
| --- | --- |
| Succès |Le mot de passe a été correctement synchronisé. |
| FilteredByTarget |Le mot de passe est défini sur **L’utilisateur doit changer le mot de passe à la prochaine ouverture de session**. Mot de passe n'a pas été synchronisé. |
| NoTargetConnection |Aucun objet dans le métaverse ou dans l'espace de connecteur Azure AD. |
| SourceConnectorNotPresent |Aucun objet trouvé dans l'espace de connecteur Active Directory local. |
| TargetNotExportedToDirectory |L'objet dans l'espace de connecteur Azure AD n'a pas encore été exporté. |
| MigratedCheckDetailsForMoreInfo |L'entrée de journal a été créée avant la version 1.0.9125.0 et est affichée dans son état hérité. |
| Error |Le service a renvoyé une erreur inconnue. |
| Unknown |Une erreur s’est produite lors de la tentative de traitement d’un lot de hachages de mot de passe.  |
| MissingAttribute |Des attributs spécifiques (par exemple, le hachage Kerberos) requis par Azure AD Domain Services ne sont pas disponibles. |
| RetryRequestedByTarget |Des attributs spécifiques (par exemple, le hachage Kerberos) requis par Azure AD Domain Services n’étaient pas disponibles précédemment. Une tentative pour resynchroniser le hachage de mot de passe de l’utilisateur est effectuée. |

## <a name="scripts-to-help-troubleshooting"></a>Scripts pour faciliter le dépannage

### <a name="get-the-status-of-password-sync-settings"></a>Obtenir l’état des paramètres de synchronisation de mot de passe

```powershell
Import-Module ADSync
$connectors = Get-ADSyncConnector
$aadConnectors = $connectors | Where-Object {$_.SubType -eq "Windows Azure Active Directory (Microsoft)"}
$adConnectors = $connectors | Where-Object {$_.ConnectorTypeName -eq "AD"}
if ($aadConnectors -ne $null -and $adConnectors -ne $null)
{
    if ($aadConnectors.Count -eq 1)
    {
        $features = Get-ADSyncAADCompanyFeature -ConnectorName $aadConnectors[0].Name
        Write-Host
        Write-Host "Password sync feature enabled in your Azure AD directory: "  $features.PasswordHashSync
        foreach ($adConnector in $adConnectors)
        {
            Write-Host
            Write-Host "Password sync channel status BEGIN ------------------------------------------------------- "
            Write-Host
            Get-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector.Name
            Write-Host
            $pingEvents =
                Get-EventLog -LogName "Application" -Source "Directory Synchronization" -InstanceId 654  -After (Get-Date).AddHours(-3) |
                    Where-Object { $_.Message.ToUpperInvariant().Contains($adConnector.Identifier.ToString("D").ToUpperInvariant()) } |
                    Sort-Object { $_.Time } -Descending
            if ($pingEvents -ne $null)
            {
                Write-Host "Latest heart beat event (within last 3 hours). Time " $pingEvents[0].TimeWritten
            }
            else
            {
                Write-Warning "No ping event found within last 3 hours."
            }
            Write-Host
            Write-Host "Password sync channel status END ------------------------------------------------------- "
            Write-Host
        }
    }
    else
    {
        Write-Warning "More than one Azure AD Connectors found. Please update the script to use the appropriate Connector."
    }
}
Write-Host
if ($aadConnectors -eq $null)
{
    Write-Warning "No Azure AD Connector was found."
}
if ($adConnectors -eq $null)
{
    Write-Warning "No AD DS Connector was found."
}
Write-Host
```

#### <a name="trigger-a-full-sync-of-all-passwords"></a>Déclencher une synchronisation complète de tous les mots de passe

> [!NOTE]
> Exécutez ce script une seule fois. Si vous devez l’exécuter plusieurs fois, le problème est ailleurs. Pour résoudre le problème, contactez le support technique Microsoft.

Vous pouvez déclencher une synchronisation complète de tous les mots de passe à l’aide du script suivant :

```powershell
$adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"
$aadConnector = "<CASE SENSITIVE AAD CONNECTOR NAME>"
Import-Module adsync
$c = Get-ADSyncConnector -Name $adConnector
$p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null
$p.Value = 1
$c.GlobalParameters.Remove($p.Name)
$c.GlobalParameters.Add($p)
$c = Add-ADSyncConnector -Connector $c
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $false
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $aadConnector -Enable $true
```

## <a name="next-steps"></a>Étapes suivantes

* [Implémenter la synchronisation du hachage de mot de passe avec Azure AD Connect Sync](how-to-connect-password-hash-synchronization.md)
* [Synchronisation Azure AD Connect : personnaliser les options de synchronisation](how-to-connect-sync-whatis.md)
* [Intégration des identités locales dans Azure Active Directory](whatis-hybrid-identity.md)
