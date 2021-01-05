---
title: Confidentialité des utilisateurs et authentification directe Azure Active Directory | Microsoft Docs
description: Cet article traite de l’authentification directe Azure Active Directory (Azure AD) et de la conformité RGPD.
services: active-directory
keywords: Authentification directe Azure AD Connect, RGPD, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/23/2018
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 396344ba90aa3850d7d23dc40d6df95f6d1f6c3f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996574"
---
# <a name="user-privacy-and-azure-active-directory-pass-through-authentication"></a>Confidentialité des utilisateurs et authentification directe Azure Active Directory


[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

## <a name="overview"></a>Vue d’ensemble

L’authentification directe Azure AD crée les types de journaux suivants, qui peuvent contenir des données personnelles :

- Fichiers journaux des traces Azure AD Connect
- Fichiers journaux des traces de l’Agent d’authentification
- Fichiers journaux des événements Windows

Améliorez la confidentialité des utilisateurs pour l’authentification directe de deux manières :

1.  Sur demande, en extrayant les données d’une personne, puis en supprimant ces données des installations
2.  En garantissant qu’aucune donnée n’est conservée plus de 48 heures

Nous recommandons vivement la deuxième option, car elle est plus facile à implémenter et à gérer. Voici les instructions pour chaque type de journal :

### <a name="delete-azure-ad-connect-trace-log-files"></a>Supprimer les fichiers journaux des traces Azure AD Connect

Vérifiez le contenu du dossier **%ProgramData%\AADConnect** et supprimez le contenu des journaux de traces (fichiers **trace -\*.log**) de ce dossier dans les 48 heures suivant l’installation ou la mise à niveau d’Azure AD Connect ou la modification de la configuration de l’authentification directe, car cette action peut créer des données couvertes par la réglementation RGPD.

>[!IMPORTANT]
>Ne supprimez pas le fichier **PersistedState.xml** de ce dossier, car il est sert à conserver l’état de l’installation précédente d’Azure AD Connect et est utilisé quand une mise à niveau est effectuée. Ce fichier ne contiendra jamais de données relatives à un individu et ne doit jamais être supprimé.

Vous pouvez passer en revue et supprimer ces fichiers journaux des traces à l’aide de l’Explorateur Windows, ou vous pouvez utiliser le script PowerShell suivant pour effectuer les actions nécessaires :

```
$Files = ((Get-Item -Path "$env:programdata\aadconnect\trace-*.log").VersionInfo).FileName 
 
Foreach ($file in $Files) { 
    {Remove-Item -Path $File -Force} 
}
```

Enregistrez le script dans un fichier avec l’extension « .PS1 ». Exécutez ce script selon les besoins.

Pour en savoir plus sur les exigences RGPD relatives à Azure AD Connect, consultez [cet article](reference-connect-user-privacy.md).

### <a name="delete-authentication-agent-event-logs"></a>Supprimer les journaux des événements de l’Agent d’authentification

Ce produit peut également créer des **Journaux des événements Windows**. Pour plus d’informations, consultez [cet article](/windows/win32/wes/windows-event-log).

Pour afficher les journaux d’activité relatifs à l’agent d’authentification directe, ouvrez l’application **Observateur d’événements** sur le serveur et cherchez dans **Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin**.

### <a name="delete-authentication-agent-trace-log-files"></a>Supprimer les fichiers journaux des traces de l’Agent d’authentification

Vous devez régulièrement vérifier le contenu de **%ProgramData%\Microsoft\Azure AD Connect Authentication Agent\Trace** et supprimer le contenu de ce dossier toutes les 48 heures. 

>[!IMPORTANT]
>Si le service Agent d’authentification est en cours d’exécution, vous ne pourrez pas supprimer le fichier journal actuel dans le dossier. Arrêtez le service avant de réessayer. Pour éviter les échecs de connexion de l’utilisateur, vous devez déjà avoir configuré l’authentification directe pour une [haute disponibilité](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability).

Vous pouvez passer en revue et supprimer ces fichiers à l’aide de l’Explorateur Windows, ou vous pouvez utiliser le script qui suit pour effectuer les actions nécessaires :

```
$Files = ((Get-childitem -Path "$env:programdata\microsoft\azure ad connect authentication agent\trace" -Recurse).VersionInfo).FileName 
 
Foreach ($file in $files) { 
    {Remove-Item -Path $File -Force} 
}
```

Pour planifier une exécution de ce script toutes les 48 heures, effectuez les étapes ci-dessous :

1.  Enregistrez le script dans un fichier avec l’extension « .PS1 ».
2.  Ouvrez le **Panneau de configuration**, puis cliquez sur **Système et sécurité**.
3.  Sous le titre **Outils d’administration**, cliquez sur **Tâches planifiées**.
4.  Dans le **Planificateur de tâches**, cliquez avec le bouton droit sur **Bibliothèque du Planificateur de tâches**, puis cliquez sur **Créer une tâche de base**.
5.  Entrez le nom de la nouvelle tâche, puis cliquez sur **Suivant**.
6.  Sélectionnez **Tous les jours** pour le **Déclencheur de tâche**, puis cliquez sur **Suivant**.
7.  Choisissez de répéter tous les deux jours, puis cliquez sur **Suivant**.
8.  Sélectionnez **Démarrer un programme** comme action, puis cliquez sur **Suivant**.
9.  Tapez **PowerShell** dans le champ Programme/script. Ensuite, dans le champ **Ajouter des arguments (facultatif)** , entrez le chemin complet du script que vous avez créé précédemment, puis cliquez sur **Suivant**.
10. L’écran suivant montre un récapitulatif de la tâche que vous êtes sur le point de créer. Vérifiez les valeurs, puis cliquez sur **Terminer** pour créer la tâche :
 
### <a name="note-about-domain-controller-logs"></a>Remarque concernant les journaux d’activité des contrôleurs de domaine

Si l’enregistrement d’audit est activé, ce produit peut générer des journaux d’activité de sécurité pour vos contrôleurs de domaine. Pour en savoir plus sur la configuration des stratégies d’audit, consultez cet [article](/previous-versions/tn-archive/dd277403(v=technet.10)).

## <a name="next-steps"></a>Étapes suivantes
* [Lire la politique de confidentialité Microsoft sur le Centre de confidentialité](https://www.microsoft.com/trustcenter)
* [**Résolution des problèmes**](tshoot-connect-pass-through-authentication.md) : découvrez comment résoudre les problèmes courants susceptibles de survenir avec cette fonctionnalité.