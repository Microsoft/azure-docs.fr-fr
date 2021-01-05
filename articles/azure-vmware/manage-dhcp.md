---
title: Gérer le protocole DHCP pour Azure VMware Solution
description: Découvrez comment créer et gérer le protocole DHCP pour votre cloud privé Azure VMware Solution.
ms.topic: how-to
ms.custom: contperf-fy21q2
ms.date: 11/09/2020
ms.openlocfilehash: bcaba4274b0e6b423e9fa490c80fc57204d4e153
ms.sourcegitcommit: d488a97dc11038d9cef77a0235d034677212c8b3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/21/2020
ms.locfileid: "97708549"
---
# <a name="manage-dhcp-for-azure-vmware-solution"></a>Gérer le protocole DHCP pour Azure VMware Solution

Les applications et les charges de travail exécutées dans un environnement de cloud privé nécessitent des services DHCP pour les affectations d’adresses IP.  Cet article explique les procédures de création et de gestion du protocole DHCP dans Azure VMware Solution de deux manières :

- Si vous utilisez NSX-T pour héberger votre serveur DHCP, vous devez [créer un serveur DHCP](#create-a-dhcp-server) et [relayer vers ce serveur](#create-dhcp-relay-service). Lorsque vous créez le serveur DHCP, vous ajoutez également un segment réseau et spécifiez la plage d’adresses IP DHCP.   

- Si vous utilisez un serveur DHCP externe tiers sur votre réseau, vous devez [créer un service de relais DHCP](#create-dhcp-relay-service). Lorsque vous créez un relais vers un serveur DHCP, que vous utilisiez NSX-T ou un tiers pour héberger votre serveur DHCP, vous devez spécifier la plage d’adresses IP DHCP.

>[!IMPORTANT]
>DHCP ne fonctionne pas pour les machines virtuelles sur le réseau Stretch HCX L2 VMware lorsque le serveur DHCP se trouve dans le centre de données local.  Par défaut, le service NSX empêche toutes les requêtes DHCP de traverser le réseau Stretch L2. Pour obtenir la solution, consultez la procédure [Envoyer des requêtes DHCP au serveur DHCP local](#send-dhcp-requests-to-the-on-premises-dhcp-server).


## <a name="create-a-dhcp-server"></a>Créer un serveur DHCP

Si vous envisagez d’utiliser NSX-T pour héberger votre serveur DHCP, créez un serveur DHCP. Ensuite, vous ajoutez un segment réseau et spécifiez la plage d’adresses IP DHCP.

1. Dans NSX-T Manager, sélectionnez **Mise en réseau** > **DHCP**, puis sélectionnez **Ajouter un serveur**.

1. Sélectionnez le **Type de serveur** **DHCP**, indiquez le nom du serveur et l’adresse IP, puis sélectionnez **Enregistrer**.

   :::image type="content" source="./media/manage-dhcp/dhcp-server-settings.png" alt-text="Ajouter un serveur DHCP" border="true":::

1. Sélectionnez **Passerelles de niveau 1**, sélectionnez les trois points suspendus sur la passerelle de niveau 1, puis sélectionnez **Modifier**.

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway.png" alt-text="Sélectionner la passerelle à utiliser" border="true":::

1. Sélectionnez **Aucune allocation d’adresse IP définie** pour ajouter un sous-réseau.

   :::image type="content" source="./media/manage-dhcp/add-subnet.png" alt-text="Ajouter un sous-réseau" border="true":::

1. Pour **Type**, sélectionnez **Serveur local DHCP**. 
   
1. Pour **Serveur DHCP**, sélectionnez **DHCP par défaut**, puis sélectionnez **Enregistrer**.

1. Sélectionnez de nouveau **Enregistrer**, puis sélectionnez **Fermer l’édition**.

### <a name="add-a-network-segment"></a>Ajouter un segment réseau

[!INCLUDE [add-network-segment-steps](includes/add-network-segment-steps.md)]


## <a name="create-dhcp-relay-service"></a>Créer un service de relais DHCP

Si vous voulez utiliser un serveur DHCP externe tiers sur votre réseau, vous devez créer un service de relais DHCP. Vous allez également spécifier la plage d’adresses IP DHCP dans NSX-T Manager. 

1. Dans NSX-T Manager, sélectionnez **Mise en réseau** > **DHCP**, puis sélectionnez **Ajouter un serveur**.

1. Sélectionnez le **Type de serveur** **Relais DHCP**, indiquez le nom du serveur et l’adresse IP, puis sélectionnez **Enregistrer**.

   :::image type="content" source="./media/manage-dhcp/create-dhcp-relay.png" alt-text="créer un service de relais DHCP" border="true":::

1. Sélectionnez **Passerelles de niveau 1**, sélectionnez les trois points suspendus sur la passerelle de niveau 1, puis sélectionnez **Modifier**.

   :::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway-relay.png" alt-text="Modifier la passerelle de niveau 1" border="true":::

1. Sélectionnez **Aucune allocation IP définie** pour définir l’allocation d’adresse IP.

   :::image type="content" source="./media/manage-dhcp/edit-ip-address-allocation.png" alt-text="Modifier l’allocation d’adresse IP" border="true":::

1. Pour **Type**, sélectionnez **Serveur DHCP**. 
   
1. Pour **Serveur DHCP**, sélectionnez **Relais DHCP**, puis sélectionnez **Enregistrer**.

1. Sélectionnez de nouveau **Enregistrer**, puis sélectionnez **Fermer l’édition**.


## <a name="specify-the-dhcp-ip-address-range"></a>Spécifier la plage d’adresses IP DHCP

1. Dans NSX-T Manager, sélectionnez **Mise en réseau** > **Segments**. 
   
1. Sélectionnez les trois points suspendus sur le nom du segment, puis **Modifier**.
   
1. Sélectionnez **Définir des sous-réseaux** pour spécifier l’adresse IP DHCP du sous-réseau. 
   
   :::image type="content" source="./media/manage-dhcp/network-segments.png" alt-text="segments réseau" border="true":::
      
1. Modifiez l’adresse IP de la passerelle si nécessaire, puis entrez l’adresse IP de la plage DHCP. 
      
   :::image type="content" source="./media/manage-dhcp/edit-subnet.png" alt-text="Modifier des sous-réseaux" border="true":::
      
1. Sélectionnez **Appliquer**, puis **Enregistrer**. Le segment est affecté à un pool de serveurs DHCP.
      
   :::image type="content" source="./media/manage-dhcp/assigned-to-segment.png" alt-text="Pool de serveurs DHCP affecté à un segment" border="true":::


## <a name="send-dhcp-requests-to-the-on-premises-dhcp-server"></a>Envoyer des requêtes DHCP au serveur DHCP local

Si vous souhaitez envoyer des requêtes DHCP de vos machines virtuelles Azure VMware Solution sur le segment étendu L2 vers le serveur DHCP local, créez un profil de segment de sécurité. 

1. Connectez-vous au vCenter local puis, sous **Accueil**, sélectionnez **HCX**.

1. Sélectionnez **Extension réseau** sous **Services**.

1. Sélectionnez l’extension réseau devant prendre en charge les requêtes DHCP d’Azure VMware Solution vers l’emplacement local. 

1. Entrez ici le nom du réseau de destination.  

   :::image type="content" source="media/manage-dhcp/hcx-find-destination-network.png" alt-text="Capture d’écran d’une extension réseau dans le client VMware vSphere" lightbox="media/manage-dhcp/hcx-find-destination-network.png":::

1. Dans Azure VMware Solution NSX-T Manager, sélectionnez **Mise en réseau** > **Segments** > **Profils de segment**. 

1. Sélectionnez **Ajouter un profil de segment**, puis **Sécurité du segment**.

   :::image type="content" source="media/manage-dhcp/add-segment-profile.png" alt-text="Capture d’écran de la procédure d’ajout d’un profil de segment dans NSX-T" lightbox="media/manage-dhcp/add-segment-profile.png":::

1. Fournissez un nom et une balise, puis définissez le bouton bascule **Filtre BPDU** sur ACTIVÉ et tous les boutons bascule DHCP sur DÉSACTIVÉ.

   :::image type="content" source="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png" alt-text="Capture d’écran montrant le filtre BPDU activé et les boutons bascule DHCP désactivés" lightbox="media/manage-dhcp/add-segment-profile-bpdu-filter-dhcp-options.png":::

1. Supprimez toutes les adresses MAC, le cas échéant, sous **Liste d'autorisation du filtre BPDU**.  Ensuite, sélectionnez **Enregistrer**.

   :::image type="content" source="media/manage-dhcp/add-segment-profile-bpdu-filter-allow-list.png" alt-text="Capture d’écran montrant des adresses MAC dans la liste d’autorisation du filtre BPDU":::

1. Sous **Mise en réseau** > **Segments** > **Segments**, dans la zone de recherche, entrez le nom réseau de la définition.

   :::image type="content" source="media/manage-dhcp/networking-segments-search.png" alt-text="Capture d’écran du champ de Mise en réseau > Segments":::

1. Sélectionnez les trois points suspendus sur le nom du segment, puis **Modifier**.

   :::image type="content" source="media/manage-dhcp/edit-network-segment.png" alt-text="Capture d’écran du bouton modifier pour le segment" lightbox="media/manage-dhcp/edit-network-segment.png":::

1. Remplacez **Sécurité du segment** par le profil du segment que vous avez créé précédemment.

   :::image type="content" source="media/manage-dhcp/edit-segment-security.png" alt-text="Capture d’écran du champ Sécurité du segment" lightbox="media/manage-dhcp/edit-segment-security.png":::

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Maintenance de l’hôte et gestion du cycle de vie](concepts-private-clouds-clusters.md#host-maintenance-and-lifecycle-management).