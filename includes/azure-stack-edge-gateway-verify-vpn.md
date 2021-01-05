---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/27/2020
ms.author: alkohli
ms.openlocfilehash: efa9e996cbfbc8954c294b4da8900b1d5bcbeeed
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465339"
---
Pour vérifier le VPN, vous pouvez créer un compte de stockage accessible uniquement par le réseau virtuel que vous avez créé. Pour créer et associer ce compte de stockage au réseau virtuel, procédez comme suit :

1. Créez un compte de stockage. Vous pouvez utiliser le compte de stockage que vous prévoyez d’utiliser avec votre appareil Azure Stack Edge. Essayez d’accéder au compte de stockage à partir d’un réseau externe (en dehors du réseau sélectionné). Le compte de stockage doit être accessible.
2. Sur le Portail Azure, accédez au compte de stockage. 
3. Accédez à **Pare-feu et réseaux virtuels**. Dans le volet droit du menu **Autoriser l’accès à partir de**, choisissez **Réseaux sélectionnés**. Dans **Sécuriser le compte de stockage avec des réseaux virtuels**, choisissez **+ Ajouter un réseau virtuel existant**.

    ![Vérification du VPN 1](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-1.png)

4. Dans le panneau **Ajouter des réseaux** :

    - Sélectionnez l’abonnement. Il s’agit du même abonnement que celui qui est associé à votre ressource Azure Stack Edge/Data Box Gateway. 
    - Dans la liste déroulante, choisissez le réseau virtuel à associer à ce compte de stockage. Sélectionnez le réseau virtuel créé à l’étape précédente.
    - Dans **Sous-réseaux**, choisissez la **_valeur par défaut_* _ et _GatewaySubnet*. Les points de terminaison de service sont activés pour cette combinaison réseau virtuel/sous-réseau. L’activation de l’accès peut prendre jusqu’à 15 minutes.
    - Sélectionnez **Activer**.

    ![Vérification du VPN 2](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-2.png)
    
4. Enregistrez les **Paramètres**.

    ![Vérification du VPN 3](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-3.png)

5. Une fois les paramètres enregistrés, les sous-réseaux pour lesquels le réseau virtuel est activé apparaissent.

    ![Vérification du VPN 4](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/verify-vpn-4.png)

5. Pour vérifier que les données passent uniquement par le VPN, procédez comme suit : 
    - Essayez d’accéder au compte de stockage à partir d’un réseau externe (en dehors du réseau sélectionné). Le compte de stockage ne doit pas être accessible. 
    - Essayez d’accéder au compte de stockage à partir du réseau virtuel/des sous-réseaux que vous avez activés dans les réseaux sélectionnés. Le compte de stockage doit être accessible. 
 
Vous ne pouvez accéder à ce compte de stockage que si le VPN est activé. Si vous désactivez le VPN, vous devrez également ajuster les paramètres réseau du compte de stockage. 

Pour plus d’informations, consultez [Configuration des pare-feu et des réseaux virtuels du Stockage Azure](../articles/storage/common/storage-network-security.md). 

