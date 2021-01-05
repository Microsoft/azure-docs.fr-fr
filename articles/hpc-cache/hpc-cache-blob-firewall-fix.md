---
title: Contourner les paramètres du pare-feu de stockage
description: Un paramètre de pare-feu réseau de compte de stockage peut entraîner un échec lors de la création d’une cible de stockage Blob Azure dans Azure HPC Cache. Cet article fournit une solution de contournement pour la limitation jusqu’à ce qu’un correctif logiciel soit en place.
author: ekpgh
ms.service: hpc-cache
ms.topic: troubleshooting
ms.date: 11/7/2019
ms.author: v-erkel
ms.openlocfilehash: 6916c79e9110a88beff65d487fac72441382c2f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092368"
---
# <a name="work-around-blob-storage-account-firewall-settings"></a>Contourner les paramètres de pare-feu de compte de stockage Blob

Un paramètre particulier utilisé dans les pare-feu de compte de stockage peut faire échouer la création de la cible de stockage d’objets blob. L’équipe Azure HPC Cache travaille sur un correctif logiciel pour résoudre ce problème, mais vous pouvez le contourner en suivant les instructions de cet article.

Le paramètre de pare-feu qui autorise l’accès uniquement à partir de « réseaux sélectionnés » peut empêcher le cache de créer une cible de stockage d’objets blob. Cette configuration se trouve dans la page de paramètres **Pare-feu et réseaux virtuels** du compte de stockage.

Le problème est que le service de cache utilise un réseau virtuel de service masqué distinct des environnements des clients. Il n’est pas possible d’autoriser explicitement ce réseau à accéder à votre compte de stockage.

Quand vous créez une cible de stockage d’objets blob, le service de cache utilise ce réseau pour vérifier si le conteneur est vide ou non. Si le pare-feu n’autorise pas l’accès à partir du réseau masqué, la vérification échoue, ainsi que la création de la cible de stockage.

Pour contourner ce problème, changez temporairement les paramètres de votre pare-feu lors de la création de la cible de stockage :

1. Accédez à la page **Pare-feu et réseaux virtuels** du compte de stockage, puis définissez le paramètre « Autoriser l’accès depuis » sur **Tous les réseaux**.
1. Créez la cible de stockage d’objets blob dans votre instance Azure HPC Cache.
1. Une fois la cible de stockage créée avec succès, redéfinissez le paramètre de pare-feu du compte sur **Réseaux sélectionnés**.

Azure HPC Cache n’utilise pas le réseau virtuel de service pour accéder à la cible de stockage terminée.

Pour obtenir de l’aide sur cette solution de contournement, [contacter le support technique Microsoft](hpc-cache-support-ticket.md).
