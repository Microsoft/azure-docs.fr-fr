---
title: Connexion Azure Peering Service
description: Découvrez la connexion Microsoft Azure Peering Service
services: peering-service
author: derekolo
ms.service: peering-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Infrastructure-services
ms.date: 05/18/2020
ms.author: derekol
ms.openlocfilehash: 0540fa9c7a29456aaf343adce83509b2d0e390e4
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91398918"
---
# <a name="peering-service-connection"></a>Connexion Peering Service

Une connexion fait généralement référence à un ensemble d’informations logiques identifiant un Peering Service. Vous la définissez en spécifiant les attributs suivants :

- Nom logique
- Partenaire de connectivité
- Emplacement physique du client
- Préfixes d’adresses IP

Le client peut établir une ou plusieurs connexions en fonction des besoins. Une connexion est également utilisée en tant qu’unité de collecte de télémétrie. Par exemple, pour choisir des alertes de télémétrie, le client doit définir la connexion à superviser.

> [!Note]
> Lorsque vous vous inscrivez à Peering Service, nous analysons votre télémétrie Windows et Microsoft 365 afin de vous fournir des mesures de latence pour vos préfixes sélectionnés.
>Pour plus d’informations sur la télémétrie des connexions, reportez-vous à [Télémétrie des connexions Peering Service](connection-telemetry.md).
>

>## <a name="how-to-register-a-connection"></a>Comment inscrire une connexion ?

**Scénario** : supposons qu’une filiale est éparpillée dans différents emplacements géographiques, comme illustré dans la figure ci-dessous. Ici, le client doit fournir un nom logique, le nom du fournisseur de services, l’emplacement physique du client et des préfixes IP (appartenant au client ou alloués par le fournisseur de services) associés à une connexion unique. Ce processus doit être répété pour inscrire Peering Service pour des connexions géo-redondantes distinctes.

![Connexions géo-redondantes](./media/peering-service-connection/peering-service-connections.png)

> [!Note]
> La filtration de niveau d’état est prise en compte pour l’emplacement physique du client lorsque la connexion est géo-localisée aux États-Unis.
>

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir le processus pas à pas d’inscription de connexion Peering Service, consultez [Inscrire Peering Service à l’aide du portail Azure](azure-portal.md).

Pour en savoir plus sur la télémétrie des connexions Peering Service, consultez [Télémétrie des connexions Peering Service](connection-telemetry.md).

Pour mesurer les données de télémétrie, consultez [Mesurer les données de télémétrie des connexions](measure-connection-telemetry.md).
