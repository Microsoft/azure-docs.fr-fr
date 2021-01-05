---
title: Fichier include
description: Fichier include
services: api-management
author: vladvino
ms.assetid: 1b813833-39c8-46be-8666-fd0960cfbf04
ms.service: api-management
ms.topic: include
ms.date: 04/14/2020
ms.author: vlvinogr
ms.custom: include file
ms.openlocfilehash: cf051d96651161658796689639b5d268230d4b5a
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559026"
---
| Ressource | Limite |
| ---------------------------------------------------------------------- | -------------------------- |
| Nombre maximal d’unités d’échelle | 10 par région<sup>1</sup> |
| Taille du cache | 5 Gio par unité<sup>2</sup> |
| Connexions back-ends simultanées<sup>3</sup> par autorité HTTP | 2 048 par unité<sup>4</sup> |
| Taille maximale de la réponse mise en cache | 2 Mio |
| Taille maximale du document de stratégie | 256 Kio<sup>5</sup> |
| Nombre maximal de domaines de passerelle personnalisés par instance de service<sup>6</sup> | 20 |
| Nombre maximal de certificats d’autorité de certification par instance de service<sup>7</sup> | 10 |
| Nombre maximal d’instances de service par abonnement<sup>8</sup> | 20 |
| Nombre maximal d’abonnements par instance de service<sup>8</sup> | 500 |
| Nombre maximal de certificats clients par instance de service<sup>8</sup> | 50 |
| Nombre maximal d’API par instance de service<sup>8</sup> | 50 |
| Nombre maximal d’opérations d’API par instance de service<sup>8</sup> | 1 000 |
| Durée totale maximale des requêtes<sup>8</sup> | 30 secondes |
| Taille maximale de la charge utile mise en mémoire tampon<sup>8</sup> | 2 Mio |
| Taille maximale de l’URL de requête<sup>9</sup> | 4096 octets |
| Nombre maximal de passerelles auto-hébergées<sup>10</sup> | 25 |

<sup>1</sup>Les limites de mise à l’échelle varient selon le niveau tarifaire. Pour plus de détails sur les niveaux tarifaires et leurs limites de mise à l’échelle, consultez [Tarification de la gestion des API](https://azure.microsoft.com/pricing/details/api-management/).<br/>
<sup>2</sup>La taille du cache par unité varie selon le niveau tarifaire. Pour consulter les niveaux tarifaires et leurs limites de mise à l’échelle, voir [Tarification de la gestion des API](https://azure.microsoft.com/pricing/details/api-management/).<br/>
<sup>3</sup>Les connexions sont regroupées et réutilisées, sauf si elles sont explicitement fermées par le back-end.<br/>
<sup>4</sup>Cette limite est appliquée par unité des niveaux De base, Standard et Premium. Le niveau Développeur est limité à 1 024. Cette limite ne s’applique pas au niveau Consommation.<br/>
<sup>5</sup>Cette limite est appliquée aux niveaux De base, Standard et Premium. Dans le niveau Consommation, la taille des documents de stratégie est limitée à 16 Kio.<br/>
<sup>6</sup>Les domaines personnalisés multiples sont pris en charge uniquement dans les niveaux Développeur et Premium.<br/>
<sup>7</sup>Les certificats d’autorité de certification ne sont pas pris en charge dans le niveau Consommation.<br/>
<sup>8</sup>Cette limite s’applique au niveau Consommation uniquement. Il n’existe aucune limite dans ces catégories pour les autres niveaux.<br/>
<sup>9</sup> S’applique au niveau Consommation uniquement. Comprend une chaîne de requête longue allant jusqu’à 2048 octets.<br/>
<sup>10</sup>Les passerelles auto-hébergées sont prises en charge uniquement dans les niveaux Développeur et Premium. Cette limite s’applique au nombre de [ressources des passerelles auto-hébergées](/rest/api/apimanagement/2019-12-01/gateway). Pour augmenter cette limite, contactez le [support](https://azure.microsoft.com/support/options/). Notez que le nombre de nœuds (ou réplicas) associés à une ressource de passerelle auto-hébergée est illimité dans le niveau Premium, mais limité à un seul nœud dans le niveau Développeur.