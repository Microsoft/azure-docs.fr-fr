---
title: Incidents Azure Security Center – corrélations intelligentes d’alertes
description: Cette rubrique explique comment la fusion utilise la corrélation des alertes intelligentes cloud pour générer des incidents de sécurité dans Azure Security Center.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: e9d5a771-bfbe-458c-9a9b-a10ece895ec1
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/02/2019
ms.author: memildin
ms.openlocfilehash: 742d7122c60b383e8ab7aa73f73b11b47843a9d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91613818"
---
# <a name="cloud-smart-alert-correlation-in-azure-security-center-incidents"></a>Corrélation des alertes intelligentes cloud dans Azure Security Center (incidents)

Azure Security Center analyse en continu les charges de travail de cloud hybride en utilisant une analytique avancée et des informations de Threat Intelligence pour vous alerter sur les activités malveillantes.

La couverture contre les menaces s’étend progressivement. Face à la nécessité de détecter jusqu’à la moindre compromission, il peut s’avérer compliqué pour les analystes de sécurité de trier les différentes alertes et d’identifier une attaque réelle. Security Center aide les analystes à faire face à la charge de travail liée aux alertes. Il facilite le diagnostic des attaques en temps réel, en mettant en corrélation les différentes alertes et les signaux de faible fidélité en incidents de sécurité.

L’analytique de fusion constitue le back-end technologique et analytique sur lequel reposent les incidents Security Center, permettant la mise en corrélation des différentes alertes et signaux contextuels. La fusion examine les différents signaux relevés sur un abonnement parmi les ressources. La fusion recherche des modèles qui révèlent la progression des attaques ou les signaux avec des informations contextuelles partagées, indiquant que vous devez utiliser une procédure de réponse unifiée pour eux.

L’analytique de fusion combine les connaissances dans le domaine de la sécurité avec l’intelligence artificielle pour analyser les alertes, ce qui permet de découvrir les nouveaux modèles d’attaque à mesure qu’ils apparaissent. 

Security Center s’appuie sur MITRE Attack Matrix pour associer les alertes à leur intention perçue, facilitant la formalisation des connaissances dans le domaine de la sécurité. De plus, en utilisant les informations collectées à chaque étape d’une attaque, Security Center peut écarter les activités qui semblent être des étapes d’une attaque, mais qui ne le sont pas en réalité.

Étant donné que les attaques se produisent souvent sur différents locataires, Security Center peut combiner des algorithmes d’intelligence artificielle pour analyser les séquences d’attaque signalées dans chaque abonnement. Cette technique identifie les séquences d’alerte comme des modèles d’alerte prédominants au lieu de simplement les associer les unes aux autres incidemment.

Dans le cadre d’une investigation d’un incident, les analystes ont souvent besoin de plus de contexte pour aboutir à un verdict sur la nature de la menace et sur les moyens de l’atténuer. Par exemple, même si une anomalie de réseau a été détectée, sans connaître les autres événements qui se produisent sur le réseau ou en lien avec la ressource ciblée, il est difficile de déterminer les actions à prendre. Pour vous aider, un incident de sécurité peut inclure des artefacts, des événements connexes et des informations. Les informations supplémentaires disponibles pour les incidents de sécurité varient en fonction du type de menace détecté et de la configuration de votre environnement. 

> [!TIP]
> Pour obtenir la liste des alertes d’incident de sécurité qui peuvent être générées par l’analytique de fusion, consultez la [table de référence des alertes](alerts-reference.md#alerts-fusion).

:::image type="content" source="./media/security-center-alerts-cloud-smart/security-incident.png" alt-text="Capture d’écran du rapport d’incident de sécurité détecté":::


Pour mieux comprendre les incidents de sécurité, consultez le [Guide pratique pour gérer les incidents de sécurité dans Azure Security Center](security-center-incident.md).

