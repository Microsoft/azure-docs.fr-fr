---
title: Configurer des adresses IP privées pour des machines virtuelles (Classic) - Portail Azure | Microsoft Docs
description: Découvrez comment configurer des adresses IP privées pour des machines virtuelles (Classic) à l’aide du portail Azure.
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
tags: azure-service-management
ms.assetid: b8ef8367-58b2-42df-9f26-3269980950b8
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/04/2016
ms.author: genli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 57df05918b590cedbf4af0464690ef2524f8ba79
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91650505"
---
# <a name="configure-private-ip-addresses-for-a-virtual-machine-classic-using-the-azure-portal"></a>Configurer des adresses IP privées pour une machine virtuelle (Classic) à l’aide du portail Azure

[!INCLUDE [virtual-networks-static-private-ip-selectors-classic-include](../../includes/virtual-networks-static-private-ip-selectors-classic-include.md)]

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Cet article traite du modèle de déploiement classique. Vous pouvez également [gérer une adresse IP privée statique dans le modèle de déploiement de Resource Manager](virtual-networks-static-private-ip-arm-pportal.md).

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

Les étapes de l’exemple suivant supposent qu’un environnement simple a déjà été créé. Si vous souhaitez exécuter les étapes telles qu’elles sont présentées dans ce document, commencez par créer l’environnement de test décrit dans [Créer un réseau virtuel](virtual-networks-create-vnet-classic-pportal.md).

## <a name="how-to-specify-a-static-private-ip-address-when-creating-a-vm"></a>Comment spécifier une adresse IP privée statique lors de la création d’une machine virtuelle
Pour créer une machine virtuelle nommée *DNS01* dans le sous-réseau *FrontEnd* d’un réseau virtuel nommé *TestVNet* avec l’adresse IP privée statique *192.168.1.101*, effectuez les étapes suivantes :

1. Dans un navigateur, accédez à https://portal.azure.com et, si nécessaire, connectez-vous avec votre compte Azure.
2. Sélectionnez **Nouveau** > **Compute** > **Windows Server 2012 R2 Datacenter** (notez que la liste **Sélectionner un modèle de déploiement** contient déjà **Classique**), puis sélectionnez **Créer**.
   
    ![Capture d’écran représentant le Portail Azure dans lequel la mosaïque Nouveau > Calculer > Windows Server 2012 R2 Datacenter est mise en surbrillance.](./media/virtual-networks-static-ip-classic-pportal/figure01.png)
3. Sous **Créer une machine virtuelle**, entrez le nom de la machine virtuelle à créer (*DNS01* dans le scénario), le compte d’administrateur local et un mot de passe.
   
    ![Capture d’écran montrant comment créer une machine virtuelle en entrant le nom de la machine virtuelle, le nom d’utilisateur de l’administrateur local et le mot de passe.](./media/virtual-networks-static-ip-classic-pportal/figure02.png)
4. Sélectionnez **Configuration facultative** > **Réseau** > **Réseau virtuel**, puis sélectionnez **TestVNet**. Si l’option **TestVNet** n’est pas disponible, assurez-vous que vous utilisez l’emplacement *USA Centre* et avez créé l’environnement de test décrit au début de cet article.
   
    ![Capture d’écran représentant l’option Configuration facultative > Réseau > Réseau virtuel > TestVNet est mise en surbrillance.](./media/virtual-networks-static-ip-classic-pportal/figure03.png)
5. Sous **Réseau**, vérifiez que le sous-réseau actuellement sélectionné est *FrontEnd*, puis sélectionnez **Adresses IP**. Sous **Affectation d’adresses IP**, sélectionnez **Statique**, puis entrez *192.168.1.101* pour **Adresse IP**, comme illustré ci-dessous.
   
    ![Capture d’écran représentant le champ Adresses IP dans lequel vous pouvez saisir l’adresse IP statique.](./media/virtual-networks-static-ip-classic-pportal/figure04.png)    
6. Sélectionnez **OK** sous **Adresses IP**, sélectionnez **OK** sous **Réseau**, puis sélectionnez **OK** sous **Configuration facultative**.
7. Sous **Créer une machine virtuelle**, sélectionnez **Créer**. Notez la vignette ci-dessous affichée dans votre tableau de bord :
   
    ![Capture d’écran représentant la mosaïque Windows Server 2012 R2 Datacenter.](./media/virtual-networks-static-ip-classic-pportal/figure05.png)

## <a name="how-to-retrieve-static-private-ip-address-information-for-a-vm"></a>Récupération d’informations d’adresse IP privée statique pour une machine virtuelle
Pour afficher les informations d’adresse IP privée statique de la machine virtuelle créée lors des étapes ci-dessus, exécutez les étapes ci-dessous.

1. Dans le portail Azure, sélectionnez **Parcourir tout** > **Machines virtuelles (Classic)**  > **DNS01** > **Tous les paramètres** > **Adresses IP**, puis examinez l’affectation d’adresses IP et l’adresse IP, telles qu’illustrées ci-dessous.
   
    ![Création d'une machine virtuelle dans le portail Azure](./media/virtual-networks-static-ip-classic-pportal/figure06.png)

## <a name="how-to-remove-a-static-private-ip-address-from-a-vm"></a>Comment supprimer une adresse IP privée statique d’une machine virtuelle

Sous **Adresses IP**, sélectionnez **Dynamique** à droite de **Affectation d’adresses IP**, sélectionnez **Enregistrer**, puis sélectionnez **Oui**, comme illustré dans l’image suivante :
   
![Capture d’écran représentant l’opération de suppression de l’adresse IP privée statique d’une machine virtuelle à l’aide de la sélection dynamique à droite de l’étiquette d’attribution d’adresse IP.](./media/virtual-networks-static-ip-classic-pportal/figure07.png)

## <a name="how-to-add-a-static-private-ip-address-to-an-existing-vm"></a>Ajout d’une adresse IP privée statique à une machine virtuelle existante

1. Sous **Adresses IP**, illustré précédemment, sélectionnez **Statique** à droite de **Affectations d’adresses IP**.
2. Saisissez *192.168.1.101* pour **Adresse IP**, sélectionnez **Enregistrer**, puis sélectionnez **Oui**.

## <a name="set-ip-addresses-within-the-operating-system"></a>Définir des adresses IP au sein du système d’exploitation

Il est recommandé de ne pas statiquement assigner l’IP privée assignée à la machine virtuelle Azure au sein du système d’exploitation d’une machine virtuelle, sauf si nécessaire. Si vous définissez manuellement l’adresse IP privée dans le système d’exploitation, assurez-vous qu’il s’agit de la même adresse que l’adresse IP privée assignée à la machine virtuelle Azure ou vous pouvez perdre la connectivité à la machine virtuelle. Vous ne devez jamais assigner manuellement l’adresse IP publique assignée à une machine virtuelle Azure au sein du système d’exploitation de la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [adresses IP publiques réservées](virtual-networks-reserved-public-ip.md) .
* En savoir plus sur les [adresses IP publiques de niveau d’instance](virtual-networks-instance-level-public-ip.md) .
* Consulter les [API REST d’adresse IP réservée](https://msdn.microsoft.com/library/azure/dn722420.aspx).

