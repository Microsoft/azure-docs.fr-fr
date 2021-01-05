---
title: Protégez le back-end d’une application monopage avec OAuth 2.0 à l’aide d’Azure Active Directory B2C et de Gestion des API Azure.
description: Protégez une API avec OAuth 2.0 en utilisant Azure Active Directory B2C, Gestion des API Azure et Easy Auth à appeler à partir d’une application monopage (SPA) JavaScript.
services: api-management, azure-ad-b2c, app-service
documentationcenter: ''
author: WillEastbury
manager: alberts
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/20/2020
ms.author: wieastbu
ms.custom: fasttrack-new, devx-track-js
ms.openlocfilehash: b157200b03bdc89e00bfa5c8264d78baf24875c1
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609120"
---
# <a name="protect-spa-backend-with-oauth-20-azure-active-directory-b2c-and-azure-api-management"></a>Protéger le back-end d’une application monopage avec OAuth 2.0 à l’aide d’Azure Active Directory B2C et de Gestion des API Azure

Ce scénario montre comment configurer votre instance de Gestion des API Azure pour protéger une API.
Nous allons utiliser le protocole OAuth 2.0 avec Azure AD B2C, en plus de Gestion des API, pour sécuriser un principal Azure Functions à l’aide d’EasyAuth.

## <a name="aims"></a>Objectifs
Nous allons voir comment le service Gestion des API peut être utilisé dans un scénario simplifié avec Azure Functions et Azure AD B2C. Vous allez créer une application JavaScript (JS) appelant une API, qui connecte les utilisateurs avec Azure AD B2C. Vous utiliserez ensuite les fonctionnalités de la stratégie validate-jwt de Gestion des API pour protéger l’API back-end.

Pour une défense en profondeur, nous utiliserons ensuite EasyAuth afin de revalider le jeton à l’intérieur de l’API back-end.

## <a name="prerequisites"></a>Prérequis
Pour suivre les étapes décrites dans cet article, vous devez avoir :
* Un compte Stockage Azure universel v2 (StorageV2) pour héberger l’application monopage JS front-end
* Une instance du service Gestion des API Azure 
* Une application de fonction Azure vide (exécutant le runtime .NET Core V2, sur un plan de consommation Windows) pour héberger l’API appelée
* Un locataire Azure AD B2C lié à un abonnement 

Bien que dans la pratique vous utiliseriez des ressources de la même région dans les charges de travail de production, pour cet article de procédures, la région de déploiement n’a pas d’importance.

## <a name="overview"></a>Vue d’ensemble
Voici une illustration des composants utilisés et du flux de l’un à l’autre une fois ce processus terminé.
![Composants utilisés et flux](../api-management/media/howto-protect-backend-frontend-azure-ad-b2c/image-arch.png "Composants utilisés et flux")

Voici un petit aperçu des étapes :

1. Créer les composants Azure AD B2C appelants (front-end, Gestion des API) et les applications API avec des étendues, et accorder l’accès à l’API
1. Créer les stratégies d’inscription ou de connexion pour permettre aux utilisateurs de se connecter avec Azure AD B2C 
1. Configurer Gestion des API avec les nouveaux ID clients et clés Azure AD B2C pour activer l’autorisation utilisateur OAuth2 dans la console de développement
1. Générer l’API de fonction
1. Configurer l’API de fonction pour activer EasyAuth avec les nouveaux ID clients et clés Azure AD B2C, et verrouiller sur l’adresse IP virtuelle APIM 
1. Générer la définition d’API dans Gestion des API
1. Configurer Oauth2 pour la configuration de l’API Gestion des API
1. Configurer la stratégie **CORS** et ajouter une stratégie **validate-jwt** afin de valider le jeton OAuth pour chaque requête entrante
1. Générer l’application appelante pour consommer l’API
1. Charger l’exemple d’application monopage JS
1. Configurer l’exemple d’application cliente JS avec les nouveaux ID clients et clés Azure AD B2C 
1. Tester l’application cliente

## <a name="configure-azure-ad-b2c"></a>Configurer Azure AD B2C 
Ouvrez le panneau Azure AD B2C dans le portail et effectuez les étapes ci-dessous.
1. Sélectionnez l’onglet **Applications**. 
1. Cliquez sur le bouton « Ajouter » et créez trois applications pour les besoins suivants :
   * Le client front-end
   * L’API de fonction back-end
   * [Facultatif] Le portail des développeurs Gestion des API (sauf si vous exécutez Gestion des API Azure dans le niveau de consommation ; ce scénario sera traité en détail plus loin)
1. Définissez WebApp/API web pour les trois applications et définissez « Autoriser un flux implicite » sur Oui pour le client frontal uniquement.
1. Maintenant, définissez l’URI d’ID d’application ; choisissez un nom unique et correspondant au service en cours de création.
1. Utilisez pour l’instant des espaces réservés pour les URL de réponse, par exemple https://localhost. Nous mettrons à jour ces URL ultérieurement.
1. Cliquez sur « Créer », puis répétez les étapes 2 à 5 pour chacune des trois applications ci-dessus, en prenant note de l’URI d’ID d’application, du nom et de l’ID d’application pour une utilisation ultérieure pour les trois applications.
1. Ouvrez l’API Management Developer Portal Application dans la liste des applications et sélectionnez l’onglet *Clés* (sous Général), puis cliquez sur « Générer la clé » pour générer une clé d’authentification.
1. Après avoir cliqué sur Enregistrer, consignez la clé à un emplacement sécurisé pour une utilisation ultérieure. Notez que cet emplacement est le SEUL où vous aurez la chance d’afficher et de copier cette clé.
1. Sélectionnez l’onglet *Étendues publiées* (sous Accès d’API).
1. Créez et nommez une étendue pour votre API de fonction, et prenez note des valeurs Portée et Valeur de portée complète renseignée, puis cliquez sur « Enregistrer ».
   > [!NOTE]
   > Les étendues Azure AD B2C sont en fait des autorisations au sein de votre API auxquelles d’autres applications peuvent demander l’accès par le biais du panneau d’accès d’API à partir de leurs applications. Concrètement, vous venez de créer des autorisations d’application pour votre API appelée.
1. Ouvrez les deux autres applications, puis regardez sous l’onglet *Accès d’API*.
1. Accordez-leur l’accès à l’étendue de l’API back-end et à celle par défaut déjà présente (« Accéder au profil de l’utilisateur »).
1. Générez-leur une clé en sélectionnant l’onglet *Clés* sous « Général » pour générer une clé d’authentification, et enregistrez ces clés à un emplacement sécurisé pour une utilisation ultérieure.

## <a name="create-a-sign-up-or-sign-in-user-flow"></a>Créer un flux d’utilisateur « Inscription et connexion »
1. Revenez à la racine (ou « Vue d’ensemble ») du panneau Azure AD B2C. 
1. Sélectionnez « Flux utilisateur (stratégies) », puis cliquez sur « Nouveau flux d’utilisateur ».
1. Choisissez le type de flux utilisateur « Inscription et connexion ».
1. Donnez un nom à la stratégie et prenez-en note pour une utilisation ultérieure.
1. Ensuite, sous « Fournisseurs d’identité », cochez « Inscription de l’identifiant utilisateur » (cela peut indiquer « S’inscrire par e-mail »), puis cliquez sur OK. 
1. Sous « Attributs utilisateur et revendications », cliquez sur « Afficher plus... », puis choisissez les options de revendication que vous souhaitez que vos utilisateurs entrent et qui doivent être retournées dans le jeton. Cochez au moins « Nom d’affichage » et « Adresse e-mail » afin de recueillir et retourner ces valeurs, cliquez sur « OK », puis sur « Créer ».
1. Sélectionnez la stratégie que vous avez créée dans la liste, puis cliquez sur « Exécuter le flux d’utilisateur ».
1. Cette action ouvre le panneau Exécuter le flux d’utilisateur. Sélectionnez l’application front-end, puis prenez note de l’adresse du domaine b2clogin.com affichée sous la liste déroulante pour « Sélectionner un domaine ».
1. Cliquez sur le lien situé en haut pour ouvrir le « point de terminaison de configuration openid connu », puis prenez note des valeurs authorization_endpoint et token_endpoint ainsi que de la valeur du lien lui-même en tant que point de terminaison de configuration openid connu.

   > [!NOTE]
   > Les stratégies B2C vous permettent d’exposer les points de terminaison de connexion Azure AD B2C pour pouvoir capturer différents composants de données et connecter des utilisateurs de différentes manières. Ici, nous avons configuré un point de terminaison d’inscription ou de connexion qui exposait un point de terminaison de configuration connu ; plus spécifiquement, notre stratégie créée a été identifiée dans l’URL par le paramètre p=.
   > 
   > Une fois cette opération effectuée, vous disposez désormais d’une plateforme d’identité Business to Consumer fonctionnelle qui connecte les utilisateurs à plusieurs applications. 
   > Si vous le souhaitez, vous pouvez cliquer ici sur le bouton « Exécuter le flux d’utilisateur » (pour suivre le processus d’inscription ou de connexion) et vous faire une idée de ce qu’il fera dans la pratique, mais l’étape de redirection à la fin échouera car l’application n’a pas encore été déployée.

## <a name="build-the-function-api"></a>Générer l’API de fonction
1. Revenez à votre locataire Azure AD standard dans le Portail Azure afin de pouvoir reconfigurer les éléments de votre abonnement. 
1. Accédez au panneau Applications de fonctions du portail Azure, ouvrez votre application de fonction vide, puis créez une fonction « Webhook + API » dans le portail par le biais du démarrage rapide.
1. Collez l’exemple de code ci-dessous dans Run.csx à la place du code existant qui s’affiche.

   ```csharp
   
   using System.Net;
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.Extensions.Primitives;
   
   public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
   {
      log.LogInformation("C# HTTP trigger function processed a request.");
      
      return (ActionResult)new OkObjectResult($"Hello World, time and date are {DateTime.Now.ToString()}");
   }
   
   ```

   > [!NOTE]
   > Le code de fonction de script C# que vous venez de coller enregistre simplement une ligne dans les journaux de fonctions et retourne le texte « Hello World » avec des données dynamiques (date et heure).

3. Sélectionnez « Intégrer » dans le panneau de gauche, puis « Éditeur avancé » dans le coin supérieur droit du volet.
4. Collez l’exemple de code ci-dessous à la place du code JSON existant.

   ```json
   {
      "bindings": [
       {
        "authLevel": "function",
        "name": "req",
        "type": "httpTrigger",
        "direction": "in",
        "methods": [
           "get"
        ],
        "route": "hello"
       },
       {
         "name": "$return",
         "type": "http",
         "direction": "out"
       }
     ]
   }
   ```

5. Revenez à l’onglet HttpTrigger1, cliquez sur « Obtenir l’URL de la fonction », puis copiez l’URL qui s’affiche.

   > [!NOTE]
   > Les liaisons que vous venez de créer indiquent simplement à Functions qu’il faut répondre aux requêtes GET HTTP anonymes envoyées à l’URL que vous venez de copier. (`https://yourfunctionappname.azurewebsites.net/api/hello?code=secretkey`) Nous disposons désormais d’une API HTTPS serverless scalable qui est capable de retourner une charge utile très simple.
   > Vous pouvez maintenant tester l’appel de cette API à partir d’un navigateur web à l’aide de l’URL ci-dessus. Vous pouvez aussi supprimer la partie ?code=secret de l’URL et prouver qu’Azure Functions retournera une erreur 401.

## <a name="configure-and-secure-the-function-api"></a>Configurer et sécuriser l’API de fonction
1. Deux zones supplémentaires de l’application de fonction doivent être configurées (Authentification et Restrictions de réseau).
1. Tout d’abord, configurons l’authentification/autorisation. Cliquez sur le nom de l’application de fonction (à côté de l’icône de fonctions &lt;Z&gt;) pour afficher la page de vue d’ensemble.
1. Sélectionnez l’onglet « Fonctionnalités de la plateforme », puis « Authentification/autorisation ».
1. Activez la fonctionnalité Authentification App Service.
1. Sous « Fournisseurs d’authentification », choisissez « Azure Active Directory », puis « Avancé » dans le commutateur Mode de gestion.
1. Collez l’ID d’application de l’API Backend Function (à partir d’Azure AD B2C dans la zone « ID client »).
1. Collez le point de terminaison de configuration open-id connu à partir de la stratégie d’inscription ou de connexion dans la zone URL de l’émetteur (nous avons enregistré cette configuration plus tôt).
1. Sélectionnez OK.
1. Définissez la liste déroulante Action à exécuter quand une demande n’est pas authentifiée sur « Se connecter avec Azure Active Directory », puis cliquez sur Enregistrer.

   > [!NOTE]
   > Votre API Function est désormais déployée et doit lever des réponses 401 si la bonne clé n’est pas fournie et retourner des données quand une requête valide est présentée.
   > Vous avez ajouté une sécurité supplémentaire de défense approfondie dans EasyAuth en configurant l’option « Se connecter avec Azure Active Directory » pour gérer les demandes non authentifiées. N’oubliez pas que cela modifiera le comportement des demandes non autorisées entre l’application de fonction principale et l’application monopage frontale, car EasyAuth émettra une redirection 302 vers AAD au lieu d’une réponse 401 Non autorisée. Nous corrigerons cela ultérieurement en utilisant Gestion des API.
   > Aucune sécurité IP n’est encore appliquée ; si on dispose d’une clé et d’un jeton OAuth2 valides, tout le monde peut appeler cette API depuis n’importe où. Dans l’idéal, nous souhaitons forcer toutes les requêtes à venir par le biais de Gestion des API.
   > Si vous utilisez le niveau de consommation Gestion des API, vous ne pourrez pas effectuer ce verrouillage par adresse IP virtuelle, car il n’y a pas d’adresse IP statique dédiée pour ce niveau. Vous devrez vous appuyer sur la méthode de verrouillage de vos appels d’API par le biais de la clé de fonction à secret partagé ; les étapes 11 à 13 ne seront donc pas possibles.

1. Fermez le panneau « Authentification / Autorisation ». 
1. Sélectionnez « Mise en réseau », puis « Restrictions d’accès ».
1. Verrouillez les adresses IP d’applications de fonction autorisées sur l’adresse IP virtuelle de l’instance Gestion des API. Cette adresse IP virtuelle est affichée dans la section Gestion des API - Vue d’ensemble du portail.
1. Si vous souhaitez continuer à interagir avec le portail Functions et effectuer les étapes facultatives ci-dessous, vous devez également ajouter ici votre propre plage CIDR ou adresse IP publique.
1. Une fois que la liste contient une entrée d’autorisation, Azure ajoute une règle de refus implicite afin de bloquer toutes les autres adresses. 

Vous devez ajouter des blocs d’adresses au format CIDR au panneau de restrictions d’adresses IP. Quand vous devez ajouter une adresse unique telle que l’adresse IP virtuelle de Gestion des API, vous devez l’ajouter au format xx.xx.xx.xx.

   > [!NOTE]
   > Désormais, votre API de fonction ne devrait plus pouvoir être appelée à partir d’un emplacement autre que par le biais de Gestion des API, ou votre adresse.
   
## <a name="import-the-function-app-definition"></a>Importer la définition de la fonction d’application
1. Ouvrez le *panneau Gestion des API*, puis ouvrez *votre instance*.
1. Sélectionnez le panneau des API dans la section Gestion des API de votre instance.
1. Dans le volet « Ajouter une nouvelle API », choisissez « Function App », puis « Complète » dans la partie supérieure de la fenêtre contextuelle.
1. Cliquez sur Parcourir, choisissez l’application de fonction dans laquelle vous hébergez l’API, puis cliquez sur Sélectionner.
1. Donnez un nom et une description à l’API pour l’utilisation interne de Gestion des API, et ajoutez-la au produit « unlimited ».
1. Pensez à prendre note de l’URL de base pour une utilisation ultérieure, puis cliquez sur Créer.

## <a name="configure-oauth2-for-api-management"></a>Configurer Oauth2 pour Gestion des API

1. Ensuite, sélectionnez le panneau OAuth 2.0 sous l’onglet Sécurité, puis cliquez sur « Ajouter ».
1. Spécifiez des valeurs pour *Nom d’affichage* et *Description* pour le point de terminaison OAuth ajouté (ces valeurs s’affichent à l’étape suivante en tant que point de terminaison Oauth2).
1. Vous pouvez entrer n’importe quelle valeur dans l’URL de la page d’inscription du client, car cette valeur ne sera pas utilisée.
1. Activez le type d’octroi *Authentification implicite* et laissez le type d’octroi Code d’autorisation activé.
1. Accédez aux champs de point de terminaison *Autorisation* et *Jeton*, puis entrez les valeurs que vous avez capturées précédemment à partir du document XML de configuration connue.
1. Faites défiler vers le bas et renseignez un *paramètre de corps supplémentaire* appelé « ressource » avec l’ID client de l’API de fonction principale obtenu à partir de l’inscription de l’application Azure AD B2C.
1. Sélectionnez « Informations d’identification du client » et affectez l’ID de l’application de la console de développeur comme ID client. Ignorez cette étape si vous utilisez le modèle de consommation Gestion des API.
1. Affectez la clé dont vous avez pris note précédemment comme clé secrète client. Ignorez cette étape si vous utilisez le modèle de consommation Gestion des API.
1. Pour finir, prenez note de la valeur redirect_uri de l’octroi de code d’authentification à partir de Gestion des API pour une utilisation ultérieure.

## <a name="set-up-oauth2-for-your-api"></a>Configurer Oauth2 pour votre API
1. Votre API apparaîtra sur le côté gauche du portail sous la section « Toutes les API ». Ouvrez votre API en cliquant dessus.
1. Sélectionnez l’onglet « Paramètres ».
1. Mettez à jour vos paramètres en sélectionnant « OAuth 2.0 » dans la case d’option d’autorisation utilisateur.
1. Sélectionnez le serveur OAuth que vous avez défini plus tôt.
1. Cochez la case « Remplacer l’étendue » et entrez l’étendue dont vous avez pris note plus tôt pour l’appel d’API back-end.

   > [!NOTE]
   > Nous disposons à présent d’une instance de Gestion des API qui sait comment obtenir des jetons d’accès à partir d’Azure AD B2C pour autoriser les requêtes et comprend notre configuration Oauth2 Azure Active Directory B2C.

## <a name="set-up-the-cors-and-validate-jwt-policies"></a>Configurer les stratégies **CORS** et **validate-jwt**

> Les instructions des sections suivantes doivent être suivies quel que soit le niveau APIM utilisé. 

1. Revenez à l’onglet Conception et sélectionnez « Toutes les API », puis cliquez sur le bouton de mode code pour afficher l’éditeur de stratégie.
1. Modifiez la section entrante et collez le code XML ci-dessous pour qu’il ressemble à ce qui suit.

   ```xml
   <inbound>
      <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
         <openid-config url="https://tenant.b2clogin.com/tenant.onmicrosoft.com/v2.0/.well-known/openid-configuration?p=B2C_1_MyDefaultPolicy" />
         <required-claims>
            <claim name="aud">
               <value>your-backend-api-application-client-id</value>
            </claim>
         </required-claims>
      </validate-jwt>
      <cors>
         <allowed-origins>
            <origin>*</origin>
         </allowed-origins>
         <allowed-methods>
           <method>GET</method>
         </allowed-methods>
         <allowed-headers>
            <header>*</header>
         </allowed-headers>
         <expose-headers>
           <header>*</header>
         </expose-headers>
       </cors>
   </inbound>
   ```
1. Modifiez l’URL openid-config afin qu’elle corresponde à votre point de terminaison Azure AD B2C connu pour la stratégie d’inscription ou de connexion.
1. Modifiez la valeur de revendication pour qu’elle corresponde à l’ID d’application valide, également appelé ID client pour l’application API back-end, puis cliquez sur Enregistrer.

   > [!NOTE]
   > Désormais, Gestion des API est capable de répondre aux requêtes cross-origin pour les applications monopages JS, et elle effectuera une limitation, une limitation du débit et une prévalidation du jeton d’authentification JWT qui est passé AVANT de transférer la requête à l’API de fonction.

   > [!NOTE]
   > La section qui suit est facultative et ne s’applique pas au niveau **Consommation**, qui ne prend pas en charge le portail des développeurs.
   > Si vous n’envisagez pas d’utiliser le portail des développeurs, ou si vous ne pouvez pas l’utiliser car vous utilisez le niveau Consommation, ignorez cette étape et accédez directement à [« Générer l’application monopage JavaScript pour consommer l’API »](#build-the-javascript-spa-to-consume-the-api).

## <a name="optional-configure-the-developer-portal"></a>[Facultatif] Configurer le portail des développeurs

1. Ouvrez le panneau Azure AD B2C et accédez à l’inscription de l’application pour le portail des développeurs.
1. Affectez comme « URL de réponse » celle que vous avez notée plus tôt quand vous avez configuré le redirect_uri de l’octroi de code d’authentification dans Gestion des API.

   Maintenant que l’autorisation utilisateur OAuth 2.0 est activée sur `Echo API`, la console de développeur obtient un jeton d’accès pour l’utilisateur avant d’appeler l’API.

1. Accédez à une opération quelconque sous `Echo API` dans le portail des développeurs, et sélectionnez **Essayer** pour accéder à la console de développeur.
1. Observez le nouvel élément dans la section **Autorisation** correspondant au serveur d’autorisation que vous venez d’ajouter.
1. Sélectionnez **Code d’autorisation** dans la liste déroulante d’autorisations. Vous êtes alors invité à vous connecter au locataire Azure AD. Si vous êtes déjà connecté avec ce compte, il est possible qu’aucune invite ne s’affiche.
1. Une fois la connexion établie, un en-tête `Authorization: Bearer` est ajouté à la requête, avec un jeton d’accès Azure AD B2C encodé en Base64. 
1. Sélectionnez **Send**, et vous pouvez appeler l’API avec succès.

   > [!NOTE]
   > Maintenant, Gestion des API est capable d’acquérir des jetons pour le portail des développeurs afin de tester votre API, et est capable de comprendre sa définition et d’afficher la page de test appropriée dans le portail des développeurs.

1. Dans le panneau de vue d’ensemble du portail Gestion des API, cliquez sur « Portail des développeurs » pour vous connecter en tant qu’administrateur de l’API.
1. Ici, vous et d’autres consommateurs sélectionnés de votre API pouvez les tester et les appeler à partir d’une console.
1. Sélectionnez « Products », choisissez « Unlimited », puis choisissez l’API que nous avons créée et cliquez sur « TRY IT ».
1. Affichez la clé d’abonnement d’API, puis copiez-la à un emplacement sécurisé, en même temps que l’URL de requête dont vous aurez besoin ultérieurement.
1. Sélectionnez également Implicite dans la liste déroulante d’authentification OAuth. Vous devrez peut-être vous authentifier ici par le biais d’une fenêtre contextuelle.
1. Cliquez sur Send et, si tout va bien, votre application de fonction doit répondre avec un message de salutation par le biais de Gestion des API avec un message 200 OK et un code JSON.

   > [!NOTE]
   > Félicitations, vous avez fait en sorte qu’Azure AD B2C, Gestion des API et Azure Functions opèrent en collaboration pour publier, sécuriser et consommer une API. Vous avez peut-être remarqué que l’API est en fait sécurisée deux fois à l’aide de cette méthode : une fois avec l’en-tête Ocp-Subscription-Key de Gestion des API, et une fois avec le jeton JWT Autorisation : porteur.
   > En effet, cet exemple étant une application monopage JavaScript, nous utilisons la clé Gestion des API uniquement pour les appels de facturation et de limitation de débit.
   > L’autorisation et l’authentification réelles sont gérées par Azure AD B2C et sont encapsulées dans le jeton JWT, qui est validée deux fois, d’abord par Gestion des API, puis par Azure Functions.

## <a name="build-the-javascript-spa-to-consume-the-api"></a>Générer l’application monopage JavaScript pour consommer l’API
1. Ouvrez le panneau des comptes de stockage dans le portail Azure. 
1. Sélectionnez le compte que vous avez créé et sélectionnez le panneau « Site web statique » dans la section Paramètres (si aucune option « Site web statique » n’est visible, vérifiez que vous avez créé un compte V2).
1. Affectez la valeur « activé » à la fonctionnalité d’hébergement web statique, affectez « index.html » comme nom du document d’index, puis cliquez sur « Enregistrer ».
1. Notez le contenu du point de terminaison principal, car il s’agit de l’emplacement où le site front-end sera hébergé. 

   > [!NOTE]
   > Vous pouvez utiliser Stockage Blob Azure + la réécriture CDN, ou Azure App Service, mais la fonctionnalité d’hébergement de site web statique du stockage d’objets blob nous fournit un conteneur par défaut pour traiter le contenu web statique / html / js / css à partir de Stockage Azure et déduira une page par défaut pour nous.

## <a name="upload-the-js-spa-sample"></a>Charger l’exemple d’application monopage JS
1. Toujours dans le panneau du compte de stockage, sélectionnez le panneau « Objets blob » dans la section Service blob, puis cliquez sur le conteneur $web qui apparaît dans le volet de droite.
1. Enregistrez le code ci-dessous dans un fichier local sur votre ordinateur sous le nom index.html, puis chargez le fichier index.html dans le conteneur $web.

   ```html
   <!doctype html>
   <html lang="en">
   <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Sample JS SPA</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
   </head>
   <body>
        <div class="container-fluid">
            <div class="row">
                <div class="col-md-12">
                    <nav class="navbar navbar-expand-lg navbar-light bg-light navbar-dark bg-dark">
                        <a class="navbar-brand" href="#">Sample Code</a>
                        <ul class="navbar-nav ml-md-auto">
                            <li class="nav-item dropdown">
                                <a class="btn btn-large btn-success" onClick="login()">Sign In</a>
                            </li>
                        </ul>
                    </nav>
                </div>
            </div>
            <div class="row">
                <div class="col-md-12">
                    <div class="jumbotron">
                        <h2>
                    <div id="message">Hello, world!</div>
                    </h2>
                        <p>
                            <a class="btn btn-primary btn-large" onClick="GetAPIData()">Call API</a>
                        </p>
                    </div>
                </div>
            </div>
        </div>
        <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
        <script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.0/js/msal.js"></script>
        <script lang="javascript">
            var applicationConfig = {
                clientID: "clientidgoeshere",
                authority: "https://tenant.b2clogin.com/tfp/tenant/policy",
                b2cScopes: ["https://tenant/app/scope"],
                webApi: 'http://functionurl',
                subKey: 'apimkeygoeshere'
            };
            var msalConfig = {
                auth: {
                        clientId: applicationConfig.clientID, 
                        authority:  applicationConfig.authority,
                        validateAuthority: false
                },
                cache: {
                        cacheLocation: "localStorage",
                        storeAuthStateInCookie: true
                }
            };
            var clientApplication = new Msal.UserAgentApplication(msalConfig);
            function login() {
                var loginRequest = {
                    scopes: applicationConfig.b2cScopes
                };
                clientApplication.loginPopup(loginRequest).then(function (loginResponse) {
                    var tokenRequest = {
                        scopes: applicationConfig.b2cScopes
                    };
                    clientApplication.acquireTokenSilent(tokenRequest).then(function (tokenResponse) {
                        document.getElementById("signinbtn").innerHTML = "Logged in as: " + clientApplication.account.name;
                        document.getElementById("callapibtn").hidden = false
                        }).catch(function (error) {
                            clientApplication.acquireTokenPopup(tokenRequest).then(function (tokenResponse) {
                                }).catch (function (error) {
                                    console.log("Error acquiring the popup:\n" + error);
                                });
                        })
                    }).catch (function (error) {
                        console.log("Error during login:\n" + error);
                });
            }
            function GetAPIData() {
                var tokenRequest = {
                    scopes: applicationConfig.b2cScopes
                }
                clientApplication.acquireTokenSilent(tokenRequest).then(function (tokenResponse) {
                    callApiWithAccessToken(tokenResponse.accessToken);
                    }).catch(function (error) {
                        clientApplication.acquireTokenPopup(tokenRequest).then(function (tokenResponse) {
                            callApiWithAccessToken(tokenResponse.accessToken);
                            
                        }).catch(function (error) {
                            console.log("Error acquiring the access token to call the Web api:\n" + error);
                        });
                    })
            }
            function callApiWithAccessToken(token)
            {
                console.log("calling "  + applicationConfig.webApi +  " with " + token);
                    // Make the api call here
                $.ajax({
                    type: "get",
                    headers: {'Authorization': 'Bearer ' + token, 'Ocp-Apim-Subscription-Key': applicationConfig.subKey},
                    url: applicationConfig.webApi
                }
                ).done(function (body) {
                    document.getElementById("message").innerHTML = "The API Said " + body;
                });
            }
        </script>
    </body>
   </html>
   
   ```

1. Accédez au point de terminaison principal du site web statique que vous avez stocké dans la dernière section.

   > [!NOTE]
   > Félicitations, vous venez de déployer une application monopage JavaScript sur Stockage Azure. Étant donné que nous n’avons pas encore configuré l’application JS avec vos clés pour l’API ou configuré l’application JS avec vos détails Azure AD B2C, la page ne fonctionnera pas si vous l’ouvrez.

## <a name="configure-the-js-spa-for-azure-ad-b2c"></a>Configurer l’application monopage JS pour Azure AD B2C
1. Nous savons maintenant où tout se trouve : nous pouvons configurer l’application monopage avec l’adresse d’API Gestion des API appropriée et les ID de clients /d’applications Azure AD B2C corrects.
1. Revenez au panneau de stockage du portail Azure et cliquez sur index.html, puis choisissez « Modifier l’objet blob ». 
1. Mettez à jour les détails de l’authentification pour qu’ils correspondent à l’application front-end que vous avez inscrite plus tôt dans B2C, en notant que les valeurs « b2cScopes » concernent le back-end d’API.
1. La clé webApi et l’URL de l’API se trouvent dans le volet de test de Gestion des API pour l’opération d’API.
1. Créez une clé d’abonnement APIM en revenant au panneau Gestion des API, en sélectionnant « Abonnements », en cliquant sur « Ajouter un abonnement », puis en enregistrant l’enregistrement. Un clic sur les points de suspension (...) en regard de la ligne créée vous permet d’afficher les clés afin de pouvoir copier la clé primaire.
1. Vous devriez avoir quelque chose ressemblant au code suivant :  

   ```javascript
   var applicationConfig =
      clientID: "{aadb2c-clientid-goeshere}",
      authority: "https://{tenant}.b2clogin.com/{tenant}/{policy}",
      b2cScopes: ["https://{tenant}/{app}/{scope}"], 
      webApi: 'http://{apim-url-for-your-function}',
      subKey: '{apim-subscription-key-goes-here}'
   };
   ```

1. Cliquez sur Enregistrer.

## <a name="set-the-redirect-uris-for-the-azure-ad-b2c-frontend-app"></a>Définir les URI de redirection pour l’application front-end Azure AD B2C
1. Ouvrez le panneau Azure AD B2C et accédez à l’inscription de l’application pour l’application front-end JavaScript.
1. Définissez l’URL de redirection sur celle que vous avez notée plus haut quand vous avez configuré le point de terminaison principal du site web statique.

   > [!NOTE] 
   > Cette configuration entraînera la réception, par un client de l’application front-end, d’un jeton d’accès avec les revendications appropriées en provenance d’Azure AD B2C.
   > L’application monopage sera en mesure de l’ajouter en tant que jeton du porteur dans l’en-tête HTTPS de l’appel à l’API back-end.
   > Gestion des API prévalidera le jeton, limitera le débit des appels au point de terminaison par la clé de l’abonné, avant de passer la requête à l’API de fonction Azure réceptrice.
   > L’application monopage affichera la réponse dans le navigateur.

   > *Félicitations, vous avez configuré Azure AD B2C, Gestion des API Azure, Azure Functions et l’autorisation Azure App Service pour opérer en parfaite harmonie !*

   > [!NOTE]
   > Maintenant que nous avons une application simple avec une API sécurisée simple, testons-la.

## <a name="test-the-client-application"></a>Tester l’application cliente
1. Ouvrez l’URL de l’exemple d’application que vous avez notée à partir du compte de stockage créé plus tôt.
1. Cliquez sur « Sign In » dans le coin supérieur droit pour afficher votre profil d’inscription ou de connexion Azure AD B2C.
1. La valeur dans la section « Logged in as » de l’écran sera renseignée à partir de votre jeton JWT.
1. Cliquez maintenant sur « Call Web Api ». La page devrait être mise à jour avec les valeurs retournées à partir de votre API sécurisée.

## <a name="and-were-done"></a>Nous avons terminé.
Les étapes ci-dessus peuvent être adaptées et modifiées pour autoriser de nombreuses utilisations différentes d’Azure AD B2C avec Gestion des API.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [Azure Active Directory et OAuth2.0](../active-directory/develop/authentication-vs-authorization.md).
* Découvrez plus de [vidéos](https://azure.microsoft.com/documentation/videos/index/?services=api-management) sur Gestion des API.
* Pour les autres méthodes permettant de sécuriser votre service principal, consultez [Authentification mutuelle des certificats](api-management-howto-mutual-certificates.md).
* [Création d’une instance du service Gestion des API](get-started-create-service-instance.md).
* [Gérer votre première API](import-and-publish.md).
