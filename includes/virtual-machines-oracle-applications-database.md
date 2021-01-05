---
author: dlepow
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 06/01/2020
ms.author: danlep
ms.openlocfilehash: b9e275cbdaf264df3d0a40f032693fd93553e520
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84317705"
---
### <a name="database-tier"></a>Couche base de données

La couche de base de données contient les instances de base de données pour l’application. La base de données peut être un système Oracle DB, Oracle RAC ou Oracle Exadata Database. 

Si vous choisissez d’utiliser Oracle DB, l’instance de base de données peut être déployée sur Azure via les images Oracle DB disponibles sur la place de marché Azure. Vous pouvez également utiliser l’interconnexion entre Azure et OCI pour déployer la base de données Oracle dans un modèle PaaS sur OCI.

Pour Oracle RAC, vous pouvez utiliser OCI dans le modèle PaaS. Nous vous recommandons d’utiliser un système RAC à deux nœuds. Bien qu’il soit possible de déployer Oracle RAC sur Azure CloudSimple dans le modèle IaaS, il ne s’agit pas d’une configuration prise en charge par Oracle. Reportez-vous aux [programmes Oracle éligibles pour connaître les environnements cloud autorisés](http://www.oracle.com/us/corporate/pricing/authorized-cloud-environments-3493562.pdf).

Enfin, pour les systèmes Exadata, utilisez l’interconnexion OCI et déployez le système Exadata dans OCI. Le diagramme d’architecture ci-dessus montre un système Exadata déployé dans OCI sur deux sous-réseaux.

Pour les scénarios de production, déployez plusieurs instances de la base de données sur deux zones de disponibilité (si vous déployez dans Azure) ou deux domaines de disponibilité (dans OCI). Utilisez Oracle Active Data Guard pour synchroniser les bases de données principale et de secours.

La couche de base de données reçoit uniquement les requêtes de la couche intermédiaire. Il est recommandé de configurer un groupe de sécurité réseau (liste de sécurité si vous déployez la base de données dans OCI) pour autoriser uniquement les demandes sur le port 1521 à partir de la couche intermédiaire et sur le port 22 du serveur bastion pour des raisons administratives.

Pour les bases de données déployées dans OCI, un réseau de cloud virtuel distinct doit être configuré avec une passerelle de routage dynamique (DRG) connectée à votre circuit FastConnect.