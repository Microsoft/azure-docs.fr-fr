---
title: Connecter les tâches Stream Analytics aux ressources d’un réseau virtuel Azure (VNET)
description: Cet article explique comment connecter une tâche Azure Stream Analytics à des ressources qui se trouvent dans un réseau virtuel.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 01/04/2021
ms.custom: devx-track-js
ms.openlocfilehash: 4701cb4122b4196b08b2a427b34d49c7784b91a7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/29/2021
ms.locfileid: "98878237"
---
# <a name="connect-stream-analytics-jobs-to-resources-in-an-azure-virtual-network-vnet"></a>Connecter les tâches Stream Analytics aux ressources d’un réseau virtuel Azure (VNet)

Vos tâches Stream Analytics créent des connexions sortantes avec vos ressources Azure d’entrée et de sortie, afin de traiter les données en temps réel et de générer des résultats. Ces ressources d’entrée et de sortie (par exemple, Azure Event Hubs et Azure SQL Database) peuvent se trouver derrière un pare-feu Azure ou dans un réseau virtuel Azure (VNet). Le service Stream Analytics fonctionne à partir de réseaux qui ne peuvent pas être inclus directement dans vos règles de réseau.

Toutefois, il existe deux façons de connecter en toute sécurité vos tâches Stream Analytics à vos ressources d’entrée et de sortie dans de tels scénarios.
* À l’aide de points de terminaison privés dans des clusters Stream Analytics.
* À l’aide du mode d’authentification Identité managée couplé avec le paramètre de mise en réseau « Autoriser les services de confiance ».

Votre tâche Stream Analytics n’accepte aucune connexion entrante.

## <a name="private-endpoints-in-stream-analytics-clusters"></a>Points de terminaison privés dans des clusters Stream Analytics.
Les [clusters Stream Analytics](./cluster-overview.md) sont des clusters de calcul dédiés à un seul locataire, dans lesquels vous pouvez exécuter vos tâches Stream Analytics. Vous pouvez créer des points de terminaison privés managés dans votre cluster Stream Analytics, ce qui permet à toutes les tâches en cours d’exécution sur votre cluster d’établir une connexion sortante sécurisée avec vos ressources d’entrée et de sortie.

La création de points de terminaison privés dans votre cluster Stream Analytics est une [opération en deux étapes](./private-endpoints.md). Cette option est idéale pour les charges de travail de diffusion en continu de taille moyenne à grande, car la taille minimale d’un cluster Stream Analytics est de 36 unités de diffusion en continu (même si les 36 unités de diffusion en continu peuvent être partagées par différentes tâches dans différents abonnements ou environnements tels que le développement, le test et la production).

## <a name="managed-identity-authentication-with-allow-trusted-services-configuration"></a>Authentification d’identité managée avec la configuration « Autoriser les services de confiance »
Certains services Azure proposent le paramètre de mise en réseau **Autoriser les services Microsoft approuvés** qui, lorsqu’il est activé, permet à vos tâches Stream Analytics de se connecter en toute sécurité à votre ressource à l’aide d’une authentification forte. Cette option vous permet de connecter vos tâches à vos ressources d’entrée et de sortie sans avoir besoin d’un cluster Stream Analytics ni de points de terminaison privés. La configuration de votre tâche pour utiliser cette technique est une opération en deux étapes :
* Utilisez le mode d’authentification Identité managée lors de la configuration de l’entrée ou de la sortie dans votre tâche Stream Analytics.
* Accordez à vos tâches Stream Analytics spécifiques un accès explicite à vos ressources cibles en attribuant un rôle Azure à l’identité managée de la tâche, telle qu’affectée par le système. 

L’activation du paramètre **Autoriser les services Microsoft approuvés** n’accorde pas d’accès permanent à une tâche. Elle vous permet de contrôler totalement quelles tâches Stream Analytics spécifiques peuvent accéder à vos ressources en toute sécurité. 

Grâce à cette technique, vos tâches peuvent se connecter aux services Azure suivants :
1. [Stockage Blob ou Azure Data Lake Storage Gen2](./blob-output-managed-identity.md) : il peut s’agir du compte de stockage de votre tâche, d’une entrée ou d’une sortie de diffusion en continu.
2. [Azure Event Hubs](./event-hubs-managed-identity.md) : il peut s’agir de l’entrée ou de la sortie de diffusion en continu de votre tâche.

Si vos travaux doivent se connecter à d’autres types d’entrée ou de sortie, vous pouvez d’abord écrire de Stream Analytics vers la sortie Event Hubs, puis vers la destination de votre choix avec Azure Functions. Si vous souhaitez écrire directement de Stream Analytics vers d’autres types de sortie sécurisés dans un réseau virtuel ou un pare-feu, la seule possibilité consiste à utiliser des points de terminaison privés dans des clusters Stream Analytics.

## <a name="next-steps"></a>Étapes suivantes

* [Créer et supprimer des points de terminaison privés dans des clusters Stream Analytics](./private-endpoints.md)
* [Se connecter à Event Hubs dans un réseau virtuel à l’aide de l’authentification Identité managée](./event-hubs-managed-identity.md)
* [Se connecter à un stockage Blob et à ADLS Gen2 dans un réseau virtuel à l’aide de l’authentification Identité managée](./blob-output-managed-identity.md)