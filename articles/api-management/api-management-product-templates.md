---
title: Modèles Produit dans Gestion des API Azure | Microsoft Docs
description: Découvrez comment personnaliser le contenu des pages Produit dans le portail des développeurs Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.assetid: 49f9254c-4c5f-4ed4-9c8d-798f44e805ee
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: 4c8cd4aa3e91c5d69c40e47683818ed8bc9be338
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86249901"
---
# <a name="product-templates-in-azure-api-management"></a>Modèles Produit dans Gestion des API Azure

Gestion des API Azure vous offre la possibilité de personnaliser le contenu des pages du portail des développeurs à l’aide d’un ensemble de modèles qui configurent leur contenu. En utilisant la syntaxe [DotLiquid](http://dotliquidmarkup.org/) et l’éditeur de votre choix, comme [DotLiquid for Designers](https://github.com/dotliquid/dotliquid/wiki/DotLiquid-for-Designers), ainsi qu’un ensemble de [ressources de chaîne](api-management-template-resources.md#strings), de [ressources de glyphe](api-management-template-resources.md#glyphs) et de [contrôles de page](api-management-page-controls.md) localisés, vous disposez d’un large choix pour configurer le contenu des pages selon vos besoins à l’aide de ces modèles.  
  
 Les modèles de cette section vous permettent de personnaliser le contenu des pages Produit dans le portail des développeurs.  
  
-   [Liste de produits](#ProductList)  
  
-   [Produit](#Product)  
  
> [!NOTE]
>  Les exemples de modèles par défaut inclus dans la documentation suivante sont susceptibles d’être modifiés et améliorés de façon régulière. Vous pouvez afficher les modèles dynamiques par défaut dans le portail des développeurs en accédant aux modèles individuels souhaités. Pour plus d’informations sur l’utilisation de modèles, consultez la page [Guide pratique de personnalisation du portail des développeurs Gestion des API à l’aide de modèles](./api-management-developer-portal-templates.md).  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
##  <a name="product-list"></a><a name="ProductList"></a> Liste de produits  
 Le modèle **Liste de produits** vous permet de personnaliser le corps de la page Liste de produits dans le portail des développeurs.  
  
 ![Liste de produits](./media/api-management-product-templates/APIM_ProductsListTemplatePage.png "APIM_ProductsListTemplatePage")  
  
### <a name="default-template"></a>Modèle par défaut  
  
```xml  
<search-control></search-control>  
<div class="row">  
    <div class="col-md-9">  
        <h2>{% localized "ProductsStrings|PageTitleProducts" %}</h2>  
    </div>  
</div>  
<div class="row">  
    <div class="col-md-12">  
    {% if products.size > 0 %}  
    <ul class="list-unstyled">  
    {% for product in products %}  
        <li>  
            <h3><a href="/products/{{product.id}}">{{product.title}}</a></h3>  
            {{product.description}}  
        </li>     
    {% endfor %}  
    </ul>  
    <paging-control></paging-control>  
    {% else %}  
    {% localized "CommonResources|NoItemsToDisplay" %}  
    {% endif %}  
    </div>  
</div>  
```  
  
### <a name="controls"></a>Commandes  
 Le modèle `Product list` peut utiliser les [contrôles de page](api-management-page-controls.md) suivants.  
  
-   [paging-control](api-management-page-controls.md#paging-control)  
  
-   [search-control](api-management-page-controls.md#search-control)  
  
### <a name="data-model"></a>Modèle de données  
  
|Propriété|Type|Description|  
|--------------|----------|-----------------|  
|Pagination|Entité [Paging](api-management-template-data-model-reference.md#Paging).|Informations de pagination de la collection de produits.|  
|Filtrage|Entité [Filtering](api-management-template-data-model-reference.md#Filtering).|Informations de filtrage de la page Liste de produits.|  
|Products|Collection d’entités [Product](api-management-template-data-model-reference.md#Product).|Produits visibles par l’utilisateur actuel.|  
  
### <a name="sample-template-data"></a>Données d’un exemple de modèle  
  
```json  
{  
    "Paging": {  
        "Page": 1,  
        "PageSize": 10,  
        "TotalItemCount": 2,  
        "ShowAll": false,  
        "PageCount": 1  
    },  
    "Filtering": {  
        "Pattern": null,  
        "Placeholder": "Search products"  
    },  
    "Products": [  
        {  
            "Id": "56f9445ffaf7560049060001",  
            "Title": "Starter",  
            "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
            "Terms": "",  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        },  
        {  
            "Id": "56f9445ffaf7560049060002",  
            "Title": "Unlimited",  
            "Description": "Subscribers have completely unlimited access to the API. Administrator approval is required.",  
            "Terms": null,  
            "ProductState": 1,  
            "AllowMultipleSubscriptions": false,  
            "MultipleSubscriptionsCount": 1  
        }  
    ]  
}  
```  
  
##  <a name="product"></a><a name="Product"></a> Produit  
 Le modèle **Produit** vous permet de personnaliser le corps de la page Produit dans le portail des développeurs.  
  
 ![Page Produit dans le portail des développeurs](./media/api-management-product-templates/APIM_ProductPage.png "APIM_ProductPage")  
  
### <a name="default-template"></a>Modèle par défaut  
  
```xml  
<h2>{{Product.Title}}</h2>  
<p>{{Product.Description}}</p>  
  
{% assign replaceString0 = '{0}' %}  
  
{% if Limits and Limits.size > 0 %}  
<h3>{% localized "ProductDetailsStrings|WebProductsUsageLimitsHeader"%}</h3>  
<ul>  
  {% for limit in Limits %}  
  <li>{{limit.DisplayName}}</li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if apis.size > 0 %}  
<p>  
  <b>  
    {% if apis.size == 1 %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockSingleApisCount" %}{% endcapture %}  
    {% else %}  
    {% capture apisCountText %}{% localized "ProductDetailsStrings|TextblockMultipleApisCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture apisCount %}{{apis.size}}{% endcapture %}  
    {{ apisCountText | replace : replaceString0, apisCount }}  
  </b>  
</p>  
  
<ul>  
  {% for api in Apis %}  
  <li>  
    <a href="/docs/services/{{api.Id}}">{{api.Name}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
  
{% if subscriptions.size > 0 %}  
<p>  
  <b>  
    {% if subscriptions.size == 1 %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockSingleSubscriptionsCount" %}{% endcapture %}  
    {% else %}  
    {% capture subscriptionsCountText %}{% localized "ProductDetailsStrings|TextblockMultipleSubscriptionsCount" %}{% endcapture %}  
    {% endif %}  
  
    {% capture subscriptionsCount %}{{subscriptions.size}}{% endcapture %}  
    {{ subscriptionsCountText | replace : replaceString0, subscriptionsCount }}  
  </b>  
</p>  
  
<ul>  
  {% for subscription in subscriptions %}  
  <li>  
    <a href="/developer#{{subscription.Id}}">{{subscription.DisplayName}}</a>  
  </li>  
  {% endfor %}  
</ul>  
{% endif %}  
{% if CannotAddBecauseSubscriptionNumberLimitReached %}  
<b>{% localized "ProductDetailsStrings|TextblockSubscriptionLimitReached" %}</b>  
{% elsif CannotAddBecauseMultipleSubscriptionsNotAllowed == false %}  
<subscribe-button></subscribe-button>  
{% endif %}  
```  
  
### <a name="controls"></a>Commandes  
 Le modèle `Product list` peut utiliser les [contrôles de page](api-management-page-controls.md) suivants.  
  
-   [subscribe-button](api-management-page-controls.md#subscribe-button)  
  
### <a name="data-model"></a>Modèle de données  
  
|Propriété|Type|Description|  
|--------------|----------|-----------------|  
|Produit|[Produit](api-management-template-data-model-reference.md#Product)|Produit spécifié.|  
|IsDeveloperSubscribed|boolean|Si l’utilisateur actuel est abonné à ce produit.|  
|SubscriptionState|nombre|État de l’abonnement. Les états possibles sont :<br /><br /> -   `0 - suspended` : l’abonnement est bloqué et l’abonné ne peut appeler aucune API du produit.<br />-   `1 - active` : l’abonnement est actif.<br />-   `2 - expired` : l’abonnement a atteint sa date d’expiration et a été désactivé.<br />-   `3 - submitted` : la demande d’abonnement a été effectuée par le développeur, mais n’a pas encore été approuvée ou rejetée.<br />-   `4 - rejected` : la demande d’abonnement a été refusée par un administrateur.<br />-   `5 - cancelled` : l’abonnement a été annulé par le développeur ou l’administrateur.|  
|limites|tableau|Cette propriété est déconseillée et ne doit pas être utilisée.|  
|DelegatedSubscriptionEnabled|boolean|Indique si la [délégation](./api-management-howto-setup-delegation.md) est activée pour cet abonnement.|  
|DelegatedSubscriptionUrl|string|Si la délégation est activée, indique l’URL de l’abonnement délégué.|  
|IsAgreed|boolean|Si le produit est associé à un contrat, indique si l’utilisateur actuel a accepté les termes du contrat.|  
|Abonnements|Collection d’entités [Subscription summary](api-management-template-data-model-reference.md#SubscriptionSummary).|Abonnements au produit.|  
|Apis|Collection d’entités [API](api-management-template-data-model-reference.md#API).|API dans ce produit.|  
|CannotAddBecauseSubscriptionNumberLimitReached|boolean|Indique si l’utilisateur actuel est autorisé à s’abonner à ce produit en fonction de la limite d’abonnement.|  
|CannotAddBecauseMultipleSubscriptionsNotAllowed|boolean|Indique si l’utilisateur actuel est autorisé à s’abonner à ce produit en fonction de l’autorisation ou non de plusieurs abonnements.|  
  
### <a name="sample-template-data"></a>Données d’un exemple de modèle  
  
```json  
{  
    "Product": {  
        "Id": "56f9445ffaf7560049060001",  
        "Title": "Starter",  
        "Description": "Subscribers will be able to run 5 calls/minute up to a maximum of 100 calls/week.",  
        "Terms": "",  
        "ProductState": 1,  
        "AllowMultipleSubscriptions": false,  
        "MultipleSubscriptionsCount": 1  
    },  
    "IsDeveloperSubscribed": true,  
    "SubscriptionState": 1,  
    "Limits": [],  
    "DelegatedSubscriptionEnabled": false,  
    "DelegatedSubscriptionUrl": null,  
    "IsAgreed": false,  
    "Subscriptions": [  
        {  
            "Id": "56f9445ffaf7560049070001",  
            "DisplayName": "Starter  (default)"  
        }  
    ],  
    "Apis": [  
        {  
            "id": "56f9445ffaf7560049040001",  
            "name": "Echo API",  
            "description": null,  
            "serviceUrl": "http://echoapi.cloudapp.net/api",  
            "path": "echo",  
            "protocols": [  
                2  
            ],  
            "authenticationSettings": null,  
            "subscriptionKeyParameterNames": null  
        }  
    ],  
    "CannotAddBecauseSubscriptionNumberLimitReached": false,  
    "CannotAddBecauseMultipleSubscriptionsNotAllowed": true  
}  
```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’utilisation de modèles, consultez la page [Guide pratique de personnalisation du portail des développeurs Gestion des API à l’aide de modèles](api-management-developer-portal-templates.md).
