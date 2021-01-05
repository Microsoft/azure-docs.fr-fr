---
title: Accéder à Key Vault derrière un pare-feu - Azure Key Vault | Microsoft Docs
description: Découvrez quels ports, hôtes ou adresses IP ouvrir pour permettre à une application cliente de coffre de clés derrière un pare-feu d’accéder à un coffre de clés.
services: key-vault
author: amitbapat
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/12/2019
ms.author: ambapat
ms.openlocfilehash: ca5842fb268c20f8ae58eb5f683229c4ae3919f4
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289161"
---
# <a name="access-azure-key-vault-behind-a-firewall"></a>Accès à Azure Key Vault derrière un pare-feu

## <a name="what-ports-hosts-or-ip-addresses-should-i-open-to-enable-my-key-vault-client-application-behind-a-firewall-to-access-key-vault"></a>Quels ports, hôtes ou adresses IP dois-je ouvrir pour permettre à mon application cliente de coffre de clés derrière un pare-feu d’accéder au coffre de clés ?

Pour accéder à un coffre de clés, votre application cliente de coffre de clés doit accéder à plusieurs points de terminaison pour différentes fonctionnalités :

* Authentification via Azure Active Directory (Azure AD).
* Gestion d’un coffre Azure Key Vault. Cela inclut la création, la lecture, la mise à jour, la suppression et la définition de stratégies d’accès par le biais d’Azure Resource Manager.
* L’accès et la gestion des objets (clés et secrets) stockés dans le coffre de clés lui-même passent par le point de terminaison spécifique à Key Vault (par exemple, `https://yourvaultname.vault.azure.net`).  

Il existe des variantes selon votre configuration et l’environnement.

## <a name="ports"></a>Ports

Tout le trafic vers le coffre de clés pour chacune des 3 fonctions (authentification, gestion et accès au plan de données) passe par le protocole HTTPS : port 443. Cependant, il faut compter occasionnellement sur un trafic HTTP (port 80) pour les CRL. Les clients qui prennent en charge le protocole OCSP ne doivent pas atteindre les CRL, mais peuvent occasionnellement atteindre [http://cdp1.public-trust.com/CRL/Omniroot2025.crl](http://cdp1.public-trust.com/CRL/Omniroot2025.crl).  

## <a name="authentication"></a>Authentification

L’application cliente de coffre de clés doit accéder aux points de terminaison Azure Active Directory pour l’authentification. Le point de terminaison utilisé dépend de la configuration du locataire Azure AD, du type de principal (principal d’utilisateur ou principal du service) et du type de compte (par exemple, compte Microsoft ou ID d’organisation).  

| Type de principal | Point de terminaison:port |
| --- | --- |
| Utilisateur utilisant un compte Microsoft<br> (Par exemple, user@hotmail.com) |**Mondial :**<br> login.microsoftonline.com:443<br><br> **Azure China :**<br> login.chinacloudapi.cn:443<br><br>**Azure US Government :**<br> login.microsoftonline.us:443<br><br>**Azure Germany :**<br> login.microsoftonline.de:443<br><br> and <br>login.live.com:443 |
| Utilisateur ou principal du service utilisant un compte professionnel ou scolaire avec Azure AD (par exemple, user@contoso.com) |**Mondial :**<br> login.microsoftonline.com:443<br><br> **Azure China :**<br> login.chinacloudapi.cn:443<br><br>**Azure US Government :**<br> login.microsoftonline.us:443<br><br>**Azure Germany :**<br> login.microsoftonline.de:443 |
| Utilisateur ou principal du service utilisant un compte professionnel ou scolaire, plus Active Directory Federation Services (AD FS) ou un autre point de terminaison fédéré (par exemple, user@contoso.com) |Tous les points de terminaison correspondant à un compte professionnel ou scolaire, plus AD FS ou d’autres points de terminaison fédérés |

D’autres scénarios complexes sont possibles. Pour plus d’informations, reportez-vous à [Azure Active Directory Authentication Flow (Flux d’authentification d’Azure Active Directory)](../../active-directory/develop/authentication-vs-authorization.md), [Intégration d’applications dans Azure Active Directory](../../active-directory/develop/active-directory-how-to-integrate.md) et [Protocoles d’authentification Active Directory](/previous-versions/azure/dn151124(v=azure.100)).  

## <a name="key-vault-management"></a>Gestion de Key Vault

Pour la gestion d’un coffre de clés Key Vault (CRUD et définition de la stratégie d’accès), l’application cliente de coffre de clés doit accéder au point de terminaison Azure Resource Manager.  

| Type d’opération | Point de terminaison:port |
| --- | --- |
| Opérations du plan de contrôle Key Vault<br> via Azure Resource Manager |**Mondial :**<br> management.azure.com:443<br><br> **Azure China :**<br> management.chinacloudapi.cn:443<br><br> **Azure US Government :**<br> management.usgovcloudapi.net:443<br><br> **Azure Germany :**<br> management.microsoftazure.de:443 |
| API Microsoft Graph |**Mondial :**<br> graph.microsoft.com:443<br><br> **Azure China :**<br> graph.chinacloudapi.cn:443<br><br> **Azure US Government :**<br> graph.microsoft.com:443<br><br> **Azure Germany :**<br> graph.cloudapi.de:443 |

## <a name="key-vault-operations"></a>Opérations Key Vault

Pour toutes les opérations de gestion et de chiffrement d’objets (clés et secrets), le client de coffre de clés doit accéder au point de terminaison du coffre de clés. Le suffixe DNS du point de terminaison varie en fonction de l’emplacement de votre coffre de clés. Le point de terminaison du coffre de clés est au format *nom du coffre*. *suffixe-dns-spécifique-à-la-région* , comme indiqué dans le tableau ci-dessous.  

| Type d’opération | Point de terminaison:port |
| --- | --- |
| Opérations telles que le chiffrement sur les clés, création/lecture/mise à jour/suppression de clés et de secrets, définition/obtention de mots-clés et autres attributs sur objets de coffre de clés (clés ou secrets) |**Mondial :**<br> &lt;nom du coffre&gt;.vault.azure.net:443<br><br> **Azure China :**<br> &lt;nom du coffre&gt;.vault.azure.cn:443<br><br> **Azure US Government :**<br> &lt;nom du coffre&gt;.vault.usgovcloudapi.net:443<br><br> **Azure Germany :**<br> &lt;nom du coffre&gt;.vault.microsoftazure.de:443 |

## <a name="ip-address-ranges"></a>Plages d’adresse IP

Le service Key Vault utilise d’autres ressources Azure telles que l’infrastructure PaaS, de sorte qu’il n’est pas possible de fournir une plage spécifique des adresses IP qu’auront les points de terminaison Key Vault à un moment donné. Si votre pare-feu prend en charge uniquement des plages d’adresses IP, consultez les documents Plages IP des centres de données Microsoft Azure disponibles aux emplacements suivants :
* [Public](https://www.microsoft.com/en-us/download/details.aspx?id=56519)
* [Gouvernement des États-Unis](https://www.microsoft.com/en-us/download/details.aspx?id=57063)
* [Allemagne](https://www.microsoft.com/en-us/download/details.aspx?id=57064)
* [Chine](https://www.microsoft.com/en-us/download/details.aspx?id=57062)

L’authentification et l’identité (Azure Active Directory) est un service global et peut basculer vers d’autres régions ou réacheminer du trafic sans avertissement. Dans ce scénario, toutes les plages d’adresses IP répertoriées dans [Adresse IP d’authentification et d’identité](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity_ip) doivent être ajoutées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

Pour toute question concernant Key Vault, rendez-vous sur la [Page de questions Microsoft Q&A consacrée à Azure Key Vault](/answers/topics/azure-key-vault.html).