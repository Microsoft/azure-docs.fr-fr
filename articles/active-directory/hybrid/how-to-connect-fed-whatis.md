---
title: Fédération avec Azure AD Connect | Microsoft Docs
description: Cette page centralise toute la documentation portant sur les opérations AD FS avec Azure AD Connect
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: f9107cf5-0131-499a-9edf-616bf3afef4d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/09/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a97142e0c512f4f95235ad08c94c852906d3efd8
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92095854"
---
# <a name="azure-ad-connect-and-federation"></a>Fédération avec Azure AD Connect
Azure Active Directory (Azure AD) Connect vous permet de configurer la fédération avec Active Directory Federation Services (AD FS) et Azure AD au niveau local. Avec l’authentification de fédération, vous pouvez autoriser les utilisateurs à se connecter aux services Azure AD avec leurs mots de passe locaux sans avoir à les saisir de nouveau, et ce, alors qu’ils sont sur le réseau d’entreprise. En utilisant l’option de fédération AD FS, vous pouvez déployer un nouveau service AD FS ou spécifier une installation existante dans une batterie de serveurs Windows Server 2012 R2.

Cette rubrique présente les fonctions associées à la fédération pour Azure AD Connect. Elle répertorie les liens vers toutes les autres rubriques qui lui sont associées. Pour consulter les liens vers Azure AD Connect, voir [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).

## <a name="azure-ad-connect-federation-topics"></a>Azure AD Connect : rubriques sur la fédération
| Rubrique | Sujet qu’elle aborde et quand la consulter |
|:--- |:--- |
| **Options de connexion de l’utilisateur via Azure AD Connect** | |
| [Présentation des options de connexion de l’utilisateur](plan-connect-user-signin.md) |Découvrez les options de connexion de l’utilisateur et leur impact sur l’expérience de connexion à Azure. |
| **Installation d’AD FS avec Azure AD Connect** | |
| [Composants requis](how-to-connect-install-custom.md#ad-fs-configuration-prerequisites) |Consultez les conditions préalables à l’installation d’AD FS via Azure AD Connect |
| [Configurer une batterie de serveurs AD FS](how-to-connect-install-custom.md#configuring-federation-with-ad-fs) |Installation d’une nouvelle batterie de serveurs AD FS avec Azure AD Connect |
| [Fédérer avec Azure AD à l’aide d’un ID de connexion de substitution](how-to-connect-fed-management.md#alternateid) | Configurer la fédération à l’aide d’un ID de connexion de substitution  |
| **Modifier la configuration AD FS** | |
| [Réparation de l’approbation](how-to-connect-fed-management.md#repairthetrust) |Rétablir l’approbation actuelle entre AD FS et Microsoft 365/Azure au niveau local. |
| [Ajouter un nouveau serveur AD FS](how-to-connect-fed-management.md#addadfsserver) |Extension d’une batterie de serveurs AD FS avec l’ajout d’un serveur AD FS après l’installation initiale |
| [Ajouter un nouveau serveur de proxy d’application web AD FS](how-to-connect-fed-management.md#addwapserver) |Extension d’une batterie de serveurs AD FS avec l’ajout d’un serveur proxy d’application web après l’installation initiale. |
| [Ajouter un nouveau domaine fédéré](how-to-connect-fed-management.md#addfeddomain) |Ajoutez un domaine à fédérer avec Azure AD. |
| [Mettre à jour le certificat TLS/SSL](how-to-connect-fed-ssl-update.md)| Mise à jour du certificat TLS/SSL pour une batterie de serveurs AD FS. |
| [Renouveler des certificats de fédération pour Microsoft 365 et Azure AD](how-to-connect-fed-o365-certs.md)|Renouvelez votre certificat O365 avec Azure AD.|
| **Autre configuration de fédération** | |
| [Fédérer plusieurs instances d’Azure AD avec une seule instance AD FS](how-to-connect-fed-single-adfs-multitenant-federation.md) | Fédérer plusieurs instances d’Azure AD avec une seule batterie de serveurs AD FS| 
| [Ajouter un logo/une illustration personnalisée de la société](how-to-connect-fed-management.md#customlogo) |Modifiez l’expérience de connexion en spécifiant le logo personnalisé qui apparaît sur la page de connexion AD FS. |
| [Ajout d’une description de connexion](how-to-connect-fed-management.md#addsignindescription) |Modifiez la description de la connexion sur la page de connexion AD FS. |
| [Modification des règles de revendication AD FS](how-to-connect-fed-management.md#modclaims) |Modifiez ou ajoutez des règles de revendication dans AD FS pour configurer la synchronisation Azure AD Connect. |


## <a name="additional-resources"></a>Ressources supplémentaires
* [Fédérer deux Azure AD avec un seul AD FS](how-to-connect-fed-single-adfs-multitenant-federation.md)
* [Déploiement des services AD FS dans Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs)
* [Déploiement des services AD FS haute disponibilité par-delà les frontières dans Azure avec Azure Traffic Manager](/windows-server/identity/ad-fs/deployment/active-directory-adfs-in-azure-with-azure-traffic-manager)