---
title: Tutoriel - Accéder à votre cloud privé
description: Découvrez comment accéder à un cloud privé Azure VMware Solution
ms.topic: tutorial
ms.date: 09/21/2020
ms.openlocfilehash: b7f21dbba5256712607241bb1258962fc22418fb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91578414"
---
# <a name="tutorial-access-an-azure-vmware-solution-private-cloud"></a>Tutoriel : Accéder à un cloud privé Azure VMware Solution

Azure VMware Solution ne vous permet pas de gérer votre cloud privé avec votre vCenter local. Vous devez procéder à une configuration supplémentaire et disposer d’une connexion à une instance vCenter locale via une jumpbox. 

Dans ce tutoriel, vous allez créer une jumpbox dans le groupe de ressources que vous avez créé dans le [tutoriel précédent](tutorial-configure-networking.md) et vous connecter à vCenter. La jumpbox est une machine virtuelle Windows (VM) sur le même réseau virtuel que vous avez créé.  Elle permet d’accéder à vCenter et NSX Manager. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une machine virtuelle Windows à utiliser pour se connecter à vCenter
> * Vous connecter à vCenter à partir de votre machine virtuelle

## <a name="create-a-new-windows-virtual-machine"></a>Créer une machine virtuelle Windows

[!INCLUDE [create-avs-jump-box-steps](includes/create-jump-box-steps.md)]

## <a name="connect-to-the-local-vcenter-of-your-private-cloud"></a>Se connecter au vCenter local de votre cloud privé

1. À partir de la jumpbox, connectez-vous au client vSphere avec VMware vCenter SSO à l’aide d’un nom d’utilisateur d’administrateur cloud et vérifiez que l’interface utilisateur s’affiche correctement.

1. Dans le portail Azure, sélectionnez votre cloud privé puis, dans la vue **Vue d’ensemble**, sélectionnez **Identité > Par défaut**. 

   Les URL et les informations d’identification de l’utilisateur pour le cloud privé vCenter et NSX-T Manager s’affichent.

   :::image type="content" source="media/tutorial-access-private-cloud/ss4-display-identity.png" alt-text="Affichez les URL et les informations d’identification vCenter et NSX Manager du cloud privé." border="true":::

1. Accédez à la machine virtuelle que vous avez créée à l’étape précédente et connectez-vous à cette machine. 

   Si vous avez besoin d’aide pour la connexion à la machine virtuelle, consultez [connexion à une machine virtuelle](../virtual-machines/windows/connect-logon.md#connect-to-the-virtual-machine) pour plus d’informations.

1. Dans la machine virtuelle Windows, ouvrez un navigateur et accédez aux URL vCenter et NSX-T Manager dans deux onglets. 

1. Sous l’onglet vCenter, entrez les informations d’identification de l’utilisateur `cloudadmin@vmcp.local` de l’étape précédente.

   :::image type="content" source="media/tutorial-access-private-cloud/ss5-vcenter-login.png" alt-text="Affichez les URL et les informations d’identification vCenter et NSX Manager du cloud privé." border="true":::

   :::image type="content" source="media/tutorial-access-private-cloud/ss6-vsphere-client-home.png" alt-text="Affichez les URL et les informations d’identification vCenter et NSX Manager du cloud privé." border="true":::

1. Sous le deuxième onglet du navigateur, connectez-vous à NSX-T Manager.

   :::image type="content" source="media/tutorial-access-private-cloud/ss10-nsx-manager-home.png" alt-text="Affichez les URL et les informations d’identification vCenter et NSX Manager du cloud privé." border="true":::



## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Windows à utiliser pour se connecter à vCenter
> * Vous connecter à vCenter à partir de votre machine virtuelle

Passez au tutoriel suivant pour découvrir comment créer un réseau virtuel afin de configurer la gestion locale pour vos clusters de cloud privé.

> [!div class="nextstepaction"]
> [Créer un réseau virtuel](tutorial-configure-networking.md)


