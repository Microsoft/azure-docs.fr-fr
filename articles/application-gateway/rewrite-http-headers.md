---
title: Réécrire des en-têtes HTTP avec Azure Application Gateway | Microsoft Docs
description: Cet article offre une vue d’ensemble de la réécriture d’en-têtes HTTP dans Azure Application Gateway
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 04/27/2020
ms.author: absha
ms.openlocfilehash: 3e8eb79d519e2f7bfbf006b852f0c5294976b727
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397147"
---
# <a name="rewrite-http-headers-with-application-gateway"></a>Réécrire des en-têtes HTTP avec Application Gateway

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les en-têtes HTTP permettent à un client et à un serveur de transmettre des informations supplémentaires avec une requête ou une réponse. En réécrivant ces en-têtes, vous pouvez effectuer des tâches importantes, comme ajouter des champs d’en-tête liés à la sécurité comme HSTS/X-XSS-Protection, supprimer des champs d’en-tête de réponse susceptibles de révéler des informations sensibles et supprimer les informations de port des en-têtes X-Forwarded-For.

Application Gateway vous permet d’ajouter, de supprimer et de mettre à jour les en-têtes de requête et de réponse HTTP pendant le déplacement des paquets de requête et de réponse entre le pool client et le pool back-end. Il vous permet également d’ajouter des conditions afin que les en-têtes spécifiés soient réécrits uniquement quand certaines conditions sont remplies.

Application Gateway prend aussi en charge plusieurs [variables serveur](#server-variables) qui vous permettent de stocker des informations supplémentaires sur les requêtes et les réponses. Vous pouvez ainsi créer plus facilement des règles de réécriture puissantes.

> [!NOTE]
>
> La prise en charge de la réécriture d’en-tête HTTP n’est disponible que pour [Standard_V2 et WAF_v2 SKU](application-gateway-autoscaling-zone-redundant.md).

![Réécriture d’en-têtes](media/rewrite-http-headers/rewrite-headers.png)

## <a name="supported-headers"></a>En-têtes pris en charge

Vous pouvez réécrire tous les en-têtes des requêtes et des réponses, à l’exclusion des en-têtes d’hôte, de connexion et de mise à niveau. Vous pouvez aussi utiliser la passerelle d’application pour créer des en-têtes personnalisés et les ajouter aux requêtes et réponses qui transitent par elle.

## <a name="rewrite-conditions"></a>Conditions de réécriture

Vous pouvez utiliser des conditions de réécriture pour évaluer le contenu des requêtes et des réponses HTTP(S) et procéder à une réécriture d’en-tête uniquement quand une ou plusieurs conditions sont remplies. La passerelle d’application utilise ces types de variables pour évaluer le contenu des requêtes et des réponses HTTP(S) :

- En-têtes HTTP de la requête.
- En-têtes HTTP de la réponse.
- Variables du serveur Application Gateway.

Vous pouvez utiliser une condition pour déterminer si une variable spécifiée est présente, si elle correspond à une valeur spécifique ou si elle correspond à des critères spéciaux. Vous devez utilisez la [bibliothèque PCRE (Perl Compatible Regular Expressions) Perl](https://www.pcre.org/) pour configurer des critères spéciaux d’expression régulière dans les conditions. Pour en savoir plus sur la syntaxe des expressions régulières, consultez la [page principale des expressions régulières Perl](https://perldoc.perl.org/perlre.html).

## <a name="rewrite-actions"></a>Actions de réécriture

Les actions de réécriture permettent de spécifier les en-têtes de requête et de réponse que vous souhaitez réécrire ainsi que la nouvelle valeur de ces en-têtes. Vous pouvez créer un nouvel en-tête, modifiez la valeur d’un en-tête existant ou supprimer un en-tête existant. La valeur d’un nouvel en-tête ou d’un en-tête existant peut être définie avec ces types de valeur :

- Texte.
- En-tête de demande. Pour spécifier un en-tête de requête, vous devez utiliser la syntaxe {http_req_ *headerName* }.
- En-tête de réponse. Pour spécifier un en-tête de réponse, vous devez utiliser la syntaxe {http_resp_ *headerName* }.
- Variable de serveur. Pour spécifier une variable de serveur, vous devez utiliser la syntaxe {var_ *serverVariable* }.
- Texte, en-tête de requête, en-tête de réponse et variable de serveur combinés.

## <a name="server-variables"></a>Variables de serveur

Application Gateway utilise des variables de serveur pour stocker des informations utiles sur le serveur, la connexion avec le client et la requête active sur la connexion. L’adresse IP du client et le type de navigateur web sont quelques exemples d’informations stockées. Les variables de serveur changent dynamiquement, par exemple, quand une nouvelle page est chargée ou qu’un formulaire est posté. Vous pouvez utiliser ces variables pour évaluer les conditions de réécriture et réécrire des en-têtes. Pour utiliser la valeur des variables de serveur afin de réécrire des en-têtes, vous devez spécifier ces variables dans la syntaxe {var_ *serverVariable* }

Application Gateway prend en charge ces variables de serveur :

| Nom de la variable | Description                                                  |
| -------------------------- | :----------------------------------------------------------- |
| add_x_forwarded_for_proxy  | Champ d’en-tête de requête client X-Forwarded-For auquel la variable `client_ip` (voir l’explication plus loin dans ce tableau) est ajoutée au format IP1, IP2, IP3 et ainsi de suite. Si le champ X-Forwarded-For ne se trouve pas dans l’en-tête de la requête cliente, la variable `add_x_forwarded_for_proxy` est égale à la variable `$client_ip`. Cette variable est particulièrement utile quand vous voulez réécrire l’en-tête X-Forwarded-For défini par Application Gateway de sorte que l’en-tête contienne uniquement l’adresse IP sans les informations de port. |
| ciphers_supported          | Liste de chiffrements pris en charge par le client.          |
| ciphers_used               | Chaîne de chiffrements utilisés pour une connexion TLS établie. |
| client_ip                  | Adresse IP du client à partir duquel la passerelle d’application a reçu la requête. S’il existe un proxy inverse devant la passerelle d’application et le client d’origine, *client_ip* retourne l’adresse IP du proxy inverse. |
| client_port                | Port client.                                                  |
| client_tcp_rtt             | Informations sur la connexion TCP cliente. Disponible sur les systèmes qui prennent en charge l’option de socket TCP_INFO. |
| client_user                | Quand l’authentification HTTP est utilisée, nom d’utilisateur fourni pour l’authentification. |
| host                       | Dans cet ordre de priorité : nom d’hôte de la ligne de la requête, nom d’hôte du champ d’en-tête de requête d’hôte ou nom de serveur correspondant à une requête. Exemple : dans la demande *http://contoso.com:8080/article.aspx?id=123&title=fabrikam* , la valeur host sera *contoso.com* |
| cookie_ *name*              | Cookie *name*.                                            |
| http_method                | Méthode utilisée pour effectuer la requête d’URL. Par exemple, GET ou POST. |
| HTTP_STATUS                | État de session. Par exemple, 200, 400 ou 403.                       |
| http_version               | Protocole de requête. Généralement, HTTP/1.0, HTTP/1.1 ou HTTP/2.0. |
| query_string               | Liste de paires variable/valeur qui suivent le « ? » dans l’URL demandée. Exemple : dans la demande *http://contoso.com:8080/article.aspx?id=123&title=fabrikam* , la valeur query_string sera *id=123&title=fabrikam* |
| received_bytes             | Longueur de la requête (incluant la ligne, l’en-tête et le corps de la requête). |
| request_query              | Arguments dans la ligne de la requête.                                |
| request_scheme             | Schéma de requête : « http » ou « https ».                            |
| request_uri                | URI complet de la requête d’origine (avec les arguments). Exemple : dans la demande *http://contoso.com:8080/article.aspx?id=123&title=fabrikam* , la valeur request_uri sera */article.aspx?id=123&title=fabrikam*   |
| sent_bytes                 | Nombre d’octets envoyés à un client.                             |
| server_port                | Port du serveur qui a accepté une requête.                 |
| ssl_connection_protocol    | Protocole d’une connexion TLS établie.        |
| ssl_enabled                | Valeur « On » si la connexion opère en mode TLS. Sinon, chaîne vide. |
| uri_path                   | Identifie la ressource spécifique dans l’hôte à laquelle le client web souhaite accéder. Il s’agit de la partie de l’URI de demande sans les arguments. Exemple : dans la demande *http://contoso.com:8080/article.aspx?id=123&title=fabrikam* , la valeur uri_path sera */article.aspx*  |

## <a name="rewrite-configuration"></a>Configuration de la réécriture

Pour configurer la réécriture d’en-tête HTTP, vous devez effectuer ces étapes.

1. Créez les objets nécessaires à la réécriture d’en-tête HTTP :

   - **Action de réécriture**  : permet de spécifier la requête et les champs d’en-tête de requête que vous souhaitez réécrire ainsi que la nouvelle valeur des en-têtes. Vous pouvez associer une ou plusieurs conditions de réécriture à une action de réécriture.

   - **Condition de réécriture**  : configuration facultative. Les conditions de réécriture évaluent le contenu des requêtes et réponses HTTP(S). L’action de réécriture se produit si la requête ou la réponse HTTP(S) correspondent à la condition de réécriture.

     Si vous associez plusieurs conditions à une action, cette dernière ne se produit que lorsque toutes les conditions sont remplies. En d’autres termes, il s’agit d’une opération AND logique.

   - **Règle de réécriture**  : contient plusieurs combinaisons d’actions/conditions de réécriture.

   - **Séquence de règle**  : permet de déterminer l’ordre dans lequel les règles de réécriture s’exécutent. Cette configuration est utile quand vous disposez de plusieurs règles de réécriture dans un jeu de réécritures. Une règle de réécriture qui présente une valeur de séquence de règle inférieure s’exécute en premier. Si vous attribuez la même séquence de règle à deux règles de réécriture, l’ordre d’exécution n’est pas déterministe.

   - **Jeu de réécritures**  : contient plusieurs règles de réécriture qui seront associées à une règle de routage de requête.

2. Attachez le jeu de réécritures ( *RewriteRuleSet* ) à une règle de routage. La configuration de réécriture est attachée à l’écouteur source via la règle de routage. Quand vous utilisez une règle de routage de base, la configuration de réécriture d’en-tête est associée à un écouteur source et correspond à une réécriture d’en-tête globale. Quand vous utilisez une règle de routage basée sur le chemin, la configuration de réécriture d’en-tête est définie sur le mappage du chemin d’URL. Dans ce cas, elle s’applique uniquement à la zone de chemin spécifique d’un site.
   > [!NOTE]
   > La réécriture d’URL modifie les en-têtes ; elle ne change pas l’URL du chemin.

Vous pouvez créer plusieurs jeux de réécritures d’en-tête HTTP et appliquer chacun d’eux à différents écouteurs. En revanche, vous ne pouvez appliquer qu’un seul jeu de réécritures à un écouteur spécifique.

## <a name="common-scenarios"></a>Scénarios courants

Voici quelques scénarios courants d’utilisation de la réécriture d’en-tête.

### <a name="remove-port-information-from-the-x-forwarded-for-header"></a>Supprimer les informations de port de l’en-tête X-Forwarded-For

Application Gateway insère un en-tête X-Forwarded-For dans toutes les requêtes avant de les transférer au back-end. Cet en-tête est une liste sont séparées par des virgules de ports IP. Dans certains scénarios, les serveurs back-end n’exigent pas que les en-têtes contiennent des adresses IP. Vous pouvez utiliser la réécriture d’en-tête pour supprimer les informations de port de l’en-tête X-Forwarded-For. Une façon d’y parvenir est de définir l’en-tête sur la variable de serveur add_x_forwarded_for_proxy :

![Supprimer le port](media/rewrite-http-headers/remove-port.png)

### <a name="modify-a-redirection-url"></a>Modifier une URL de redirection

Quand une application back-end envoie une réponse de redirection, vous pouvez souhaiter rediriger le client vers une URL différente de celle spécifiée par l’application back-end. Par exemple, cela peut être le cas quand un service d’application est hébergé derrière une passerelle d’application et demande au client d’effectuer une redirection vers son chemin relatif. (Par exemple, une redirection de contoso.azurewebsites.net/path1 vers contoso.azurewebsites.net/path2.)

App Service étant un service multilocataire, il utilise l’en-tête d’hôte de la requête pour router celle-ci vers le point de terminaison approprié. Les services d’application ont un nom de domaine par défaut *.azurewebsites.net (par exemple, contoso.azurewebsites.net) qui est différent du nom de domaine de la passerelle d’application (par exemple, contoso.com). Sachant que le nom d’hôte de la requête d’origine du client est le nom de domaine de la passerelle d’application (contoso.com), la passerelle d’application remplace le nom d’hôte par contoso.azurewebsites.net. Ce changement vise à permettre au service d’application de router la requête vers le point de terminaison approprié.

Quand le service d’application envoie une réponse de redirection, il utilise le nom d’hôte qui figure dans l’en-tête d’emplacement de sa réponse, qui est identique à celui situé dans la requête qu’il reçoit de la passerelle d’application. Par conséquent, le client adresse directement la requête à contoso.azurewebsites.net/path2 au lieu de passer par la passerelle d’application (contoso.com/path2). Or, il n’est pas souhaitable de contourner la passerelle d’application.

Vous pouvez résoudre ce problème en définissant le nom d’hôte de l’en-tête d’emplacement sur le nom de domaine de la passerelle d’application.

Voici les étapes à suivre pour remplacer le nom d’hôte :

1. Créez une règle de réécriture avec une condition qui détermine si l’en-tête d’emplacement de la réponse contient azurewebsites.net. Entrez le modèle `(https?):\/\/.*azurewebsites\.net(.*)$`.
1. Exécutez une action de façon à réécrire l’en-tête d’emplacement et lui attribuer le nom d’hôte de la passerelle d’application. Pour ce faire, entrez `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` comme valeur d’en-tête.

![Modifier l’en-tête d’emplacement](media/rewrite-http-headers/app-service-redirection.png)

### <a name="implement-security-http-headers-to-prevent-vulnerabilities"></a>Implémenter des en-têtes HTTP de sécurité pour éviter les vulnérabilités

Vous pouvez corriger plusieurs vulnérabilités de sécurité en implémentant les en-têtes nécessaires dans la réponse de l’application. Il s’agit notamment des en-têtes de sécurité X-XSS-Protection, Strict-Transport-Security et Content-Security-Policy. Vous pouvez utiliser Application Gateway pour définir ces en-têtes pour toutes les réponses.

![En-tête de sécurité](media/rewrite-http-headers/security-header.png)

### <a name="delete-unwanted-headers"></a>Supprimer les en-têtes indésirables

Vous pouvez souhaiter supprimer les en-têtes d’une réponse HTTP qui présentent des informations sensibles. Par exemple, vous pouvez souhaiter supprimer certaines informations comme les détails concernant le système d’exploitation, la bibliothèque ou le nom du serveur back-end. Vous pouvez utiliser la passerelle d’application pour supprimer ces en-têtes :

![Suppression d’en-tête](media/rewrite-http-headers/remove-headers.png)

### <a name="check-for-the-presence-of-a-header"></a>Vérifier la présence d’un en-tête

Vous pouvez évaluer un en-tête de requête ou de réponse HTTP pour déterminer s’il contient une variable d’en-tête ou de serveur. Cette évaluation est utile si vous voulez procéder à une réécriture d’en-tête qu’en présence d’un certain en-tête.

![Vérification de la présence d’un en-tête](media/rewrite-http-headers/check-presence.png)

## <a name="limitations"></a>Limites

- Si une réponse contient plusieurs en-têtes de même nom, la réécriture de la valeur de l’un de ces en-têtes entraîne la suppression des autres en-têtes de la réponse. Cela peut généralement se produire avec l’en-tête Set-Cookie, car une réponse peut en contenir plusieurs. Tel est le cas quand vous utilisez un service d’application avec une passerelle d’application et que vous avez configuré l’affinité de session basée sur les cookies sur la passerelle d’application. Dans ce cas, la réponse contient deux en-têtes Set-Cookie : un utilisé par le service d’application, par exemple `Set-Cookie: ARRAffinity=ba127f1caf6ac822b2347cc18bba0364d699ca1ad44d20e0ec01ea80cda2a735;Path=/;HttpOnly;Domain=sitename.azurewebsites.net`, et un autre pour l’affinité de la passerelle d’application, par exemple `Set-Cookie: ApplicationGatewayAffinity=c1a2bd51lfd396387f96bl9cc3d2c516; Path=/`. La réécriture de l’un des en-têtes Set-Cookie dans ce scénario peut entraîner la suppression de l’autre en-tête Set-Cookie de la réponse.

- Les réécritures ne sont pas prises en charge quand la passerelle d’application est configurée pour rediriger les demandes ou afficher une page d’erreur personnalisée.

- La réécriture des en-têtes de connexion, de mise à niveau et d’hôte n’est actuellement pas prise en charge.

- Les noms d’en-tête peuvent contenir n’importe quel caractère alphanumérique et des symboles spécifiques tels que définis dans la [RFC 7230](https://tools.ietf.org/html/rfc7230#page-27). Nous ne prenons actuellement pas en charge le caractère spécial de soulignement (\_) dans le nom d’en-tête.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment réécrire les en-têtes HTTP, consultez :

- [Réécrire les en-têtes HTTP à partir du portail Azure](./rewrite-http-headers-portal.md)
- [Réécrire les en-têtes HTTP à partir d’Azure PowerShell](add-http-header-rewrite-rule-powershell.md)