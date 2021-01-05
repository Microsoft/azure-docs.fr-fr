---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/30/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: e3ad8fcb6cd5cf68b02ac522d0e5ef5a18ba8a3c
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93144991"
---
Les connexions d’authentification par certificat Azure natif de point à site utilisent les éléments suivants, que vous pouvez configurer dans cet exercice :

* Une passerelle VPN RouteBased.
* La clé publique (fichier .cer) d’un certificat racine, chargée sur Azure. Une fois le certificat chargé, il est considéré comme un certificat approuvé et est utilisé pour l’authentification.
* Un certificat client généré à partir du certificat racine. Le certificat client installé sur chaque ordinateur client qui se connecte au réseau virtuel. Ce certificat est utilisé pour l’authentification du client.
* Une configuration de client VPN. Le client VPN est configuré à l’aide des fichiers config du client VPN. Ces fichiers contiennent les informations nécessaires permettant au client de se connecter au réseau virtuel. Les fichiers configurent le client VPN existant qui est natif au système d’exploitation. Chaque client qui se connecte doit être configuré à l’aide des paramètres dans les fichiers de configuration.
