---
title: Entonnoirs Azure Application Insights
description: Apprenez à utiliser les entonnoirs pour découvrir de quelle façon les clients interagissent avec votre application.
ms.topic: conceptual
author: NumberByColors
ms.author: daviste
ms.date: 07/17/2017
ms.reviewer: mbullwin
ms.openlocfilehash: a81c4f32385cf06238f36ca0162923f93d682807
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87323721"
---
# <a name="discover-how-customers-are-using-your-application-with-application-insights-funnels"></a>Découvrir comment les clients utilisent votre application avec les entonnoirs Application Insights

Comprendre l’expérience de vos utilisateurs est primordial pour votre entreprise. Si votre application implique plusieurs étapes, vous devez savoir si la plupart des clients vont au bout du processus, ou s’ils arrêtent celui-ci à un moment donné. La progression via une série d’étapes dans une application web est appelée *entonnoir*. Vous pouvez utiliser les entonnoirs Azure Application Insights pour obtenir des informations sur vos utilisateurs et suivre les taux de conversion étape par étape. 

## <a name="create-your-funnel"></a>Créer votre entonnoir
Avant de créer votre entonnoir, choisissez la question à laquelle vous souhaitez répondre. Par exemple, vous souhaitez peut-être savoir combien d’utilisateurs sont en train d’afficher la page d’accueil, de consulter le profil d’un client ou de créer un ticket. Dans cet exemple, les propriétaires de la société Fabrikam Fiber souhaitent connaître le pourcentage de clients créant correctement un ticket client.

Voici les étapes qu’ils suivent pour créer leur entonnoir.

1. Dans l’outil Entonnoirs Application Insights, sélectionnez **Nouveau**.
1. Dans le menu déroulant **Intervalle de temps**, sélectionnez **90 derniers jours**. Sélectionnez **Mes entonnoirs** ou **Entonnoirs partagés**.
1. Dans la liste déroulante **Étape 1**, sélectionnez **Index**. 
1. Dans la liste **Étape 2**, sélectionnez **Client**.
1. Dans la liste **Étape 3**, sélectionnez **Créer**.
1. Donnez un nom à l’entonnoir, puis sélectionnez **Enregistrer**.

La capture d’écran suivante montre un exemple du type de données que l’outil Entonnoirs génère. Les propriétaires de Fabrikam peuvent constater qu’au cours des 90 derniers jours, 54,3 pourcent de leurs clients ayant visité la page d’accueil ont créé un ticket client. Ils peuvent également constater que 2 700 de leurs clients sont parvenus à l’index à partir de la page d’accueil. Cela peut indiquer un problème d’actualisation.


![Capture d’écran de l’outil Entonnoirs avec des données](media/usage-funnels/funnel1.png)

### <a name="funnels-features"></a>Fonctionnalités des entonnoirs
La capture d’écran précédente inclut cinq zones mises en surbrillance. Il s’agit des fonctionnalités des entonnoirs. La liste suivante explique en détail chaque zone correspondante dans la capture d’écran :
1. Si votre application est échantillonnée, une bannière d’échantillonnage est affichée. La sélection de la bannière ouvre un volet contextuel, qui explique comment désactiver l’échantillonnage. 
2. Vous pouvez exporter votre entonnoir vers [Power BI](./export-power-bi.md).
3. Sélectionnez une étape pour plus d’informations sur la droite. 
4. Le graphique de conversion historique indique les taux de conversion au cours des 90 derniers jours. 
5. Comprenez mieux vos utilisateurs en accédant à l’outil correspondant. Vous pouvez utiliser des filtres à chaque étape. 

## <a name="next-steps"></a>Étapes suivantes
  * [Vue d’ensemble de l’utilisation](usage-overview.md)
  * [Utilisateurs, Sessions et Événements](usage-segmentation.md)
  * [Rétention](usage-retention.md)
  * [Classeurs](../platform/workbooks-overview.md)
  * [Ajouter du contexte utilisateur](usage-send-user-context.md)
  * [Exporter vers Power BI](./export-power-bi.md)

