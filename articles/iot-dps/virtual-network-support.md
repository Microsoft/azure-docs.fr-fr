---
title: Prise en charge des réseaux virtuels dans le service Azure IoT Device Provisioning (DPS)
description: Utilisation du modèle de connectivité de réseaux virtuels avec Azure IoT Device Provisioning Service (DPS)
services: iot-hub
author: wesmc7777
ms.service: iot-dps
ms.topic: conceptual
ms.date: 06/30/2020
ms.author: wesmc
ms.openlocfilehash: f1409a931195d236b2729e629e4603c606137593
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94959779"
---
# <a name="azure-iot-hub-device-provisioning-service-dps-support-for-virtual-networks"></a>Prise en charge des réseaux virtuels dans le service Azure IoT Hub Device Provisioning (DPS)

Cet article présente le modèle de connectivité de réseau virtuel pour le provisionnement des appareils IoT avec des hubs IoT à l’aide de DPS. Ce modèle fournit une connectivité privée entre les appareils, DPS et le hub IoT au sein d’un réseau virtuel Azure appartenant à un client. 

Dans la plupart des scénarios où DPS est configuré avec un réseau virtuel, votre IoT Hub est également configuré dans le même réseau virtuel. Pour plus d’informations spécifiques sur la prise en charge et la configuration du réseau virtuel pour des hubs IoT, consultez [Prise en charge des réseaux virtuels par IoT Hub](../iot-hub/virtual-network-support.md).



## <a name="introduction"></a>Introduction

Par défaut, les noms d'hôtes DPS sont mappés à un point de terminaison public avec une adresse IP routable publiquement via Internet. Ce point de terminaison public est visible par tous les clients. L’accès au point de terminaison public peut être tenté par des appareils IoT sur des réseaux étendus et des réseaux locaux.

Pour plusieurs raisons, les clients peuvent souhaiter limiter la connectivité aux ressources Azure, telles que DPS. Les raisons sont les suivantes :

* Empêchez l’exposition de la connexion sur l’Internet public. Vous pouvez réduire l’exposition en introduisant des couches supplémentaires de sécurité via l’isolement réseau pour votre IoT Hub et vos ressources DPS

* Permettre une expérience de connectivité privée à partir de vos ressources réseau sur site en garantissant que vos données et votre trafic sont transmis directement au réseau principal Azure.

* Empêcher les attaques par exfiltration à partir de réseaux locaux sensibles. 

* Suivre des modèles de connectivité établis à l'échelle d’Azure en utilisant des [points de terminaison privés](../private-link/private-endpoint-overview.md).

Les approches courantes de limitation de la connectivité incluent les [règles de filtre IP DPS](./iot-dps-ip-filtering.md) et le réseau virtuel (VNET) avec des [points de terminaison privés](../private-link/private-endpoint-overview.md). L’objectif de cet article est de décrire l’approche de réseau virtuel pour DPS à l’aide de points de terminaison privés. 

Les appareils qui fonctionnent sur des réseaux locaux peuvent utiliser un [réseau privé virtuel (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou l’homologation privée [ExpressRoute](https://azure.microsoft.com/services/expressroute/) pour se connecter à un réseau virtuel dans Azure et accéder aux ressources DPS via des points de terminaison privés. 

Un point de terminaison privé est une adresse IP privée attribuée à l'intérieur d'un réseau virtuel appartenant au client, qui permet l’accès à une ressource Azure. En ayant un point de terminaison privé pour votre ressource DPS, vous pouvez autoriser les appareils qui fonctionnent à l’intérieur de votre réseau virtuel à demander l’approvisionnement par votre ressource DPS sans autoriser le trafic vers le point de terminaison public.


## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que les conditions préalables suivantes sont remplies :

* Votre ressource DPS est déjà créée et liée à vos hubs IoT. Pour obtenir des conseils sur la configuration d’une nouvelle ressource DPS, consultez [Configurer le Service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md)

* Vous provisionné réseau virtuel Azure avec un sous-réseau dans lequel sera créé le point de terminaison privé. Pour plus d’informations, consultez [Créer un réseau virtuel à l’aide d’Azure CLI](../virtual-network/quick-create-cli.md).

* Pour les appareils qui fonctionnent à l'intérieur de réseaux locaux, configurez un [réseau privé virtuel (VPN)](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou un peering privé [ExpressRoute](https://azure.microsoft.com/services/expressroute/) dans votre réseau virtuel Azure.

## <a name="private-endpoint-limitations"></a>Limitations de point de terminaison privé

Notez les limitations actuelles suivantes pour DPS lors de l’utilisation de points de terminaison privés :

* Les points de terminaison privé ne fonctionnent pas avec DPS lorsque la ressource DPS et hub lié se situent dans des clouds différents. Par exemple, [Azure Government et Azure international](../azure-government/documentation-government-welcome.md).

* À l’heure actuelle, les [stratégies d’allocation personnalisées avec Azure Functions](how-to-use-custom-allocation-policies.md) pour Data Protection Manager ne fonctionnent pas lorsque la fonction Azure est bloquée dans un réseau virtuel et des points de terminaison privés. 

* La prise en charge actuelle du réseau virtuel DPS concerne l’entrée de données dans DPS uniquement. La sortie de données, qui est le trafic de DPS vers IoT Hub, utilise un mécanisme de service à service interne plutôt qu’un réseau virtuel dédié. La prise en charge d’un verrouillage de sortie basé sur un réseau virtuel complet entre DPS et IoT Hub n’est pas disponible actuellement.

* La stratégie d’allocation de latence la plus faible est utilisée pour affecter un appareil à IoT Hub avec la latence la plus faible. Cette stratégie d’allocation n’est pas fiable dans un environnement de réseau virtuel. 

## <a name="set-up-a-private-endpoint"></a>Créer un point de terminaison privé

Pour configurer un point de terminaison privé, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com/), ouvrez votre ressource DPS, puis cliquez sur l’onglet **Mise en réseau**. Cliquez sur **Connexions de points de terminaison privés** et sur **+ Point de terminaison privé**.

    ![Ajouter un nouveau point de terminaison privé pour DPS](./media/virtual-network-support/networking-tab-add-private-endpoint.png)

2. Sur la page _Créer un point de terminaison privé - Notions de base_, entrez les informations mentionnées dans le tableau ci-dessous.

    ![Créer des points de terminaison privés - Notions de base](./media/virtual-network-support/create-private-endpoint-basics.png)

    | Champ | Valeur |
    | :---- | :-----|
    | **Abonnement** | Choisissez l’abonnement Azure de votre choix pour contenir le point de terminaison privé.  |
    | **Groupe de ressources** | Choisissez ou créez un groupe de ressources pour contenir le point de terminaison privé |
    | **Nom**       | Entrez un nom pour votre point de terminaison privé |
    | **Région**     | La région choisie doit être la même que la région qui contient le réseau virtuel, mais elle ne doit pas nécessairement être la même que la ressource DPS. |

    Cliquez sur **Suivant : Ressource** pour configurer la ressource vers laquelle pointe le point de terminaison privé.

3. Sur la page _Créer un point de terminaison privé - Ressource_, entrez les informations mentionnées dans le tableau ci-dessous.

    ![Créer une ressource de point de terminaison privé](./media/virtual-network-support/create-private-endpoint-resource.png)

    | Champ | Valeur |
    | :---- | :-----|
    | **Abonnement**        | Choisissez l’abonnement Azure qui contient la ressource DPS vers laquelle votre point de terminaison privé va pointer.  |
    | **Type de ressource**       | Choisissez **Microsoft.Devices/ProvisioningServices**. |
    | **Ressource**            | Sélectionnez la ressource DPS vers laquelle le point de terminaison privé va mapper. |
    | **Sous-ressource cible** | Sélectionnez **iotDps**. |

    > [!TIP]
    > Les informations sur le paramètre **Se connecter à une ressource Azure par ID de ressource ou alias** sont fournies dans la section [Demander un point de terminaison privé](#request-a-private-endpoint) de cet article.


    Cliquez sur **Suivant : Configuration** pour configurer le réseau virtuel pour le point de terminaison privé.

4. Sur la page _Créer un point de terminaison privé - Configuration_, choisissez votre réseau virtuel et votre sous-réseau où le point de terminaison privé sera créé.
 
    Cliquez sur **Suivant : Balises**, et spécifiez si nécessaire les balises de votre ressource.

    ![Configurer un point de terminaison privé](./media/virtual-network-support/create-private-endpoint-configuration.png)

6. Cliquez sur **Vérifier + créer**, puis sur **Créer** pour créer votre ressource de point de terminaison privé.


## <a name="request-a-private-endpoint"></a>Demander un point de terminaison privé

Vous pouvez demander un point de terminaison privé à une ressource DPS par ID de ressource. Pour effectuer cette demande, le propriétaire de la ressource doit vous fournir l’ID de ressource. 

1. L’ID de ressource est fourni sous l’onglet Propriétés de la ressource DPS, comme indiqué ci-dessous.

    ![Onglet Propriétés DPS](./media/virtual-network-support/dps-properties.png)

    > [!CAUTION]
    > N’oubliez pas que l’ID de ressource contient l’ID d’abonnement. 

2. Une fois que vous disposez de l’ID de ressource, suivez la procédure ci-dessus dans [Configurer un point de terminaison privé](#set-up-a-private-endpoint) jusqu’à l’étape 3 sur la page _Créer un point de terminaison privé - Ressource_. Cliquez sur **Connectez-vous à une ressource Azure par ID de ressource ou alias** et entrez les informations dans le tableau suivant. 

    | Champ | Valeur |
    | :---- | :-----|
    | **ID de ressource ou alias** | Entrez l’ID de ressource pour la ressource DPS. |
    | **Sous-ressource cible** | Entrez **iotDps** |
    | **Message de requête** | Entrez un message de demande pour le propriétaire de la ressource DPS.<br>Par exemple, <br>`Please approve this new private endpoint`<br>`for IoT devices in site 23 to access this DPS instance`  |

    Cliquez sur **Suivant : Configuration** pour configurer le réseau virtuel pour le point de terminaison privé.

3. Sur la page _Créer un point de terminaison privé - Configuration_, choisissez le réseau virtuel et le sous-réseau où le point de terminaison privé sera créé.
 
    Cliquez sur **Suivant : Balises**, et spécifiez si nécessaire les balises de votre ressource.

4. Cliquez sur **Vérifier + créer**, puis sur **Créer** pour créer votre demande de point de terminaison privé.

5. Le propriétaire DPS verra la demande de point de terminaison privé dans la liste **Connexions des points de terminaison privés** sous l’onglet Mise en réseau DPS. Sur cette page, le propriétaire peut **approuver** ou **rejeter** la demande de point de terminaison privé, comme indiqué ci-dessous.

    ![Approbation du DPS](./media/virtual-network-support/approve-dps-private-endpoint.png)


## <a name="pricing-private-endpoints"></a>Tarification des points de terminaison privés

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure](https://azure.microsoft.com/pricing/details/private-link).



## <a name="next-steps"></a>Étapes suivantes

Utilisez les liens ci-dessous pour en savoir plus sur les fonctionnalités de sécurité DPS :

* [Sécurité](./concepts-service.md#attestation-mechanism)
* [Prise en charge de TLS 1.2](tls-support.md)