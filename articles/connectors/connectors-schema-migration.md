---
title: Migrer des applications vers le schéma le plus récent
description: Comment migrer des définitions JSON de workflow d’application logique vers la version la plus récente de schéma de langage de définition de workflow
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 08/25/2018
ms.openlocfilehash: 114b8b32d4abb1fd9b7e641625cd1b132470bafd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87281444"
---
# <a name="migrate-logic-apps-to-latest-schema-version"></a>Migrer des applications logiques vers la dernière version du schéma

Pour migrer vos applications logiques existantes vers le schéma le plus récent, procédez comme suit : 

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

2. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. Dans la barre d’outils, choisissez **Mettre à jour le schéma**.

   > [!NOTE]
   > Lorsque vous choisissez **Mettre à jour le schéma**, Azure Logic Apps exécute automatiquement les étapes de migration avec le code de sortie correspondant. Vous pouvez utiliser cette sortie pour mettre à jour la définition de votre application logique. Toutefois, veillez à respecter les meilleures pratiques comme décrit dans la section **Meilleures pratiques** suivante.

   ![Mettre à jour le schéma](./media/connectors-schema-migration/update-schema.png)

   La page Mettre à jour le schéma apparaît et affiche un lien vers un document qui décrit les améliorations incluses dans le nouveau schéma.

## <a name="best-practices"></a>Meilleures pratiques

Voici quelques-unes des meilleures pratiques pour la migration de vos applications logiques vers la dernière version du schéma :

* Copiez le script migré vers une nouvelle application logique. Ne remplacez pas l’ancienne version avant d’avoir terminé vos tests et vérifiez que votre application migrée fonctionne comme prévu.

* Testez votre application logique **avant** sa mise en production.

* Une fois la migration terminée, commencez à mettre à jour vos applications logiques pour utiliser les [API managées](../connectors/apis-list.md) lorsque cela est possible. Par exemple, commencez à utiliser Dropbox v2 partout où vous utilisez DropBox v1.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez à [migrer manuellement vos applications logiques](../logic-apps/logic-apps-schema-2016-04-01.md)

