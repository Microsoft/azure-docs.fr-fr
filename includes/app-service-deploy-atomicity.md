---
title: Fichier Include
description: Fichier Include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 01/07/2020
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: feed8b21833d4244d027d64d5e6547b94e4fa66f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96004338"
---
## <a name="what-happens-to-my-app-during-deployment"></a>Que se passe-t-il au niveau de mon application pendant le déploiement ?

Toutes les méthodes de déploiement officiellement prises en charge apportent des modifications aux fichiers dans le dossier `/home/site/wwwroot` de votre application. Ces fichiers sont utilisés pour exécuter votre application. Par conséquent, le déploiement peut échouer si des fichiers sont verrouillés. L’application peut également se comporter de façon imprévisible pendant le déploiement, car tous les fichiers ne sont pas tous mis à jour en même temps. Cela n’est pas souhaitable pour une application faisant face au client. Il existe différentes manières d’éviter ces problèmes :

- [Exécutez votre application directement à partir du package ZIP](../articles/app-service/deploy-run-package.md) sans l’en extraire.
- Arrêtez votre application ou activez le mode hors connexion pendant le déploiement. Pour plus d’informations, consultez [Gérer les fichiers verrouillés au cours du déploiement](https://github.com/projectkudu/kudu/wiki/Dealing-with-locked-files-during-deployment).
- Effectuez le déploiement sur un [emplacement de préproduction](../articles/app-service/deploy-staging-slots.md) avec [échange automatique](../articles/app-service/deploy-staging-slots.md#configure-auto-swap) activé. 
