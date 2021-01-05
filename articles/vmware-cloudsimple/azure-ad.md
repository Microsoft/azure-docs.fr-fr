---
title: Azure VMware Solution by CloudSimple - Utiliser Azure AD comme source d’identité sur le cloud privé
description: Décrit comment ajouter Azure AD en tant que fournisseur d’identité sur votre cloud privé CloudSimple pour authentifier les utilisateurs qui accèdent à CloudSimple à partir d’Azure
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 93922986dfe0b2b4e8ba0923931df601cc12428b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90532526"
---
# <a name="use-azure-ad-as-an-identity-provider-for-vcenter-on-cloudsimple-private-cloud"></a>Utiliser Azure AD comme fournisseur d’identité pour vCenter sur un cloud privé CloudSimple

Vous pouvez configurer votre vCenter cloud privé CloudSimple pour l’authentification auprès d’Azure Active Directory (Azure AD) pour que vos administrateurs VMware accèdent à vCenter. Une fois la source d’identité d’authentification unique configurée, l'utilisateur **cloudowner** peut ajouter des utilisateurs à partir de la source d’identité à vCenter.  

Vous pouvez configurer votre domaine Active Directory et vos contrôleurs de domaine de l’une des manières suivantes :

* Domaine et contrôleurs de domaine Active Directory exécutés localement
* Domaine et contrôleurs de domaine Active Directory exécutés sur Azure en tant que machines virtuelles dans votre abonnement Azure
* Nouveaux domaine et contrôleurs de domaine Active Directory exécutés dans votre cloud privé CloudSimple
* Service Azure Active Directory

Ce guide explique les tâches requises pour configurer Azure AD en tant que source d’identité.  Pour plus d’informations sur l’utilisation d’Active Directory en local ou d’Active Directory s’exécutant dans Azure, consultez [Configurer des sources d’identité vCenter pour utiliser Active Directory](set-vcenter-identity.md) afin d’obtenir des instructions détaillées sur la configuration de la source d’identité.

## <a name="about-azure-ad"></a>À propos d’Azure AD

Azure AD est le service Microsoft de gestion des répertoires et des identités basé sur le cloud mutualisé.  Azure AD fournit un mécanisme d’authentification scalable, cohérent et fiable permettant aux utilisateurs de s’authentifier et d’accéder à différents services sur Azure.  Il fournit également des services LDAP sécurisés pour tous les services tiers afin d’utiliser Azure AD en tant que source d’authentification/d’identité.  Azure AD combine les services d’annuaire principaux, la gouvernance avancée des identités et la gestion de l’accès aux applications, qui peuvent être utilisés pour donner accès à votre cloud privé pour les utilisateurs qui administrent le cloud privé.

Pour utiliser Azure AD comme source d’identité avec vCenter, vous devez configurer Azure AD et Azure AD Domain Services. Suivez ces instructions :

1. [Comment configurer Azure AD et Azure AD Domain Services](#set-up-azure-ad-and-azure-ad-domain-services)
2. [Comment configurer une source d’identité sur votre cloud privé vCenter](#set-up-an-identity-source-on-your-private-cloud-vcenter)

## <a name="set-up-azure-ad-and-azure-ad-domain-services"></a>Configurer Azure AD et Azure AD Domain Services

Avant de commencer, vous aurez besoin d’un accès à votre abonnement Azure avec des privilèges d’administrateur général.  Les étapes suivantes fournissent des instructions générales. Les détails sont contenus dans la documentation Azure.

### <a name="azure-ad"></a>Azure AD

> [!NOTE]
> Si vous avez déjà Azure AD, vous pouvez ignorer cette section.

1. Configurez Azure AD sur votre abonnement, comme décrit dans la [documentation Azure AD](../active-directory/fundamentals/active-directory-whatis.md).
2. Activez Azure Active Directory Premium sur votre abonnement, comme décrit dans [S’inscrire à Azure Active Directory Premium](../active-directory/fundamentals/active-directory-get-started-premium.md).
3. Configurez un nom de domaine personnalisé et vérifiez le nom de domaine personnalisé comme décrit dans [Ajouter un nom de domaine personnalisé à Azure Active Directory](../active-directory/fundamentals/add-custom-domain.md).
    1. Configurez un enregistrement DNS sur votre bureau d’enregistrement de domaines avec les informations fournies sur Azure.
    2. Configurez le nom de domaine personnalisé comme domaine principal.

Vous pouvez éventuellement configurer d’autres fonctionnalités d’Azure AD.  Celles-ci ne sont pas obligatoires pour l’activation de l’authentification vCenter avec Azure AD.

### <a name="azure-ad-domain-services"></a>Azure AD Domain Services

> [!NOTE]
> Il s’agit d’une étape importante pour l’activation d’Azure AD en tant que source d’identité pour vCenter.  Pour éviter tout problème, assurez-vous que toutes les étapes sont effectuées correctement.

1. Activez Azure AD Domain Services comme indiqué dans [Activer Azure Active Directory Domain Services à l’aide du portail Azure](../active-directory-domain-services/tutorial-create-instance.md).
2. Configurez le réseau qui sera utilisé par Azure AD Domain Services comme indiqué dans [Activer Azure Active Directory Domain Services à l’aide du portail Azure](../active-directory-domain-services/tutorial-create-instance.md).
3. Configurez le groupe d’administrateurs pour la gestion d’Azure AD Domain Services comme indiqué dans [Activer Azure Active Directory Domain Services à l’aide du portail Azure](../active-directory-domain-services/tutorial-create-instance.md).
4. Mettez à jour les paramètres DNS de vos Azure AD Domain Services comme décrit dans [Activer Azure Active Directory Domain Services](../active-directory-domain-services/tutorial-create-instance.md).  Si vous souhaitez vous connecter à AD via Internet, configurez l’enregistrement DNS pour l’adresse IP publique d’Azure AD Domain Services sur le nom de domaine.
5. Activez la synchronisation de hachage de mot de passe pour les utilisateurs.  Cette étape active la synchronisation des hachages de mot de passe requise pour l’authentification NT LAN Manager (NTLM) et Kerberos avec Azure AD Domain Services. Une fois la synchronisation des informations de hachage de mot de passe configurée, les utilisateurs peuvent se connecter au domaine managé à l’aide de leurs informations d’identification d’entreprise. Consultez [Activer la synchronisation de hachage de mot de passe pour Azure Active Directory Domain Services](../active-directory-domain-services/tutorial-create-instance.md).
    1. Si des utilisateurs cloud uniquement sont présents, ils doivent modifier leur mot de passe à l’aide du <a href="https://myapps.microsoft.com/" target="_blank">panneau d’accès Azure AD</a> pour s’assurer que les hachages de mot de passe sont stockés au format requis par NTLM ou Kerberos.  Suivez les instructions sous [Activer la synchronisation de hachage de mot de passe avec votre domaine managé pour les comptes d’utilisateurs uniquement dans le cloud](../active-directory-domain-services/tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds).  Cette étape doit être effectuée pour les utilisateurs individuels et tout nouvel utilisateur créé dans votre annuaire Azure AD à l’aide des cmdlets Azure AD PowerShell ou du portail Azure. Les utilisateurs qui ont besoin d’accéder à Azure AD Domain Services doivent utiliser le <a href="https://myapps.microsoft.com/" target="_blank">panneau d’accès Azure AD</a> et accéder à leur profil pour modifier le mot de passe.

        > [!NOTE]
        > Si votre organisation utilise des comptes d’utilisateur dans le cloud uniquement, tous les utilisateurs ayant besoin d’Azure Active Directory Domain Services doivent modifier leur mot de passe. Un compte d’utilisateur uniquement dans le cloud est un compte qui a été créé dans votre répertoire Azure AD à l’aide du portail Azure ou d’applets de commande PowerShell Azure AD. Ces comptes d’utilisateurs ne sont pas synchronisés à partir d’un répertoire local.

    2. Si vous synchronisez des mots de passe à partir de votre annuaire Active Directory local, suivez les étapes de la [documentation de Active Directory](../active-directory-domain-services/tutorial-configure-password-hash-sync.md).

6.  Configurez le protocole LDAP sécurisé sur votre Azure Active Directory Domain Services comme décrit dans [Configurer le protocole LDAPS (LDAP sécurisé) pour un domaine managé Azure AD Domain Services](../active-directory-domain-services/tutorial-configure-ldaps.md).
    1. Chargez un certificat pour une utilisation par le protocole LDAP sécurisé, comme décrit dans la rubrique Azure [Obtenir un certificat pour le protocole LDAP sécurisé](../active-directory-domain-services/tutorial-configure-ldaps.md#create-a-certificate-for-secure-ldap).  CloudSimple recommande d’utiliser un certificat signé émis par une autorité de certification pour s’assurer que vCenter peut approuver le certificat.
    2. Activez le protocole LDAP sécurisé comme indiqué dans [Configurer le protocole LDAPS (LDAP sécurisé) pour un domaine managé Azure AD Domain Services](../active-directory-domain-services/tutorial-configure-ldaps.md).
    3. Enregistrez la partie publique du certificat (sans la clé privée) au format .cer pour une utilisation avec vCenter lors de la configuration de la source d’identité.
    4. Si l’accès Internet à Azure AD Domain Services est requis, activez l’option « Autoriser l’accès sécurisé à LDAP sur Internet ».
    5. Ajoutez la règle de sécurité de trafic entrant pour le NSG Azure AD Domain Services pour le port TCP 636.

## <a name="set-up-an-identity-source-on-your-private-cloud-vcenter"></a>Configurer une source d’identité sur votre cloud privé vCenter

1. [Élevez les privilèges](escalate-private-cloud-privileges.md) pour votre cloud privé vCenter.
2. Collectez les paramètres de configuration requis pour la configuration de la source d’identité.

    | **Option** | **Description** |
    |------------|-----------------|
    | **Nom** | Nom de la source d’identité. |
    | **DN de base pour les utilisateurs** | Nom unique de base pour les utilisateurs.  Pour Azure AD, utilisez : `OU=AADDC Users,DC=<domain>,DC=<domain suffix>`  Exemple : `OU=AADDC Users,DC=cloudsimplecustomer,DC=com`.|
    | **Nom de domaine** | Nom de domaine complet du domaine, par exemple « example.com ». Ne fournissez pas d’adresse IP dans cette zone de texte. |
    | **Alias de domaine** | *(facultatif)* Nom NetBIOS du domaine. Ajoutez le nom NetBIOS du domaine Active Directory en tant qu’alias de la source d’identité si vous utilisez des authentifications SSPI. |
    | **DN de base pour les groupes** | Nom unique de base pour les groupes. Pour Azure AD, utilisez : `OU=AADDC Users,DC=<domain>,DC=<domain suffix>`  Exemple : `OU=AADDC Users,DC=cloudsimplecustomer,DC=com`|
    | **URL du serveur principal** | Serveur LDAP du contrôleur de domaine principal pour le domaine.<br><br>Utilisez le format  `ldaps://hostname:port`. Le port est généralement 636 pour les connexions LDAPS. <br><br>Un certificat établissant une relation de confiance pour le point de terminaison LDAPS du serveur Active Directory est nécessaire quand vous utilisez `ldaps://` dans l’URL LDAP principale ou secondaire. |
    | **URL du serveur secondaire** | Adresse d’un serveur LDAP de contrôleur de domaine secondaire utilisé pour le basculement. |
    | **Choisir un certificat** | Si vous souhaitez utiliser LDAPS avec votre serveur LDAP Active Directory ou votre source d’identité serveur OpenLDAP, un bouton Choisir un certificat s’affiche une fois que vous avez entré `ldaps://` dans la zone de texte URL. Une URL secondaire n’est pas nécessaire. |
    | **Nom d’utilisateur** | ID d’un utilisateur dans le domaine qui dispose au minimum d’un accès en lecture seule au DN de base pour les utilisateurs et les groupes. |
    | **Mot de passe** | Mot de passe de l’utilisateur spécifié dans Nom d’utilisateur. |

3. Connectez-vous à votre cloud privé vCenter une fois les privilèges élevés.
4. Suivez les instructions dans [Ajouter une source d’identité sur vCenter](set-vcenter-identity.md#add-an-identity-source-on-vcenter) en utilisant les valeurs de l’étape précédente pour configurer Azure Active Directory comme source d’identité.
5. Ajoutez des utilisateurs/groupes à partir d’Azure AD à des groupes vCenter comme décrit dans la rubrique VMware [Ajouter des membres à un groupe d’authentification unique vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

> [!CAUTION]
> Les nouveaux utilisateurs doivent être ajoutés uniquement à *Cloud-Owner-Group*, à *Cloud-Global-Cluster-Admin-Group*, à *Cloud-Global-Storage-Admin-Group*, à *Cloud-Global-Network-Admin-Group* ou à *Cloud-Global-VM-Admin-Group*.  Les utilisateurs ajoutés au groupe *Administrateurs* seront automatiquement supprimés.  Seuls les comptes de service doivent être ajoutés au groupe *Administrateurs*.

## <a name="next-steps"></a>Étapes suivantes

* [Découvrir le modèle d’autorisation de cloud privé](learn-private-cloud-permissions.md)
