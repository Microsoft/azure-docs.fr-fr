---
title: Conception d’Azure Functions pour une entrée identique
description: Mise en œuvre de l’idempotence dans Azure Functions
author: craigshoemaker
ms.author: cshoe
ms.date: 9/12/2019
ms.topic: article
ms.openlocfilehash: 15af60ac5a862e6fb20e65ba6fbb92482420b7c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74226874"
---
# <a name="designing-azure-functions-for-identical-input"></a>Conception d’Azure Functions pour une entrée identique

La réalité d’une architecture pilotée par les événements et basée sur les messages dicte la nécessité d’accepter des demandes identiques tout en préservant l’intégrité des données et la stabilité du système.

Pour illustrer cela, considérons un bouton d’appel d’ascenseur. Quand vous appuyez sur le bouton, il s’allume et un ascenseur est envoyé à votre étage. Quelques instants plus tard, une autre personne vous rejoint dans le hall d’entrée. Cette personne vous sourit et appuie une deuxième fois sur le bouton éclairé. Vous lui souriez en retour tout en ricanant dans votre fort intérieur en vous rappelant que la commande d’appel d’un ascenseur est idempotente.

Le fait d’appuyer sur un bouton d’appel d’ascenseur une deuxième, une troisième ou une quatrième fois n’a aucune incidence sur le résultat final. En appuyant sur le bouton, quel que soit le nombre de fois où vous appuyez, l’ascenseur est envoyé à votre étage. Les systèmes idempotents, tels que les ascenseurs, produisent le même résultat quel que soit le nombre de fois où des commandes identiques sont exécutées.

Lorsqu’il s’agit de créer des applications, tenez compte des scénarios suivants :

- Que se passe-t-il si votre application de contrôle des stocks tente de supprimer un même produit plusieurs fois ?
- Comment votre application de ressources humaines se comporte-t-elle face à plusieurs demandes de création d’un enregistrement d’employé pour la même personne ?
- Où passe votre argent si votre application bancaire obtient 100 demandes pour effectuer le même retrait ?

Il existe de nombreux contextes dans lesquels les demandes adressées à une fonction peuvent recevoir des commandes identiques. Voici quelques situations de ce type :

- Stratégies relatives aux nouvelles tentatives envoyant la même demande plusieurs fois
- Réexécution des commandes mises en cache pour l’application
- Erreurs d’application envoyant plusieurs demandes identiques

Pour protéger l’intégrité des données et du système, une application idempotente intègre une logique pouvant inclure les comportements suivants :

- Vérification de l’existence des données avant de tenter de les supprimer
- Vérification de l’existence des données avant de tenter d’exécuter une action de création
- Réconciliation d’une logique visant à créer une cohérence éventuelle dans les données
- Contrôles des accès concurrentiels
- Détection des doublons
- Validation de l’actualisation des données
- Logique de protection pour vérifier les données d’entrée

En définitive, l’idempotence est obtenue en garantissant qu’une action donnée est possible et qu’elle est exécutée une seule fois.
