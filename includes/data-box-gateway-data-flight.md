---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: e177fabdad7a1517a8371fe1d3483cb5c9310d2c
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580970"
---
Pour les données à la volée :

- Standard TLS 1.2 est utilisé pour les données qui transitent entre l’appareil et Azure. Il n’existe aucun protocole de secours pour TLS 1.1 et versions antérieures. Les communications de l’agent seront bloquées si TLS 1.2 n’est pas pris en charge. TLS 1.2 est également requis pour la gestion du portail et du Kit de développement logiciel (SDK).
- Lorsque les clients accèdent à votre appareil via l’interface utilisateur web locale d’un navigateur, Standard TLS 1.2 est utilisé comme protocole sécurisé par défaut.

    - La meilleure pratique consiste à configurer votre navigateur pour utiliser TLS 1.2.
    - Si le navigateur ne prend pas en charge TLS 1.2, vous pouvez utiliser TLS 1.1 ou TLS 1.0.
- Nous vous recommandons d’utiliser SMB 3.0 avec chiffrement pour protéger les données lorsque vous les copiez depuis vos serveurs de données.
