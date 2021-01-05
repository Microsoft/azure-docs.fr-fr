---
title: En-têtes HTTP spécifiques à Verizon pour le moteur de règles Azure CDN | Microsoft Docs
description: Cet article décrit comment utiliser les en-têtes HTTP spécifiques à Verizon avec le moteur de règles Azure CDN.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2018
ms.author: allensu
ms.openlocfilehash: e20f6ce9540d357b61ae2cfdf0e8f96d127dc6c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84343215"
---
# <a name="verizon-specific-http-headers-for-azure-cdn-rules-engine"></a>En-têtes HTTP spécifiques à Verizon pour le moteur de règles Azure CDN

Pour les produits **Azure CDN Premium de Verizon**, lorsqu’une requête HTTP est envoyée au serveur d’origine, le serveur de point de présence (POP) peut ajouter un ou plusieurs en-têtes réservés (ou en-têtes spéciaux de proxy) dans la requête client au POP. Ces en-têtes s’ajoutent aux en-têtes de transfert standard reçus. Pour plus d’informations sur les en-têtes de requête standard, consultez [Champs de la requête](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields).

Si vous souhaitez éviter que l’un de ces en-têtes réservés ne soit ajouté dans la requête POP d’Azure CDN (Content Delivery Network) au serveur d’origine, vous devez créer une règle avec la [fonctionnalité En-têtes spéciaux de proxy](https://docs.vdms.com/cdn/Content/HRE/F/Proxy-Special-Headers.htm) dans le moteur de règles. Dans cette règle, excluez l’en-tête que vous souhaitez supprimer de la liste par défaut des en-têtes dans le champ d’en-têtes. Si vous avez activé la [fonctionnalité En-têtes de réponse de cache de débogage](https://docs.vdms.com/cdn/Content/HRE/F/Debug-Cache-Response-Headers.htm), veillez à ajouter les en-têtes `X-EC-Debug` nécessaires. 

Par exemple, pour supprimer l’en-tête `Via`, le champ d’en-têtes de la règle doit inclure la liste suivante d’en-têtes : *X-Forwarded-For, X-Forwarded-Proto, X-Host, X-Midgress, X-Gateway-List, X-EC-Name, Host*. 

![Règle En-têtes spéciaux de proxy](./media/cdn-http-headers/cdn-proxy-special-header-rule.png)

Le tableau suivant décrit les en-têtes qui peuvent être ajoutés par le POP du CDN Verizon dans la requête :

En-tête de requête | Description | Exemple
---------------|-------------|--------
[Via](#via-request-header) | Identifie le serveur POP ayant traité par proxy la requête à un serveur d’origine. | HTTP/1.1 ECS (dca/1A2B)
X-Forwarded-For | Indique l’adresse IP du demandeur.| 10.10.10.10
X-Forwarded-Proto | Indique le protocole de la requête. | http
X-Host | Indique le nom d’hôte de la requête. | cdn.mydomain.com
X-Midgress | Indique si la requête a été traitée par proxy via un serveur CDN supplémentaire. Par exemple, un serveur de protection serveur à origine POP ou un serveur de passerelle serveur à ADN POP. <br />Cet en-tête est ajouté à la requête uniquement lorsque le trafic midgress a lieu. Dans ce cas, l’en-tête est défini sur 1 pour indiquer que la requête a été traitée par proxy via un serveur CDN supplémentaire.| 1
[Hôte](#host-request-header) | Identifie l’hôte et le port où le contenu demandé est disponible. | marketing.mydomain.com:80
[X-Gateway-List](#x-gateway-list-request-header) | ADN : identifie la liste de basculement des serveurs de passerelle ADN affectés à une origine client. <br />Origin Shield (Protection d’origine) : indique l’ensemble des serveurs de protection d’origine affectés à une origine client. | `icn1,hhp1,hnd1`
X-EC- _&lt;name&gt;_ | Les en-têtes de requête qui commencent par *X-EC* (par exemple, X-EC-Tag, [X-EC-Debug](cdn-http-debug-headers.md)) sont réservés pour une utilisation par le CDN.| waf-production

## <a name="via-request-header"></a>En-tête de requête Via
Le format par le biais duquel l’en-tête de la requête `Via` identifie un serveur POP est spécifié par la syntaxe suivante :

`Via: Protocol from Platform (POP/ID)` 

Les termes utilisés dans la syntaxe sont définis comme suit :
- Protocol : indique la version du protocole (par exemple, HTTP/1.1) utilisé pour traiter la requête par proxy. 

- Platform : indique la plateforme sur laquelle le contenu a été demandé. Les codes suivants sont valides pour ce champ : 

    Code | Plateforme
    -----|---------
    ECAcc | HTTP Large
    ECS   | HTTP Small
    ECD   | ADN (Application delivery network)

- POP : indique le [POP](cdn-pop-abbreviations.md) qui a traité la requête. 

- ID : à usage interne uniquement.

### <a name="example-via-request-header"></a>Exemple d’en-tête de requête Via

`Via: HTTP/1.1 ECD (dca/1A2B)`

## <a name="host-request-header"></a>En-tête de requête Host
Les serveurs POP remplaceront l’en-tête `Host` lorsque les deux conditions suivantes sont remplies :
- La source du contenu demandé est un serveur d’origine client.
- L’option HTTP Host Header (En-tête Host HTTP) de l’origine client correspondante n’est pas vide.

L’en-tête de la requête `Host` sera remplacé pour refléter la valeur définie dans l’option En-tête Host HTTP.
Si l’option En-tête Host HTTP de l’origine client est vide, alors l’en-tête de la requête `Host` qui est soumis par le demandeur est transmis au serveur d’origine du client.

## <a name="x-gateway-list-request-header"></a>En-tête de requête X-Gateway-List
Un serveur POP ajoute/remplace l’en-tête de requête « X-Gateway-List » lorsqu’une des conditions suivantes est remplie :
- La requête pointe vers la plateforme ADN.
- La requête est envoyée à un serveur d’origine client qui est protégé par la fonctionnalité Protection d’origine.

