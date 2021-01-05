---
title: Connecter un réseau virtuel Azure à CloudSimple à l’aide d’ExpressRoute - Azure VMware Solution by CloudSimple
description: Indique comment obtenir des informations de peering permettant de connecter le réseau virtuel Azure à votre environnement CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 6b20ee4e04a4443529ecceca8c6fc2206f7df39d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77563996"
---
# <a name="connect-azure-virtual-network-to-cloudsimple-using-expressroute"></a>Connecter un réseau virtuel Azure à CloudSimple à l’aide d’ExpressRoute

Vous pouvez étendre le réseau de votre cloud privé à votre réseau virtuel et vos ressources Azure. Vous pouvez utiliser la connexion ExpressRoute pour accéder aux ressources s’exécutant dans votre abonnement Azure à partir de votre cloud privé.

## <a name="request-authorization-key"></a>Demande de clé d’autorisation

Une clé d’autorisation est requise pour connecter ExpressRoute à votre cloud privé et à votre réseau virtuel Azure. Pour obtenir une clé, envoyer un ticket avec <a href="https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest" target="_blank">Support</a>.  Utilisez les informations suivantes dans votre demande :

* Type de problème : **Technique**
* Abonnement : **Sélectionnez l’abonnement dans lequel le service CloudSimple est déployé**
* Service : **VMware Solution by CloudSimple**
* Type de problème : **Demande de service**
* Sous-type de problème : **Clé d’autorisation pour la connexion au réseau virtuel Azure**
* Objet : **Demande de clé d’autorisation pour la connexion au réseau virtuel Azure**

## <a name="get-peering-information-from-cloudsimple-portal"></a>Recevoir des informations de peering à partir du portail CloudSimple

Pour configurer la connexion, vous devez établir une connexion entre le réseau virtuel Azure et votre environnement CloudSimple.  Dans le cadre de la procédure, vous devez fournir l’URI du circuit pair et la clé d’autorisation. Obtenez l’URI et la clé d'autorisation à partir du [portail CloudSimple](access-cloudsimple-portal.md).  Sélectionnez **Réseau** dans le menu latéral, puis **Connexion réseau Azure**. Ou sélectionnez **Compte** dans le menu latéral, puis **Connexion réseau Azure**.

Copiez l’URI du circuit pair et la clé d’autorisation pour chacune des régions à l’aide de l’icône de *copie*. Pour chaque région CloudSimple que vous souhaitez connecter :

1. Cliquez sur **Copier** pour copier l’URI. Collez-le dans un fichier qui permettra de l’ajouter au portail Azure.  
2. Cliquez sur **Copier** pour copier la clé d’autorisation et collez-la également dans le fichier.

Copiez la clé d’autorisation et l’URI du circuit pair qui se trouve à l’état **Disponible**.  L’état **Utilisé** indique que la clé a déjà été utilisée pour créer une connexion de réseau virtuel.

![Page de connexion de réseau virtuel](media/virtual-network-connection.png)

Pour plus d’informations sur la configuration du réseau virtuel Azure au lien CloudSimple, consultez la page [Connecter votre environnement de cloud privé CloudSimple au réseau virtuel Azure à l’aide d’ExpressRoute](azure-expressroute-connection.md).

## <a name="next-steps"></a>Étapes suivantes

* [Connexion entre un réseau virtuel Azure et le cloud privé](azure-expressroute-connection.md)
* [Se connecter à un réseau local à l’aide d’Azure ExpressRoute](on-premises-connection.md)
