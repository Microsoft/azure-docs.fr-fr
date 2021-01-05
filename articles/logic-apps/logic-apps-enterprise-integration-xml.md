---
title: Messages XML et fichiers plats
description: Traiter, valider et transformer des messages XML dans Azure Logic Apps avec Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 02/27/2017
ms.openlocfilehash: 3cb3472572abbd891f8d36ea036b371c1224f38c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74792153"
---
# <a name="xml-messages-and-flat-files-in-azure-logic-apps-with-enterprise-integration-pack"></a>Messages XML et fichiers plats dans Azure Logic Apps avec Enterprise Integration Pack

Dans [Azure Logic Apps](logic-apps-overview.md), vous pouvez traiter les messages XML que vous envoyez et recevez à l’aide d’Enterprise Integration Pack. Si vous utilisiez BizTalk Server, Enterprise Integration Pack fournit des capacités similaires pour transformer et valider des messages, utiliser des fichiers plats et même XPath pour enrichir ou extraire des propriétés spécifiques d’un message. Si vous découvrez cet environnement, ces fonctionnalités étendent les possibilités de traitement des messages dans le workflow de votre application logique. Par exemple, dans un scénario d’entreprise à entreprise (B2B) où vous travaillez avec des schémas XML spécifiques, vous pouvez utiliser Enterprise Integration Pack pour améliorer la façon dont votre entreprise traite ces messages.

Par exemple, Enterprise Integration Pack comprend les fonctionnalités suivantes :

* [Validation XML](logic-apps-enterprise-integration-xml-validation.md) : valider un message XML entrant ou sortant par rapport à un schéma spécifique.

* [Transformation XML](logic-apps-enterprise-integration-transform.md) : convertir ou personnaliser un message XML en fonction de vos exigences ou de celles d’un partenaire en utilisant des mappages.

* [Encodage de fichier plat et décodage de fichier plat](logic-apps-enterprise-integration-flatfile.md) : encoder ou décoder un fichier plat.

  Par exemple, SAP accepte et envoie des fichiers IDOC dans un format de fichier plat. De nombreuses plates-formes d’intégration créent des messages XML, y compris Logic Apps. Vous pouvez donc créer une application logique qui utilise l’encodeur de fichier plat afin de « convertir » des fichiers XML en fichiers plats.

* [XPath](workflow-definition-language-functions-reference.md#xpath) : enrichir un message et extraire des propriétés spécifiques du message. Les propriétés extraites peuvent ensuite servir à acheminer le message vers une destination ou un point de terminaison intermédiaire.

## <a name="sample"></a>Exemple

[Déployez une application logique pleinement opérationnelle](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-veter-pipeline) (exemple GitHub) à l’aide des fonctionnalités XML d’Azure Logic Apps.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)
