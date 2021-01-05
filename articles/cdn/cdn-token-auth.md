---
title: Sécurisation des ressources CDN Azure avec l’authentification du jeton| Microsoft Docs
description: Découvrez comment utiliser l’authentification du jeton pour sécuriser l’accès à vos ressources Azure CDN.
services: cdn
documentationcenter: .net
author: zhangmanling
manager: zhangmanling
editor: ''
ms.assetid: 837018e3-03e6-4f9c-a23e-4b63d5707a64
ms.service: azure-cdn
ms.devlang: multiple
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 11/17/2017
ms.author: mazha
ms.openlocfilehash: 21ef06f37e6840df08b1477f9c0ff24f6e15d1a3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95978033"
---
# <a name="securing-azure-cdn-assets-with-token-authentication"></a>Sécurisation des ressources CDN Azure avec l’authentification du jeton

[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>Vue d’ensemble

L’authentification du jeton est un mécanisme qui vous permet d’empêcher Azure Content Delivery Network (CDN) de fournir des ressources à des clients non autorisés. L’authentification par jeton vise généralement à empêcher le *hotlinking* de contenu, dans le cadre duquel un autre site web, par exemple, un forum de discussion, utilise vos ressources sans autorisation. Le « hotlinking » peut avoir un impact sur vos coûts de distribution de contenu. En activant l’authentification du jeton sur CDN, les requêtes sont authentifiées par le serveur de périphérie CDN avant que CDN livre le contenu. 

## <a name="how-it-works"></a>Fonctionnement

L’authentification du jeton s’assure que les requêtes sont générées par un site de confiance en vérifiant qu’elles comportent une valeur de jeton contenant des informations codées sur le demandeur. Le contenu est fourni à un demandeur uniquement si les informations codées respectent les exigences définies. Dans le cas contraire, les requêtes sont refusées. Vous pouvez procéder à la configuration à l’aide d’un ou de plusieurs des paramètres suivants :

- Pays/région : autorisez ou refusez les requêtes provenant des pays/régions spécifiés par leur [code pays/région](/previous-versions/azure/mt761717(v=azure.100)).
- URL : autorisez uniquement les requêtes qui correspondent à la ressource ou au chemin d’accès spécifique.
- Hôte : autorisez ou refusez les requêtes utilisant les hôtes spécifiés dans l’en-tête de requête.
- Référent : autorisez ou refusez une requête provenant du référent spécifié.
- Adresse IP : autorisez uniquement les requêtes provenant d’une adresse ou d’un sous-réseau IP spécifique.
- Protocole : autorisez ou refusez des requêtes en fonction du protocole utilisé pour demander le contenu.
- Date/heure d’expiration : attribuez une période de date et d’heure pour vous assurer qu’un lien reste valide uniquement pendant une période limitée.

Pour plus d’informations, consultez les exemples de configuration détaillés pour chaque paramètre dans [Configuration de l’authentification du jeton](#setting-up-token-authentication).

>[!IMPORTANT] 
> Si l’autorisation par jeton est activée pour un chemin d’accès sur ce compte, le mode de cache standard est le seul mode qui puisse être utilisé pour la mise en cache de la chaîne de requête. Pour plus d’informations, consultez [Contrôle du comportement de mise en cache du CDN Azure avec des chaînes de requête](cdn-query-string-premium.md).

## <a name="reference-architecture"></a>Architecture de référence

Le diagramme des flux de travail suivant décrit comment Azure CDN applique l’authentification du jeton pour utiliser votre application web.

![Flux de travail de l’authentification du jeton dans CDN](./media/cdn-token-auth/cdn-token-auth-workflow2.png)

## <a name="token-validation-logic-on-cdn-endpoint"></a>Logique de validation de jeton sur le point de terminaison CDN
    
L’organigramme suivant décrit comment Azure CDN valide une demande du client lorsque l’authentification du jeton est configurée sur le point de terminaison CDN.

![Logique de validation du jeton dans CDN](./media/cdn-token-auth/cdn-token-auth-validation-logic.png)

## <a name="setting-up-token-authentication"></a>Configuration de l’authentification du jeton

1. Sur le [Portail Azure](https://portal.azure.com), accédez à votre profil CDN, puis sélectionnez **Gérer** pour ouvrir le portail supplémentaire.

    ![Bouton Gérer du profil CDN](./media/cdn-token-auth/cdn-manage-btn.png)

2. Placez le curseur sur **HTTP Large**, puis sélectionnez **Token Auth (Authentification par jeton)** dans le menu volant. Vous pouvez ensuite configurer la clé de chiffrement et les paramètres de chiffrement comme suit :

   1. Créez une ou plusieurs clés de chiffrement. Une clé de chiffrement respecte la casse et peut contenir n’importe quelle combinaison de caractères alphanumériques. D’autres types de caractères, y compris les espaces, ne sont pas autorisés. La longueur maximale est de 250 caractères. Pour vous assurer que vos clés de chiffrement sont aléatoires, nous vous recommandons de les créer à l’aide de l’outil [OpenSSL](https://www.openssl.org/). 

      La syntaxe de l’outil OpenSSL est la suivante :

      ```rand -hex <key length>```

      Par exemple :

      ```OpenSSL> rand -hex 32``` 

      Pour éviter les temps d’arrêt, créez une clé primaire et une clé de sauvegarde. Une clé de sauvegarde fournit un accès ininterrompu à votre contenu pendant que la mise à jour de votre clé primaire est en cours.
    
   2. Entrez une clé de chiffrement unique dans la zone **Clé primaire**, puis tapez éventuellement une clé de sauvegarde dans la zone **Clé de sauvegarde**.

   3. Sélectionnez la version minimale de chiffrement pour chaque clé dans sa liste **Minimum Encryption Version (Version minimale de chiffrement)** , puis sélectionnez **Update (Mettre à jour)** :
      - **V2** : indique que la clé peut être utilisée pour générer des jetons de version 2.0 et version 3.0. Utilisez cette option uniquement si vous effectuez la transition depuis une clé de chiffrement version 2.0 héritée vers une clé de version 3.0.
      - **V3** : (recommandé) indique que la clé peut être uniquement utilisée pour générer des jetons de version 3.0.

      ![Clé de configuration de l’authentification du jeton CDN](./media/cdn-token-auth/cdn-token-auth-setupkey.png)
    
   4. Utilisez l’outil de chiffrement pour définir les paramètres de chiffrement et générer un jeton. Avec l’outil de chiffrement, vous pouvez autoriser ou refuser des requêtes en fonction de la date/heure d’expiration, du pays/de la région, du référent, du protocole et de l’IP du client (dans toute combinaison). Bien qu’il n’existe aucune restriction sur le nombre et la combinaison de paramètres pouvant être associés pour former un jeton, la longueur totale d’un jeton est limitée à 512 caractères. 

      ![Outil de chiffrement CDN](./media/cdn-token-auth/cdn-token-auth-encrypttool.png)

      Entrez les valeurs pour un ou plusieurs des paramètres de chiffrement suivants dans la zone **Encrypt Tool (Outil de chiffrement)** : 

      > [!div class="mx-tdCol2BreakAll"] 
      > <table>
      > <tr>
      >   <th>Nom du paramètre</th> 
      >   <th>Description</th>
      > </tr>
      > <tr>
      >    <td><b>ec_expire</b></td>
      >    <td>Affecte à un jeton une date/heure d’expiration au-delà de laquelle le jeton expire. Les requêtes soumises après la date/heure d’expiration sont refusées. Ce paramètre utilise un timestamp Unix, qui est basé sur le nombre de secondes écoulées depuis l’époque Unix standard `1/1/1970 00:00:00 GMT`. (Vous pouvez utiliser des outils en ligne pour convertir l’heure standard en heure Unix, et inversement.)> 
      >    Par exemple, si vous souhaitez que le jeton expire à `12/31/2016 12:00:00 GMT`, utilisez la valeur de timestamp Unix `1483185600`. 
      > </tr>
      > <tr>
      >    <td><b>ec_url_allow</b></td> 
      >    <td>Permet d’adapter les jetons à une ressource ou à un chemin d’accès particulier. Ce paramètre restreint l’accès aux demandes dont l’URL commence par un chemin d’accès relatif spécifique. Les URL sont sensibles à la casse. Entrez plusieurs chemins d’accès en les séparant par une virgule ; n’ajoutez pas d’espaces. Selon vos exigences, vous pouvez définir des valeurs différentes pour fournir différents niveaux d’accès.> 
      >    Par exemple, pour l’URL `http://www.mydomain.com/pictures/city/strasbourg.png`, ces requêtes sont autorisées pour les valeurs d’entrée suivantes : 
      >    <ul>
      >       <li>Valeur d’entrée `/` : toutes les requêtes sont autorisées.</li>
      >       <li>Valeur d’entrée `/pictures` : les requêtes suivantes sont autorisées : <ul>
      >          <li>`http://www.mydomain.com/pictures.png`</li>
      >          <li>`http://www.mydomain.com/pictures/city/strasbourg.png`</li>
      >          <li>`http://www.mydomain.com/picturesnew/city/strasbourgh.png`</li>
      >       </ul></li>
      >       <li>Valeur d’entrée `/pictures/` : seules les requêtes contenant le chemin d’accès `/pictures/` sont autorisées. Par exemple : `http://www.mydomain.com/pictures/city/strasbourg.png`.</li>
      >       <li>Valeur d’entrée `/pictures/city/strasbourg.png` : seules les requêtes concernant ce chemin d’accès et cette ressource spécifiques sont autorisées.</li>
      >    </ul>
      > </tr>
      > <tr>
      >    <td><b>ec_country_allow</b></td> 
      >    <td>Autorise uniquement les requêtes provenant d’un ou de plusieurs pays (ou régions) spécifiés. Les requêtes provenant de tous les autres pays/régions sont refusées. Utilisez un [code de pays/région ISO 3166](/previous-versions/azure/mt761717(v=azure.100)) de deux lettres pour chaque pays/région, en séparant les codes par une virgule ; n’ajoutez pas d’espace. Par exemple, pour autoriser l’accès aux requêtes provenant uniquement des États-Unis et de France, entrez `US,FR`.</td>
      > </tr>
      > <tr>
      >    <td><b>ec_country_deny</b></td> 
      >    <td>Refuse les requêtes provenant d’un ou de plusieurs pays (ou régions) spécifiés. Les requêtes provenant de tous les autres pays/régions sont autorisées. L’implémentation est identique à celle du paramètre <b>ec_country_allow</b>. Si un code de pays/région est présent dans les paramètres <b>ec_country_allow</b> et <b>ec_country_deny</b>, le paramètre <b>ec_country_allow</b> est prioritaire.</td>
      > </tr>
      > <tr>
      >    <td><b>ec_ref_allow</b></td>
      >    <td>Autorise uniquement les requêtes provenant du référent spécifié. Un référent identifie l’URL de la page web qui est liée à la ressource demandée. N’incluez pas le protocole dans la valeur du paramètre.> 
      >    Les types d’entrée autorisés sont les suivants :
      >    <ul>
      >       <li>Un nom d’hôte ou un nom d’hôte et un chemin d’accès.</li>
      >       <li>Plusieurs référents. Pour ajouter plusieurs référents, séparez-les par une virgule ; n’ajoutez pas d’espace. Si vous spécifiez une valeur de référent, mais que la configuration du navigateur ne permet pas l’envoi des informations sur le référent dans la requête, cette requête est refusée par défaut.</li> 
      >       <li>Requêtes manquant d’informations ou contenant des informations vides sur le référent. Par défaut, le paramètre <b>ec_ref_allow</b> bloque ces types de requêtes. Pour autoriser ces requêtes, entrez le texte « missing » ou une valeur vide (à l’aide d’une virgule de fin).</li> 
      >       <li>Sous-domaines. Pour autoriser des sous-domaines, entrez un astérisque (\*). Par exemple, pour autoriser tous les sous-domaines de `contoso.com`, entrez `*.contoso.com`.</li>
      >    </ul> 
      >    Par exemple, pour autoriser l’accès aux requêtes provenant de `www.contoso.com`, à l’ensemble des sous-domaines sous `contoso2.com` et aux requêtes manquant de référents ou contenant des valeurs vides pour les référents, entrez `www.contoso.com,*.contoso.com,missing`.</td>
      > </tr>
      > <tr> 
      >    <td><b>ec_ref_deny</b></td>
      >    <td>Refuse les requêtes provenant du référent spécifié. L’implémentation est identique à celle du paramètre <b>ec_ref_allow</b>. Si un référent est présent dans les deux paramètres <b>ec_ref_allow</b> et <b>ec_ref_deny</b>, le paramètre <b>ec_ref_allow</b> est prioritaire.</td>
      > </tr>
      > <tr> 
      >    <td><b>ec_proto_allow</b></td> 
      >    <td>Autorise uniquement les requêtes provenant du protocole spécifié. Les valeurs valides sont `http`, `https` ou `http,https`.</td>
      > </tr>
      > <tr>
      >    <td><b>ec_proto_deny</b></td>
      >    <td>Refuse les requêtes du protocole spécifié. L’implémentation est identique à celle du paramètre <b>ec_proto_allow</b>. Si un protocole est présent dans les deux paramètres <b>ec_proto_allow</b> et <b>ec_proto_deny</b>, le paramètre <b>ec_proto_allow</b> est prioritaire.</td>
      > </tr>
      > <tr>
      >    <td><b>ec_clientip</b></td>
      >    <td>Restreint l’accès à l’adresse IP de demandeur spécifié. Les adresses IPV4 et IPV6 sont prises en charge. Vous pouvez spécifier une adresse IP de requête unique ou des adresses IP associées à un sous-réseau spécifique. Par exemple, `11.22.33.0/22` autorise les requêtes provenant des adresses IP 11.22.32.1 à 11.22.35.254.</td>
      > </tr>
      > </table>

   5. Après avoir entré les valeurs des paramètres de chiffrement, sélectionnez une clé à chiffrer (si vous avez créé à la fois une clé primaire et une clé de sauvegarde) dans la liste **Key To Encrypt (Clé à chiffrer)** .
    
   6. Sélectionnez une version de chiffrement dans la liste **Version de chiffrement** liste : **V2** pour la version 2 ou **V3** pour la version 3 (recommandé). 

   7. Sélectionnez **Encrypt (Chiffrer)** pour générer le jeton.

      Lorsque le jeton est généré, il est affiché dans la zone **Generated Token (Jeton généré)** . Pour utiliser le jeton, ajoutez-le en tant que chaîne de requête à la fin du fichier dans le chemin de l’URL. Par exemple : `http://www.domain.com/content.mov?a4fbc3710fd3449a7c99986b`.
        
   8. Vous pouvez tester votre jeton avec l’outil de déchiffrement afin d’afficher les paramètres de votre jeton. Collez la valeur du jeton dans la zone **Token to Decrypt (Jeton à déchiffrer)** . Sélectionnez la clé de chiffrement à utiliser dans la liste **Key To Decrypt (Clé pour déchiffrer)** , puis sélectionnez **Decrypt (Déchiffrer)** .

      Une fois que le jeton est déchiffré, ses paramètres sont affichés dans la zone **Original Parameters (Paramètres d’origine)** .

   9. Personnalisez éventuellement le type de code de réponse qui est retourné lorsqu’une requête est refusée. Sélectionnez **Enabled (Activé)** , puis sélectionnez le code de réponse dans la liste **Code de réponse**. Le **Header Name (Nom d’en-tête)** est automatiquement défini sur **Location (Emplacement)** . Sélectionnez **Save (Enregistrer)** pour implémenter le nouveau code de réponse. Pour certains codes de réponse, vous devez également entrer l’URL de votre page d’erreur dans la zone **Header Value (Valeur d’en-tête)** . Le code de réponse **403** (Interdit) est sélectionné par défaut. 

3. Sélectionnez **Rules Engine (Moteur de règles)** sous **HTTP Large**. Le moteur de règles permet de définir les chemins d’accès pour appliquer la fonctionnalité, d’activer la fonctionnalité d’authentification du jeton et d’activer d’autres fonctionnalités associées à l’authentification du jeton. Pour plus d’informations, consultez [Moteur des règles Azure CDN](./cdn-verizon-premium-rules-engine-reference.md).

   1. Sélectionnez une règle existante ou créez-en une pour définir la ressource ou le chemin d’accès pour lesquels vous souhaitez appliquer l’authentification du jeton. 
   2. Pour activer l’authentification du jeton sur une règle, sélectionnez **[Token Auth (Authentification du jeton)](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth.htm)** dans la liste **Fonctionnalités**, puis sélectionnez **Enabled (Activé)** . Sélectionnez **Update (Mettre à jour)** si vous mettez à jour une règle ou **Add (Ajouter)** si vous en créez une.
        
      ![Exemple d’activation de l’authentification du jeton via le moteur de règles dans CDN](./media/cdn-token-auth/cdn-rules-engine-enable2.png)

4. Dans le moteur de règles, vous pouvez également activer d’autres fonctionnalités associées à l’authentification. Pour activer l’une des fonctionnalités suivantes, sélectionnez-la dans la liste **Features (Fonctionnalités)** , puis sélectionnez **Enabled (Activé)** .
    
   - **[Token Auth Denial Code (Code de refus d’authentification du jeton)](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Denial-Code.htm)**  : détermine le type de réponse à retourner à un utilisateur quand une requête est refusée. Les règles définies ici remplacent les codes de réponse de la section **Custom Denial Handling (Gestion personnalisée des refus)** de la page d’authentification basée sur le jeton.

   - **[Token Auth Ignore URL Case (Ignorer la casse de l’URL pour l’authentification du jeton)](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Ignore-URL-Case.htm)**  : détermine si la casse de l’URL utilisée pour valider le jeton est prise en compte.

   - **[Token Auth Parameter (Paramètre d’authentification du jeton)](https://docs.vdms.com/cdn/Content/HRE/F/Token-Auth-Parameter.htm)**  : renomme le paramètre de chaîne de requête d’authentification du jeton affiché dans l’URL demandée. 
        
     ![Exemple de paramètres d’authentification du jeton via le moteur de règles dans CDN](./media/cdn-token-auth/cdn-rules-engine2.png)

5. Vous pouvez personnaliser votre jeton en accédant au code source dans [GitHub](https://github.com/VerizonDigital/ectoken).
   Les langages disponibles sont notamment :
    
   - C
   - C#
   - PHP
   - Perl
   - Java
   - Python    

## <a name="azure-cdn-features-and-provider-pricing"></a>Tarification du fournisseur et des fonctionnalités du CDN Azure

Pour plus d’informations sur les fonctionnalités, consultez la page [Fonctionnalités du produit Azure CDN](cdn-features.md). Pour des informations sur les tarifs, consultez [Tarifs Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).