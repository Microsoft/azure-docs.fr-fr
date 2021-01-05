---
title: Fichier include
description: fichier
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
author: emuench
ms.author: mingshen
ms.date: 10/09/2020
ms.openlocfilehash: a1bc7cf1fd339ca3660c7b39326f37d2763c74b2
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92283077"
---
Si vous avez besoin d’une configuration supplémentaire, utilisez une tâche planifiée exécutée au démarrage pour apporter des modifications finales à la machine virtuelle après sont déploiement. Tenez également compte des éléments suivants :

- S’il s’agit d’une tâche à exécution unique, la tâche doit se supprimer une fois qu’elle s’est terminée avec succès.
- Les configurations ne doivent pas s’appuyer sur des lecteurs autres que C ou D, car seuls ces deux lecteurs sont toujours présents (le lecteur C est le disque du système d’exploitation et le lecteur D est le disque local temporaire).

Pour plus d’informations sur les personnalisations Linux, consultez l’article [Extensions et fonctionnalités de machine virtuelle pour Linux](../../virtual-machines/extensions/features-linux.md).