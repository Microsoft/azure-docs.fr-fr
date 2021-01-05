---
title: Fichier Include
description: Fichier Include
services: active-directory
author: msaburnley
ms.service: active-directory
ms.topic: include
ms.date: 10/15/2019
ms.author: ajburnle
ms.custom: include file
ms.openlocfilehash: e971af5d415ea788f97ab11b58541e629c7eb0ed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76694615"
---
## <a name="lifecycle"></a>Cycle de vie

Dans l’onglet **Cycle de vie**, vous spécifiez l’expiration de l’attribution du package d’accès à un utilisateur. Vous pouvez également spécifier si les utilisateurs peuvent étendre leurs attributions.

1. Dans la section **Expiration**, définissez **L’affectation de package d’accès expire** sur **Date**, **Nombre de jours** ou **Jamais**.

    Pour **Date**, sélectionnez une date d’expiration.

    Pour **Nombre de jours**, spécifiez un nombre de jours compris entre 0 et 3 660.

    Selon votre sélection, l’attribution du package d’accès à un utilisateur expire à une date définie, un nombre de jours définis après approbation ou n’expire jamais.

1. Cliquez sur **Afficher les paramètres avancés d’expiration** pour afficher plus de paramètres.

    ![Package d’accès - Paramètres d’expiration du cycle de vie](./media/active-directory-entitlement-management-lifecycle-policy/expiration.png)

1. Pour autoriser un utilisateur à étendre son attribution, définissez **Autoriser un utilisateur à étendre son accès** sur **Oui**.

    Si les extensions sont autorisées dans la stratégie, l’utilisateur recevra un e-mail 14 jours ainsi qu’un jour avant l’expiration de son attribution l’invitant à étendre l’attribution. Si l’utilisateur envoie une demande d’extension de l’accès, la date de l’extension doit être au plus tard la date d’expiration des affectations définie dans la stratégie utilisée pour accorder à l’utilisateur l’accès au package d’accès. Par exemple, si la stratégie indique que les affectations sont configurées pour expirer le 30 juin, l’extension maximale qu’un utilisateur peut demander est le 30 juin.

    Si l’accès d’un utilisateur est étendu, il ne pourra pas demander le package d’accès après la date d’extension spécifiée (date définie dans le fuseau horaire de l’utilisateur qui a créé la stratégie).

1. Pour demander une approbation pour accorder une extension, définissez **Exiger une approbation pour accorder l'extension** sur **Oui**.

    Les mêmes paramètres d’approbation que ceux spécifiés sous l’onglet **Requêtes** seront utilisés.

1. Cliquez sur **Suivant** ou **Mettre à jour**.
