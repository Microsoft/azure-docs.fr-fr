---
title: Menaces - Outil Microsoft de modélisation des menaces - Azure | Documents Microsoft
description: Page de catégories des menaces pour l’outil Microsoft de modélisation des menaces, contenant les catégories de toutes les menaces générées exposées.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/17/2017
ms.author: rodsan
ms.openlocfilehash: eb006482b851e9094b82ec3d0753b74c05296994
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68727829"
---
# <a name="microsoft-threat-modeling-tool-threats"></a>Menaces de l’outil Microsoft de modélisation des menaces

L’outil de modélisation des menaces est un élément essentiel de Microsoft Security Development Lifecycle (SDL). Il permet aux architectes logiciels d’identifier et de corriger les problèmes de sécurité potentiels au plus tôt, lorsqu’ils sont relativement simples et peu coûteux à résoudre. Par conséquent, il réduit considérablement le coût total de développement. En outre, nous avons conçu cet outil en pensant aux personnes qui ne sont pas expertes en sécurité et nous avons facilité la modélisation des menaces pour tous les développeurs en fournissant des conseils éclairés sur la création et l’analyse des modèles de menace.

> Visitez **[l’outil de modélisation des menaces](threat-modeling-tool.md)** pour démarrer dès aujourd'hui !

L’outil de modélisation des menaces vous aide à répondre à certaines questions, par exemple :

* Comment une personne malveillante peut-elle modifier les données d’authentification ?
* Quelles sont les répercussions lorsqu’une personne malveillante est en mesure de lire les données de profil utilisateur ?
* Que se passe-t-il si l’accès à la base de données de profils utilisateur est refusé ?

## <a name="stride-model"></a>Modèle STRIDE

Pour vous aider à formuler ces questions spécifiques, Microsoft utilise le modèle STRIDE qui classifie les différents types de menaces et simplifie les conversations de sécurité globale.

| Category | Description |
| -------- | ----------- |
| **Usurpation d’identité** | Implique l’accès illégal aux informations d’authentification d’un autre utilisateur, comme le nom d’utilisateur et le mot de passe, ainsi que leur utilisation |
| **Falsification** | Implique la modification malveillante de données. On peut citer par exemple des modifications non autorisées apportées aux données persistantes, comme celles contenues dans une base de données, et l’altération de données circulant entre deux ordinateurs sur un réseau ouvert, tel qu’Internet |
| **Répudiation** | Associée aux utilisateurs refusant d’effectuer une action sans que d’autres parties aient un moyen de prouver le contraire : par exemple, un utilisateur effectue une opération illégale dans un système qui ne permet pas de tracer les opérations interdites. La non-répudiation fait référence à la capacité d’un système à contrer les menaces de répudiation. Par exemple, un utilisateur qui achète un article peut être obligé de donner sa signature à la réception de l’article. Le fournisseur peut ensuite utiliser l’accusé de réception signé comme preuve de réception du colis par l’utilisateur |
| **Divulgation d’informations** | Implique l’exposition d’informations à des personnes qui ne devraient pas pour avoir accès à ce service, par exemple, la capacité des utilisateurs à lire un fichier auxquels ils n’ont pas été autorisés à accéder, ou la capacité d’un intrus à lire les données en transit entre deux ordinateurs |
| **Déni de service** | Les attaques par déni de service (DoS) refusent un service aux utilisateurs valides, par exemple en rendant un serveur web temporairement indisponible ou inutilisable. Vous devez vous protéger contre certains types de menaces DoS, simplement pour améliorer la fiabilité et la disponibilité du système |
| **Élévation de privilège** | Un utilisateur sans privilège obtient un accès privilégié et donc suffisant pour compromettre ou détruire tout le système. Les menaces d’élévation de privilège incluent les situations dans lesquelles une personne malveillante a percé toutes les défenses du système et fait désormais partie intégrante du système de confiance proprement dit : cette situation est très dangereuse |

## <a name="next-steps"></a>Étapes suivantes

Reportez-vous à **[Atténuations de l’outil de modélisation des menaces](threat-modeling-tool-mitigations.md)** pour découvrir les différentes méthodes pour atténuer ces menaces avec Azure.