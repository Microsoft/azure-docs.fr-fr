---
title: Planification du déploiement d’Azure VMware Solution
description: Cet article décrit un workflow de déploiement d’Azure VMware Solution.  Le résultat final est un environnement prêt pour la création et la migration des machines virtuelles.
ms.topic: tutorial
ms.date: 10/16/2020
ms.openlocfilehash: 1d1deebd56717c63a745500198eb8df9e3687b49
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558689"
---
# <a name="planning-the-azure-vmware-solution-deployment"></a>Planification du déploiement d’Azure VMware Solution

Cet article vous présente le processus de planification permettant d’identifier et de collecter les données utilisées pendant le déploiement. À mesure que vous planifiez votre déploiement, veillez à documenter les informations que vous collectez pour les référencer facilement pendant le déploiement.

Les processus de ce démarrage rapide génèrent un environnement prêt pour la production en vue de la création de machines virtuelles et de la migration. 

>[!IMPORTANT]
>Avant de créer votre ressource Azure VMware Solution, référez-vous à l’article [Guide pratique pour activer la ressource Azure VMware Solution](enable-azure-vmware-solution.md) afin de soumettre un ticket de support pour que vos hôtes soient alloués. Une fois que l’équipe du support technique a reçu votre demande, il lui faut jusqu’à cinq jours ouvrés pour confirmer votre requête et vous allouer des hôtes. Si vous disposez d’un cloud privé Azure VMware Solution et que vous souhaitez que davantage d’hôtes vous soient alloués, vous passerez par le même processus. 


## <a name="subscription"></a>Abonnement

Identifiez l’abonnement que vous prévoyez d’utiliser pour déployer Azure VMware Solution.  Vous pouvez soit créer un nouvel abonnement, soit réutiliser un abonnement existant.

>[!NOTE]
>L’abonnement doit être associé à un Contrat Entreprise Microsoft ou à un plan Azure pour fournisseurs de solutions cloud. Pour plus d’informations, voir [Comment activer la ressource Azure VMware Solution](enable-azure-vmware-solution.md).

## <a name="resource-group"></a>Resource group

Identifiez le groupe de ressources que vous souhaitez utiliser pour votre Azure VMware Solution.  En règle générale, un groupe de ressources est créé spécifiquement pour Azure VMware Solution, mais vous pouvez utiliser un groupe de ressources existant.

## <a name="region"></a>Région

Identifiez la région dans laquelle vous souhaitez déployer Azure VMware Solution.  Pour plus d’informations, consultez le guide [Disponibilité des produits Azure par région](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware).

## <a name="resource-name"></a>Nom de la ressource

Définissez le nom de la ressource que vous allez utiliser au cours du déploiement.  Le nom de la ressource est le nom convivial et descriptif que vous utilisez pour votre cloud privé Azure VMware Solution.

>[!IMPORTANT]
>Le nom ne doit pas comporter plus de 40 caractères. Si le nom dépasse cette limite, vous ne pourrez pas créer d’adresses IP publiques à utiliser avec le cloud privé. 

## <a name="size-hosts"></a>Dimensionner les hôtes

Identifiez les tailles d’hôtes que vous souhaitez utiliser lors du déploiement d’Azure VMware Solution.  Pour obtenir une liste complète, consultez la documentation relative aux [clouds privés et clusters Azure VMware Solution](concepts-private-clouds-clusters.md#hosts).

## <a name="number-of-hosts"></a>Nombre d’hôtes

Définissez le nombre d’hôtes que vous souhaitez déployer dans le cloud privé Azure VMware Solution.  Le nombre minimal d’hôtes est de trois, et la valeur maximale est de 16 par cluster.  Pour plus d’informations, consultez la documentation relative aux [clouds privés et clusters Azure VMware Solution](concepts-private-clouds-clusters.md#clusters).

Vous pouvez toujours étendre le cluster ultérieurement si vous devez aller au-delà du chiffre de déploiement initial.

## <a name="vcenter-admin-password"></a>Mot de passe de l’administrateur du vCenter
Définissez le mot de passe d'administrateur vCenter.  Pendant le déploiement, vous allez créer un mot de passe d’administrateur vCenter. Le mot de passe est adressé au compte d’administrateur cloudadmin@vsphere.local lors de la build vCenter. Vous l’utiliserez pour vous connecter à vCenter.

## <a name="nsx-t-admin-password"></a>Mot de passe administrateur NSX-T
Définissez le mot de passe administrateur NSX-T.  Pendant le déploiement, vous allez créer un mot de passe d’administrateur NSX-T. Le mot de passe est attribué à l’utilisateur administrateur dans le compte NSX au cours de la build NSX. Vous l’utiliserez pour vous connecter à NSX-T Manager.

## <a name="ip-address-segment"></a>Segment d’adresse IP

La première étape de la planification du déploiement consiste à planifier la segmentation IP.  Azure VMware Solution ingère un réseau /22 que vous fournissez. Le décompose ensuite en segments plus petits, puis utilise ces segments IP pour vCenter, VMware HCX, NSX-T et vMotion.

Azure VMware Solution se connecte à votre Réseau virtuel Microsoft Azure via un circuit ExpressRoute interne. Dans la plupart des cas, il se connecte à votre centre de données via ExpressRoute Global Reach. 

Azure VMware Solution, votre environnement Azure existant et tous les itinéraires Exchange de votre environnement local (généralement). Dans ce cas, le bloc d’adresses réseau CIDR /22 que vous définissez dans cette étape ne doit pas chevaucher tout ce que vous possédez déjà localement ou dans Azure.

**Exemple :** 10.0.0.0/22

Pour plus d’informations, consultez la [Liste de contrôle de planification du réseau](tutorial-network-checklist.md#routing-and-subnet-considerations).

:::image type="content" source="media/pre-deployment/management-vmotion-vsan-network-ip-diagram.png" alt-text="Identifier : segment d’adresse IP" border="false":::  

## <a name="ip-address-segment-for-virtual-machine-workloads"></a>Segment d’adresse IP pour les charges de travail des machines virtuelles

Identifiez un segment IP pour créer votre premier réseau (segment NSX) dans votre cloud privé.  En d’autres termes, vous souhaitez créer un segment réseau dans Azure VMware Solution afin de pouvoir déployer des machines virtuelles sur Azure VMware Solution.   

Même si vous envisagez seulement d’étendre des réseaux L2, créez un segment réseau qui validera l’environnement.

N’oubliez pas que tous les segments IP créés doivent être uniques dans votre espace Azure et votre empreinte locale.  

**Exemple :** 10.0.4.0/24

:::image type="content" source="media/pre-deployment/nsx-segment-diagram.png" alt-text="Identifier : segment d’adresse IP pour les charges de travail des machines virtuelles" border="false":::     

## <a name="optional-extend-networks"></a>(Facultatif) Étendre des réseaux

Vous pouvez étendre les segments réseau à partir d’un site local vers Azure VMware Solution et, si vous le faites, identifier ces réseaux maintenant.  

N’oubliez pas les points suivants :

- Si vous envisagez d’étendre des réseaux en local, ces réseaux doivent se connecter à un [commutateur vDS (vSphere Distributed Switch)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) dans votre environnement VMware local.  
- Si le ou les réseaux que vous souhaitez étendre se trouvent sur un [commutateur vSS (vSphere Standard Switch)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html), ils ne peuvent pas être étendus.

## <a name="azure-virtual-network-to-attach-azure-vmware-solution"></a>Réseau virtuel Azure pour joindre Azure VMware Solution

Pour accéder à votre cloud privé Azure VMware Solution, le circuit ExpressRoute, fourni avec Azure VMware Solution, doit être attaché à un réseau virtuel Azure.  Pendant le déploiement, vous pouvez définir un nouveau réseau virtuel ou en choisir un existant.

Le circuit ExpressRoute d’Azure VMware Solution se connecte à une passerelle ExpressRoute dans le réseau virtuel Azure que vous définissez dans cette étape.  

>[!IMPORTANT]
>Vous pouvez utiliser une passerelle ExpressRoute existante pour vous connecter à Azure VMware Solution, à condition qu’elle ne dépasse pas la limite de quatre circuits ExpressRoute par réseau virtuel.  Toutefois, pour accéder à Azure VMware Solution à partir d’un site local via ExpressRoute, vous devez disposer de ExpressRoute Global Reach, car la passerelle ExpressRoute ne fournit pas de routage transitif entre ses circuits connectés.  

Si vous souhaitez connecter le circuit ExpressRoute à partir d’Azure VMware Solution à une passerelle ExpressRoute existante, vous pouvez le faire après le déploiement.  

Donc, en résumé, voulez-vous connecter Azure VMware Solution à une passerelle Express Route existante ?  

* **Oui** = Identifiez le réseau virtuel qui n’est pas utilisé pendant le déploiement.
* **No** = Identifiez un réseau virtuel existant ou créez-en un nouveau lors du déploiement.

Dans les deux cas, documentez ce que vous souhaitez faire au cours de cette étape.

>[!NOTE]
>Ce réseau virtuel est visible par votre environnement local et votre solution Azure VMware Solution. Vous devez donc vous assurer que le segment IP que vous utilisez dans ce réseau virtuel et les sous-réseaux ne se chevauchent pas.

:::image type="content" source="media/pre-deployment/azure-vmware-solution-expressroute-diagram.png" alt-text="Identité : réseau virtuel Azure pour joindre Azure VMware Solution" border="false":::

## <a name="vmware-hcx-network-segments"></a>Segments réseau VMware HCX

VMware HCX est une technologie regroupée avec Azure VMware Solution. Les principaux cas d’utilisation de VMware HCX sont les migrations de charge de travail et la récupération d’urgence. Si vous envisagez d’effectuer l’une ou l’autre, il est préférable de planifier la mise en réseau maintenant.   Dans le cas contraire, vous pouvez passer à l’étape suivante.

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez rassemblé et documenté les informations nécessaires, passez à la section suivante pour créer votre cloud privé Azure VMware Solution.

> [!div class="nextstepaction"]
> [Déployer Azure VMware Solution](deploy-azure-vmware-solution.md)
