---
title: Guide pratique pour créer un labo avec une ressource partagée | Azure Lab Services
description: Découvrez comment créer un laboratoire qui nécessite une ressource partagée entre les étudiants.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: d4bf4127dc163bb5f034e077b84664828374ba87
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94647968"
---
# <a name="how-to-create-a-lab-with-a-shared-resource-in-azure-lab-services"></a>Guide pratique pour créer un labo avec une ressource partagée dans Azure Lab Services

Parfois, lors de la création d’un laboratoire de classe, il peut y avoir des ressources devant être partagées entre tous les étudiants d’un laboratoire.  Par exemple, vous avez un serveur de licences ou un SQL Server pour une classe de base de données.  Cet article décrit les étapes permettant d’activer la ressource partagée pour un laboratoire.  Nous parlerons également de la façon de limiter l’accès à cette ressource partagée.

## <a name="architecture"></a>Architecture

Comme indiqué dans le diagramme ci-dessous, nous disposons d’un compte de laboratoire doté d’un laboratoire.  Le compte de laboratoire aura les paramètres de peering de réseaux virtuels, de sorte que le réseau virtuel du laboratoire soit connecté au réseau de la ressource partagée.  Dans le diagramme ci-dessous, il existe deux réseaux virtuels avec des plages d’adresses IP qui ne se chevauchent pas.  Ces plages d’adresses IP sont simplement des exemples.  Notez également que le réseau virtuel de ressources partagées correspond au même abonnement que le compte de labo.

![Services Lab avec architecture des ressources partagées](./media/how-to-create-a-lab-with-shared-resource/shared-resource-architecture.png)

## <a name="setup-shared-resource"></a>Configurer une ressource partagée

Le réseau virtuel de la ressource partagée doit être créé avant le laboratoire.  Pour plus d'informations sur la création d'un réseau virtuel, consultez [Créer un réseau virtuel](../virtual-network/quick-create-portal.md).  Il est important de planifier les plages de réseaux virtuels de sorte qu’elles ne se chevauchent pas avec l’adresse IP des ordinateurs de laboratoire.  Pour plus d’informations sur la planification de votre réseau, consultez l’article [Planifier des réseaux virtuels](../virtual-network/virtual-network-vnet-plan-design-arm.md). Dans notre exemple, la ressource partagée se trouve dans un réseau virtuel avec la plage 10.2.0.0/16.  Si ce n’est pas déjà fait, [créez un sous-réseau](../virtual-network/virtual-network-manage-subnet.md#add-a-subnet) pour contenir la ressource partagée.  Dans l’exemple, nous utilisons la plage 10.2.0.0/24, mais votre plage peut être différente selon les besoins de votre réseau.

La ressource partagée peut être un logiciel s’exécutant sur une machine virtuelle ou un service fourni par Azure. La ressource partagée doit être disponible via une adresse IP privée.  En rendant la ressource partagée disponible uniquement par le biais de l’adresse IP privée, vous limitez l’accès à cette ressource.

Le diagramme montre également un groupe de sécurité réseau (NSG) qui peut être utilisé pour limiter le trafic provenant de la machine virtuelle de l’étudiant.  Par exemple, vous pouvez écrire une règle de sécurité qui indique que le trafic provenant des adresses IP de la machine virtuelle de l’étudiant peut uniquement accéder à une ressource partagée et rien d’autre.  Pour plus d’informations sur le réglage de règles de sécurité, consultez [Gérer un groupe de sécurité réseau](../virtual-network/manage-network-security-group.md#work-with-security-rules). Si vous souhaitez restreindre l’accès à une ressource partagée à un laboratoire spécifique, récupérez l’adresse IP du laboratoire à partir des [paramètres du labo dans le compte de labo](manage-labs.md#view-labs-in-a-lab-account) et définissez une règle de trafic entrant pour autoriser l’accès uniquement à partir de cette adresse IP.  N’oubliez pas d’autoriser les ports 49152 à 65535 pour cette adresse IP.  Si vous le souhaitez, vous pouvez trouver l’adresse IP privée des machines virtuelles de l’étudiant à l’aide de la [page du pool de machines virtuelles](how-to-set-virtual-machine-passwords.md).

Si votre ressource partagée est une machine virtuelle Azure qui exécute des logiciels nécessaires, vous devrez peut-être modifier les règles de pare-feu par défaut pour la machine virtuelle.

### <a name="tips-for-shared-resources---license-server"></a>Conseils pour les ressources partagées - Serveurs de licences
L’une des ressources partagées les plus courantes est le serveur de licences. Voici quelques conseils pour en réussir la configuration.
#### <a name="server-region"></a>Région du serveur
Le serveur de licences doit être connecté au réseau virtuel qui est appairé au labo. Il doit donc se trouver dans la même région que le compte du labo.

#### <a name="static-private-ip-and-mac-address"></a>Adresse MAC et IP privée statique
Les machines virtuelles ont une adresse IP privée dynamique par défaut. [Avant de configurer un logiciel, définissez l’adresse IP privée sur statique](../virtual-network/virtual-networks-static-private-ip-arm-pportal.md). Cela permet de définir l’adresse IP privée et l’adresse MAC comme des adresses statiques.  

#### <a name="control-access"></a>Contrôler l’accès
Il est essentiel de contrôler l’accès au serveur de licences.  Une fois la machine virtuelle configurée, l’accès sera toujours nécessaire pour la maintenance, la résolution des problèmes et la mise à jour.  Voici les différentes manières de le faire.
- [Définir l’accès juste-à-temps dans Azure Security Center.](../security-center/security-center-just-in-time.md?tabs=jit-config-asc%252cjit-request-asc)
- [Définir un groupe de sécurité réseau pour restreindre l’accès.](../virtual-network/network-security-groups-overview.md)
- [Configurer Azure Bastion pour permettre un accès sécurisé au serveur de licences.](https://azure.microsoft.com/services/azure-bastion/)

## <a name="lab-account"></a>Compte de laboratoire

Pour utiliser une ressource partagée, le compte de labo doit être configuré pour utiliser un [réseau virtuel en peering](how-to-connect-peer-virtual-network.md).  Dans ce cas, nous procéderons au peering du réseau virtuel qui contient la ressource partagée.

>[!WARNING]
>Le laboratoire de votre classe doit être créé **après** le peering du compte de labo au réseau virtuel de ressources partagées.  
Machine modèle

Une fois le peering de votre compte de labo au réseau virtuel terminé, la machine de modèle doit maintenant avoir accès à la ressource partagée.  Vous devrez peut-être mettre à jour les règles de pare-feu, en fonction de la ressource partagée dont vous souhaitez partager l’accès.