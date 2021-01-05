---
title: Dépanner le protocole LDAP sécurisé dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment résoudre les problèmes liés au protocole LDAP sécurisé (LDAPS) pour un domaine managé Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 445c60da-e115-447b-841d-96739975bdf6
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: e21ec7d8fdf7a5c7a8ba7a9f22aa5709581cbae6
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618397"
---
# <a name="troubleshoot-secure-ldap-connectivity-issues-to-an-azure-active-directory-domain-services-managed-domain"></a>Résoudre les problèmes de connectivité LDAP sécurisé dans un domaine managé Azure Active Directory Domain Services

Les applications et services qui utilisent le protocole LDAP (Lightweight Directory Access Protocol) pour communiquer avec Azure Active Directory Domain Services (Azure AD DS) peuvent être [configurés pour utiliser le protocole LDAP sécurisé](tutorial-configure-ldaps.md). Un certificat approprié et les ports réseau requis doivent être ouverts pour que le protocole LDAP sécurisé fonctionne correctement.

Cet article vous aide à résoudre les problèmes liés à l’accès LDAP sécurisé dans Azure AD DS.

## <a name="common-connection-issues"></a>Problèmes de connexion courants

Si vous ne parvenez pas à vous connecter à un domaine managé Azure AD DS à l’aide du protocole LDAP sécurisé, passez en revue les étapes de résolution des problèmes suivantes. Après chaque étape de résolution d’un problème, réessayez de vous connecter au domaine managé :

* La chaîne émetteur du certificat LDAP sécurisé doit être approuvée sur le client. Vous pouvez ajouter l’autorité de certification racine au magasin de certificats racines de confiance sur le client pour établir la relation d’approbation.
    * Assurez-vous d’[exporter et d’appliquer le certificat aux ordinateurs clients][client-cert].
* Vérifiez que le certificat LDAP sécurisé pour votre domaine managé présente le nom DNS dans l’attribut *Objet* ou *Autres noms de l’objet*.
    * Passez en revue les [conditions requises pour le certificat LDAP sécurisé][certs-prereqs] et créez un certificat de remplacement si nécessaire.
* Vérifiez que le client LDAP, tel que *ldp.exe* se connecte au point de terminaison LDAP sécurisé à l’aide d’un nom DNS, et non de l’adresse IP.
    * Le certificat appliqué au domaine managé n’inclut pas les adresses IP du service, mais uniquement les noms DNS.
* Vérifiez le nom de DNS auquel la client LDAP se connecte. Il doit correspondre à l’adresse IP publique pour le protocole LDAP sécurisé sur le domaine managé.
    * Si le nom DNS correspond à l’adresse IP interne, mettez à jour l’enregistrement DNS pour qu’il corresponde à l’adresse IP externe.
* Pour la connectivité externe, le groupe de sécurité réseau doit inclure une règle qui autorise le trafic vers le port TCP 636 à partir d’Internet.
    * Si vous pouvez vous connecter au domaine managé à l’aide du protocole LDAP sécurisé à partir de ressources directement connectées au réseau virtuel, mais pas à partir de connexions externes, vous devez [créer une règle de groupe de sécurité réseau pour autoriser le trafic LDAP sécurisé][ldaps-nsg].

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[configure-ldaps]: tutorial-configure-ldaps.md
[certs-prereqs]: tutorial-configure-ldaps.md#create-a-certificate-for-secure-ldap
[client-cert]: tutorial-configure-ldaps.md#export-a-certificate-for-client-computers
[ldaps-nsg]: tutorial-configure-ldaps.md#lock-down-secure-ldap-access-over-the-internet
