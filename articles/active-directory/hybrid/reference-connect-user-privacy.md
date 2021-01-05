---
title: Azure AD Connect et confidentialité des utilisateurs | Microsoft Docs
description: Ce document explique comment obtenir la conformité RGPD avec Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 05/21/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8f1d1bf35d03f0f7c4d11e4cc7e9b6f98f38d8ab
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997696"
---
# <a name="user-privacy-and-azure-ad-connect"></a>Confidentialité des utilisateurs et Azure AD Connect 

[!INCLUDE [Privacy](../../../includes/gdpr-intro-sentence.md)]

>[!NOTE] 
>Cet article porte sur Azure AD Connect et la confidentialité des utilisateurs.  Pour plus d’informations sur Azure AD Connect Health et la confidentialité des utilisateurs, consultez [cet article](reference-connect-health-user-privacy.md).

Améliorez la confidentialité de l’utilisateur pour les installations Azure AD Connect de deux manières :

1.  Sur demande, en extrayant les données d’une personne, puis en supprimant ces données des installations
2.  En garantissant qu’aucune donnée n’est conservée plus de 48 heures

L’équipe Azure AD Connect recommande la deuxième option, car elle est beaucoup plus facile à implémenter et à tenir à jour.

Un serveur de synchronisation Azure AD Connect stocke les données de confidentialité utilisateur suivantes :
1.  Données relatives à un individu stockées dans la **base de données Azure AD Connect**
2.  Données stockées dans les fichiers du **Journal des événements Windows** qui peuvent contenir des informations sur un individu
3.  Données stockées dans les **fichiers journaux de l’installation Azure AD Connect** qui peuvent contenir des informations sur un individu

Les clients Azure AD Connect doivent respecter les directives suivantes lors de la suppression de données utilisateur :
1.  Supprimer régulièrement le contenu du dossier où se trouvent les fichiers journaux de l’installation Azure AD Connect (au moins toutes les 48 heures)
2.  Ce produit peut également créer des journaux d’activité d’événements.  Pour plus d’informations sur les journaux d’activité d’événements, consultez [cette documentation](/windows/win32/wes/windows-event-log).

Les données relatives à un individu sont automatiquement supprimées de la base de données Azure AD Connect lorsqu’elles sont supprimées du système source dont elles proviennent. L’obtention de la conformité RGPD ne nécessite aucune action de la part des administrateurs.  Toutefois, elle nécessite que les données d’Azure AD Connect soient synchronisées avec votre source de données au moins tous les deux jours.

## <a name="delete-the-azure-ad-connect-installation-log-file-folder-contents"></a>Supprimer le contenu du dossier où se trouvent les journaux d’installation Azure AD Connect
Vérifiez régulièrement le contenu du dossier **c:\programdata\aadconnect** et supprimez-le, à l’exception du fichier **PersistedState.Xml**. Ce fichier contient l’état de l’installation précédente d’Azure AD Connect, et est utilisé lorsqu’une mise à niveau est effectuée. Ce fichier ne contient pas de données relatives à un individu et ne doit pas être supprimé.

>[!IMPORTANT]
>Ne supprimez pas le fichier PersistedState.xml.  Ce fichier ne contient aucune information utilisateur. Il contient l’état de l’installation précédente.

Vous pouvez passer en revue et supprimer ces fichiers à l’aide de l’Explorateur Windows, ou vous pouvez utiliser un script comme celui qui suit pour effectuer les actions nécessaires :


```
$Files = ((Get-childitem -Path "$env:programdata\aadconnect" -Recurse).VersionInfo).FileName
Foreach ($file in $files) {
If ($File.ToUpper() -ne "$env:programdata\aadconnect\PERSISTEDSTATE.XML".toupper()) # Do not delete this file
    {Remove-Item -Path $File -Force}
    } 
```

### <a name="schedule-this-script-to-run-every-48-hours"></a>Planifier une exécution de ce script toutes les 48 heures
Suivez les étapes ci-dessous pour planifier l’exécution de ce script toutes les 48 heures.

1.  Enregistrez le script dans un fichier portant l’extension **&#46;PS1**, ouvrez le Panneau de configuration, puis cliquez sur **Systèmes et sécurité**.
    ![Système](./media/reference-connect-user-privacy/gdpr2.png)

2.  Sous le titre Outils d’administration, cliquez sur **Tâches planifiées**.
    ![Tâche](./media/reference-connect-user-privacy/gdpr3.png)
3.  Dans le Planificateur de tâches, cliquez avec le bouton droit sur **Bibliothèque du Planificateur de tâches**, puis cliquez sur **Créer une tâche de base...**
4.  Entrez le nom de la nouvelle tâche, puis cliquez sur **Suivant**.
5.  Sélectionnez **Tous les jours** pour le déclencheur, puis cliquez sur **Suivant**.
6.  Choisissez de répéter tous les **2 jours**, puis cliquez sur **Suivant**.
7.  Sélectionnez **Démarrer un programme**, puis cliquez sur **Suivant**.
8.  Tapez **PowerShell** dans le champ Programme/script. Ensuite, dans le champ **Ajouter des arguments (facultatif)** , entrez le chemin complet du script que vous avez créé précédemment, puis cliquez sur **Suivant**.
9.  L’écran suivant montre un récapitulatif de la tâche que vous êtes sur le point de créer. Vérifiez les valeurs, puis cliquez sur **Terminer** pour créer la tâche.



## <a name="next-steps"></a>Étapes suivantes
* [Lire la politique de confidentialité Microsoft sur le Centre de confidentialité](https://www.microsoft.com/trustcenter)
* [Azure AD Connect Health et confidentialité des utilisateurs](reference-connect-health-user-privacy.md)