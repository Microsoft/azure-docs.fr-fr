---
title: Configurer Azure Analysis Services pour des sources de données VNet| Microsoft Docs
description: Découvrez comment configurer un serveur Azure Analysis Services pour l’utilisation d’une passerelle afin de gérer des sources de données sur un réseau virtuel Microsoft Azure (VNet).
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: c3a0ed0f2c9a52d10d206075d229cd62cd3909db
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/28/2021
ms.locfileid: "108145750"
---
# <a name="use-gateway-for-data-sources-on-an-azure-virtual-network-vnet"></a>Utiliser la passerelle pour les sources de données sur un réseau virtuel Azure (VNet)

Cet article décrit la propriété de serveur **AlwaysUseGateway** d’Azure Analysis Services à utiliser lorsque les sources de données se trouvent sur un [réseau virtuel Azure (VNet)](../virtual-network/virtual-networks-overview.md).

## <a name="server-access-to-vnet-data-sources"></a>Serveur d’accès aux sources de données du réseau virtuel

Si vos sources de données sont accessibles via un réseau virtuel, votre serveur Azure Analysis Services doit se connecter à ces sources de données comme si elles étaient accessibles en local, dans votre propre environnement. Vous pouvez configurer la propriété de serveur **AlwaysUseGateway** pour indiquer le serveur qui accédera à toutes les sources de données via une [passerelle locale](analysis-services-gateway.md). 

Les sources de données Azure SQL Managed Instance s’exécutent dans Azure VNet avec une adresse IP privée. Si le point de terminaison public est activé sur l’instance, aucune passerelle n’est requise. Si le point de terminaison public n’est pas activé, une passerelle de données locale est requise et la propriété AlwaysUseGateway doit avoir la valeur true.

> [!NOTE]
> Cette propriété est effective uniquement quand une [passerelle de données locale](analysis-services-gateway.md) est installée et configurée. La passerelle peut se trouver sur le réseau virtuel.

## <a name="configure-alwaysusegateway-property"></a>Configurer la propriété AlwaysUseGateway

1. Dans SSMS > serveur > **Propriétés** > **Général**, sélectionnez **Afficher les propriétés avancées (toutes)** .
2. Dans **ASPaaS\AlwaysUseGateway**, sélectionnez **true**.

    ![Toujours utiliser la propriété de la passerelle](media/analysis-services-vnet-gateway/aas-ssms-always-property.png)


## <a name="see-also"></a>Voir aussi
[Connexion à des sources de données locales](analysis-services-gateway.md)   
[Installer et configurer une passerelle de données locale](analysis-services-gateway-install.md)   
[Réseau virtuel Azure (VNET)](../virtual-network/virtual-networks-overview.md)   

