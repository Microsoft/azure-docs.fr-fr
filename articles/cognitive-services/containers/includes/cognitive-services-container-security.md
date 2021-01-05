---
title: Sécurité du conteneur
titleSuffix: Azure Cognitive Services
description: Apprendre à sécuriser votre conteneur
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/11/2020
ms.author: aahi
ms.openlocfilehash: bb9b0da609169288521d21ee6d5e412a786c7549
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94573864"
---
## <a name="azure-cognitive-services-container-security"></a>Sécurité des conteneurs Azure Cognitive Services

Quand vous développez des applications, la sécurité doit être l’un de vos principaux soucis. L’importance que vous accordez à la sécurité est un indicateur de réussite. Quand vous concevez une solution logicielle qui comprend des conteneurs Cognitive Services, il est essentiel de comprendre les limitations et les capacités disponibles. Pour plus d’informations sur la sécurité réseau, voir [Configurer des réseaux virtuels Azure Cognitive Services][az-security].

> [!IMPORTANT]
> Par défaut, l’API du conteneur Cognitive Services *n’est pas sécurisée*. Cela est dû au fait que le conteneur s’exécute le plus souvent dans le cadre d’un pod qui est protégé de l’extérieur par un pont réseau. Toutefois, il est possible d’activer l’authentification, celle-ci fonctionnant de la même façon que l’authentification utilisée lors de l’accès aux [services Cognitive Services basés sur le cloud][request-authentication].

Le diagramme ci-dessous illustre l’approche **non sécurisée** par défaut :

![Sécurité du conteneur](../media/container-security.svg)

Une autre approche *sécurisée* pour les consommateurs de conteneurs Cognitive Services consiste à ajouter un composant frontal à un conteneur afin de maintenir le point de terminaison de conteneur privé. Prenons l’exemple d’un scénario dans lequel nous utilisons [Istio][istio] comme passerelle d’entrée. Istio prend en charge HTTPS/TLS et l’authentification par certificat client. Dans ce scénario, le composant frontal Istio expose l’accès au conteneur et présente le certificat client qui est préalablement approuvé avec Istio.

[Nginx][nginx] est un autre choix populaire dans la même catégorie. Istio et Nginx jouent le rôle de maillage de service et offrent des fonctionnalités supplémentaires, notamment l’équilibrage de charge, le routage et le contrôle du débit.

### <a name="container-networking"></a>Mise en réseau de conteneurs

Les conteneurs Cognitive Services sont requis pour envoyer des informations relatives aux compteurs à des fins de facturation. Si vous n’ajoutez pas à la liste verte les différents canaux réseau sur lesquels s’appuient les conteneurs Cognitive Services, le conteneur ne fonctionnera pas.

#### <a name="allow-list-cognitive-services-domains-and-ports"></a>Ajouter à la liste verte les domaines et ports Cognitive Services

L’hôte doit ajouter à la liste verte le **port 443** et les domaines suivants :

* `*.cognitive.microsoft.com`
* `*.cognitiveservices.azure.com`

#### <a name="disable-deep-packet-inspection"></a>Désactiver l’inspection approfondie des paquets

> L’[inspection approfondie des paquets](https://en.wikipedia.org/wiki/Deep_packet_inspection) est un type de traitement des données qui inspecte en détail les données envoyées sur un réseau informatique et prend généralement une mesure en bloquant, en reroutant ou en enregistrant ces données en conséquence.

Désactivez l’inspection approfondie des paquets sur les canaux sécurisés que les conteneurs Cognitive Services créent sur des serveurs Microsoft. Dans le cas contraire, le conteneur ne fonctionnera pas correctement.

[istio]: https://istio.io/
[nginx]: https://www.nginx.com
[request-authentication]: ../../authentication.md
[az-security]: ../../cognitive-services-virtual-networks.md
