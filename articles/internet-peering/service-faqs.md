---
title: Peering Service - FAQ
titleSuffix: Azure
description: Peering Service - FAQ
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: reference
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 507e503b881df123ffc6694c53b0e9cc9b6a8872
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75774024"
---
# <a name="peering-service---faqs"></a>Peering Service - FAQ

Vous pouvez consulter les informations ci-dessous si vous avez des questions d’ordre général.

**Un opérateur peut-il utiliser son Peering direct existant avec Microsoft pour prendre en charge Peering Service ?**

Oui, un opérateur peut tirer parti de son infrastructure réseau de Peering existante pour prendre en charge Peering Service. Une infrastructure Peering Service nécessite une diversité pour prendre en charge la haute disponibilité. Si l’infrastructure de Peering existante a déjà une diversité, aucune nouvelle infrastructure n’est requise. Si l’infrastructure de Peering existante a besoin de la diversité, elle peut être augmentée.

**Un opérateur peut-il utiliser un nouveau Peering direct avec Microsoft pour prendre en charge Peering Service ?**

Oui, c’est également possible. Microsoft collaborera avec l’opérateur pour créer un nouveau Peering direct afin de prendre en charge Peering Service.  

**Pourquoi le Peering direct est-il obligatoire pour prendre en charge Peering Service ?**

L’un des principaux objectifs de Peering Service consiste à fournir une connectivité aux services en ligne de Microsoft par le biais d’un fournisseur de services correctement connecté. Les infrastructures réseau de Peering sont toujours dans la plage de Gbits/s, et constituent donc un élément fondamental pour une connectivité à haut débit entre les opérateurs et Microsoft.

**Quelles sont les exigences en matière de diversité pour qu’un Peering direct prenne en charge Peering Service ?**

Une infrastructure de Peering doit prendre en charge la redondance locale et la géoredondance. La redondance locale est définie comme deux ensembles différents de chemins dans un site de Peering donné. La géoredondance exige que l’opérateur dispose d’une connectivité supplémentaire sur un site de périphérie Microsoft différent en cas de défaillance du site principal. Pendant la courte durée de défaillance, l’opérateur peut router le trafic par le biais du site de secours.

**L’opérateur offre déjà un contrat de niveau de service (SLA) et une solution Internet de qualité professionnelle ; en quoi cette offre est-elle différente ?**

Certains opérateurs proposent un contrat SLA et une solution Internet de qualité professionnelle sur leur partie du réseau. Avec Peering Service, Microsoft offre un contrat SLA pour le trafic sur la partie Microsoft du réseau. La sélection de Peering Service permet au client de bénéficier d’un contrat SLA de bout en bout. Le contrat SLA de son site vers la périphérie Microsoft sur le réseau de l’ISP peut être couvert par l’ISP. Le contrat SLA sur le Réseau Microsoft mondial de la périphérie Microsoft vers l’application des utilisateurs finaux est désormais couvert par Microsoft.

**Si un fournisseur de services a déjà établi un Peering avec Microsoft à l’aide d’une infrastructure de Peering, quel type de modification est nécessaire pour prendre en charge Peering Service ?**

* Des modifications logicielles permettant d’identifier un utilisateur de Peering Service et son trafic. Il peut être nécessaire de modifier la stratégie de routage pour échanger le trafic d’un utilisateur à la périphérie Microsoft la plus proche par le biais de la connexion Peering Service.
* Assurez-vous que la connectivité dispose d’une redondance locale et d’une géoredondance.
