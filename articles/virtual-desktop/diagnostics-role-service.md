---
title: Problèmes de diagnostic de Windows Virtual Desktop – Azure
description: Comment utiliser la fonctionnalité de diagnostic de Windows Virtual Desktop pour diagnostiquer des problèmes.
author: Heidilohr
ms.topic: troubleshooting
ms.date: 09/21/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 70676bd1a07acdfcbba071a906b390ed66d70074
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91279856"
---
# <a name="identify-and-diagnose-windows-virtual-desktop-issues"></a>Identifier et diagnostiquer les problèmes liés à Windows Virtual Desktop

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/diagnostics-role-service-2019.md).

Windows Virtual Desktop offre une fonctionnalité de diagnostic qui permet à l’administrateur d’identifier les problèmes via une seule interface. Pour en savoir plus sur les fonctionnalités de diagnostic de Windows Virtual Desktop, consultez [Utiliser Log Analytics pour la fonctionnalité de diagnostic](diagnostics-log-analytics.md).

Les connexions qui n’atteignent pas Windows Virtual Desktop ne figureront pas dans les résultats de diagnostic, car le service de rôle de diagnostics fait partie de Windows Virtual Desktop. Des problèmes de connexion à Windows Virtual Desktop peuvent survenir lorsque l’utilisateur rencontre des problèmes de connectivité au réseau.

## <a name="common-error-scenarios"></a>Scénarios d’erreur courants

La table WVDErrors effectue le suivi des erreurs pour tous les types d’activités. La colonne appelée « ServiceError » fournit un indicateur supplémentaire : « True » ou « False ». Cet indicateur vous signale si l’erreur est liée au service.

* Si la valeur est « True », l’équipe du service a peut-être déjà étudié ce problème. Si cela impacte l’expérience utilisateur et si l’erreur s’affiche un grand nombre de fois, nous vous recommandons d’envoyer un ticket de support pour Windows Virtual Desktop.
* Si la valeur est « False », il s’agit peut-être d’une mauvaise configuration que vous pouvez corriger. Le message d’erreur peut vous indiquer par où commencer.

Le tableau suivant répertorie les erreurs courantes que pourraient rencontrer vos administrateurs.

>[!NOTE]
>Cette liste inclut les erreurs les plus courantes et est mise à jour régulièrement. Pour être sûr de disposer des informations les plus à jour, veillez à consulter cet article au moins une fois par mois.

## <a name="management-errors"></a>Erreurs de gestion

|Message d’erreur|Solution suggérée|
|---|---|
|La création de la clé d’inscription a échoué. |Impossible de créer le jeton d’inscription. Réessayez de le créer avec un délai d’expiration plus court (entre 1 heure et 1 mois). |
|La suppression de la clé d’inscription a échoué.|Impossible de supprimer le jeton d’inscription. Réessayez de le supprimer. Si cela ne fonctionne toujours pas, utilisez PowerShell pour vérifier si le jeton est toujours présent. Si c’est le cas, supprimez-le avec PowerShell.|
|Le changement du mode maintenance de l’hôte de session a échoué |Impossible de modifier le mode maintenance sur la machine virtuelle. Vérifiez l’état de la machine virtuelle. Si la machine virtuelle est indisponible, il n’est pas possible de modifier le mode maintenance.|
|La déconnexion de sessions utilisateur a échoué |Impossible de déconnecter l’utilisateur de la machine virtuelle. Vérifiez l’état de la machine virtuelle. Si la machine virtuelle est indisponible, il n’est pas possible de déconnecter la session utilisateur. Si la machine virtuelle est disponible, vérifiez l’état de la session utilisateur pour voir si elle est déconnectée. |
|La fermeture de session de tous les utilisateurs de l’hôte de session a échoué |Impossible de déconnecter les utilisateurs de la machine virtuelle. Vérifiez l’état de la machine virtuelle. S’il n’est pas disponible, il n’est pas possible de déconnecter les utilisateurs. Vérifiez l’état de la session utilisateur pour voir s’ils sont déconnectés. Vous pouvez forcer la déconnexion avec PowerShell. |
|La dissociation de l’utilisateur du groupe d’applications a échoué|Impossible d’annuler la publication d’un groupe d’applications pour un utilisateur. Vérifiez si l’utilisateur est disponible sur Azure AD. Vérifiez si l’utilisateur fait partie d’un groupe d’utilisateurs sur lequel le groupe d’applications est publié. |
|Une erreur s’est produite lors de la récupération des emplacements disponibles |Vérifiez l’emplacement de la machine virtuelle utilisée dans l’Assistant Création d’un pool d’hôtes. Si l’image n’est pas disponible dans cet emplacement, ajoutez une image à cet emplacement ou choisissez un autre emplacement de machine virtuelle. |

### <a name="connection-error-codes"></a>Codes d’erreur de connexion

|Code numérique|Code d'erreur|Solution suggérée|
|---|---|---|
|-2147467259|ConnectionFailedAdTrustedRelationshipFailure|L’hôte de la session n’est pas correctement joint à Active Directory.|
|-2146233088|ConnectionFailedUserHasValidSessionButRdshIsUnhealthy|Les connexions ont échoué car l’hôte de la session est indisponible. Vérifiez l’intégrité de l’hôte de la session.|
|-2146233088|ConnectionFailedClientDisconnect|Si vous rencontrez souvent cette erreur, assurez-vous que l’ordinateur est connecté au réseau.|
|-2146233088|ConnectionFailedNoHealthyRdshAvailable|La session à laquelle l’utilisateur hôte a tenté de se connecter n’est pas viable. Déboguez la machine virtuelle.|
|-2146233088|ConnectionFailedUserNotAuthorized|L’utilisateur n’est pas autorisé à accéder à l’application publiée ou au bureau. Cette erreur peut apparaître après la suppression de ressources publiées par l’administrateur. Demandez à l’utilisateur d’actualiser le flux dans l’application de bureau à distance.|
|2|FileNotFound|L’application à laquelle l’utilisateur tente d’accéder est incorrectement installée ou son chemin d’accès est invalide.<br><br>Lors de la publication de nouvelles applications alors que l’utilisateur dispose d’une session active, celui-ci ne peut pas accéder à cette application. Vous devez arrêter et redémarrer la session pour que l’utilisateur puisse accéder à l’application. |
|3|InvalidCredentials|Le nom d’utilisateur ou le mot de passe saisi par l’utilisateur ne correspond à aucun nom d’utilisateur ou mot de passe existant. Vérifiez que vous n’avez fait aucune faute de frappe et réessayez.|
|8|ConnectionBroken|La connexion entre le client et passerelle ou serveur a été interrompue. Aucune action n’est nécessaire, sauf si cela se produit de manière inattendue.|
|14|UnexpectedNetworkDisconnect|La connexion au réseau a été perdue. Demandez à l’utilisateur de se reconnecter.|
|24|ReverseConnectFailed|La machine virtuelle hôte n’a aucune une ligne directe vers la passerelle Bureau à distance. Vérifiez que l’adresse IP de la passerelle peut être résolue.|

## <a name="error-cant-add-user-assignments-to-an-app-group"></a>Erreur : Impossible d’ajouter des affectations d’utilisateurs à un groupe d’applications

Après l’affectation d’un utilisateur à un groupe d’applications, le portail Azure affiche un avertissement indiquant « Fin de session » ou « Problèmes d’authentification - Extension Microsoft_Azure_WVD ». La page d’affectation n’est alors pas chargée, et ensuite, les pages cessent de se charger dans le portail Azure (par exemple, Azure Monitor, Log Analytics, Service Health, etc.).

**Cause :** Il y a un problème avec la stratégie d’accès conditionnel. Le portail Azure tente d’obtenir un jeton pour Microsoft Graph, qui dépend de SharePoint Online. Le client dispose d’une stratégie d’accès conditionnel appelée « Microsoft Office 365 Data Storage Terms of Use », qui oblige les utilisateurs à accepter les conditions d’utilisation pour accéder au stockage des données. Toutefois, les utilisateurs ne se sont pas encore connectés, donc le portail Azure ne peut pas récupérer le jeton.

**Correctif :** Avant de se connecter au portail Azure, l’administrateur doit d’abord se connecter à SharePoint et accepter les conditions d’utilisation. Ensuite, il devrait être en mesure de se connecter au portail Azure normalement.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les rôles dans Windows Virtual Desktop, consultez [Environnement Windows Virtual Desktop](environment-setup.md).

Pour obtenir une liste des cmdlets PowerShell disponibles pour Windows Virtual Desktop, consultez le document de [référence PowerShell](/powershell/windows-virtual-desktop/overview).
