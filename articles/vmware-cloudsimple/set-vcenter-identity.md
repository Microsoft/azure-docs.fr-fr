---
title: Azure VMware Solution by CloudSimple – Configurer des sources d’identité vCenter sur le cloud privé
description: Décrit comment configurer votre cloud privé vCenter pour s’authentifier auprès d’Active Directory afin que les administrateurs VMware accèdent à vCenter
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 60090dec56a177fac6ddad946d97142b484355af
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92425747"
---
# <a name="set-up-vcenter-identity-sources-to-use-active-directory"></a>Configurer des sources d’identité vCenter pour utiliser Active Directory

## <a name="about-vmware-vcenter-identity-sources"></a>À propos des sources d’identité VMware vCenter

VMware vCenter prend en charge différentes sources d’identité pour l’authentification des utilisateurs qui accèdent à vCenter.  Votre vCenter cloud privé CloudSimple peut être configuré pour s’authentifier auprès d’Active Directory pour que vos administrateurs VMware accèdent à vCenter. Une fois l’installation terminée, l’utilisateur **cloudowner** peut ajouter des utilisateurs de la source d’identité à vCenter.  

Vous pouvez configurer votre domaine Active Directory et vos contrôleurs de domaine de l’une des manières suivantes :

* Domaine et contrôleurs de domaine Active Directory exécutés localement
* Domaine et contrôleurs de domaine Active Directory exécutés sur Azure en tant que machines virtuelles dans votre abonnement Azure
* Nouveaux domaine et contrôleurs de domaine Active Directory exécutés dans votre cloud privé
* Service Azure Active Directory

Ce guide explique les tâches à effectuer pour configurer un domaine et des contrôleurs de domaine Active Directory exécutés localement ou en tant que machines virtuelles dans vos abonnements.  Si vous souhaitez utiliser Azure AD comme source d’identité, consultez [Utiliser Azure AD comme fournisseur d’identité pour vCenter sur un cloud privé CloudSimple](azure-ad.md) pour obtenir des instructions détaillées sur la configuration de la source d’identité.

Avant d’[ajouter une source d’identité](#add-an-identity-source-on-vcenter), [augmentez temporairement vos privilèges vCenter](escalate-private-cloud-privileges.md).

> [!CAUTION]
> Les nouveaux utilisateurs doivent être ajoutés uniquement à *Cloud-Owner-Group* , à *Cloud-Global-Cluster-Admin-Group* , à *Cloud-Global-Storage-Admin-Group* , à *Cloud-Global-Network-Admin-Group* ou à *Cloud-Global-VM-Admin-Group*.  Les utilisateurs ajoutés au groupe *Administrateurs* seront automatiquement supprimés.  Seuls des comptes de service doivent être ajoutés au groupe *Administrateurs* , et les comptes de service en doivent pas être utilisés pour se connecter à l’interface utilisateur web de vSphere.   


## <a name="identity-source-options"></a>Options de la source d’identité

* [Ajouter un domaine Active Directory local en tant que source d’identité d’authentification unique](#add-on-premises-active-directory-as-a-single-sign-on-identity-source)
* [Configurer un nouveau domaine Active Directory sur un cloud privé](#set-up-new-active-directory-on-a-private-cloud)
* [Configurer Active Directory sur Azure](#set-up-active-directory-on-azure)

> [!IMPORTANT]
> **Active Directory (l’authentification intégrée de Windows) n’est pas pris en charge.** Seule l’option Active Directory sur LDAP est prise en charge en tant que source d’identité.

## <a name="add-on-premises-active-directory-as-a-single-sign-on-identity-source"></a>Ajouter un domaine Active Directory local en tant que source d’identité d’authentification unique

Pour configurer votre domaine Active Directory local en tant que source d’identité d’authentification unique, vous devez disposer des éléments suivants :

* [Connexion VPN de site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) de votre centre de données local vers votre cloud privé
* Adresse IP du serveur DNS local ajoutée à vCenter et au contrôleur PSC (Platform Services Controller)

Utilisez les informations du tableau suivant lors de la configuration de votre domaine Active Directory.

| **Option** | **Description** |
|------------|-----------------|
| **Nom** | Nom de la source d’identité. |
| **DN de base pour les utilisateurs** | Nom unique de base pour les utilisateurs. |
| **Nom de domaine** | Nom de domaine complet du domaine, par exemple « example.com ». Ne fournissez pas d’adresse IP dans cette zone de texte. |
| **Alias de domaine** | Nom NetBIOS du domaine. Ajoutez le nom NetBIOS du domaine Active Directory en tant qu’alias de la source d’identité si vous utilisez des authentifications SSPI. |
| **DN de base pour les groupes** | Nom unique de base pour les groupes. |
| **URL du serveur principal** | Serveur LDAP du contrôleur de domaine principal pour le domaine.<br><br>Utilisez le format `ldap://hostname:port` ou `ldaps://hostname:port`. Le port est généralement 389 pour les connexions LDAP et 636 pour les connexions LDAPS. Pour les déploiements de contrôleurs à domaines multiples Active Directory, le port est généralement 3268 pour LDAP et 3269 pour LDAPS.<br><br>Un certificat établissant une relation de confiance pour le point de terminaison LDAPS du serveur Active Directory est nécessaire lorsque vous utilisez `ldaps://` dans l'URL LDAP principale ou secondaire. |
| **URL du serveur secondaire** | Adresse d’un serveur LDAP de contrôleur de domaine secondaire utilisé pour le basculement. |
| **Choisir un certificat** | Si vous souhaitez utiliser LDAPS avec votre serveur LDAP Active Directory ou votre source d'identité serveur OpenLDAP, un bouton Choisir un certificat s'affiche une fois que vous avez entré `ldaps://` dans la zone de texte URL. Une URL secondaire n’est pas nécessaire. |
| **Nom d’utilisateur** | ID d’un utilisateur dans le domaine qui dispose au minimum d’un accès en lecture seule au DN de base pour les utilisateurs et les groupes. |
| **Mot de passe** | Mot de passe de l’utilisateur spécifié dans Nom d’utilisateur. |

Une fois que vous avez les informations du tableau précédent, vous pouvez ajouter votre domaine Active Directory local en tant que source d’identité d’authentification unique sur vCenter.

> [!TIP]
> Vous trouverez plus d’informations sur les sources d’identité d’authentification unique dans la [page de documentation VMware](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.psc.doc/GUID-B23B1360-8838-4FF2-B074-71643C4CB040.html).

## <a name="set-up-new-active-directory-on-a-private-cloud"></a>Configurer un nouveau domaine Active Directory sur un cloud privé

Vous pouvez configurer un nouveau domaine Active Directory sur votre cloud privé et l’utiliser comme source d’identité pour l’authentification unique.  Le domaine Active Directory peut faire partie d’une forêt Active Directory existante ou peut être configuré en tant que forêt indépendante.

### <a name="new-active-directory-forest-and-domain"></a>Nouvelle forêt et nouveau domaine Active Directory

Pour configurer une nouvelle forêt et un nouveau domaine Active Directory, vous avez besoin des éléments suivants :

* Une ou plusieurs machines virtuelles exécutant Microsoft Windows Server à utiliser en tant que contrôleurs de domaine pour la nouvelle forêt et le nouveau domaine Active Directory
* Une ou plusieurs machines virtuelles exécutant le service DNS pour la résolution de noms

Pour obtenir des étapes détaillées, consultez [Installer une nouvelle forêt Active Directory Windows Server 2012](/windows-server/identity/ad-ds/deploy/install-a-new-windows-server-2012-active-directory-forest--level-200-).

> [!TIP]
> Pour disposer d’une haute disponibilité des services, nous vous recommandons de configurer plusieurs contrôleurs de domaine et serveurs DNS.

Après avoir configuré la forêt et le domaine Active Directory, vous pouvez [ajouter une source d’identité sur vCenter](#add-an-identity-source-on-vcenter) pour votre nouveau domaine Active Directory.

### <a name="new-active-directory-domain-in-an-existing-active-directory-forest"></a>Nouveau domaine Active Directory dans une forêt Active Directory existante

Pour configurer un nouveau domaine Active Directory dans une forêt Active Directory existante, vous avez besoin des éléments suivants :

* Connexion VPN de site à site à votre emplacement de forêt Active Directory
* Serveur DNS pour résoudre le nom de votre forêt Active Directory existante

Pour obtenir des étapes détaillées, consultez [Installer un nouveau domaine enfant ou domaine d’arborescence Active Directory Windows Server 2012](/windows-server/identity/ad-ds/deploy/install-a-new-windows-server-2012-active-directory-child-or-tree-domain--level-200-).

Après avoir configuré le domaine Active Directory, vous pouvez [ajouter une source d’identité sur vCenter](#add-an-identity-source-on-vcenter) pour votre nouveau domaine Active Directory.

## <a name="set-up-active-directory-on-azure"></a>Configurer Active Directory sur Azure

Active Directory exécuté sur Azure est similaire à Active Directory exécuté localement.  Pour configurer Active Directory exécuté sur Azure en tant que source d’identité d’authentification unique sur vCenter, il faut que le serveur vCenter et le PSC disposent d’une connectivité au réseau virtuel Azure sur lequel s’exécutent les services Active Directory.  Vous pouvez établir cette connectivité avec la [connexion de réseau virtuel Azure à l’aide d’ExpressRoute](azure-expressroute-connection.md) allant du réseau virtuel Azure sur lequel Active Directory Services s’exécute au cloud privé CloudSimple.

Une fois la connexion réseau établie, suivez les étapes fournies dans [Ajouter un domaine Active Directory local en tant que source d’identité d’authentification unique](#add-on-premises-active-directory-as-a-single-sign-on-identity-source) pour l’ajouter en tant que source d’identité.  

## <a name="add-an-identity-source-on-vcenter"></a>Ajouter une source d’identité sur vCenter

1. [Augmentez les privilèges](escalate-private-cloud-privileges.md) sur votre cloud privé.

2. Connectez-vous à vCenter pour votre cloud privé.

3. Sélectionnez **Accueil > Administration**.

    ![Administration](media/OnPremAD01.png)

4. Sélectionnez **Authentification unique > Configuration**.

    ![Authentification unique](media/OnPremAD02.png)

5. Ouvrez l’onglet **Sources d’identité** et cliquez sur **+** pour ajouter une nouvelle source d’identité.

    ![Sources d’identité](media/OnPremAD03.png)

6. Sélectionnez **Active Directory en tant que serveur LDAP** , puis cliquez sur **Suivant**.

    ![Capture d'écran mettant en évidence l'option Active Directory en tant que serveur LDAP.](media/OnPremAD04.png)

7. Spécifiez les paramètres de la source d’identité pour votre environnement, puis cliquez sur **Suivant**.

    ![Active Directory](media/OnPremAD05.png)

8. Vérifiez les paramètres, puis cliquez sur **Terminer**.
