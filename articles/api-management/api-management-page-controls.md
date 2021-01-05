---
title: Contrôles de page Gestion des API Azure | Microsoft Docs
description: Découvrez les contrôles de page disponibles pour une utilisation dans les modèles du portail des développeurs dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: apimpm
ms.openlocfilehash: b826c986a0af26bb79fb0823e4e8626f0165d460
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86250020"
---
# <a name="azure-api-management-page-controls"></a>Contrôles de page Gestion des API Azure
Gestion des API Azure fournit les contrôles suivants à utiliser dans les modèles du portail des développeurs.  
  
Pour utiliser un contrôle, placez-le à l’emplacement souhaité dans le modèle du portail des développeurs. Certains contrôles, tels que [app-actions](#app-actions), ont des paramètres, comme illustré dans l’exemple suivant :  
  
```xml  
<app-actions params="{ appId: '{{app.id}}' }"></app-actions>  
```  
  
Les valeurs des paramètres sont transmises comme partie du modèle de données pour le modèle. Dans la plupart des cas, vous pouvez simplement coller l’exemple fourni pour chaque contrôle afin qu’il fonctionne correctement. Pour plus d’informations sur les valeurs de paramètre, vous pouvez voir la section de modèle de données pour chaque modèle dans lequel un contrôle peut être utilisé.  

Pour plus d’informations sur l’utilisation de modèles, consultez la page [Guide pratique de personnalisation du portail des développeurs Gestion des API à l’aide de modèles](./api-management-developer-portal-templates.md).  

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]
  
## <a name="developer-portal-template-page-controls"></a>Contrôles de page du modèle dans le portail des développeurs  
  
-   [app-actions](#app-actions)  
-   [basic-signin](#basic-signin)  
-   [paging-control](#paging-control)  
-   [fournisseurs](#providers)  
-   [search-control](#search-control)  
-   [sign-up](#sign-up)  
-   [subscribe-button](#subscribe-button)  
-   [subscription-cancel](#subscription-cancel)  
  
##  <a name="app-actions"></a><a name="app-actions"></a> app-actions  
 Le contrôle `app-actions` fournit une interface utilisateur pour interagir avec les applications sur la page de profil utilisateur dans le portail des développeurs.  
  
 ![contrôle app&#45;actions](./media/api-management-page-controls/APIM-app-actions-control.png "contrôle APIM app-actions")  
  
### <a name="usage"></a>Usage  
  
```xml  
<app-actions params="{ appId: '{{app.id}}' }"></app-actions>  
```  
  
### <a name="parameters"></a>Paramètres  
  
|Paramètre|Description|  
|---------------|-----------------|  
|appId|ID de l’application.|  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `app-actions` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Applications](api-management-user-profile-templates.md#Applications)  
  
##  <a name="basic-signin"></a><a name="basic-signin"></a> basic-signin  
 Le contrôle `basic-signin` fournit un contrôle pour la collecte des informations de connexion utilisateur dans la page de connexion du portail des développeurs.  
  
 ![contrôle basic&#45;signin](./media/api-management-page-controls/APIM-basic-signin-control.png "contrôle APIM basic-signin")  
  
### <a name="usage"></a>Usage  
  
```xml  
<basic-SignIn></basic-SignIn>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `basic-signin` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Connexion](api-management-page-templates.md#SignIn)  
  
##  <a name="paging-control"></a><a name="paging-control"></a> paging-control  
 Le contrôle `paging-control` fournit des fonctionnalités de pagination sur les pages du portail des développeurs qui affichent une liste d’éléments.  
  
 ![contrôle paging](./media/api-management-page-controls/APIM-paging-control.png "contrôle APIM paging")  
  
### <a name="usage"></a>Usage  
  
```xml  
<paging-control></paging-control>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `paging-control` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Liste d’API](api-management-api-templates.md#APIList)  
  
-   [Liste des problèmes](api-management-issue-templates.md#IssueList)  
  
-   [Liste de produits](api-management-product-templates.md#ProductList)  
  
##  <a name="providers"></a><a name="providers"></a> providers  
 Le contrôle `providers` fournit un contrôle pour la sélection des fournisseurs d’authentification sur la page de connexion du portail des développeurs.  
  
 ![contrôle providers](./media/api-management-page-controls/APIM-providers-control.png "contrôle APIM providers")  
  
### <a name="usage"></a>Usage  
  
```xml  
<providers></providers>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `providers` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Connexion](api-management-page-templates.md#SignIn)  
  
##  <a name="search-control"></a><a name="search-control"></a> search-control  
 Le contrôle `search-control` fournit des fonctionnalités de recherche sur les pages du portail des développeurs affichant une liste d’éléments.  
  
 ![contrôle search](./media/api-management-page-controls/APIM-search-control.png "contrôle APIM search")  
  
### <a name="usage"></a>Usage  
  
```xml  
<search-control></search-control>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `search-control` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Liste d’API](api-management-api-templates.md#APIList)  
  
-   [Liste de produits](api-management-product-templates.md#ProductList)  
  
##  <a name="sign-up"></a><a name="sign-up"></a> sign-up  
 Le contrôle `sign-up` fournit un contrôle pour la collecte des informations de profil utilisateur sur la page d’inscription du portail des développeurs.  
  
 ![contrôle sign&#45;up](./media/api-management-page-controls/APIM-sign-up-control.png "contrôle APIM sign-up")  
  
### <a name="usage"></a>Usage  
  
```xml  
<sign-up></sign-up>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `sign-up` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Inscription](api-management-page-templates.md#SignUp)  
  
##  <a name="subscribe-button"></a><a name="subscribe-button"></a> subscribe-button  
 `subscribe-button` fournit un contrôle pour abonner un utilisateur à un produit.  
  
 ![contrôle subscribe&#45;button](./media/api-management-page-controls/APIM-subscribe-button-control.png "contrôle APIM subscribe-button")  
  
### <a name="usage"></a>Usage  
  
```xml  
<subscribe-button></subscribe-button>  
```  
  
### <a name="parameters"></a>Paramètres  
 Aucun.  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `subscribe-button` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Produit](api-management-product-templates.md#Product)  
  
##  <a name="subscription-cancel"></a><a name="subscription-cancel"></a> subscription-cancel  
 Le contrôle `subscription-cancel` fournit un contrôle pour annuler l’abonnement à un produit sur la page de profil utilisateur du portail des développeurs.  
  
 ![contrôle subscription&#45;cancel](./media/api-management-page-controls/APIM-subscription-cancel-control.png "contrôle APIM subscription-cancel")  
  
### <a name="usage"></a>Usage  
  
```xml  
<subscription-cancel params="{ subscriptionId: '{{subscription.id}}', cancelUrl: '{{subscription.cancelUrl}}' }">  
</subscription-cancel>  
  
```  
  
### <a name="parameters"></a>Paramètres  
  
|Paramètre|Description|  
|---------------|-----------------|  
|subscriptionId|ID de l’abonnement à annuler.|  
|cancelUrl|URL permettant d’annuler l’abonnement.|  
  
### <a name="developer-portal-templates"></a>Modèles du portail des développeurs  
 Le contrôle `subscription-cancel` peut être utilisé dans les modèles suivants du portail des développeurs :  
  
-   [Produit](api-management-product-templates.md#Product)

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur l’utilisation de modèles, consultez la page [Guide pratique de personnalisation du portail des développeurs Gestion des API à l’aide de modèles](api-management-developer-portal-templates.md).
