---
title: Tutoriel - Intégrer Azure Spring Cloud à des solutions d’équilibrage de charge Azure
description: Comment intégrer Azure Spring Cloud à des solutions d’équilibrage de charge Azure
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-java
ms.openlocfilehash: cd0b9d1369fb1c0e662de83b7056da0ff7c83bd1
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090826"
---
# <a name="integrate-azure-spring-cloud-with-azure-load-balance-solutions"></a>Intégrer Azure Spring Cloud à des solutions d’équilibrage de charge Azure

**Cet article s’applique à :** ✔️ Java ✔️ C#

Azure Spring Cloud prend en charge les microservices sur Azure.  L’augmentation de l’activité peut nécessiter plusieurs centres de données avec la gestion de plusieurs instances d’Azure Spring Cloud.

Azure fournit déjà différentes solutions d’équilibrage de charge. Il existe trois options pour intégrer Azure Spring Cloud à des solutions d’équilibrage de charge Azure :

1.  Intégrer Azure Spring Cloud à Azure Traffic Manager
2.  Intégrer Azure Spring Cloud à Azure Application Gateway
3.  Intégrer Azure Spring Cloud à Azure Front Door

## <a name="prerequisites"></a>Prérequis

* Azure Spring Cloud : [Guide pratique pour créer un service Azure Spring Cloud](./spring-cloud-quickstart.md)
* Azure Traffic Manager : [Guide pratique pour créer un profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md)
* Azure Application Gateway : [Guide pratique pour créer une passerelle Application Gateway](../application-gateway/quick-create-portal.md)
* Azure Front Door : [Guide pratique pour créer un profil Front Door](../frontdoor/quickstart-create-front-door.md)

## <a name="integrate-azure-spring-cloud-with-azure-traffic-manager"></a>Intégrer Azure Spring Cloud à Azure Traffic Manager

Pour intégrer Azure Spring Cloud à Traffic Manager, ajoutez ses points de terminaison publics en tant que points de terminaison de Traffic Manager, puis configurez un domaine personnalisé pour Traffic Manager et Azure Spring Cloud.

### <a name="add-endpoint-in-traffic-manager"></a>Ajouter un point de terminaison dans Traffic Manager
Ajoutez des points de terminaison dans Traffic Manager :
1.  Spécifiez **Type** comme étant *Point de terminaison externe*.
1.  Entrez le nom de domaine complet (FQDN) de chaque point de terminaison public Azure Spring Cloud.
1. Cliquez sur **OK**.

    ![Traffic Manager 1](media/spring-cloud-load-balancers/traffic-manager-1.png) ![Traffic Manager 2](media/spring-cloud-load-balancers/traffic-manager-2.png)

### <a name="configure-custom-domain"></a>Configurer un domaine personnalisé
Pour terminer la configuration :
1.  Connectez-vous au site web de votre fournisseur de domaine, puis créez un mappage d’enregistrement CNAME à partir de votre domaine personnalisé vers le nom de domaine Azure par défaut de Traffic Manager.
1.  Suivez les instructions dans [Guide pratique pour ajouter un domaine personnalisé à Azure Spring Cloud](spring-cloud-tutorial-custom-domain.md).
1. Ajoutez la liaison de domaine personnalisé ci-dessus à Traffic Manager dans le service App Service correspondant Azure Spring Cloud et chargez-y le certificat SSL.

    ![Traffic Manager 3](media/spring-cloud-load-balancers/traffic-manager-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-app-gateway"></a>Intégrer Azure Spring Cloud à Azure Application Gateway

Pour une intégration au service Azure Spring Cloud, effectuez les configurations suivantes :

### <a name="configure-backend-pool"></a>Configurer le pool de back-ends
1. Spécifiez **Type de cible** comme *Adresse IP* ou *FQDN*.
1. Entrez vos points de terminaison publics Azure Spring Cloud.

    ![Application Gateway 1](media/spring-cloud-load-balancers/app-gateway-1.png)

### <a name="add-custom-probe"></a>Ajouter une sonde personnalisée
1. Sélectionnez **Sondes d’intégrité**, puis **Ajouter** pour ouvrir la boîte de dialogue **Sonde** personnalisée. 
1. Il est essentiel de sélectionner *Oui* pour l’option **Choisir un nom d’hôte à partir des paramètres HTTP du back-end**.

    ![Application Gateway 2](media/spring-cloud-load-balancers/app-gateway-2.png)

### <a name="configure-http-setting"></a>Configurer un paramètre HTTP
1.  Sélectionnez **Paramètres HTTP**, puis **Ajouter** pour ajouter un paramètre HTTP.
1.  **Remplacer par le nouveau nom d’hôte** : sélectionnez *Oui*.
1.  **Remplacement du nom d’hôte** : sélectionnez **Choisir un nom d’hôte à partir d’une cible de back-end**.
1.  **Utiliser une sonde personnalisée** : sélectionnez *Oui* et choisissez la sonde personnalisée créée ci-dessus.

    ![Application Gateway 3](media/spring-cloud-load-balancers/app-gateway-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-front-door"></a>Intégrer Azure Spring Cloud à Azure Front Door

Pour effectuer une intégration au service Azure Spring Cloud et configurer le pool de back-ends : 
1. **Ajoutez le pool de back-ends**.
1. Spécifiez le point de terminaison de back-end en ajoutant l’hôte.

    ![Front Door 1](media/spring-cloud-load-balancers/front-door-1.png)

1.  Spécifiez **Type d’hôte du back-end** comme étant *Hôte personnalisé*.
1.  Entrez le nom de domaine complet (FQDN) de vos points de terminaison publics Azure Spring Cloud dans **Nom de l’hôte du back-end**.
1.  Acceptez l’**En-tête de l’hôte du back-end** par défaut, qui est identique au **Nom de l’hôte du back-end**.

    ![Front Door 2](media/spring-cloud-load-balancers/front-door-2.png)

## <a name="next-steps"></a>Étapes suivantes
* [Guide pratique pour créer un profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md)
* [Guide pratique pour créer une passerelle Application Gateway](../application-gateway/quick-create-portal.md)
* [Guide pratique pour créer un profil Front Door](../frontdoor/quickstart-create-front-door.md)