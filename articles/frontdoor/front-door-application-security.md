---
title: Azure Front Door – Sécurité de la couche Application | Microsoft Docs
description: Cet article explique comment Azure Front Door permet de protéger et de sécuriser vos back-ends d’application.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 4ee50b4c7da27df3630c1b4d263f076da44189bc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89399937"
---
# <a name="application-layer-security-with-front-door"></a>Sécurité de la couche Application avec Front Door
Azure Front Door offre une fonction de protection des applications web contre les attaques réseau et les types courants de code malveillant exploitant une faille de sécurité web, comme l’injection de code SQL le cross-site scripting (XSS). Activée pour les frontends HTTP(S), la sécurité de la couche Application de Front Door est distribuée mondialement et toujours activée, ce qui arrête les attaques malveillantes au niveau du périmètre de réseau, loin de vos backends. Avec une sécurité accrue et une optimisation des performances, Front Door offre des expériences web rapides et sécurisées pour vos utilisateurs finaux.

## <a name="application-protection"></a>Protection des applications
La protection des applications par Front Door est configurée sur chaque environnement Edge dans le monde entier, conformément aux applications, et empêche automatiquement le trafic autre que HTTP(S) d’atteindre vos applications web. Notre architecture distribuée multi-locataire permet une protection globale à grande échelle sans sacrifier les performances. Pour les charges de travail HTTP(S), le service de protection des applications web de Front Door fournit un moteur de règles enrichi pour les règles personnalisées, un ensemble de règles préconfiguré contre les attaques courantes et une journalisation détaillée pour toutes les requêtes qui correspondent à une règle. Des actions flexibles, notamment autoriser, bloquer ou journaliser uniquement, sont prises en charge.

## <a name="custom-access-control-rules"></a>Règles de contrôle d’accès personnalisées
- **Listes d’adresses IP autorisées et bloquées :** vous pouvez configurer des règles personnalisées pour contrôler l’accès à vos applications web en fonction d’une liste d’adresses IP de clients. IP v4 et IP v6 sont pris en charge
- **Contrôle d’accès en fonction de l’emplacement géographique :** vous pouvez configurer des règles personnalisées pour contrôler l’accès à vos applications web en fonction du code de pays d’où provient une adresse IP de client.
- **Filtrage des paramètres HTTP :** vous pouvez configurer des règles d’accès personnalisées basées sur la correspondance de paramètres de requête HTTP(S), notamment les en-têtes, l’URL et les chaînes de requête.

## <a name="azure-managed-rules"></a>Règles gérées par Azure
- Un ensemble préconfiguré de règles par rapport à aux principales vulnérabilités OWASP courantes est activé par défaut. Dans la préversion, l’ensemble de règles inclut le contrôle des requêtes sqli et xss. Des règles supplémentaires seront ajoutées. Vous pouvez choisir de commencer avec l’action « journaliser uniquement » pour valider que les règles préconfigurées fonctionnent comme prévu pour vos applications. 

## <a name="rate-limiting"></a>Limitation du débit
- Une règle de contrôle du débit consiste à limiter un trafic élevé anormal en provenance de toute adresse IP de client.  Vous pouvez définir un seuil sur le nombre de requêtes web autorisées par une adresse IP de client pendant une durée d’une minute.

## <a name="centralized-protection-policy"></a>Stratégie de protection centralisée
- Vous pouvez définir plusieurs règles de protection et les ajouter à une stratégie par ordre de priorité. Les règles personnalisées ont une priorité plus élevée qu’un ensemble de règles géré pour autoriser les exceptions. Une seule stratégie est associée à votre application web.  La même stratégie de protection des applications web est répliquée sur tous les serveurs Edge à tous les emplacements, afin de garantir une stratégie de sécurité cohérente dans toutes les régions.

## <a name="configuration"></a>Configuration
- Pendant la préversion, vous pouvez utiliser des API REST, PowerShell ou l’interface CLI pour créer et déployer des stratégies et des règles de protection des applications de Front Door. L’accès au portail sera pris en charge avant la mise à disposition générale du service. 


## <a name="monitoring"></a>Surveillance
Front Door permet de superviser les applications web contre les attaques à l’aide de métriques en temps réel qui sont intégrées à Azure Monitor pour assurer le suivi des alertes et superviser facilement les tendances.

## <a name="pricing"></a>Tarifs
La sécurité de la couche Application de Front Door est gratuite pendant la préversion.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).
