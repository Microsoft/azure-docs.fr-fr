---
title: Fichier Include
description: Fichier Include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 12/18/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 8c5c0c8f649e7cbab2c16688717de1aaabfb93c5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75474111"
---
Cette référence détaille les objectifs de performance et d’extensibilité du stockage Azure. Les objectifs d’extensibilité et de performances répertoriés ici sont des objectifs haut de gamme mais réalisables. Dans tous les cas, le taux de demande et la bande passante atteints par votre compte de stockage dépendent de la taille des objets stockés, des modèles d’accès utilisés et du type de charge de travail de votre application.

Veillez à tester votre service afin de déterminer si ses performances répondent à vos besoins. Dans la mesure du possible, évitez les pics soudains de trafic et assurez-vous que le trafic est bien réparti sur toutes les partitions.

Lorsque votre application atteint la limite de gestion d’une partition concernant la charge de travail, Stockage Azure commence à renvoyer des codes d’erreur 503 (Serveur occupé) ou 500 (Délai d’expiration de l’opération). Si vous rencontrez des erreurs 503, nous vous recommandons de modifier votre application pour utiliser une stratégie d’interruption exponentielle pour les nouvelles tentatives. L’interruption exponentielle diminue la charge sur la partition et atténue les pics de trafic pour cette partition.
