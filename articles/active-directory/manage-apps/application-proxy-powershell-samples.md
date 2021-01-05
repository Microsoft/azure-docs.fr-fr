---
title: Exemples PowerShell pour le proxy d’application Azure AD
description: Utilisez ces exemples PowerShell pour le proxy d’application Azure AD pour obtenir des informations sur les applications et les connecteurs du proxy d’application dans votre annuaire, affecter des utilisateurs et des groupes à des applications et obtenir des informations sur les certificats.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 12/05/2019
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8b0955e67c5a35cbea96490a0395522fa145dad1
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96858363"
---
# <a name="azure-ad-powershell-examples-for-azure-ad-application-proxy"></a>Exemples Azure AD PowerShell pour le proxy d’application Azure AD

Le tableau suivant contient des liens vers des exemples de scripts PowerShell pour le proxy d’application Azure AD. Ces exemples requièrent [AzureAD v2 PowerShell pour le module Graph](/powershell/azure/active-directory/install-adv2) ou [AzureAD v2 PowerShell pour la version préliminaire du module Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview), sauf indication contraire.


Pour plus d’informations sur les applets de commande utilisées dans ces exemples, consultez [Gestion des applications de proxy d’application](/powershell/module/azuread/#application_proxy_application_management) et [Gestion des connecteurs de proxy d’application](/powershell/module/azuread/#application_proxy_connector_management).

| Lien | Description |
|---|---|
|**Applications de proxy d’application**||
| [Répertorier les informations de base pour toutes les applications de proxy d’application](scripts/powershell-get-all-app-proxy-apps-basic.md) | Liste les informations de base (AppId, DisplayName, ObjId) de toutes les applications de proxy d’application de votre répertoire. |
| [Répertorier les informations étendues pour toutes les applications de proxy d’application](scripts/powershell-get-all-app-proxy-apps-extended.md) | Répertorie les informations étendues (AppId, DisplayName, ExternalUrl, InternalUrl, ExternalAuthenticationType) relatives à toutes les applications de proxy d’application de votre répertoire.  |
| [Répertorier toutes les applications de proxy d’application par groupe de connecteurs](scripts/powershell-get-all-app-proxy-apps-by-connector-group.md) | Répertorie des informations sur toutes les applications de proxy d’application de votre répertoire et les groupes de connecteurs auxquels les applications sont affectées. |
| [Obtenir toutes les applications de proxy d’application avec une stratégie de durée de vie de jeton](scripts/powershell-get-all-app-proxy-apps-with-policy.md) | Répertorie toutes les applications de proxy d’application de votre répertoire avec une stratégie de durée de vie des jetons et ses détails. Cet exemple nécessite la [version en préversion du module AzureAD v2 PowerShell pour Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview). |
|**Groupes de connecteurs**||
| [Récupérer tous les connecteurs et groupes de connecteurs dans le répertoire](scripts/powershell-get-all-connectors.md) | Répertorie tous les connecteurs et groupes de connecteurs dans votre répertoire. |
| [Déplacer toutes les applications affectées à un groupe de connecteurs vers un autre groupe de connecteurs](scripts/powershell-move-all-apps-to-connector-group.md) | Déplace toutes les applications actuellement affectées à un groupe de connecteurs vers un autre groupe de connecteurs. |
|**Utilisateurs et groupe affectés**||
| [Afficher les utilisateurs et les groupes affectés à une application de proxy d’application](scripts/powershell-display-users-group-of-app.md) | Répertorie les utilisateurs et les groupes affectés à une application de proxy d’application spécifique. |
| [Affecter un utilisateur à une application](scripts/powershell-assign-user-to-app.md) | Affecte un utilisateur spécifique à une application. |
| [Affecter un groupe à une application](scripts/powershell-assign-group-to-app.md) | Affecte un groupe spécifique à une application. |
|**Configuration de l’URL externe**||
| [Récupérer toutes les applications de proxy d’application à l’aide de domaines par défaut (. msappproxy.net)](scripts/powershell-get-all-default-domain-apps.md)  | Répertorie toutes les applications de proxy d’application à l’aide de domaines par défaut (. msappproxy.net). |
| [Récupérer toutes les applications de proxy d’application à l’aide de la publication générique](scripts/powershell-get-all-wildcard-apps.md) | Répertorie toutes les applications de proxy d’application à l’aide de la publication de caractères génériques. |
|**Configuration de domaine personnalisée**||
| [Répertorier toutes les applications de proxy d’application utilisant des domaines personnalisés et informations de certificat](scripts/powershell-get-all-custom-domains-and-certs.md) | Répertorie toutes les applications de proxy d’application qui utilisent des domaines personnalisés et des informations de certificat associées aux domaines personnalisés. |
| [Répertorier toutes les applications d’application proxy Azure AD publiées sans certificat chargé](scripts/powershell-get-all-custom-domain-no-cert.md) | Liste toutes les applications de proxy d’application qui utilisent des domaines personnalisés, mais n’ont pas de certificat TLS/SSL valide chargé. |
| [Récupérer toutes les applications d’application Azure AD proxy publiées avec le même certificat](scripts/powershell-get-custom-domain-identical-cert.md) | Répertorie toutes les applications d’application proxy Azure AD publiées avec le certificat identique. |
| [Récupérer toutes les applications d’application Azure AD proxy publiées avec le certificat identique et remplacement](scripts/powershell-get-custom-domain-replace-cert.md) | Pour les applications Azure AD proxy qui sont publiées avec un certificat identique, vous permet de remplacer le certificat en bloc. |
