---
title: Supprimer des données personnelles - Proxy d’application Azure Active Directory
description: Supprimez des données personnelles des connecteurs installés sur des appareils pour des proxy d'application Azure Active Directory.
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/21/2018
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99fb466debd5a2ee4bb659aea3494469a8bbe8e1
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658415"
---
# <a name="remove-personal-data-for-azure-active-directory-application-proxy"></a>Supprimer des données personnelles pour des proxy d’application Azure Active Directory

Le proxy d'application Azure Active Directory nécessite l’installation de connecteurs sur vos appareils, ce qui signifie qu’il pourrait y avoir des données personnelles sur vos appareils. Cet article explique comment supprimer ces données personnelles afin d’améliorer la confidentialité.

## <a name="where-is-the-personal-data"></a>Où se trouvent ces données personnelles ?

Le proxy d’application peut écrire des données personnelles dans les types de journaux suivants :

- Journaux d’événements des connecteurs
- Journaux des événements Windows

## <a name="remove-personal-data-from-windows-event-logs"></a>Supprimer des données personnelles de journaux des événements Windows

Pour obtenir des informations sur la configuration de la conservation des données dans les journaux des événements Windows, consultez [Paramètres de journaux des événements](https://technet.microsoft.com/library/cc952132.aspx). Pour en savoir plus sur les journaux des événements Windows, consultez [Utilisation des journaux des événements](/windows/win32/wes/using-windows-event-log).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-hybrid-note.md)]

## <a name="remove-personal-data-from-connector-event-logs"></a>Supprimer des données personnelles de journaux des événements des connecteurs

Pour s’assurer que les journaux d’activité du proxy d’application ne contiennent aucune donnée personnelle, vous pouvez :

- Supprimer ou afficher les données si nécessaire, ou
- Désactiver la journalisation

Utiliser les sections suivantes pour supprimer les données personnelles de journaux des événements des connecteurs. Vous devez terminer le processus de suppression sur tous les appareils sur lesquels le connecteur est installé.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

### <a name="view-or-export-specific-data"></a>Afficher ou exporter des données spécifiques

Pour afficher ou exporter des données spécifiques, recherchez des entrées associées dans chaque journal des événements du connecteur. Les journaux d’activité se trouvent dans `C:\ProgramData\Microsoft\Microsoft AAD Application Proxy Connector\Trace`.

Étant donné que les journaux d’activité sont des fichiers texte, vous pouvez utiliser [findstr](/windows-server/administration/windows-commands/findstr) pour rechercher les entrées de texte associées à un utilisateur.  

Pour rechercher des données personnelles, recherchez UserID (ID utilisateur) dans les fichiers journaux.

Pour rechercher des données personnelles consignées par une application qui utilise la délégation Kerberos contrainte, recherchez ces composants du type de nom d'utilisateur :

- Nom d’utilisateur principal local
- Partie correspondant au nom d’utilisateur dans le nom d’utilisateur principal
- Partie correspondant au nom d’utilisateur dans le nom d’utilisateur principal local
- Nom de compte du gestionnaire des comptes de sécurité (SAM) local

### <a name="delete-specific-data"></a>Supprimer des données spécifiques

Pour supprimer des données spécifiques :

1. Redémarrez le service du connecteur de proxy d’application Microsoft Azure AD afin de générer un nouveau fichier journal. Le nouveau fichier journal vous permet de supprimer ou de modifier les anciens fichiers journaux. 
1. Suivez le processus [Afficher ou exporter des données spécifiques](#view-or-export-specific-data), détaillé précédemment, pour trouver les informations à supprimer. Recherchez dans tous les journaux d’activité du connecteur.
1. Supprimez les fichiers journaux adéquats ou sélectionnez les champs contenant des données personnelles et supprimez-les. De plus, vous pouvez supprimer tous les anciens fichiers journaux si vous n’en avez plus besoin.

### <a name="turn-off-connector-logs"></a>Désactiver les journaux d’activité du connecteur

Pour vous assurer que les journaux d’activité du connecteur ne contiennent aucune donnée personnelle, vous pouvez arrêter la génération de journaux d’activité. Pour arrêter la génération de journaux d’activité du connecteur, supprimez la ligne en surbrillance suivante de `C:\Program Files\Microsoft AAD App Proxy Connector\ApplicationProxyConnectorService.exe.config`.

![Affiche un extrait de code avec le code à supprimer mis en surbrillance](./media/application-proxy-remove-personal-data/01.png)

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une vue d’ensemble du proxy d’application, consultez [Offrir un accès à distance sécurisé aux applications locales](application-proxy.md).