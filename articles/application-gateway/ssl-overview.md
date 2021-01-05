---
title: Activation du protocole TLS de bout en bout sur Azure Application Gateway
description: Cet article fournit une vue d’ensemble de la prise en charge TLS de bout en bout pour Application Gateway.
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: victorh
ms.openlocfilehash: c39401289ffc6f27c292168adaa15c5163a3967b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001284"
---
# <a name="overview-of-tls-termination-and-end-to-end-tls-with-application-gateway"></a>Présentation de la terminaison TLS et du chiffrement TLS de bout en bout avec Application Gateway

Transport Layer Security (TLS), anciennement appelé Secure Sockets Layer (SSL), est la technologie de sécurité standard pour établir une liaison chiffrée entre un serveur web et un navigateur. Cette liaison garantit que toutes les données transmises entre le serveur web et les navigateurs restent privées et chiffrées. Application Gateway prend en charge la terminaison TLS sur la passerelle ainsi que le chiffrement TLS de bout en bout.

## <a name="tls-termination"></a>Arrêt TLS

Application Gateway prend en charge l’arrêt TLS au niveau de la passerelle, après lequel le trafic transite généralement de façon non chiffrée vers les serveurs backend. La terminaison TLS établie au niveau de la passerelle offre un certain nombre d’avantages :

- **Amélioration des performances** – Lors du déchiffrement TLS, la négociation initiale est l’opération qui dégrade le plus les performances. Pour améliorer les performances, le serveur qui effectue le déchiffrement met en cache les ID de session TLS et gère les tickets de session TLS. Si cela est fait au niveau de la passerelle d’application, toutes les requêtes d’un même client peuvent utiliser les valeurs mises en cache. Si cela est fait sur les serveurs principaux, le client doit s’authentifier chaque fois que les requêtes du client sont adressées à un serveur différent. L’utilisation de tickets TLS peut aider à atténuer ce problème, mais ces tickets ne sont pas pris en charge par tous les clients et peuvent être difficiles à configurer et à gérer.
- **Meilleure utilisation des serveurs principaux**  - Le traitement SSL/TLS nécessite une grande quantité de ressources processeur et, à mesure qu’il s’intensifie, la taille des clés augmente. Ce travail n’est plus dévolu aux serveurs principaux si bien qu’ils peuvent se concentrer sur ce qu’ils savent faire le mieux : diffuser du contenu.
- **Routage intelligent** – En déchiffrant le trafic, la passerelle d’application a accès au contenu de la requête, comme les en-têtes, l’URI, etc. et peut utiliser ces données pour acheminer les requêtes.
- **Gestion des certificats** - Il suffit d’acheter et d’installer les certificats sur la passerelle d’application et non sur tous les serveurs principaux. Cela permet d’économiser du temps et de l’argent.

Pour configurer l’arrêt TLS, un certificat TLS/SSL doit être ajouté à l’écouteur pour permettre à la passerelle d’application de dériver une clé symétrique conformément aux spécifications du protocole TLS/SSL. La clé symétrique est ensuite utilisée pour chiffrer et déchiffrer le trafic envoyé à la passerelle. Le certificat TLS/SSL doit être partagé au format Personal Information Exchange (PFX). Ce format de fichier permet d’exporter la clé privée requise par la passerelle d’application pour effectuer le chiffrement et le déchiffrement du trafic.

> [!IMPORTANT] 
> Le certificat sur l’écouteur exige le chargement de la totalité de la chaîne de certificats (le certificat racine de l’autorité de certification, les intermédiaires et le certificat feuille) pour établir la chaîne d’approbation. 


> [!NOTE] 
>
> La passerelle d’application ne permet pas de créer un certificat numérique, ni d’envoyer une demande de certificat numérique à une autorité de certification.

Pour que la connexion TLS fonctionne, assurez-vous que le certificat TLS/SSL remplit les conditions suivantes :

- La date/heure actuelle est dans la plage de dates « Valide à partir du » et « Valide jusqu’au » sur le certificat.
- Le « nom commun » (CN) du certificat correspond à l’en-tête de l’hôte dans la requête. Par exemple, si le client adresse une requête à `https://www.contoso.com/`, le nom commun doit être `www.contoso.com`.

### <a name="certificates-supported-for-tls-termination"></a>Certificats pris en charge pour un arrêt TLS

Application Gateway prend en charge les types de certificat suivants :

- Certificat délivré par une autorité de certification (CA) : Un certificat CA est un certificat numérique émis par une autorité de certification (CA).
- Certificat EV (à validation étendue) : Un certificat EV est un certificat conforme aux directives standard du secteur. La barre de recherche du navigateur devient verte et le nom de la société s’affiche.
- Certificat générique : Ce certificat prend en charge un nombre quelconque de sous-domaines basés sur *.site.com, adresse dans laquelle l’astérisque correspond au sous-domaine. Cependant, il ne prend pas en charge l’adresse site.com ; ainsi, le certificat numérique ne gère pas le cas où un utilisateur accède à votre site web sans avoir saisi « www » au début de l’adresse.
- Certificats auto-signés : Les navigateurs clients ne font pas confiance à ces certificats et avertiront l’utilisateur que le certificat du service virtuel ne fait pas partie d’une chaîne d’approbation. Les certificats auto-signés conviennent pour les tests ou les environnements dans lesquels les administrateurs contrôlent les clients et peuvent ignorer en toute sécurité les alertes de sécurité du navigateur. Les charges de travail de production ne doivent jamais utiliser de certificats auto-signés.

Pour plus d’informations, consultez [Configurer l’arrêt TLS avec Application Gateway](./create-ssl-portal.md).

### <a name="size-of-the-certificate"></a>Taille du certificat
Consultez la section [Limites d’Application Gateway](../azure-resource-manager/management/azure-subscription-service-limits.md#application-gateway-limits) pour connaître la taille maximale des certificats TLS/SSL pris en charge.

## <a name="end-to-end-tls-encryption"></a>Chiffrement TSL de bout en bout

Il se peut que vous ne vouliez pas d’une communication non chiffrée vers les serveurs principaux. Il se peut que vous ayez des exigences de sécurité ou de conformité spécifiques, ou que l’application n’accepte qu’une connexion sécurisée. Azure Application Gateway dispose d’un chiffrement TLS de bout en bout pour prendre en charge ces exigences.

Le chiffrement TLS de bout en bout vous permet de chiffrer et transmettre en toute sécurité des données sensibles au serveur principal quand vous utilisez des fonctionnalités d’équilibrage de charge de couche 7 d’Application Gateway. Il s’agit notamment de la fonctionnalité d’affinité basée sur les cookies, du routage basé sur l’URL, de la prise en charge du routage basé sur des sites, de la possibilité de remplacer ou d’injecter des en-têtes X-Forwarded-*.

Quand elle est configurée avec un mode de communication TLS de bout en bout, la passerelle Application Gateway ferme les sessions TLS au niveau de la passerelle et déchiffre le trafic utilisateur. Il applique ensuite les règles configurées pour sélectionner une instance de pool principal appropriée vers laquelle acheminer le trafic. Le service Application Gateway établit ensuite une nouvelle connexion TLS vers le serveur principal, puis chiffre à nouveau les données à l’aide du certificat de clé publique de ce serveur avant de transmettre la requête au serveur principal. Toute réponse du serveur web passe par le même processus vers l’utilisateur final. Pour activer le chiffrement TLS de bout en bout, définissez le [Paramètre HTTP du serveur principal](./configuration-overview.md#http-settings) sur HTTPS, qui est ensuite appliqué à un pool de serveurs principaux.

Pour la référence (SKU) Application Gateway et WAF v1, la stratégie TLS s’applique au trafic frontal et principal. Au niveau frontal, Application Gateway agit en tant que serveur et applique la stratégie. Sur le serveur principal, Application Gateway agit en tant que client et envoie les informations de protocole/chiffrement en tant que préférence pendant la négociation TLS.

Pour la référence (SKU) Application Gateway et WAF v2, la stratégie TLS s’applique uniquement au trafic frontal, et tous les chiffrements sont offerts au serveur principal qui contrôle la sélection de chiffrements spécifiques et de la version TLS pendant la poignée de main.

Application Gateway communique uniquement avec les serveurs back-end qui ont mis leur certificat en liste verte avec Application Gateway, ou dont les certificats sont signés par des autorités de certification reconnues, et lorsque le nom commun du certificat correspond au nom de l’hôte dans les paramètres HTTP du serveur back-end. Cela inclut les services Azure approuvés, tels que Azure App Service/Web Apps et la Gestion des API Azure.

Si les certificats des membres dans le pool principal ne sont pas signés par des autorités de certification connues, chaque instance du pool principal pour lequel un chiffrement TLS de bout en bout est activé doit être configurée avec un certificat afin de permettre une communication sécurisée. L’ajout du certificat permet à la passerelle d’application de communiquer uniquement avec des instances de serveur principal connues. Il sécurise la communication de bout en bout.

> [!NOTE] 
>
> Le certificat ajouté au **Paramètre HTTP du serveur principal** pour authentifier les serveurs principaux peut être le même que le certificat ajouté à **l’écouteur** pour la terminaison TLS au niveau de la passerelle d’application ou être différent pour une sécurité accrue.

![scénario TLS de bout en bout][1]

Dans cet exemple, les demandes utilisant TLS1.2 sont acheminées vers les serveurs principaux dans Pool1 à l’aide du chiffrement TLS de bout en bout.

## <a name="end-to-end-tls-and-allow-listing-of-certificates"></a>Chiffrement TLS de bout en bout et mise en liste verte de certificats

Application Gateway communique uniquement avec des instances de serveur back-end connues qui ont mis leur certificat en liste verte avec la passerelle Application Gateway. Il existe des différences de processus de configuration du protocole TLS de bout en bout en lien avec la version d’Application Gateway utilisée. La section suivante les décrit individuellement.

## <a name="end-to-end-tls-with-the-v1-sku"></a>Protocole TLS de bout en bout avec la référence (SKU) v1

Pour activer le protocole TLS de bout en bout avec les serveurs principaux et pour que le service Application Gateway route les requêtes vers ceux-ci, les sondes d’intégrité doivent opérer correctement et retourner une réponse saine.

Pour des sondes d’intégrité HTTPS, la référence (SKU) Application Gateway v1 utilise une concordance exacte du certificat d’authentification (clé publique du certificat de serveur principal et non du certificat racine) à télécharger dans les paramètres HTTP.

Seules les connexions à des serveurs back-end connus et sur liste verte sont alors autorisées. Les sondes d’intégrité considèrent les serveurs principaux restants comme non sains. Les certificats auto-signés sont uniquement destinés à des fins de test et ne sont pas recommandés pour les charges de travail de production. Avant de pouvoir être utilisés, ces certificats doivent être mis sur liste verte avec la passerelle d’application comme décrit dans les étapes précédentes.

> [!NOTE]
> La configuration de l’authentification et du certificat racine approuvé n’est pas requise pour des services Azure approuvés, tels qu’Azure App Service. Ils sont considérés comme approuvés par défaut.

## <a name="end-to-end-tls-with-the-v2-sku"></a>Protocole TLS de bout en bout avec la référence (SKU) v2

Les certificats d’authentification ont été déconseillés et remplacés par des certificats racines approuvés dans la référence SKU v2 d’Application Gateway. Ils fonctionnent comme les certificats d’authentification, avec quelques différences importantes :

- Les certificats signés par des autorités de certification connues dont le nom commun correspond au nom de l’hôte dans les paramètres HTTP du serveur principal ne nécessitent pas d’étape supplémentaire pour que le protocole TLS de bout en bout fonctionne. 

   Par exemple, si les certificats de serveur principal sont émis par une autorité de certification connue et a un nom commun contoso.com et que le de paramètres HTTP du serveur principal est également défini sur contoso.com, aucune étape supplémentaire n’est nécessaire. Vous pouvez définir le paramètre du protocole HTTP du serveur principal sur HTTPS et le contrôle d’intégrité et le chemin des données seront de ce fait compatibles TLS. Si vous utilisez Azure App Service ou d’autres services web Azure en tant que back-end, ceux-ci sont également implicitement approuvés et aucune étape supplémentaire n’est nécessaire pour le protocole TLS de bout en bout.
   
> [!NOTE] 
>
> Pour qu’un certificat TLS/SSL soit approuvé, ce certificat du serveur principal doit avoir été émis par une autorité de certification connue. Si le certificat n’a pas été émis par une autorité de certification approuvée, le service Application Gateway vérifie si le certificat de l’autorité de certification émettrice a été émis par une autorité de certification approuvée, et ainsi de suite jusqu’à ce qu’une autorité de certification approuvée soit détectée (auquel cas une connexion sécurisée est établie) ou qu’aucune autorité de certification approuvée ne soit trouvée (auquel cas le service Application Gateway marque le serveur principal comme étant non sain). Par conséquent, il est recommandé que le certificat de serveur principal contienne à la fois les autorités de certification racines et intermédiaires.

- Si le certificat est auto-signé ou signé par des intermédiaires inconnus, un certificat racine approuvé doit être défini pour pouvoir activer le protocole TLS de bout en bout dans la référence SKU v2. Le service Application Gateway communique uniquement avec des serveurs principaux dont le certificat racine du certificat de serveur correspond à une des entrées de la liste des certificats racines approuvés dans le paramètre HTTP de serveur principal associé au pool.

- En plus de la concordance du certificat racine, le service Application Gateway v2 vérifie si le paramètre Host spécifié dans le paramètre HTTP de serveur principal correspond à celui du nom commun présenté par le certificat TLS/SSL du serveur principal. Lorsque vous tentez d’établir une connexion TLS au serveur principal, le service Application Gateway v2 définit l’extension SNI d’indication du nom de serveur sur l’hôte spécifié dans le paramètre HTTP de serveur principal.

- Si vous choisissez l’option **Choisir le nom d’hôte à partir de la cible back-end** au lieu du champ Hôte dans le paramètre HTTP du serveur back-end, l’en-tête SNI est toujours défini sur le nom de domaine complet du pool back-end, et le nom commun dans le certificat TLS/SSL du serveur back-end doit correspondre à son nom de domaine complet. Les membres du pool principal avec des adresses IP ne sont pas pris en charge dans ce scénario.

- Le certificat racine est un des certificats de serveur principal codés en base64.

## <a name="sni-differences-in-the-v1-and-v2-sku"></a>Différences de SNI dans les références (SKU) v1 et v2

Comme mentionné précédemment, le service Application Gateway met fin au trafic TLS en provenance du client au niveau de l’écouteur d’Application Gateway (appelons-le connexion frontale), déchiffre le trafic, applique les règles nécessaires pour déterminer le serveur principal auquel la demande doit être transférée, et établit une nouvelle session TLS avec le serveur principal (appelons-la connexion principale).

Les tableaux suivants décrivent les différences de SNI entre les références (SKU) v1 et v2 en termes de connexions frontale et principale.

### <a name="frontend-tls-connection-client-to-application-gateway"></a>Connexion TLS frontale (du client au service Application Gateway)

---
Scénario | v1 | v2 |
| --- | --- | --- |
| Si le client spécifie un en-tête SNI et que tous les écouteurs multisites sont activés avec l’indicateur « Exiger SNI » | Retourne le certificat approprié et, si le site n’existe pas (selon le nom_serveur), la connexion est réinitialisée. | Retourne un certificat approprié s’il en est un de disponible, ou retourne le certificat du premier écouteur HTTPS configuré (dans l’ordre).|
| Si le client ne spécifie pas d’en-tête SNI et si tous les en-têtes multisites sont activés avec l’indicateur « Exiger SNI » | Réinitialise la connexion. | Retourne le certificat du premier écouteur HTTPS configuré (dans l’ordre).
| Si le client ne spécifie pas d’en-tête SNI et si un écouteur de base est configuré avec un certificat | Retourne le certificat configuré dans l’écouteur de base du client (certificat par défaut ou de secours). | Retourne le certificat du premier écouteur HTTPS configuré (dans l’ordre). |

### <a name="backend-tls-connection-application-gateway-to-the-backend-server"></a>Connexion TLS principale (du service Application Gateway au serveur principal)

#### <a name="for-probe-traffic"></a>Pour le trafic de la sonde

---
Scénario | v1 | v2 |
| --- | --- | --- |
| En-tête SNI (nom_serveur) durant la poignée de main TLS en tant que nom de domaine complet (FQDN) | Défini comme nom de domaine complet (FQDN) à partir du pool principal. En vertu de la norme [RFC 6066](https://tools.ietf.org/html/rfc6066), la présence d’adresses IPv4 et IPv6 littérales n’est pas autorisée dans un nom d’hôte SNI. <br> **Remarque :** Le nom de domaine complet (FQDN) dans le pool principal doit être résolu par DNS en adresse IP (publique ou privée) du serveur principal. | L’en-tête SNI (nom_serveur) est défini en tant que nom d’hôte à partir de la sonde personnalisée attachée aux paramètres HTTP (s’ils sont configurés), ou bien à partir du nom d’hôte mentionné dans les paramètres HTTP, ou encore à partir du nom de domaine complet (FQDN) mentionné dans le pool principal. L’ordre de priorité est Sonde personnalisée > Paramètres HTTP > Pool principal. <br> **Remarque :** Si les noms d’hôte configurés dans les paramètres HTTP et la sonde personnalisée diffèrent, en fonction de la priorité, l’en-tête SNI est défini en tant que nom d’hôte à partir de la sonde personnalisée.
| Si l’adresse du pool principal est une adresse IP (v1) ou si le nom d’hôte de la sonde personnalisée est configuré en tant qu’adresse IP (v2) | L’en-tête SNI (nom_serveur) n’est pas défini. <br> **Remarque :** Dans ce cas, le serveur back-end doit pouvoir retourner un certificat par défaut/de secours, et doit être mis en liste verte dans les paramètres HTTP sous le certificat d’authentification. Si aucun certificat par défaut/de secours n’est configuré sur le serveur principal, et si l’en-tête SNI est attendu, le serveur peut réinitialiser la connexion et entraîner des échecs de sonde. | Dans l’ordre de priorité mentionné précédemment, s’ils ont une adresse IP en tant que nom d’hôte, l’en-tête SNI n’est pas défini conformément à la norme [RFC 6066](https://tools.ietf.org/html/rfc6066). <br> **Remarque :** L’en-tête SNI n’est pas non plus défini dans des sondes v2 si aucune sonde personnalisée n’est configurée et qu’aucun nom d’hôte n’est défini sur les paramètres HTTP ou le pool principal. |

> [!NOTE] 
> Si aucune sonde personnalisée n’est configurée, Application Gateway envoie une sonde par défaut au format suivant : \<protocol\>://127.0.0.1:\<port\>/. Par exemple, pour une sonde HTTPS par défaut, elle est envoyée en tant que https://127.0.0.1:443/. Notez que l’adresse 127.0.0.1 mentionnée ici est utilisée uniquement comme en-tête d’hôte HTTP et, conformément à la norme RFC 6066, n’est pas utilisée en tant qu’en-tête SNI. Pour plus d’informations sur les erreurs de sonde d’intégrité, consultez le [Guide de résolution des problèmes d’intégrité du serveur principal](application-gateway-backend-health-troubleshooting.md).

#### <a name="for-live-traffic"></a>Pour le trafic en direct

---
Scénario | v1 | v2 |
| --- | --- | --- |
| En-tête SNI (nom_serveur) durant la poignée de main TLS en tant que nom de domaine complet (FQDN) | Défini comme nom de domaine complet (FQDN) à partir du pool principal. En vertu de la norme [RFC 6066](https://tools.ietf.org/html/rfc6066), la présence d’adresses IPv4 et IPv6 littérales n’est pas autorisée dans un nom d’hôte SNI. <br> **Remarque :** Le nom de domaine complet (FQDN) dans le pool principal doit être résolu par DNS en adresse IP (publique ou privée) du serveur principal. | L’en-tête SNI (nom_serveur) est défini en tant que nom d’hôte à partir des paramètres HTTP. Autrement, si *option PickHostnameFromBackendAddress* est sélectionnée ou si aucun nom d’hôte n’est mentionné, il est défini comme nom de domaine complet (FQDN) dans la configuration du pool principal.
| Si l’adresse du pool principal est une adresse IP ou si aucun nom d’hôte n’est défini dans les paramètres HTTP | L’en-tête SNI n’est pas défini conformément à la norme [RFC 6066](https://tools.ietf.org/html/rfc6066) si l’entrée du pool principal n’est pas un nom de domaine complet (FQDN). | L’en-tête SNI est défini en tant que nom d’hôte à partir du nom de domaine complet (FQDN) d’entrée du client, et le nom commun du certificat principal doit correspondre à ce nom d’hôte.

## <a name="next-steps"></a>Étapes suivantes

Après avoir découvert le chiffrement TLS de bout en bout, accédez à [Configurer le TLS de bout en bout avec Application Gateway avec PowerShell](application-gateway-end-to-end-ssl-powershell.md) pour créer une passerelle d’application à l’aide du chiffrement TLS de bout en bout.

<!--Image references-->

[1]: ./media/ssl-overview/scenario.png