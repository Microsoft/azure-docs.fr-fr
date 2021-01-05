---
title: Réécrire des en-têtes HTTP et une URL avec Azure Application Gateway | Microsoft Docs
description: Cet article offre une vue d’ensemble de la réécriture d’en-têtes HTTP et d’URL dans Azure Application Gateway
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: surmb
ms.openlocfilehash: 93af3183ae9e969d14a35ce4e365d48895ef4e79
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92216672"
---
# <a name="rewrite-http-headers-and-url-with-application-gateway"></a>Réécrire des en-têtes HTTP et une URL à l’aide d’Application Gateway

 Application Gateway vous permet de réécrire certains contenus de requêtes et de réponses. Avec cette fonctionnalité, vous pouvez traduire des URL, interroger des paramètres de chaîne et modifier des en-têtes de requête et de réponse. Elle vous permet également d’ajouter des conditions assurant que l’URL ou les en-têtes spécifiés sont réécrits uniquement quand certaines conditions sont remplies. Ces conditions sont basées sur les informations de requête et de réponse.

>[!NOTE]
>Les fonctionnalités de réécriture d’en-tête HTTP et d’URL sont disponibles uniquement pour la [référence SKU Application Gateway v2](application-gateway-autoscaling-zone-redundant.md)

## <a name="rewrite-types-supported"></a>Types de réécriture pris en charge

### <a name="request-and-response-headers"></a>En-têtes de requête et de réponse

Les en-têtes HTTP permettent à un client et à un serveur de transmettre des informations supplémentaires avec une requête ou une réponse. En réécrivant ces en-têtes, vous pouvez effectuer des tâches importantes, comme ajouter des champs d’en-tête liés à la sécurité comme HSTS/X-XSS-Protection, supprimer des champs d’en-tête de réponse susceptibles de révéler des informations sensibles et supprimer les informations de port des en-têtes X-Forwarded-For.

Application Gateway vous permet d’ajouter, de supprimer et de mettre à jour les en-têtes de requête et de réponse HTTP pendant le déplacement des paquets de requête et de réponse entre le pool client et le pool back-end.

Pour savoir comment réécrire des en-têtes de requête et de réponse avec Application Gateway à l’aide du portail Azure, consultez [cette page](rewrite-url-portal.md).

![img](./media/rewrite-http-headers-url/header-rewrite-overview.png)


**En-têtes pris en charge**

Vous pouvez réécrire tous les en-têtes des requêtes et des réponses à l’exclusion des en-têtes de connexion et de mise à niveau. Vous pouvez aussi utiliser la passerelle d’application pour créer des en-têtes personnalisés et les ajouter aux requêtes et réponses qui transitent par elle.

### <a name="url-path-and-query-string-preview"></a>Chemin et chaîne de requête de l’URL (préversion)

Avec la fonctionnalité de réécriture d’URL d’Application Gateway, vous pouvez :

* Réécrire le nom d’hôte, le chemin et la chaîne de requête de l’URL de requête. 

* Choisir de réécrire l’URL de toutes les requêtes sur un écouteur ou uniquement des requêtes qui correspondent à une ou plusieurs des conditions que vous avez définies. Ces conditions sont basées sur les propriétés de requête et de réponse (requête, en-tête, en-tête de réponse et variables de serveur).

* Choisir de router la requête (en sélectionnant le pool de back-ends) en fonction de l’URL d’origine ou de l’URL réécrite.

Pour savoir comment réécrire une URL avec Application Gateway à l’aide du portail Azure, consultez [cette page](rewrite-url-portal.md).

![Schéma décrivant le processus de réécriture d’une URL avec Application Gateway.](./media/rewrite-http-headers-url/url-rewrite-overview.png)

>[!NOTE]
> La fonctionnalité de réécriture d’URL est en préversion et est disponible uniquement pour les références SKU Standard_v2 et WAF_v2 d’Application Gateway. Son utilisation n’est pas recommandée en environnement de production. Pour en savoir plus sur les préversions, consultez les [conditions d’utilisation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="rewrite-actions"></a>Actions de réécriture

Les actions de réécriture permettent de spécifier l’URL, les en-têtes de requête ou les en-têtes de réponse que vous souhaitez réécrire ainsi que les nouvelles valeurs correspondantes. La valeur d’une URL, d’un nouvel en-tête ou d’un en-tête existant peut être définie avec les types suivants :

* Texte
* En-tête de demande. Pour spécifier un en-tête de requête, vous devez utiliser la syntaxe {http_req_*headerName*}.
* En-tête de réponse. Pour spécifier un en-tête de réponse, vous devez utiliser la syntaxe {http_resp_*headerName*}.
* Variable de serveur. Pour spécifier une variable de serveur, vous devez utiliser la syntaxe {var_*serverVariable*}. Consultez la liste des variables de serveur prises en charge.
* Texte, en-tête de requête, en-tête de réponse et variable de serveur combinés. 

## <a name="rewrite-conditions"></a>Conditions de réécriture

Vous pouvez utiliser des conditions de réécriture (configuration facultative) pour évaluer le contenu des requêtes et des réponses HTTP(S) et procéder à une réécriture uniquement quand une ou plusieurs conditions sont remplies. La passerelle d’application utilise ces types de variables pour évaluer le contenu des requêtes et des réponses :

* En-têtes HTTP de la requête
* En-têtes HTTP de la réponse
* Variables du serveur Application Gateway

Vous pouvez utiliser une condition pour déterminer si une variable spécifiée est présente, si elle correspond à une valeur spécifique ou si elle correspond à des critères spéciaux. 


### <a name="pattern-matching"></a>Critères spéciaux 

Application Gateway utilise des expressions régulières pour les critères spéciaux dans la condition. Vous pouvez utiliser la [bibliothèque PCRE (Perl Compatible Regular Expressions) Perl](https://www.pcre.org/) pour configurer des critères spéciaux d’expression régulière dans les conditions. Pour en savoir plus sur la syntaxe des expressions régulières, consultez la [page principale des expressions régulières Perl](https://perldoc.perl.org/perlre.html).

### <a name="capturing"></a>Capture

Pour capturer une sous-chaîne en vue d’une utilisation ultérieure, placez des parenthèses autour du sous-modèle qui lui correspond dans la définition d’expression régulière de la condition. La première paire de parenthèses stocke sa sous-chaîne dans le groupe 1, la deuxième paire dans le groupe 2 et ainsi de suite. Vous pouvez utiliser autant de parenthèses que vous le souhaitez. Perl continue simplement de définir des variables numérotées pour vous afin de représenter ces chaînes capturées. Voici quelques exemples tirés de ce [document de référence](https://docstore.mik.ua/orelly/perl/prog3/ch05_07.htm) : 

*  /(\d)(\d)/ # Deux chiffres correspondants, capturés dans les groupes 1 et 2 

* /(\d+)/ # Un ou plusieurs chiffres correspondants, capturés dans le groupe 1 

* /(\d)+/ # Un chiffre correspondant une ou plusieurs fois, le dernier étant capturé dans le groupe 1

Dès lors que les sous-chaînes sont capturées, vous pouvez les référencer dans l’ensemble d’actions en utilisant le format suivant :

* Pour une capture d’en-tête de requête, vous devez utiliser {http_req_headerName_groupNumber}. Par exemple, {http_req_User-Agent_1} ou {http_req_User-Agent_2}.
* Pour une capture d’en-tête de réponse, vous devez utiliser {http_resp_headerName_groupNumber}. Par exemple, {http_resp_Location_1} ou {http_resp_Location_2}.
* Pour une variable de serveur, vous devez utiliser {var_serverVariableName_groupNumber}. Par exemple, {var_uri_path_1} ou {var_uri_path_2}.

Si vous souhaitez utiliser la valeur entière, vous ne devez pas mentionner le nombre. Utilisez simplement le format {http_req_headerName}, etc. sans groupNumber.

## <a name="server-variables"></a>Variables de serveur

Application Gateway utilise des variables de serveur pour stocker des informations utiles sur le serveur, la connexion avec le client et la requête active sur la connexion. L’adresse IP du client et le type de navigateur web sont quelques exemples d’informations stockées. Les variables de serveur changent dynamiquement, par exemple, quand une nouvelle page est chargée ou qu’un formulaire est posté. Vous pouvez utiliser ces variables pour évaluer les conditions de réécriture et réécrire des en-têtes. Pour utiliser la valeur des variables de serveur afin de réécrire des en-têtes, vous devez spécifier ces variables dans la syntaxe {var_*serverVariableName*}.

Application Gateway prend en charge les variables de serveur suivantes :

|   Nom de la variable    |                   Description                                           |
| ------------------------- | ------------------------------------------------------------ |
| add_x_forwarded_for_proxy | Champ d’en-tête de requête client X-Forwarded-For auquel la variable `client_ip` (voir l’explication plus loin dans ce tableau) est ajoutée au format IP1, IP2, IP3 et ainsi de suite. Si le champ X-Forwarded-For ne se trouve pas dans l’en-tête de requête client, la variable `add_x_forwarded_for_proxy` est égale à la variable `$client_ip`.   Cette variable est particulièrement utile quand vous voulez réécrire l’en-tête X-Forwarded-For défini par Application Gateway pour que l’en-tête contienne uniquement l’adresse IP sans les informations de port. |
| ciphers_supported         | Liste de chiffrements pris en charge par le client.               |
| ciphers_used              | Chaîne de chiffrements utilisés pour une connexion TLS établie. |
| client_ip                 | Adresse IP du client à partir duquel la passerelle d’application a reçu la requête. S’il existe un proxy inverse devant la passerelle d’application et le client d’origine, `client_ip` retourne l’adresse IP du proxy inverse. |
| client_port               | Port client.                                             |
| client_tcp_rtt            | Informations sur la connexion TCP cliente. Disponible sur les systèmes qui prennent en charge l’option de socket TCP_INFO. |
| client_user               | Quand l’authentification HTTP est utilisée, nom d’utilisateur fourni pour l’authentification. |
| host                      | Dans cet ordre de priorité : nom d’hôte de la ligne de la requête, nom d’hôte du champ d’en-tête de requête d’hôte ou nom de serveur correspondant à une requête. Exemple : dans la requête `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, la valeur host sera `contoso.com` |
| cookie_*name*             | Cookie *name*.                                           |
| http_method               | Méthode utilisée pour effectuer la requête d’URL. Par exemple, GET ou POST. |
| HTTP_STATUS               | État de session. Par exemple, 200, 400 ou 403.           |
| http_version              | Protocole de requête. Généralement, HTTP/1.0, HTTP/1.1 ou HTTP/2.0. |
| query_string              | Liste de paires variable/valeur suivant le « ? » dans l’URL demandée. Exemple : dans la requête `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, la valeur query_string sera `id=123&title=fabrikam` |
| received_bytes            | Longueur de la requête (incluant la ligne, l’en-tête et le corps de la requête). |
| request_query             | Arguments dans la ligne de la requête.                           |
| request_scheme            | Schéma de requête : « http » ou « https ».                           |
| request_uri               | URI complet de la requête d’origine (avec les arguments). Exemple : dans la requête `http://contoso.com:8080/article.aspx?id=123&title=fabrikam*`, la valeur request_uri sera `/article.aspx?id=123&title=fabrikam` |
| sent_bytes                | Nombre d’octets envoyés à un client.                        |
| server_port               | Port du serveur qui a accepté une requête.              |
| ssl_connection_protocol   | Protocole d’une connexion TLS établie.               |
| ssl_enabled               | Valeur « On » si la connexion opère en mode TLS. Sinon, chaîne vide. |
| uri_path                  | Identifie la ressource spécifique dans l’hôte à laquelle le client web souhaite accéder. Il s’agit de la partie de l’URI de requête sans les arguments. Exemple : dans la requête `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, la valeur uri_path sera `/article.aspx` |

 

## <a name="rewrite-configuration"></a>Configuration de la réécriture

Pour configurer une règle de réécriture, vous devez créer un ensemble de règles de réécriture et y ajouter la configuration des règles de réécriture.

Un ensemble de règles de réécriture contient les éléments suivants :

* **Association avec règle de routage de requête** : la configuration de réécriture est associée à l’écouteur source par le biais de la règle de routage. Quand vous utilisez une règle de routage de base, la configuration de réécriture est associée à un écouteur source et correspond à une réécriture d’en-tête globale. Quand vous utilisez une règle de routage basée sur le chemin, la configuration de réécriture est définie sur le mappage du chemin d’URL. Dans ce cas, elle s’applique uniquement à la zone de chemin spécifique d’un site. Vous pouvez créer plusieurs jeux de réécritures et appliquer chacun d’eux à différents écouteurs. En revanche, vous ne pouvez appliquer qu’un seul jeu de réécritures à un écouteur spécifique.

* **Condition de réécriture** : cette configuration est facultative. Les conditions de réécriture évaluent le contenu des requêtes et réponses HTTP(S). L’action de réécriture se produit si la requête ou la réponse HTTP(S) correspondent à la condition de réécriture. Si vous associez plusieurs conditions à une action, cette dernière ne se produit que lorsque toutes les conditions sont remplies. En d’autres termes, il s’agit d’une opération AND logique.

* **Type de réécriture** :  3 types de réécriture sont disponibles :
   * Réécriture des en-têtes de requête 
   * Réécriture des en-têtes de réponse
   * Réécriture d’URL : Le type de réécriture d’URL a 3 composants.
      * **Chemin d’URL** : valeur vers laquelle le chemin doit être réécrit. 
      * **Chaîne de requête de l’URL** : valeur vers laquelle la chaîne de requête doit être réécrite. 
      * **Réévaluer le mappage du chemin** : utilisé pour déterminer si le mappage du chemin d’URL doit être réévalué ou non. Si cette option est désactivée, le chemin d’URL d’origine est utilisé pour la correspondance avec le chemin de modèle dans le mappage du chemin d’URL. Si la valeur est true, le mappage du chemin d’URL est réévalué pour que la correspondance avec le chemin réécrit soit vérifiée. L’activation de ce commutateur permet de router la requête vers un autre pool de back-ends après la réécriture.

### <a name="common-scenarios-for-header-rewrite"></a>Scénarios courants de réécriture d’en-tête

#### <a name="remove-port-information-from-the-x-forwarded-for-header"></a>Supprimer les informations de port de l’en-tête X-Forwarded-For

Application Gateway insère un en-tête X-Forwarded-For dans toutes les requêtes avant de les transférer au back-end. Cet en-tête est une liste sont séparées par des virgules de ports IP. Dans certains scénarios, les serveurs back-end n’exigent pas que les en-têtes contiennent des adresses IP. Vous pouvez utiliser la réécriture d’en-tête pour supprimer les informations de port de l’en-tête X-Forwarded-For. Une façon d’y parvenir est de définir l’en-tête sur la variable de serveur add_x_forwarded_for_proxy. Vous pouvez également utiliser la variable client_ip :

![Supprimer le port](./media/rewrite-http-headers-url/remove-port.png)


#### <a name="modify-a-redirection-url"></a>Modifier une URL de redirection

Quand une application back-end envoie une réponse de redirection, vous pouvez souhaiter rediriger le client vers une URL différente de celle spécifiée par l’application back-end. Par exemple, cela peut être le cas quand un service d’application est hébergé derrière une passerelle d’application et demande au client d’effectuer une redirection vers son chemin relatif. (Par exemple, une redirection de contoso.azurewebsites.net/path1 vers contoso.azurewebsites.net/path2.)

App Service étant un service multilocataire, il utilise l’en-tête d’hôte de la requête pour router celle-ci vers le point de terminaison approprié. Les services d’application ont un nom de domaine par défaut *.azurewebsites.net (par exemple, contoso.azurewebsites.net) qui est différent du nom de domaine de la passerelle d’application (par exemple, contoso.com). Sachant que le nom d’hôte de la requête d’origine du client est le nom de domaine de la passerelle d’application (contoso.com), la passerelle d’application remplace le nom d’hôte par contoso.azurewebsites.net. Ce changement vise à permettre au service d’application de router la requête vers le point de terminaison approprié.

Quand le service d’application envoie une réponse de redirection, il utilise le nom d’hôte qui figure dans l’en-tête d’emplacement de sa réponse, qui est identique à celui situé dans la requête qu’il reçoit de la passerelle d’application. Par conséquent, le client adresse directement la requête à contoso.azurewebsites.net/path2 au lieu de passer par la passerelle d’application (contoso.com/path2). Or, il n’est pas souhaitable de contourner la passerelle d’application.

Vous pouvez résoudre ce problème en définissant le nom d’hôte de l’en-tête d’emplacement sur le nom de domaine de la passerelle d’application.

Voici les étapes à suivre pour remplacer le nom d’hôte :

1. Créez une règle de réécriture avec une condition qui détermine si l’en-tête d’emplacement de la réponse contient azurewebsites.net. Entrez le modèle `(https?):\/\/.*azurewebsites\.net(.*)$`.
2. Exécutez une action de façon à réécrire l’en-tête d’emplacement et lui attribuer le nom d’hôte de la passerelle d’application. Pour ce faire, entrez `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` comme valeur d’en-tête. Vous pouvez également utiliser la variable de serveur `host` pour définir le nom d’hôte afin qu’il corresponde à la requête d’origine.

![Modifier l’en-tête d’emplacement](./media/rewrite-http-headers-url/app-service-redirection.png)

#### <a name="implement-security-http-headers-to-prevent-vulnerabilities"></a>Implémenter des en-têtes HTTP de sécurité pour éviter les vulnérabilités

Vous pouvez corriger plusieurs vulnérabilités de sécurité en implémentant les en-têtes nécessaires dans la réponse de l’application. Il s’agit notamment des en-têtes de sécurité X-XSS-Protection, Strict-Transport-Security et Content-Security-Policy. Vous pouvez utiliser Application Gateway pour définir ces en-têtes pour toutes les réponses.

![En-tête de sécurité](./media/rewrite-http-headers-url/security-header.png)

### <a name="delete-unwanted-headers"></a>Supprimer les en-têtes indésirables

Vous pouvez souhaiter supprimer les en-têtes d’une réponse HTTP qui présentent des informations sensibles. Par exemple, vous pouvez souhaiter supprimer certaines informations comme les détails concernant le système d’exploitation, la bibliothèque ou le nom du serveur back-end. Vous pouvez utiliser la passerelle d’application pour supprimer ces en-têtes :

![Suppression d’en-tête](./media/rewrite-http-headers-url/remove-headers.png)

#### <a name="check-for-the-presence-of-a-header"></a>Vérifier la présence d’un en-tête

Vous pouvez évaluer un en-tête de requête ou de réponse HTTP pour déterminer s’il contient une variable d’en-tête ou de serveur. Cette évaluation est utile si vous voulez procéder à une réécriture d’en-tête qu’en présence d’un certain en-tête.

![Vérification de la présence d’un en-tête](./media/rewrite-http-headers-url/check-presence.png)

### <a name="common-scenarios-for-url-rewrite"></a>Scénarios courants de réécriture d’URL

#### <a name="parameter-based-path-selection"></a>Sélection du chemin en fonction des paramètres

Pour les scénarios dans lesquels vous souhaitez choisir le pool de back-ends en fonction de la valeur d’un en-tête, d’une partie de l’URL ou d’une chaîne de requête, vous pouvez utiliser conjointement la fonctionnalité de réécriture d’URL et le routage basé sur le chemin.  Par exemple, si vous avez un site web de vente en ligne, que la catégorie de produit est transmise comme chaîne de requête dans l’URL et que vous souhaitez router la requête vers le back-end en fonction de la chaîne de requête, effectuez les étapes suivantes :

**Étape 1 :**  Créez un mappage de chemin comme indiqué dans l’image ci-dessous.

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-1.png" alt-text="Scénario de réécriture d’URL 1-1.":::

**Étape 2 (a) :** Créez un jeu de réécritures avec 3 règles de réécriture : 

* La première règle inclut une condition qui vérifie *category=shoes* dans la variable *query_string* et une action qui réécrit le chemin d’URL vers /*listing1* et pour laquelle l’option **Réévaluer le mappage du chemin** est activée.

* La deuxième règle inclut une condition qui vérifie *category=bags* dans la variable *query_string* et une action qui réécrit le chemin d’URL vers /*listing2* et pour laquelle l’option **Réévaluer le mappage du chemin** est activée.

* La troisième règle inclut une condition qui vérifie *category=accessories* dans la variable *query_string* et une action qui réécrit le chemin d’URL vers /*listing3* et pour laquelle l’option **Réévaluer le mappage du chemin** est activée.

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-2.png" alt-text="Scénario de réécriture d’URL 1-1.":::

 

**Étape 2 (b) :** Associez ce jeu de réécritures au chemin par défaut de la règle basée sur le chemin ci-dessus.

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-3.png" alt-text="Scénario de réécriture d’URL 1-1.":::

À présent, si l’utilisateur demande *contoso.com/listing?category=any*, la chaîne sera mise en correspondance avec le chemin par défaut, car aucun des modèles de chemin du mappage du chemin (/listing1, /listing2, /listing3) ne correspond. Étant donné que vous avez associé le jeu de réécritures ci-dessus à ce chemin, ce jeu de réécritures sera évalué. Étant donné que la chaîne de requête ne correspondra pas à la condition des 3 règles de réécriture de ce jeu de réécritures, aucune action de réécriture n’aura lieu. La requête sera donc routée sans modification vers le back-end associé au chemin par défaut (*GenericList*).

 Si l’utilisateur demande *contoso.com/listing?category=shoes*, le chemin par défaut sera de nouveau mis en correspondance. Toutefois, dans ce cas, la condition de la première règle correspondra. L’action associée à la condition sera donc exécutée, entraînant la réécriture du chemin d’URL vers /*listing1* et la réévaluation du mappage du chemin. Quand le mappage du chemin est réévalué, la requête correspond au chemin associé au modèle */listing1* et est routée vers le back-end associé à ce modèle, à savoir ShoesListBackendPool.

>[!NOTE]
>Ce scénario peut être étendu à n’importe quel chemin d’URL, valeur d’en-tête ou de cookie, chaîne de requête ou variable de serveur selon la condition définie et vous permet essentiellement de router les requêtes en fonction de ces conditions.

#### <a name="rewrite-query-string-parameters-based-on-the-url"></a>Réécrire les paramètres de chaîne de requête en fonction de l’URL

Prenons l’exemple d’un site web de commerce en ligne. Le lien visible par l’utilisateur doit être simple et lisible, mais le serveur back-end a besoin des paramètres de la chaîne de requête pour afficher le contenu approprié.

Dans ce cas, Application Gateway peut capturer les paramètres à partir de l’URL et ajouter les paires clé-valeur de la chaîne de requête à partir de celles de l’URL. Par exemple, supposons que l’utilisateur souhaite réécrire `https://www.contoso.com/fashion/shirts` en `https://www.contoso.com/buy.aspx?category=fashion&product=shirts`. Pour cela, il peut utiliser la configuration de réécriture d’URL suivante.

**Condition** - Si la variable de serveur `uri_path` correspond au modèle `/(.+)/(.+)`

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario2-1.png" alt-text="Scénario de réécriture d’URL 1-1.":::

**Action** - Définir le chemin d’URL sur `buy.aspx` et la chaîne de requête sur `category={var_uri_path_1}&product={var_uri_path_2}`

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario2-2.png" alt-text="Scénario de réécriture d’URL 1-1.":::

Pour obtenir un guide pas à pas permettant de réaliser le scénario décrit ci-dessus, consultez [Réécriture d’URL avec Azure Application Gateway – Portail Azure](rewrite-url-portal.md).

### <a name="url-rewrite-vs-url-redirect"></a>Réécriture d’URL et redirection d’URL

Dans le cadre d’une réécriture d’URL, Application Gateway réécrit l’URL avant que la requête ne soit envoyée au back-end. Cela ne change rien à ce que les utilisateurs voient dans le navigateur. En effet, les modifications sont masquées pour l’utilisateur.

Dans le cadre d’une redirection d’URL, Application Gateway envoie une réponse de redirection au client avec la nouvelle URL. Ceci oblige alors le client à renvoyer sa requête à la nouvelle URL fournie dans la redirection. L’URL que l’utilisateur voit dans le navigateur sera mise à jour vers la nouvelle URL.

:::image type="content" source="./media/rewrite-http-headers-url/url-rewrite-vs-redirect.png" alt-text="Scénario de réécriture d’URL 1-1.":::

## <a name="limitations"></a>Limites

- Si une réponse contient plusieurs en-têtes de même nom, la réécriture de la valeur de l’un de ces en-têtes entraîne la suppression des autres en-têtes de la réponse. Cela peut généralement se produire avec l’en-tête Set-Cookie, car une réponse peut en contenir plusieurs. Tel est le cas quand vous utilisez un service d’application avec une passerelle d’application et que vous avez configuré l’affinité de session basée sur les cookies sur la passerelle d’application. Dans ce cas, la réponse contient deux en-têtes Set-Cookie : un utilisé par le service d’application, par exemple `Set-Cookie: ARRAffinity=ba127f1caf6ac822b2347cc18bba0364d699ca1ad44d20e0ec01ea80cda2a735;Path=/;HttpOnly;Domain=sitename.azurewebsites.net`, et un autre pour l’affinité de la passerelle d’application, par exemple `Set-Cookie: ApplicationGatewayAffinity=c1a2bd51lfd396387f96bl9cc3d2c516; Path=/`. La réécriture de l’un des en-têtes Set-Cookie dans ce scénario peut entraîner la suppression de l’autre en-tête Set-Cookie de la réponse.
- Les réécritures ne sont pas prises en charge quand la passerelle d’application est configurée pour rediriger les demandes ou afficher une page d’erreur personnalisée.
- Les noms d’en-tête peuvent contenir n’importe quel caractère alphanumérique et des symboles spécifiques tels que définis dans la [RFC 7230](https://tools.ietf.org/html/rfc7230#page-27). Nous ne prenons actuellement pas en charge le caractère spécial de soulignement (_) dans les noms d’en-tête.
- Les en-têtes de connexion et de mise à niveau ne peuvent pas être réécrits.

## <a name="next-steps"></a>Étapes suivantes

- [Découvrez comment réécrire des en-têtes HTTP avec Application Gateway à l’aide du portail Azure](rewrite-http-headers-portal.md).
- [Découvrez comment réécrire une URL avec Application Gateway à l’aide du portail Azure](rewrite-url-portal.md).
