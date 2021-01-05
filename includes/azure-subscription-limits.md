---
title: Fichier Include
description: Fichier include
author: rothja
ms.service: azure-resource-manager
ms.topic: include
ms.date: 05/18/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: 74f9c5304237ec77a629bc914d95c345882b784f
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95554532"
---
| Ressource | Limite par défaut | Limite maximale |
| --- | --- | --- |
| Processeurs virtuels par [abonnement](https://azure.microsoft.com/pricing/)<sup>1</sup> |20 |10 000 |
| [Coadministrateurs](../articles/cost-management-billing/manage/add-change-subscription-administrator.md) par abonnement |200 |200 |
| [Comptes de stockage](../articles/storage/common/storage-account-create.md) par abonnement<sup>2</sup> |100 |100 |
| [Services cloud](../articles/cloud-services/cloud-services-choose-me.md) par abonnement |20 |200 |
| [Réseaux locaux](/previous-versions/azure/reference/jj157100(v=azure.100)) par abonnement |10 |500 |
| Serveurs DNS par abonnement |9 |100 |
| Adresses IP réservées par abonnement |20 |100 |
| [Groupes d'affinités](/previous-versions/azure/virtual-network/virtual-networks-migrate-to-regional-vnet) par abonnement |256 |256 |
| Longueur de nom d’abonnement (caractères) | 64 | 64 |

<sup>1</sup>Les très petites instances comptent comme un processeur virtuel pour la limite de processeurs virtuels, malgré l’utilisation d’un cœur de processeur partiel.

<sup>2</sup>La limite du compte de stockage comprend les comptes de stockage Standard et Premium.