---
title: 'Tutoriel : Ajouter un domaine personnalisé à votre point de terminaison'
titleSuffix: Azure Content Delivery Network
description: Utilisez ce tutoriel pour ajouter un domaine personnalisé à un point de terminaison Azure Content Delivery Network afin que votre nom de domaine soit visible dans votre URL.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.topic: tutorial
ms.date: 11/06/2020
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 03ed47ee97f52aca708118f202fad583753549bf
ms.sourcegitcommit: 46c5ffd69fa7bc71102737d1fab4338ca782b6f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94331204"
---
# <a name="tutorial-add-a-custom-domain-to-your-endpoint"></a>Tutoriel : Ajouter un domaine personnalisé à votre point de terminaison

Ce didacticiel montre comment ajouter un domaine personnalisé à un point de terminaison de réseau de distribution de contenu (CDN) Azure. 

Le nom du point de terminaison dans votre profil CDN est un sous-domaine d’azureedge.net. Par défaut, lors de la distribution de contenu, le domaine du profil CDN est inclus dans l’URL.

Par exemple : **https://contoso.azureedge.net/photo.png**.

Azure CDN offre la possibilité d’associer un domaine personnalisé à un point de terminaison CDN. Cette option distribue le contenu avec un domaine personnalisé dans votre URL au lieu du domaine par défaut.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Créez un enregistrement CNAME DNS.
> - Associez le domaine personnalisé à votre point de terminaison CDN.
> - Vérifiez le domaine personnalisé.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Avant d’effectuer les étapes de ce tutoriel, créez un profil CDN et au moins un point de terminaison CDN. 
    * Pour plus d’informations, consultez [Démarrage rapide : Créer un point de terminaison et un profil de réseau de distribution de contenu Azure](cdn-create-new-endpoint.md).

* Si vous n’avez pas de domaine personnalisé, achetez-en un auprès d’un fournisseur de domaine. 
    * Pour plus d’informations, consultez [Acheter un nom de domaine personnalisé](../app-service/manage-custom-dns-buy-domain.md).

* Si vous utilisez Azure pour héberger vos [domaines DNS](../dns/dns-overview.md), déléguez votre domaine personnalisé à Azure DNS. 
    * Pour plus d’informations, voir [Délégation de domaine à Azure DNS](../dns/dns-delegate-domain-azure-dns.md).

* Si vous utilisez un fournisseur de domaine pour gérer votre domaine DNS, passez à [Créer un enregistrement CNAME DNS](#create-a-cname-dns-record).


## <a name="create-a-cname-dns-record"></a>Créer un enregistrement CNAME DNS

Avant de pouvoir utiliser un domaine personnalisé avec un point de terminaison Azure CDN, vous devez créer un enregistrement de nom canonique (CNAME) avec Azure DNS ou votre fournisseur DNS pointant vers votre point de terminaison CDN. 

Un enregistrement CNAME est un enregistrement DNS qui mappe un nom de domaine source à un nom de domaine de destination. 

Pour Azure CDN, le domaine source est votre domaine personnalisé et le domaine de destination est votre point de terminaison CDN. 

Azure CDN route le trafic adressé au domaine personnalisé source vers le nom d’hôte du point de terminaison CDN de destination après avoir vérifié l’enregistrement CNAME.

Un domaine personnalisé et son sous-domaine peuvent être associés à un seul point de terminaison à la fois. 

Utilisez plusieurs enregistrements CNAME pour différents sous-domaines du même domaine personnalisé pour différents services Azure.

Vous pouvez mapper un domaine personnalisé ayant différents sous-domaines au même point de terminaison CDN.

> [!NOTE]
> Ce tutoriel utilise le type d’enregistrement CNAME. Si vous utilisez les types d’enregistrements A ou AAAA, suivez les mêmes étapes ci-dessous et remplacez CNAME par le type d’enregistrement de votre choix.

---
# <a name="azure-dns"></a>[**Azure DNS**](#tab/azure-dns)

Azure DNS utilise des enregistrements d’alias pour les ressources Azure au sein du même abonnement.

Pour ajouter un enregistrement d’alias pour votre point de terminaison Azure CDN

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le menu de gauche, sélectionnez **Toutes les ressources**, puis la zone Azure DNS pour votre domaine personnalisé.

3. Dans la zone DNS de votre domaine personnalisé, sélectionnez **+ Jeu d’enregistrements**.

4. Dans **Ajouter un jeu d’enregistrements**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom  | Entrez l’alias que vous souhaitez utiliser pour votre point de terminaison CDN. Par exemple, **www**. |
    | Type  | Sélectionnez **CNAME**. |
    | Jeu d’enregistrements d’alias | Sélectionnez **Oui**. |
    | Type d’alias | Sélectionnez **Ressource Azure**. |
    | Sélectionnez un abonnement | Sélectionnez votre abonnement. |
    | Ressource Azure | Sélectionnez votre point de terminaison Azure CDN. |

5. Remplacez la valeur **TTL** de l’enregistrement par votre valeur.

6. Sélectionnez **OK**.

# <a name="dns-provider"></a>[**Fournisseur DNS**](#tab/dns-provider)

## <a name="map-the-temporary-cdnverify-subdomain"></a>Mapper le sous-domaine temporaire cdnverify

Lorsque vous mappez un domaine existant qui est en production, plusieurs points demandent une attention particulière. 

Il se peut que votre domaine personnalisé connaisse un court temps d’arrêt lorsque vous l’inscrivez dans le portail Azure.

Pour éviter l’interruption du trafic web, mappez votre domaine personnalisé à votre nom d’hôte de point de terminaison CDN avec le sous-domaine **cdnverify** Azure. Ce processus crée un mappage CNAME temporaire. 

Avec cette méthode, les utilisateurs peuvent accéder à votre domaine sans interruption pendant que le mappage DNS est en cours. 

Si les considérations relatives aux temps d’arrêt de production ne sont pas un problème, vous pouvez directement mapper votre domaine personnalisé à votre point de terminaison CDN. Passez à [Mapper le domaine personnalisé permanent](#map-the-permanent-custom-domain).

Pour créer un enregistrement CNAME avec le sous-domaine cdnverify :

1. Connectez-vous au site web du fournisseur DNS pour votre domaine personnalisé.

2. Créez une entrée d’enregistrement CNAME pour votre domaine personnalisé et renseignez les champs comme indiqué dans le tableau suivant (les noms de champs peuvent varier) :

    | Source                    | Type  | Destination                     |
    |---------------------------|-------|---------------------------------|
    | cdnverify. www.contoso.com | CNAME | cdnverify.contoso.azureedge.NET |

    - Source : entrez votre nom de domaine personnalisé, y compris le sous-domaine cdnverify, au format suivant : **cdnverify.\<custom-domain-name>**
        - Par exemple : **cdnverify. www.contoso.com**.

    - Type : entrez ou sélectionnez **CNAME**.

    - Destination : entrez votre nom d’hôte du point de terminaison CDN, y compris le sous-domaine cdnverify, au format suivant : **cdnverify.\<endpoint-name>.azureedge.net**. 
        - Par exemple : **cdnverify.contoso.azureedge.net**.

3. Enregistrez vos modifications.

## <a name="map-the-permanent-custom-domain"></a>Mapper le domaine personnalisé permanent

Dans cette section, vous allez mapper le domaine personnalisé permanent au point de terminaison CDN. 

Pour créer un enregistrement CNAME pour votre domaine personnalisé :

1. Connectez-vous au site web du fournisseur de domaine pour votre domaine personnalisé.

2. Créez une entrée d’enregistrement CNAME pour votre domaine personnalisé et renseignez les champs comme indiqué dans le tableau suivant (les noms de champs peuvent varier) :

    | Source          | Type  | Destination           |
    |-----------------|-------|-----------------------|
    | www.contoso.com | CNAME | contoso.azureedge.net |

    - Source : entrez votre nom de domaine personnalisé.
        - Par exemple : **www.contoso.com**.

    - Type : entrez ou sélectionnez **CNAME**.

    - Destination : entrez le nom d’hôte de votre point de terminaison CDN au format suivant : **\<endpoint-name>.azureedge.net**. 
        - Par exemple : **contoso.azureedge.net**.

3. Enregistrez vos modifications.

4. Si vous avez déjà créé un enregistrement temporaire CNAME du sous-domaine cdnverify, supprimez-le. 

5. Si vous utilisez ce domaine personnalisé en production pour la première fois, suivez les étapes pour [Associer le domaine personnalisé à votre point de terminaison CDN](#associate-the-custom-domain-with-your-cdn-endpoint) et [Vérifier le domaine personnalisé](#verify-the-custom-domain).

---
## <a name="associate-the-custom-domain-with-your-cdn-endpoint"></a>Associer le domaine personnalisé à votre point de terminaison CDN

Une fois que vous avez enregistré votre domaine personnalisé, vous pouvez l’ajouter à votre point de terminaison CDN. 

1. Connectez-vous au [Portail Azure](https://portal.azure.com/), puis accédez au profil CDN contenant le point de terminaison que vous souhaitez mapper à un domaine personnalisé.
    
2. Sur la page **Profil CDN**, sélectionnez le point de terminaison CDN à associer avec le domaine personnalisé.

    :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-endpoint-selection.png" alt-text="Sélection de point de terminaison CDN" border="true":::
    
3. Sélectionnez **+ Domaine personnalisé**. 

   :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-custom-domain-button.png" alt-text="Bouton Ajouter un domaine personnalisé" border="true":::

4. Dans **Ajouter un domaine personnalisé**, le **Nom d’hôte du point de terminaison** pré-rempli est dérivé de votre URL de point de terminaison CDN : **\<endpoint-hostname>** .zureedge.net. Cette valeur n’est pas modifiable.

5. Pour **Nom d’hôte personnalisé**, entrez votre domaine personnalisé, y compris le sous-domaine, à utiliser comme domaine source de votre enregistrement CNAME. 
    1. Par exemple, **www.contoso.com** ou **cdn.contoso.com**. **N’utilisez pas le nom de sous-domaine cdnverify**.

    :::image type="content" source="media/cdn-map-content-to-custom-domain/cdn-add-custom-domain.png" alt-text="Ajouter un domaine personnalisé" border="true":::

6. Sélectionnez **Ajouter**.

   Azure vérifie que l’enregistrement CNAME existe pour le nom de domaine personnalisé que vous avez entré. Si l'enregistrement CNAME est correct, votre domaine personnalisé sera validé. 

   La propagation des nouveaux paramètres de domaine personnalisé sur tous les nœuds de périphérie CDN peut prendre un certain temps : 
    - Pour les profils du **CDN Azure Standard fourni par Microsoft**, la propagation s’effectue généralement dans un délai de 10 minutes. 
    - Pour les profils du **CDN Azure Standard fourni par Akamai**, la propagation s’effectue généralement dans un délai d’une minute. 
    - Dans le cas des profils **Azure CDN Standard fourni par Verizon** et **Azure CDN Premium fourni par Verizon**, la propagation s’effectue généralement dans un délai de 10 minutes.   


## <a name="verify-the-custom-domain"></a>Vérifier le domaine personnalisé

Une fois que vous avez terminé l’inscription de votre domaine personnalisé, vérifiez qu’il référence votre point de terminaison CDN.
 
1. Assurez-vous d’avoir un contenu public qui est mis en cache au point de terminaison. Par exemple, si votre point de terminaison CDN est associé à un compte de stockage, Azure CDN met en cache le contenu dans un conteneur public. Vérifiez que votre conteneur autorise l’accès public et qu’il contient au moins un fichier afin de tester le domaine personnalisé.

2. Dans votre navigateur, accédez à l’adresse du fichier à l’aide du domaine personnalisé. Par exemple, si votre domaine personnalisé est `www.contoso.com`, l’URL vers le fichier mis en cache doit être similaire à l’URL suivante : `http://www.contoso.com/my-public-container/my-file.jpg`. Vérifiez que le résultat est identique à celui que vous obtenez lorsque vous accédez au point de terminaison CDN directement à **\<endpoint-hostname>** .azureedge.net.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne voulez plus associer votre point de terminaison avec un domaine personnalisé, supprimez le domaine personnalisé en effectuant les étapes suivantes :
 
1. Dans votre profil CDN, sélectionnez le point de terminaison avec le domaine personnalisé que vous souhaitez supprimer.

2. À partir de la page **Point de terminaison**, sous Domaines personnalisés, cliquez sur le domaine personnalisé que vous souhaitez supprimer, puis sélectionnez **Supprimer** dans le menu contextuel. Sélectionnez **Oui**.

   Le domaine personnalisé est dissocié de votre point de terminaison.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> - Créez un enregistrement CNAME DNS.
> - Associez le domaine personnalisé à votre point de terminaison CDN.
> - Vérifiez le domaine personnalisé.

Passer au tutoriel suivant pour apprendre à configurer HTTPS sur un domaine personnalisé Azure CDN.

> [!div class="nextstepaction"]
> [Tutoriel : Configurer HTTPS sur un domaine personnalisé Azure CDN](cdn-custom-ssl.md)