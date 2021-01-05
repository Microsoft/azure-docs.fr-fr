---
title: Résoudre les alertes du protocole LDAP sécurisé dans Azure Active Directory Domain Services | Microsoft Docs
description: Découvrez comment détecter et résoudre les alertes courantes du protocole LDAP sécurisé pour Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 81208c0b-8d41-4f65-be15-42119b1b5957
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 15c1f3a1731edf7b45061646d43688b4aacc6104
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620305"
---
# <a name="known-issues-secure-ldap-alerts-in-azure-active-directory-domain-services"></a>Problèmes connus : Alertes du protocole LDAP sécurisé dans Azure Active Directory Domain Services

Les applications et services qui utilisent le protocole LDAP (Lightweight Directory Access Protocol) pour communiquer avec Azure Active Directory Domain Services (Azure AD DS) peuvent être [configurés pour utiliser le protocole LDAP sécurisé](tutorial-configure-ldaps.md). Un certificat approprié et les ports réseau requis doivent être ouverts pour que le protocole LDAP sécurisé fonctionne correctement.

Cet article vous aide à comprendre et à résoudre les alertes courantes avec un accès LDAP sécurisé dans Azure AD DS.

## <a name="aadds101-secure-ldap-network-configuration"></a>AADDS101 : Configuration du réseau LDAP sécurisé

### <a name="alert-message"></a>Message d’alerte

*Le protocole LDAP sécurisé sur Internet est activé pour le domaine managé. Toutefois, l’accès au port 636 n’est pas verrouillé à l’aide d’un Groupe de sécurité réseau (NSG). Cela peut exposer les comptes d’utilisateurs du domaine managé à des attaques de mots de passe par force brute.*

### <a name="resolution"></a>Résolution

Lorsque vous activez le protocole LDAP sécurisé, il est recommandé de créer des règles supplémentaires qui restreignent l’accès LDAPS entrant à des adresses IP spécifiques. Ces règles protègent le domaine managé contre les attaques par force brute. Pour mettre à jour le groupe de sécurité réseau afin de restreindre l’accès au port TCP 636 pour le protocole LDAP sécurisé, procédez comme suit :

1. Dans le Portail Azure, recherchez et sélectionnez **Groupes de sécurité réseau**.
1. Sélectionnez le groupe de sécurité réseau associé à votre domaine managé, par exemple *AADDS-contoso.com-NSG*, puis sélectionnez **Règles de sécurité de trafic entrant**
1. Sélectionnez **+ Ajouter** afin de créer une règle pour le port TCP 636. Si nécessaire, sélectionnez **Avancé** dans la fenêtre pour créer une règle.
1. Pour la **Source**, sélectionnez *Adresses IP* dans le menu déroulant. Entrez les adresses IP sources auxquelles vous souhaitez accorder l’accès pour le trafic LDAP sécurisé.
1. Choisissez *Toutes* en tant que **Destination**, puis entrez *636* dans **Plages de ports de destination**.
1. Définissez **Protocole** sur *TCP* et **Action** sur *Autoriser*.
1. Spécifiez la priorité de la règle, puis saisissez un nom tel que *RestrictLDAPS*.
1. Lorsque vous êtes prêt, sélectionnez **Ajouter** pour créer la règle.

L’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

> [!TIP]
> Le port TCP 636 n’est pas la seule règle nécessaire au bon fonctionnement d’Azure AD DS. Pour en savoir plus, consultez [Groupes de sécurité réseau et ports Azure AD DS requis](network-considerations.md#network-security-groups-and-required-ports).

## <a name="aadds502-secure-ldap-certificate-expiring"></a>AADDS502 : Expiration du certificat LDAP sécurisé

### <a name="alert-message"></a>Message d’alerte

*Le certificat LDAP sécurisé pour le domaine managé expirera le [date]].*

### <a name="resolution"></a>Résolution

Créez un certificat LDAP sécurisé de remplacement en suivant les étapes de [création d’un certificat pour le protocole LDAP sécurisé](tutorial-configure-ldaps.md#create-a-certificate-for-secure-ldap). Appliquez le certificat de remplacement à Azure AD DS et distribuez-le à tous les clients qui se connectent à l’aide du protocole LDAP sécurisé.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
