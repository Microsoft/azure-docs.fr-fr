---
title: Basculement entre plusieurs points de terminaison avec Traffic Manager
titleSuffix: Azure Content Delivery Network
description: Découvrez comment configurer le basculement entre plusieurs points de terminaison Azure Content Delivery Network avec Azure Traffic Manager.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 10/08/2020
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: d2d3bd43a0f17167e855d7e678a96cd79fe42237
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92777739"
---
# <a name="failover-across-multiple-endpoints-with-azure-traffic-manager"></a>Basculement entre plusieurs points de terminaison avec Azure Traffic Manager

Lorsque vous configurez Azure Content Delivery Network (CDN), vous pouvez sélectionner le fournisseur optimal et le niveau tarifaire adapté à vos besoins. 

Avec son infrastructure distribuée globalement, Azure CDN crée par défaut une redondance locale et géographique, ainsi qu’un équilibrage de charge global pour améliorer les performances et la disponibilité du service. 

Si un emplacement n’est pas disponible pour traiter le contenu, les demandes sont automatiquement routées vers un autre emplacement. Chaque demande client est traitée suivant le point de présence (POP) optimal. Le routage automatique s’appuie sur des facteurs comme l’emplacement de la demande et la charge du serveur.
 
Si vous avez plusieurs profils CDN, vous pouvez améliorer davantage la disponibilité et les performances avec Azure Traffic Manager. 

Utilisez Azure Traffic Manager avec Azure CDN pour équilibrer la charge entre plusieurs points de terminaison CDN dans les cas de figure suivants :
 
* Basculement
* Équilibrage géographique de la charge 

Dans un scénario de basculement standard, toutes les demandes clients sont dirigées vers le profil CDN principal. 

Si le profil n’est pas disponible, les demandes sont dirigées vers le profil secondaire.  Les demandes reprennent vers le profil principal lorsqu’il est de nouveau en ligne.

Une telle utilisation d’Azure Traffic Manager garantit que votre application web est toujours disponible. 

Cet article fournit de l’aide ainsi qu’un exemple de configuration du basculement avec des profils issus des sources suivantes : 

* **Azure CDN Standard de Verizon**
* **Azure CDN Standard par Akamai**

**Azure CDN de Microsoft** est également pris en charge.

## <a name="create-azure-cdn-profiles"></a>Création de profils Azure CDN
Créez au moins deux points de terminaison et profils Azure CDN avec des fournisseurs différents.

1. Créez deux profils CDN :
    * **Azure CDN Standard de Verizon**
    * **Azure CDN Standard par Akamai** 

    Créez les profils en suivant la procédure décrite dans [Création d’un profil CDN](cdn-create-new-endpoint.md#create-a-new-cdn-profile).
 
   ![CDN - Plusieurs profils](./media/cdn-traffic-manager/cdn-multiple-profiles.png)

2. Dans chacun des nouveaux profils, créez au moins un point de terminaison en suivant les étapes décrites pour la [création d’un point de terminaison CDN](cdn-create-new-endpoint.md#create-a-new-cdn-endpoint).

## <a name="create-traffic-manager-profile"></a>Création d’un profil Traffic Manager
Créez un profil Azure Traffic Manager et configurez l’équilibrage de charge entre vos points de terminaison CDN. 

1. Créez un profil Azure Traffic Manager en suivant les étapes décrites pour la [création d’un profil Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md). 

    * **Méthode de routage** : sélectionnez **Priorité** .

2. Ajoutez vos points de terminaison CDN dans votre profil Traffic Manager en suivant les étapes décrites pour l’[ajout de points de terminaison Traffic Manager](../traffic-manager/quickstart-create-traffic-manager-profile.md#add-traffic-manager-endpoints).

    * **Type** : sélectionnez **Points de terminaison externes** .
    * **Priorité** : entrez un nombre.

    Par exemple, créez **cdndemo101akamai.azureedge.net** avec une priorité de  **1** et **cdndemo101verizon.azureedge.net** avec une priorité de  **2** .

   ![CDN - Points de terminaison Traffic Manager](./media/cdn-traffic-manager/cdn-traffic-manager-endpoints.png)


## <a name="configure-custom-domain-on-azure-cdn-and-azure-traffic-manager"></a>Configuration d’un domaine personnalisé sur Azure CDN et Azure Traffic Manager
Après avoir configuré vos profils CDN et Traffic Manager, suivez cette procédure pour ajouter le mappage DNS et inscrire un domaine personnalisé pour les points de terminaison CDN. Pour cet exemple, le nom de domaine personnalisé est **cdndemo101.dustydogpetcare.online** .

1. Accédez au site web du fournisseur de votre domaine personnalisé, tel que GoDaddy, et créez deux entrées DNS CNAME. 

    a. Pour la première entrée CNAME, mappez votre domaine personnalisé, avec le sous-domaine cdnverify, au point de terminaison CDN. Cette entrée est une étape obligatoire pour inscrire le domaine personnalisé auprès du point de terminaison CDN que vous avez ajouté à Traffic Manager à l’étape 2.

      Par exemple : 

      `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101akamai.azureedge.net`  

    b. Pour la seconde entrée CNAME, mappez votre domaine personnalisé, sans le sous-domaine cdnverify, au point de terminaison CDN. Cette entrée mappe le domaine personnalisé à Traffic Manager. 

      Par exemple : 
      
      `cdndemo101.dustydogpetcare.online  CNAME  cdndemo101.trafficmanager.net`   

    > [!NOTE]
    > Si votre domaine est actuellement en ligne et ne peut pas être interrompu, effectuez cette étape en dernier. Vérifiez que les points de terminaison CDN les et domaines de Traffic Manager sont actifs avant de mettre à jour votre DNS de domaine personnalisé sur Traffic Manager.
    >


2.  À partir de votre profil Azure CDN, sélectionnez le premier point de terminaison CDN (Akamai). Sélectionnez **Ajouter un domaine personnalisé** et saisissez **cdndemo101.dustydogpetcare.online** . Vérifiez que la coche validant le domaine personnalisé est verte. 

    Azure CDN utilise le sous-domaine **cdnverify** pour valider le mappage DNS et terminer ce processus d’inscription. Pour plus d’informations, voir [Créer un enregistrement CNAME DNS](cdn-map-content-to-custom-domain.md#create-a-cname-dns-record). Cette étape permet à Azure CDN de reconnaître le domaine personnalisé afin de répondre à ses requêtes.
    
    > [!NOTE]
    > Pour activer TLS sur des profils **Azure CDN d’Akamai** , vous devez définir directement le cname du domaine personnalisé sur votre point de terminaison. L’utilisation de cdnverify pour l’activation de TLS n’est pas encore prise en charge. 
    >

3.  Revenez au site web du fournisseur de votre domaine personnalisé. Mettez à jour le premier mappage DNS que vous avez créé. Faites correspondre un domaine personnalisé avec votre deuxième point de terminaison CDN.
                             
    Par exemple : 

    `cdnverify.cdndemo101.dustydogpetcare.online  CNAME  cdnverify.cdndemo101verizon.azureedge.net`  

4. À partir de votre profil Azure CDN, sélectionnez le deuxième point de terminaison CDN (Verizon) et répétez l’étape 2. Sélectionnez **Ajouter un domaine personnalisé** , puis entrez **cdndemo101.dustydogpetcare.online** .
 
Une fois que vous avez suivi cette procédure, votre service multi-CDN doté de fonctionnalités de basculement est configuré avec Azure Traffic Manager. 

Vous avez accès aux URL de test issues de votre domaine personnalisé. 

Pour tester la fonctionnalité, désactivez le point de terminaison CDN principal et vérifiez que la requête est correctement déplacée sur le point de terminaison CDN secondaire. 

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez configurer d’autres méthodes de routage (par exemple, géographique) pour équilibrer la charge entre différents points de terminaison CDN. 

Pour plus d’informations, voir [Configurer la méthode de routage du trafic géographique à l’aide de Traffic Manager](../traffic-manager/traffic-manager-configure-geographic-routing-method.md).