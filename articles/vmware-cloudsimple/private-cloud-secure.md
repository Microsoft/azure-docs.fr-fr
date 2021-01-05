---
title: Azure VMware Solutions by CloudSimple – Sécurisation d’un cloud privé
description: Indique comment sécuriser le cloud privé Azure VMware Solutions by CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 000e43a7861bd155ebbd7175db96dd323731464b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518209"
---
# <a name="how-to-secure-your-private-cloud-environment"></a>Sécurisation de votre environnement cloud privé

Définissez le contrôle d’accès en fonction du rôle (RBAC) pour le service et le portail CloudSimple ainsi que le cloud privé à partir d’Azure.  Les utilisateurs, les groupes et les rôles pouvant accéder aux vCenter du cloud privé sont indiqués à l’aide de l’authentification unique VMware.  

## <a name="azure-rbac-for-cloudsimple-service"></a>Azure RBAC pour le service CloudSimple

La création du service CloudSimple requiert un rôle **Propriétaire** ou **Contributeur** sur l’abonnement Azure.  Par défaut, tous les propriétaires et contributeurs peuvent créer un service CloudSimple et accéder au portail CloudSimple pour créer et gérer des clouds privés.  Seul un service CloudSimple peut être créé par région.  Pour restreindre l’accès à des administrateurs précis, suivez la procédure ci-dessous.

1. Créez un service CloudSimple dans un nouveau **groupe de ressources** sur le portail Azure
2. Spécifiez Azure RBAC pour le groupe de ressources.
3. Acheter des nœuds et utiliser le même groupe de ressources que celui du service CloudSimple

Seuls les utilisateurs disposant de privilèges **Propriétaire** ou **Contributeur** sur le groupe de ressources peuvent voir le service CloudSimple et lancer le portail CloudSimple.

Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../role-based-access-control/overview.md)

## <a name="rbac-for-private-cloud-vcenter"></a>RBAC pour le cloud privé vCenter

Un utilisateur par défaut `CloudOwner@cloudsimple.local` est créé dans le domaine SSO vCenter lors de la création d’un cloud privé.  L’utilisateur CloudOwner dispose des privilèges lui permettant de gérer vCenter. Des sources d’identité supplémentaires sont ajoutées au SSO vCenter pour donner accès à différents utilisateurs.  Les groupes et les rôles prédéfinis sont configurés sur le vCenter qui peut être utilisé pour ajouter des utilisateurs supplémentaires.

### <a name="add-new-users-to-vcenter"></a>Ajouter de nouveaux utilisateurs dans VCenter

1. [Élevez des privilèges](escalate-private-cloud-privileges.md) pour l’utilisateur **CloudOwner\@cloudsimple.local** sur le cloud privé.
2. Connectez-vous à vCenter avec **CloudOwner\@cloudsimple.local**
3. [Ajouter des utilisateurs avec authentification unique sur vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-72BFF98C-C530-4C50-BF31-B5779D2A4BBB.html).
4. Ajoutez des utilisateurs pour les [groupes avec authentification unique sur vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

Pour plus d’informations sur les groupes et les rôles prédéfinis, consultez la page [Modèle d’autorisation de cloud privé CloudSimple de VMware vCenter](learn-private-cloud-permissions.md).

### <a name="add-new-identity-sources"></a>Ajouter de nouvelles sources d’identité

Vous pouvez ajouter des fournisseurs d’identité supplémentaires pour le domaine SSO vCenter de votre Cloud privé.  Les fournisseurs d’identité fournissent une authentification et les groupes SSO vCenter accordent une autorisation aux utilisateurs.

* [Utilisez Active Directory en tant que fournisseur d’identité](set-vcenter-identity.md) sur le cloud privé vCenter.
* [Utiliser Azure AD en tant que fournisseur d’identité](azure-ad.md) sur le cloud privé vCenter

1. [Élevez des privilèges](escalate-private-cloud-privileges.md) pour l’utilisateur **CloudOwner\@cloudsimple.local** sur le cloud privé.
2. Connectez-vous à vCenter avec **CloudOwner\@cloudsimple.local**
3. Ajoutez des utilisateurs à partir du fournisseur d’identité à [des groupes avec authentification unique sur vCenter](https://docs.vmware.com/en/VMware-vSphere/5.5/com.vmware.vsphere.security.doc/GUID-CDEA6F32-7581-4615-8572-E0B44C11D80D.html).

## <a name="secure-network-on-your-private-cloud-environment"></a>Sécurisez le réseau de votre environnement de cloud privé

La sécurité du réseau de l’environnement cloud privé est assurée par la sécurisation de l’accès au réseau et le contrôle du trafic réseau entre les ressources.

### <a name="access-to-private-cloud-resources"></a>Accéder aux ressources de cloud privé

Vous pouvez accéder au cloud privé vCenter et aux ressources grâce à une connexion réseau sécurisée :

* **[Connexion ExpressRoute](on-premises-connection.md)** . ExpressRoute fournit une connexion sécurisée, à bande passante élevée et à faible latence à partir de votre environnement local.  L’utilisation de la connexion permet à vos services, réseaux et utilisateurs locaux d’accéder à votre cloud privé vCenter.
* **[Passerelle VPN de site à site](vpn-gateway.md)** . Le VPN de site à site permet d’accéder à vos ressources du cloud privé à partir d’un emplacement local grâce à un tunnel sécurisé.  Vous pouvez indiquer quels réseaux locaux peuvent envoyer et recevoir le trafic réseau sur votre cloud privé.
* **[Passerelle VPN de point à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway)** . Utilisez une connexion VPN de point à site pour pouvoir accéder facilement à distance à votre cloud privé vCenter.

### <a name="control-network-traffic-in-private-cloud"></a>Contrôlez le trafic réseau dans le cloud privé

Les tables et les règles de pare-feu contrôlent le trafic réseau dans le cloud privé.  La table de pare-feu vous permet de contrôler le trafic réseau entre un réseau source ou une adresse IP et un réseau ou une adresse IP de destination en fonction de la combinaison des règles définies dans la table.

1. Créez une [table de pare-feu](firewall.md#add-a-new-firewall-table).
2. [Ajoutez des règles](firewall.md#create-a-firewall-rule) à la table de pare-feu.
3. [Attachez une table de pare-feu à un réseau local virtuel/sous-réseau](firewall.md#attach-vlans-subnet).
