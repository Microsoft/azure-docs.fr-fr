---
title: Configuration des écouteurs Azure Application Gateway
description: Cet article explique comment configurer les écouteurs Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: 15f68e8cbca65e7b970944f7ca5ef1952140cc6b
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397637"
---
# <a name="application-gateway-listener-configuration"></a>Configuration des écouteurs Application Gateway

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Un écouteur est une entité logique qui vérifie les demandes de connexion entrante en utilisant le port, le protocole, l’hôte et l’adresse IP. Quand vous configurez l’écouteur, vous devez entrer des valeurs qui correspondent aux valeurs indiquées dans la demande entrante sur la passerelle.

Quand vous créez une passerelle d’application à l’aide du portail Azure, vous créez également un écouteur par défaut en choisissant le protocole et le port pour l’écouteur. Vous pouvez choisir d’activer ou non la prise en charge du protocole HTTP2 sur l’écouteur. Une fois que vous avez créé la passerelle d’application, vous pouvez modifier les paramètres de cet écouteur par défaut ( *appGatewayHttpListener* ) ou créer des écouteurs.

## <a name="listener-type"></a>Type d’écouteur

Quand vous créez un écouteur, vous choisissez entre le type [*de base* et le type *multisite*](./application-gateway-components.md#types-of-listeners).

- Si vous souhaitez que toutes vos demandes (pour tous les domaines) soient acceptées et transmises aux pools back-end, choisissez le type « de base ». Découvrez [comment créer une passerelle d’application avec un écouteur de base](./quick-create-portal.md).

- Si vous souhaitez transférer les requêtes HTTP vers différents pools principaux en fonction de l’en-tête d’ *hôte* ou des noms d’hôte, choisissez le type d’écouteur « multisite » ; dans ce cas, vous devez également spécifier un nom d’hôte qui correspond à la demande entrante. En effet, Application Gateway s’appuie sur des en-têtes d’hôte HTTP 1.1 pour héberger plusieurs sites web sur la même adresse IP publique et le même port. Pour en savoir plus, consultez [Hébergement de plusieurs sites à l’aide d’Application Gateway](multiple-site-overview.md).

### <a name="order-of-processing-listeners"></a>Ordre de traitement des

Pour la référence SKU v1, les demandes sont mises en correspondance en fonction de l’ordre des règles et du type d’écouteur. Si une règle avec un écouteur de base occupe la première place, elle est traitée en premier et accepte toutes les demandes pour ce port et cette combinaison d’adresses IP. Pour éviter cela, configurez d’abord les règles avec des écouteurs multisites et poussez la règle avec l’écouteur de base vers la dernière position de liste.

Pour la référence SKU v2, les écouteurs multisites sont traités avant les écouteurs de base.

## <a name="front-end-ip-address"></a>Adresse IP front-end

Choisissez l’adresse IP front-end que vous prévoyez d’associer à cet écouteur. L’écouteur écoute les demandes entrantes sur cette adresse IP.

## <a name="front-end-port"></a>Port front-end

Choisissez le port front-end. Sélectionnez un port existant ou créez-en un. Choisissez une valeur dans la [plage de ports autorisée](./application-gateway-components.md#ports). Vous pouvez utiliser non seulement les ports connus, comme les ports 80 et 443, mais aussi tout port personnalisé autorisé qui convient. Un port peut être utilisé pour des écouteurs publics ou privés.

## <a name="protocol"></a>Protocol

Choisissez HTTP ou HTTPS :

- Si vous choisissez HTTP, le trafic entre le client et la passerelle d’application est non chiffré.

- Sélectionnez HTTPS si vous voulez [un arrêt TLS](features.md#secure-sockets-layer-ssltls-termination) ou un [chiffrement TLS de bout en bout](./ssl-overview.md). Le trafic entre le client et la passerelle d’application est chiffré. Et la connexion TLS s’arrête à la passerelle d’application. Si vous voulez un chiffrement TLS de bout en bout, vous devez choisir le protocole HTTPS et configurer le paramètre **HTTP du back-end**. Ce dernier permet de veiller à ce que le trafic soit rechiffré quand il passe de la passerelle d’application au back-end.


Pour configurer l’arrêt TLS et le chiffrement TLS de bout en bout, vous devez ajouter un certificat à l’écouteur pour permettre à la passerelle d’application de dériver une clé symétrique. Il s’agit d’une exigence de la spécification du protocole TLS. La clé symétrique sert à chiffrer et déchiffrer le trafic envoyé à la passerelle. Le certificat de passerelle doit être au format Personal Information Exchange (PFX). Ce format vous permet d’exporter la clé privée que la passerelle utilise pour chiffrer et déchiffrer le trafic.

## <a name="supported-certificates"></a>Certificats pris en charge

Voir [Présentation de la terminaison TLS et du chiffrement TLS de bout en bout avec Application Gateway](ssl-overview.md#certificates-supported-for-tls-termination)

## <a name="additional-protocol-support"></a>Prise en charge d’autres protocoles

### <a name="http2-support"></a>Prise en charge de HTTP2

La prise en charge du protocole HTTP/2 est disponible pour les clients se connectant aux écouteurs de passerelle d’application uniquement. La communication avec les pools de serveurs back-end s’effectue par le biais du portocole HTTP/1.1. Par défaut, la prise en charge du protocole HTTP/2 est désactivée. L’extrait de code Azure PowerShell suivant montre comment activer cette prise en charge :

```azurepowershell
$gw = Get-AzApplicationGateway -Name test -ResourceGroupName hm

$gw.EnableHttp2 = $true

Set-AzApplicationGateway -ApplicationGateway $gw
```

### <a name="websocket-support"></a>Prise en charge de WebSocket

La prise en charge de WebSocket est activée par défaut. Il n’existe aucun paramètre configurable par l’utilisateur pour l’activer ou la désactiver. Vous pouvez utiliser des WebSockets avec des écouteurs HTTP et HTTPS.

## <a name="custom-error-pages"></a>Pages d’erreur personnalisées

Vous pouvez définir une erreur personnalisée au niveau global ou au niveau de l’écouteur. Mais la création de pages d’erreur personnalisées au niveau global à partir du portail Azure n’est actuellement pas prise en charge. Vous pouvez configurer une page d’erreur personnalisée pour une erreur de pare-feu d’application web 403 ou une page de maintenance 502 au niveau de l’écouteur. Vous devez également spécifier une URL d’objet blob accessible publiquement pour le code d’état d’erreur donné. Pour plus d’informations, consultez [Créer des pages d’erreur personnalisées Application Gateway](./custom-error.md).

![Codes d’erreur Application Gateway](/azure/application-gateway/media/custom-error/ag-error-codes.png)

Pour configurer une page d’erreur personnalisée globale, consultez [Configuration Azure PowerShell](./custom-error.md#azure-powershell-configuration).

## <a name="tls-policy"></a>Stratégie de protocole TLS

Vous pouvez centraliser la gestion des certificats TLS/SSL et réduire la surcharge de chiffrement-déchiffrement d’une batterie de serveurs back-end. Cette gestion TLS centralisée permet également de spécifier une stratégie TLS centrale adaptée à vos besoins de sécurité. Vous pouvez choisir une stratégie TLS *par défaut* , *prédéfinie* ou *personnalisée*.

Vous configurez la stratégie TLS pour contrôler les versions du protocole TLS. Vous pouvez configurer une passerelle d’application afin qu’elle utilise une version de protocole minimale pour les négociations TLS à partir de TLS 1.0, TLS 1.1 et TLS 1.2. Par défaut, SSL 2.0 et 3.0 sont désactivés et ne sont pas configurables. Pour plus d’informations, consultez [Vue d’ensemble de la stratégie TLS Application Gateway](./application-gateway-ssl-policy-overview.md).

Après avoir créé un écouteur, vous l’associez à une règle de routage des demandes. Cette règle détermine la manière dont les demandes reçues sur l’écouteur sont routées vers le back-end.

## <a name="next-steps"></a>Étapes suivantes

- [Découvrir les règles de routage des requêtes](configuration-request-routing-rules.md).