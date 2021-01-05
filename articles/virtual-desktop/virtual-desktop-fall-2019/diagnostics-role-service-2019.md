---
title: Problèmes de diagnostic de Windows Virtual Desktop (classique) – Azure
description: Comment utiliser la fonctionnalité de diagnostic de Windows Virtual Desktop (classique) pour diagnostiquer des problèmes.
author: Heidilohr
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 7e652f04b42b132e7c1307503b1764dda7b2036b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88009339"
---
# <a name="identify-and-diagnose-issues-in-windows-virtual-desktop-classic"></a>Identifier et diagnostiquer les problèmes dans Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../diagnostics-role-service.md).

Windows Virtual Desktop offre une fonctionnalité de diagnostic qui permet à l’administrateur d’identifier les problèmes via une seule interface. Les rôles Windows Virtual Desktop journalisent une activité de diagnostic à chaque fois qu’un utilisateur interagit avec le système. Chaque journal contient des informations importantes telles que les rôles Windows Virtual Desktop impliqués dans la transaction, les messages d’erreur, les informations d’abonnés, et les informations d’utilisateurs. Les activités de diagnostics sont créées par des actions de l’administrateur et de l’utilisateur, et peuvent être divisées en trois catégories principales :

* Activités d’ajout de flux : l’utilisateur déclenche ces activités à chaque fois qu’il essaie de se connecter à son flux via des applications Bureau à distance Microsoft.
* Activités de connexion : l’utilisateur déclenche ces activités à chaque fois qu’il essaie de se connecter à un bureau ou RemoteApp via des applications Bureau à distance Microsoft.
* Activités de gestion : l’administrateur déclenche ces activités à chaque fois qu’il effectue des opérations de gestion sur le système, telles que la création de pools d’hôte, l’attribution d’utilisateurs à des groupes d’applications et la création d’attributions de rôles.

Les connexions qui n’atteignent pas Windows Virtual Desktop ne figureront pas dans les résultats de diagnostic, car le service de rôle de diagnostics fait partie de Windows Virtual Desktop. Des problèmes de connexion à Windows Virtual Desktop peuvent survenir lorsque l’utilisateur rencontre des problèmes de connectivité au réseau.

Tout d’abord, si vous ne l’avez pas déjà fait, [téléchargez et importez le module PowerShell Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) à utiliser dans votre session PowerShell. Exécutez ensuite l’applet de commande suivante pour vous connecter à votre compte :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="diagnose-issues-with-powershell"></a>Diagnostiquer les problèmes avec PowerShell

Windows Virtual Desktop Diagnostics utilise une seule cmdlet PowerShell mais contient de nombreux paramètres facultatifs pour aider à limiter et isoler les problèmes. Les sections suivantes répertorient les cmdlets que vous pouvez exécuter pour diagnostiquer les problèmes. La plupart des filtres peuvent être appliqués ensemble. Les valeurs figurant entre crochets, comme `<tenantName>`, doivent être remplacées par les valeurs qui s’appliquent à votre situation.

>[!IMPORTANT]
>La fonctionnalité de diagnostic est destinée à la résolution des problèmes d’un seul utilisateur. Toutes les requêtes utilisant PowerShell doivent inclure l’un des paramètres *-UserName* ou *-ActivityID*. Pour les fonctionnalités de surveillance, utilisez Log Analytics. Pour plus d’informations sur l’envoi de données de diagnostic à votre espace de travail, voir [Utiliser Log Analytics pour la fonctionnalité de diagnostic](diagnostics-log-analytics-2019.md).

### <a name="filter-diagnostic-activities-by-user"></a>Filtrer les activités de diagnostic par utilisateur

Le paramètre **-UserName** retourne une liste d’activités de diagnostic initiées par l’utilisateur spécifié, comme montré dans l’exemple de cmdlet suivant.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN>
```

Le paramètre **-UserName** peut aussi être combiné à d’autres paramètres de filtrage facultatifs.

### <a name="filter-diagnostic-activities-by-time"></a>Filtrer les activités de diagnostic par période

Vous pouvez filtrer la liste des activités de diagnostic avec les paramètres **-StartTime** et **-EndTime**. Le paramètre **-StartTime** retourne une liste des activités de diagnostic à partir d’une date spécifique, comme montré dans l’exemple suivant.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -StartTime "08/01/2018"
```

Le paramètre **-EndTime** peut être ajouté à une cmdlet avec le paramètre **-StartTime** pour spécifier une période pour laquelle vous souhaitez recevoir des résultats. L’exemple de cmdlet suivant retourne une liste d’activités de diagnostic entre le 1er août et le 10 août.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -StartTime "08/01/2018" -EndTime "08/10/2018"
```

Les paramètres **-StartTime** et **-EndTime** peuvent aussi être combinés à d’autres paramètres de filtrage facultatifs.

### <a name="filter-diagnostic-activities-by-activity-type"></a>Filtrer les activités de diagnostic par type d’activité

Vous pouvez aussi filtrer les activités de diagnostic par type d’activité avec le paramètre **-ActivityType**. La cmdlet suivante retourne une liste de connexions d’utilisateur :

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -ActivityType Connection
```

La cmdlet suivante retourne une liste de tâches de gestion d’administrateur :

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityType Management
```

La cmdlet **Get-RdsDiagnosticActivities** ne prend actuellement pas en charge la spécifications de Feed (Flux) comme ActivityType (type d’activité).

### <a name="filter-diagnostic-activities-by-outcome"></a>Filtrer les activités de diagnostic par résultat

Vous pouvez filtrer la liste des activités de diagnostic par résultat avec le paramètre **-Outcome**. L’exemple de cmdlet suivant retourne une liste d’activités de diagnostic réussies.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -UserName <UserUPN> -Outcome Success
```

L’exemple de cmdlet suivant retourne une liste d’activités de diagnostic échouées.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -Outcome Failure
```

Le paramètre **-Outcome** peut aussi être combiné à d’autres paramètres de filtrage facultatifs.

### <a name="retrieve-a-specific-diagnostic-activity-by-activity-id"></a>Récupérer une activité de diagnostic spécifique par ID d’activité

Le paramètre **-ActivityId** retourne une activité de diagnostic spécifique (s’il y en a une), comme montré dans l’exemple de cmdlet suivant.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityId <ActivityIdGuid>
```

### <a name="view-error-messages-for-a-failed-activity-by-activity-id"></a>Afficher les messages d’erreur pour une activité ayant échoué par ID d’activité

Pour afficher les messages d’erreur relatifs à une activité ayant échoué, vous devez exécuter la cmdlet avec le paramètre **-Detailed**. Vous pouvez afficher la liste des erreurs en exécutant la cmdlet **Select-Object**.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantname> -ActivityId <ActivityGuid> -Detailed | Select-Object -ExpandProperty Errors
```

### <a name="retrieve-detailed-diagnostic-activities"></a>Récupérer des activités de diagnostics détaillées

Le paramètre **-Detailed** fournit des informations supplémentaires pour chaque activité de diagnostic retournée. Le format de chaque activité varie en fonction de son type d’activité. Le paramètre **-Detailed** peut être ajouté à n’importe quelle requête **Get-RdsDiagnosticActivities**, comme montré dans l’exemple suivant.

```powershell
Get-RdsDiagnosticActivities -TenantName <tenantName> -ActivityId <ActivityGuid> -Detailed
```

## <a name="common-error-scenarios"></a>Scénarios d’erreur courants

Les scénarios d’erreur sont classés en interne au service et en externe à Windows Virtual Desktop.

* Problème interne : spécifie les scénarios qui ne peuvent pas être atténués par l’administrateur client et qui doivent être résolus en tant que problème de support. Lorsque vous fournissez des commentaires via [Windows Virtual Desktop Tech Community](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop), incluez l’ID d’activité, puis spécifiez une période approximative pour l’erreur.
* Problème externe : fait référence aux scénarios qui peuvent être atténués par l’administrateur système. Il s’agit de problèmes externes à Windows Virtual Desktop.

Le tableau suivant répertorie les erreurs courantes que pourraient rencontrer vos administrateurs.

>[!NOTE]
>Cette liste inclut les erreurs les plus courantes et est mise à jour régulièrement. Pour être sûr de disposer des informations les plus à jour, veillez à consulter cet article au moins une fois par mois.

### <a name="external-management-error-codes"></a>Codes d’erreur de gestion externe

|Code numérique|Code d'erreur|Solution suggérée|
|---|---|---|
|1322|ConnectionFailedNoMappingOfSIDinAD|L’utilisateur n’est pas membre d’Azure Active Directory. Suivez les instructions du [Centre d’administration Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) pour les ajouter.|
|3|UnauthorizedAccess|L’utilisateur qui a tenté d’exécuter la cmdlet PowerShell d’administration n’est pas autorisé à le faire ou a mal tapé son nom d’utilisateur.|
|1 000|TenantNotFound|Le nom de l’abonné que vous avez saisi ne correspond à aucun abonné existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|1006|TenantCannotBeRemovedHasSessionHostPools|Vous ne pouvez pas supprimer un abonné tant qu’il contient des objets. Supprimez d’abord les pools d’hôte de la session, puis réessayez.|
|2000|HostPoolNotFound|Le nom du pool d’hôte que vous avez saisi ne correspond à aucun pool d’hôte existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|2005|HostPoolCannotBeRemovedHasApplicationGroups|Vous ne pouvez pas supprimer un pool d’hôte tant qu’il contient des objets. Supprimez d’abord tous les groupes d’applications dans le pool d’hôte.|
|2004|HostPoolCannotBeRemovedHasSessionHosts|Supprimez tous les hôtes de la session avant de supprimer le pool d’hôte de la session.|
|5001|SessionHostNotFound|L’hôte de la session interrogé est peut-être hors ligne. Vérifiez l’état du pool d’hôte.|
|5008|SessionHostUserSessionsExist |Vous devez déconnecter tous les utilisateurs de l’hôte de la session avant d’exécuter l’activité de gestion souhaitée.|
|6000|AppGroupNotFound|Le nom du groupe d’applications que vous avez saisi ne correspond à aucun groupe d’applications existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|6022|RemoteAppNotFound|Le nom de l’application RemoteApp que vous avez saisi ne correspond à aucune application RemoteApp. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|6010|PublishedItemsExist|Le nom de la ressource que vous tentez de publier est identique à une ressource existante. Changez le nom de la ressource, puis réessayez.|
|7002|NameNotValidWhiteSpace|N’insérez pas d’espace blanc dans le nom.|
|8000|InvalidAuthorizationRoleScope|Le nom du rôle que vous avez saisi ne correspond à aucun nom de rôle existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez. |
|8001|UserNotFound |Le nom d’utilisateur que vous avez saisi ne correspond à aucun nom d’utilisateur existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|8005|UserNotFoundInAAD |Le nom d’utilisateur que vous avez saisi ne correspond à aucun nom d’utilisateur existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|8008|TenantConsentRequired|Suivez les instructions [ici](tenant-setup-azure-active-directory.md#grant-permissions-to-windows-virtual-desktop) pour donner votre consentement à votre abonné.|

### <a name="external-connection-error-codes"></a>Codes d’erreur de connexion externe

|Code numérique|Code d'erreur|Solution suggérée|
|---|---|---|
|-2147467259|ConnectionFailedAdErrorNoSuchMember|L’utilisateur n’est pas membre d’Active Directory. Suivez les instructions du [Centre d’administration Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) pour les ajouter.|
|-2147467259|ConnectionFailedAdTrustedRelationshipFailure|L’hôte de la session n’est pas correctement joint à Active Directory.|
|-2146233088|ConnectionFailedUserHasValidSessionButRdshIsUnhealthy|Les connexions ont échoué car l’hôte de la session est indisponible. Vérifiez l’intégrité de l’hôte de la session.|
|-2146233088|ConnectionFailedClientDisconnect|Si vous rencontrez souvent cette erreur, assurez-vous que l’ordinateur est connecté au réseau.|
|-2146233088|ConnectionFailedNoHealthyRdshAvailable|La session à laquelle l’utilisateur hôte a tenté de se connecter n’est pas viable. Déboguez la machine virtuelle.|
|-2146233088|ConnectionFailedUserNotAuthorized|L’utilisateur n’est pas autorisé à accéder à l’application publiée ou au bureau. Cette erreur peut apparaître après la suppression de ressources publiées par l’administrateur. Demandez à l’utilisateur d’actualiser le flux dans l’application de bureau à distance.|
|2|FileNotFound|L’application à laquelle l’utilisateur tente d’accéder est incorrectement installée ou son chemin d’accès est invalide.|
|3|InvalidCredentials|Le nom d’utilisateur ou le mot de passe saisi par l’utilisateur ne correspond à aucun nom d’utilisateur ou mot de passe existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|8|ConnectionBroken|La connexion entre le client et passerelle ou serveur a été interrompue. Aucune action n’est nécessaire, sauf si cela se produit de manière inattendue.|
|14|UnexpectedNetworkDisconnect|La connexion au réseau a été perdue. Demandez à l’utilisateur de se reconnecter.|
|24|ReverseConnectFailed|La machine virtuelle hôte n’a aucune une ligne directe vers la passerelle Bureau à distance. Vérifiez que l’adresse IP de la passerelle peut être résolue.|
|1322|ConnectionFailedNoMappingOfSIDinAD|L’utilisateur n’est pas membre d’Active Directory. Suivez les instructions du [Centre d’administration Active Directory](/windows-server/identity/ad-ds/get-started/adac/active-directory-administrative-center) pour les ajouter.|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les rôles dans Windows Virtual Desktop, consultez [Environnement Windows Virtual Desktop](environment-setup-2019.md).

Pour obtenir une liste des cmdlets PowerShell disponibles pour Windows Virtual Desktop, consultez le document de [référence PowerShell](/powershell/windows-virtual-desktop/overview).
