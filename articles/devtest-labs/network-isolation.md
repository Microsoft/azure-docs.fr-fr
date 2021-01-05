---
title: Isolement réseau dans Azure DevTest Labs
description: Découvrir l’isolement réseau dans Azure DevTest Labs.
ms.topic: article
ms.date: 08/25/2020
ms.openlocfilehash: 28bfa048a8a6666deb58a8ecfff38779c91d95b8
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352930"
---
# <a name="network-isolation-in-devtest-labs"></a>Isolement réseau dans DevTest Labs

Un [réseau virtuel Azure](../virtual-network/virtual-networks-overview.md) agit en tant que limite de sécurité, isolant vos ressources Azure de l’Internet public. Vous pouvez également joindre un réseau virtuel Azure à votre réseau local pour être en mesure de vous connecter en toute sécurité à vos ressources locales. Dans DevTest Labs, vous pouvez choisir d’[isoler toutes les machines virtuelles lab](devtest-lab-configure-vnet.md) et les [environnements sur votre réseau](connect-environment-lab-virtual-network.md) pour vous assurer que les ressources du labo suivent les stratégies de mise en réseau de l’organisation. 

En tant que propriétaire de labo, vous pouvez également choisir d’isoler complètement le labo, ce qui signifie qu’en plus d’isoler les ordinateurs virtuels et les environnements sur le réseau sélectionné, vous pouvez également isoler le compte de stockage du labo et les coffres de clés créés dans votre abonnement. Cet article vous guide dans la création d’un labo isolé du réseau. 

Consultez également les articles suivants :

- [Utilisation du compte de stockage du labo par DevTest Labs](encrypt-storage.md)
- [Utilisation des coffres de clés par DevTest Labs](devtest-lab-store-secrets-in-key-vault.md)
 
> [!NOTE]
> L’isolement réseau n’est actuellement pris en charge que pour les créations de nouveaux labos.

## <a name="steps-to-enable-network-isolation-during-lab-creation"></a>Étapes permettant d’activer l’isolement réseau pendant la création du labo

1. Pendant la création du labo, accédez à l’onglet **mise en réseau**.
1. Vous pouvez sélectionner un réseau **par défaut** que le labo créera pour vous, ou sélectionner un réseau existant dans la liste déroulante. Vous ne pourrez sélectionner que les réseaux qui se trouvent dans la même région et le même abonnement que le labo. 

    > [!div class="mx-imgBorder"]
    > ![Créer un laboratoire](./media/network-isolation/create-lab.png)
1. Sélectionner un sous-réseau.

    > [!div class="mx-imgBorder"]
    > ![Créer un sous-réseau](./media/network-isolation/create-lab-subnet.png)
1. Si vous choisissez d’isoler des ressources de labo (compte de stockage lab et coffre de clés) sur le réseau par défaut, aucune action supplémentaire n’est nécessaire et le labo gérera l’isolement des ressources à l’avenir.
 
    > [!div class="mx-imgBorder"]
    > ![Isolement réseau](./media/network-isolation/isolate-lab-resources.png)
1. Si vous choisissez d’isoler des ressources de labo (compte de stockage du labo et coffre de clés) sur un réseau existant que vous avez sélectionné, vous devez suivre les étapes ci-dessous pour vous assurer que le labo continue de fonctionner en mode isolé. 
 
    > [!div class="mx-imgBorder"]
    > ![Isoler des ressources](./media/network-isolation/isolate-my-vnet.png)

    > [!IMPORTANT]
    > Le propriétaire du labo doit effectuer ces étapes avant de configurer et/ou de créer des ressources dans le labo.

### <a name="steps-to-follow-post-lab-creation"></a>Étapes de la création d’un labo de publication

1. Dans la page d’accueil du labo, sélectionnez le **groupe de ressources** dans la page **Vue d’ensemble**. Vous devez voir la page **Groupe de ressources** pour le groupe de ressources qui contient le labo. 
 
   > [!div class="mx-imgBorder"]
   > ![Contoso labo](./media/network-isolation/contoso-lab.png)
1. Sélectionnez le compte de stockage Azure du labo. La Convention d’affectation de noms pour le compte de stockage de labo est : a<*labNameWithoutInvalidCharacters*> *<4-digit number*>. Par exemple, si le nom du labo est contosolab, le nom du compte de stockage peut être acontosolab1234.
 
   > [!div class="mx-imgBorder"]
   > ![Test Contoso](./media/network-isolation/contoso-test.png)
1. Sur le compte de stockage, accédez à Pare-feux et réseaux virtuels et vérifiez que la case Autoriser les services Microsoft approuvés à accéder à ce compte de stockage est cochée. Étant donné que [DevTest Labs est un service Microsoft approuvé](../storage/common/storage-network-security.md#trusted-microsoft-services), cette option permet au labo de fonctionner normalement en mode isolé du réseau. 

   > [!div class="mx-imgBorder"]
   > ![Pare-feu de Contoso lab](./media/network-isolation/contoso-lab-firewalls-vnets.png)
1. Ensuite, cliquez sur **+Ajouter un réseau virtuel existant**, sélectionnez le réseau virtuel et le sous-réseau que vous avez sélectionnés lors de la création du labo, puis cliquez sur **Activer**. 

   > [!div class="mx-imgBorder"]
   > ![Contoso my vnet](./media/network-isolation/contoso-lab-my-vnet.png)
5.  Une fois que le point de terminaison de service a été activé pour le réseau virtuel sélectionné, cliquez sur **Ajouter**. 

   > [!div class="mx-imgBorder"]
   > ![Ajouter](./media/network-isolation/contoso-firewall-add.png)
 
Avec cela, le stockage Azure autorise des connexions entrantes à partir du réseau virtuel ajouté et permet au labo de fonctionner correctement en mode isolé du réseau. 

Vous pouvez également choisir d’automatiser ces étapes afin de configurer ce paramètre pour plusieurs laboratoires. 

[En savoir plus sur la gestion des règles d’accès réseau par défaut pour le stockage Azure à l’aide de PowerShell et de l’interface CLI](../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#powershell)

## <a name="things-to-remember-while-using-a-lab-in-a-network-isolated-mode"></a>Points à retenir lors de l’utilisation d’un labo en mode isolé du réseau

### <a name="accessing-labs-storage-account-outside-the-lab"></a>Accès au compte de stockage du labo en dehors du labo 

Dans un laboratoire isolé du réseau, pour des actions telles que le chargement d’un disque dur virtuel vers le compte de stockage du labo afin de créer des images personnalisées, le propriétaire du labo devra activer explicitement l’accès au compte de stockage à partir d’un point de terminaison autorisé. Pour ce faire, vous pouvez créer une machine virtuelle et accéder en toute sécurité au compte de stockage du labo à partir de cette machine virtuelle. 

[En savoir plus sur l’accès privé à un compte de stockage à partir d’une machine virtuelle](../private-link/tutorial-private-endpoint-storage-portal.md)

### <a name="exporting-usage-data-from-the-lab"></a>Exportation des données d’utilisation à partir du labo 

Dans un labo isolé du réseau, pour [exporter des données d’utilisation personnelles pour le labo](personal-data-delete-export.md), le propriétaire du labo devra explicitement fournir un compte de stockage et générer un objet blob au sein du compte pour stocker les données. 

Si aucun compte de stockage n’est fourni, cette opération échoue en mode isolé du réseau, car le compte de stockage du labo n’est pas accessible pour que le labo l’utilise dans le cas où aucun compte de stockage n’est fourni par le client. 

[En savoir plus sur l’exportation des données d’utilisation de labo dans un compte de stockage spécifié](personal-data-delete-export.md#azure-powershell)

## <a name="next-steps"></a>Étapes suivantes

[Créer ou modifier des laboratoires automatiquement à l’aide de modèles Azure Resource Manager et PowerShell](devtest-lab-use-arm-and-powershell-for-lab-resources.md)