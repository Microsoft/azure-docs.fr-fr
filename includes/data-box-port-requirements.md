---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 09/06/2020
ms.author: alkohli
ms.openlocfilehash: b9ff5968b4bb406f1a96780985b5c6fe64ca976c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89505900"
---
| N° de port| Entrant ou sortant | Étendue de ports| Obligatoire| Notes |
|--------|-----|-----|-----------|----------|
| TCP 80 (HTTP)|Dans|LAN|Oui|Ce port est utilisé pour se connecter aux API REST de stockage Data Box Blob sur HTTP. Si vous ne vous connectez pas à des API REST, la connexion est automatiquement redirigée vers l’interface utilisateur web locale sur le port 8443. |
| TCP 443 (HTTPS)|Dans|LAN|Oui|Ce port est utilisé pour se connecter aux API REST de stockage Data Box Blob sur HTTPS. Si vous ne vous connectez pas à des API REST, la connexion est automatiquement redirigée vers l’interface utilisateur web locale sur le port 8443. |
| TCP 8443 (HTTPS-Alt)|Dans|LAN|Oui|Il s’agit d’un autre port pour le protocole HTTPS. il est utilisé lors de la connexion à l’interface utilisateur web locale pour la gestion des appareils. |
| TCP 445 (SMB)|Sortie/Entrée|LAN|Dans certains cas<br>Voir les remarques|Ce port est requis seulement si vous vous connectez via SMB. |
| TCP 2049 (NFS)|Sortie/Entrée|LAN|Dans certains cas<br>Voir les remarques|Ce port est requis seulement si vous vous connectez via NFS. |
| TCP 111 (NFS)|Sortie/Entrée|LAN|Dans certains cas<br>Voir les remarques|Ce port est utilisé pour le mappage rpcbind/port et requis uniquement si vous vous connectez via NFS.  |
