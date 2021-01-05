---
title: Ajouter un segment réseau NSX-T
description: Étapes à suivre pour ajouter un segment réseau NSX-T pour Azure VMware Solution.
ms.topic: include
ms.date: 11/09/2020
ms.openlocfilehash: 5b97f0b280fa12eff39c9601bb73e439dba8e9fd
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335065"
---
<!-- Used in manage-dhcp.md and tutorial-nsx-t-network-segment.md -->

1. Dans NSX-T Manager, sélectionnez **Networking** (Réseaux)  > **Segments**, puis sélectionnez **Add Segment** (Ajouter un segment). 

   :::image type="content" source="../media/nsxt/nsxt-segments-overview.png" alt-text="Capture d’écran montrant comment ajouter un nouveau segment":::

1. Sélectionnez **Add Segment** (Ajouter un segment), puis entrez un nom pour le segment.

1. Sélectionnez la passerelle de niveau 1 (TNTxx-T1) comme valeur **Connected Gateway** (Passerelle connectée) et laissez Flexible comme **Type**.

1. Sélectionnez la zone de transport (**Transport Zone**) de superposition préconfigurée (TNTxx-OVERLAY-TZ), puis sélectionnez **Set Subnets** (Définir des sous-réseaux). 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-specs.png" alt-text="Définissez les paramètres Segment Name, Connected Gateway and Type et Transport Zone, puis sélectionnez Set Subnet.":::

1. Entrez l’adresse IP de la passerelle, puis sélectionnez **ADD**. 

   >[!IMPORTANT]
   >L’adresse IP doit se trouver dans un bloc d’adresses RFC1918 sans chevauchement, ce qui garantit la connexion aux machines virtuelles sur le nouveau segment.

   :::image type="content" source="../media/nsxt/nsxt-create-segment-gateway.png" alt-text="Définissez l’adresse IP de la passerelle pour le nouveau segment, puis sélectionnez ADD.":::

1. Sélectionnez **Apply** (Appliquer), puis **Save** (Enregistrer).

1. Sélectionnez **No** (Non) pour refuser l’option qui permet de continuer la configuration du segment. 

   :::image type="content" source="../media/nsxt/nsxt-create-segment-continue-no.png" alt-text="Refusez de poursuivre la configuration du segment réseau nouvellement créé en sélectionnant NO.":::

1. Confirmez la présence du nouveau segment réseau. Dans cet exemple, **ls01** est le nouveau segment réseau.

   1. Dans NSX-T Manager, sélectionnez **Networking** (Réseaux)  > **Segments**. 

      :::image type="content" source="../media/nsxt/nsxt-new-segment-overview-2.png" alt-text="Vérifiez que le nouveau segment réseau est présent dans NSX-T.":::

   1. Dans vCenter, sélectionnez **Networking (Réseaux) > SDDC-Datacenter**.

      :::image type="content" source="../media/nsxt/vcenter-with-ls01-2.png" alt-text="Vérifiez que le nouveau segment réseau est présent dans vCenter.":::