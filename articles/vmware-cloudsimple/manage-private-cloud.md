---
title: Gérer le cloud privé Azure VMware Solution by CloudSimple
description: Décrit les fonctionnalités disponibles pour gérer les ressources et les activités de votre cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 06/10/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: b0aff8298ac2335e5216252eb2cbd5a393cc37d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86524994"
---
# <a name="manage-private-cloud-resources-and-activity"></a>Gérer les ressources et les activités de cloud privé

Les clouds privés sont gérés à partir du portail CloudSimple.  Vérifiez l’état, les ressources disponibles, les activités sur le cloud privé et d’autres paramètres à partir du portail CloudSimple.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="access-the-cloudsimple-portal"></a>Accéder au portail CloudSimple

Accédez au [portail CloudSimple](access-cloudsimple-portal.md).

## <a name="view-the-list-of-private-clouds"></a>Afficher la liste des clouds privés

L’onglet **Clouds privés** de la page **Ressources** répertorie tous les clouds privés dans votre abonnement. Les détails fournis incluent le nom, le nombre de clusters vSphere, l’emplacement, l’état actuel du cloud privé et des informations sur les ressources.

![Page Clouds privés](media/manage-private-cloud.png)

Sélectionnez un cloud privé pour avoir accès à des informations et à des actions supplémentaires.

## <a name="private-cloud-summary"></a>Résumé du cloud privé

Affichez un résumé complet du cloud privé sélectionné.  La page de résumé inclut les serveurs DNS déployés sur le cloud privé.  Vous pouvez configurer la redirection DNS des serveurs DNS locaux vers vos serveurs DNS de cloud privé.  Pour plus d’informations sur la redirection DNS, consultez l’article [Configure DNS for name resolution for Private Cloud vCenter from on-premises](./on-premises-dns-setup.md) (Configurer DNS pour la résolution de noms pour Private Cloud vCenter sur site).

![Résumé du cloud privé](media/private-cloud-summary.png)

### <a name="available-actions"></a>Actions disponibles

* [Lancer le client vSphere](./vcenter-access.md). Accédez au serveur vCenter de ce cloud privé.
* [Acheter des nœuds](create-nodes.md). Ajoutez des nœuds à ce cloud privé.
* [Étendre](expand-private-cloud.md). Ajoutez des nœuds à ce cloud privé.
* **Actualiser**. Mettez à jour les informations de cette page.
* **Supprimer**. Vous pouvez supprimer le cloud privé à tout moment. **Avant de le supprimer, assurez-vous que vous avez sauvegardé l’ensemble des systèmes et des données.** La suppression d’un cloud privé supprime toutes les machines virtuelles, la configuration de vCenter et les données. Cliquez sur **Supprimer** dans la section de résumé du cloud privé sélectionné. Après la suppression, toutes les données du cloud privé sont effacées via un processus d’effacement sécurisé et conforme à des normes strictes.
* [Modifier les privilèges vSphere](escalate-private-cloud-privileges.md).  Élevez vos privilèges sur ce cloud privé.

## <a name="private-cloud-vlanssubnets"></a>Réseaux VLAN/sous-réseaux du cloud privé

Affichez la liste des réseaux VLAN/sous-réseaux définis pour le cloud privé sélectionné.  Cette liste inclut les réseaux VLAN/sous-réseaux de gestion créés au moment de la création du cloud privé.

![Cloud privé - Réseaux VLAN/sous-réseaux](media/private-cloud-vlans-subnets.png) 

### <a name="available-actions"></a>Actions disponibles

* [Ajouter des réseaux VLAN/sous-réseaux](./create-vlan-subnet.md). Ajoutez un réseau VLAN/sous-réseau à ce cloud privé.

Sélectionnez un réseau VLAN/sous-réseau pour avoir accès aux actions suivantes.
* [Attacher une table de pare-feu](./firewall.md). Attachez une table de pare-feu à ce cloud privé.
* **Modifier**
* **Supprimer** (uniquement les réseaux VLAN/sous-réseaux définis par l’utilisateur)

## <a name="private-cloud-activity"></a>Activité du cloud privé

Affichez les informations suivantes sur le cloud privé sélectionné.  Les informations d’activité forment une liste filtrée de toutes les activités sur le cloud privé sélectionné.  Cette page peut afficher jusqu’à 25 activités récentes.

* Alertes récentes
* Événements récents
* Tâches récentes
* Audit récent

![Cloud privé - Activité](media/private-cloud-activity.png)

## <a name="cloud-racks"></a>Racks de cloud

Les racks de cloud sont les principaux composants de votre cloud privé. Chaque rack fournit une unité de capacité. CloudSimple configure automatiquement les racks de cloud en fonction de vos sélections lors de la création ou de l’extension d’un cloud privé.  Affichez la liste complète des racks de cloud, y compris le cloud privé auquel chacun est attribué.

![Cloud privé - Racks de cloud](media/private-cloud-cloudracks.png)

## <a name="vsphere-management-network"></a>Réseau de gestion vSphere

Liste des ressources de gestion et des machines virtuelles VMware actuellement configurées sur le cloud privé. Ces informations incluent le version du logiciel, le nom de domaine complet (FQDN) et l’adresse IP des ressources.

![Cloud privé - Réseau de gestion vSphere](media/private-cloud-vsphere-management-network.png)

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des machines virtuelles VMware sur Azure](quickstart-create-vmware-virtual-machine.md)
* Découvrir plus en détail les [clouds privés](cloudsimple-private-cloud.md)
