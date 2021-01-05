---
title: Infrastructure et connectivité à SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Configurez l’infrastructure de connectivité requise pour utiliser SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/12/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4f6f8a057651ce56731c180a2ccbb05e35d8fefc
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967548"
---
# <a name="sap-hana-large-instances-deployment"></a>Déploiement de SAP HANA (grandes instances) 

Cet article suppose que vous avez acheté SAP HANA sur Azure (grandes instances) depuis Microsoft. Avant de lire cet article, pour comprendre le contexte général, consultez les [termes courants concernant les grandes Instances HANA](hana-know-terms.md) et les [références SKU des grandes instances HANA](hana-available-skus.md).


Microsoft a besoin des informations suivantes pour déployer les unités de grande instance HANA :

- Nom du client.
- Coordonnées professionnelles (y compris l’adresse e-mail et le numéro de téléphone).
- Coordonnées techniques (y compris l’adresse e-mail et le numéro de téléphone).
- Coordonnées de mise en réseau technique (y compris l’adresse e-mail et le numéro de téléphone).
- Région de déploiement Azure (par exemple, USA Ouest, Australie Est ou Europe Nord).
- Référence SKU (configuration) de SAP HANA sur Azure (grandes instances).
- Pour chaque région de déploiement Azure :
    - Une plage d’adresses IP /29 pour les connexions ER-P2P qui relient les réseaux virtuels Azure aux grandes instances HANA.
    - Un bloc CIDR /24 utilisé pour le pool d’adresses IP du serveur de grandes instances HANA.
    - Facultatif avec [ExpressRoute Global Reach](../../../expressroute/expressroute-global-reach.md) pour activer le routage direct depuis un site local vers des unités de grandes instances HANA ou le routage entre des unités de grandes instances HANA dans différentes régions Azure, vous devez réserver une autre plage d’adresses IP /29. Cette plage ne peut pas chevaucher les autres plages d’adresses IP que vous avez définies auparavant.
- Les valeurs de plage d’adresses IP utilisées dans l’attribut Espace d’adressage du réseau virtuel de chaque réseau virtuel Azure qui se connecte à de grandes instances HANA.
- Données pour chaque système de grandes instances HANA :
  - Nom d’hôte souhaité, idéalement avec le nom de domaine complet.
  - Adresse IP souhaitée pour l’unité de grande instance HANA en dehors de la plage d’adresses du pool d’adresses IP du serveur. (Les 30 premières adresses IP dans la plage d’adresses du pool d’adresses IP du serveur sont réservées à un usage interne au sein des grandes instances HANA.)
  - Nom de SID SAP HANA pour l’instance HANA (requis pour créer les volumes de disque liés à SAP HANA nécessaires). Microsoft a besoin de l’identificateur de sécurité HANA pour créer les autorisations pour sidadm sur les volumes NFS. Ces volumes sont attachés à l’unité de grande instance HANA. L’identificateur de sécurité HANA est également utilisé comme l’un des composants de nom des volumes de disque qui sont montés. Si vous souhaitez exécuter plusieurs instances HANA sur l’unité, vous devez répertorier plusieurs identificateurs de sécurité HANA. Chacun d’eux obtient un ensemble distinct de volumes assignés.
  - Dans le système d’exploitation Linux, l’utilisateur sidadm a un ID de groupe. Cet ID est nécessaire pour créer les volumes de disque liés à SAP HANA nécessaires. L’installation de SAP HANA crée généralement le groupe sapsys avec un ID de groupe de 1 001. L’utilisateur sidadm fait partie de ce groupe.
  - Dans le système d’exploitation Linux, l’utilisateur sidadm a un ID d’utilisateur. Cet ID est nécessaire pour créer les volumes de disque liés à SAP HANA nécessaires. Si vous exécutez plusieurs instances HANA sur l’unité, répertoriez tous les utilisateurs sidadm. 
- ID de l’abonnement Azure auquel les grandes instances HANA Azure vont être directement connectées. Cet ID d’abonnement désigne l’abonnement Azure qui va être facturé avec les unités de grande instance HANA.

Une fois les informations précédentes fournies, Microsoft approvisionne SAP HANA sur Azure (grandes instances). Microsoft vous envoie des informations pour lier vos réseaux virtuels Azure aux grandes instances HANA. Vous pouvez également accéder aux unités de grande instance HANA.

Utilisez la séquence suivante pour vous connecter aux grandes instances HANA après leur déploiement par Microsoft :

1. [Connexion de machines virtuelles Azure à de grandes instances HANA](hana-connect-azure-vm-large-instances.md)
2. [Connexion d’un réseau virtuel à de grandes instances HANA ExpressRoute](hana-connect-vnet-express-route.md)
3. [Conditions requises supplémentaires du réseau (facultatives)](hana-additional-network-requirements.md)
