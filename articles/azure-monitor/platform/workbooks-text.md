---
title: Paramètres texte des classeurs Azure Monitor
description: Créer des rapports complexes en toute simplicité grâce à des classeurs paramétrables prédéfinis et personnalisés. Apprenez-en davantage sur les paramètres de texte de classeur.
services: azure-monitor
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 0ac4c348b776cee2356f77424698b078947f4ad3
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143557"
---
# <a name="workbook-text-parameters"></a>Paramètres texte de classeur

Les paramètres de zone de texte offrent un moyen simple de collecter des entrées de texte à partir des utilisateurs du classeur. Ils sont utilisés lorsqu’il n’est pas pratique d’utiliser un menu déroulant pour collecter l’entrée (par exemple, un seuil arbitraire ou des filtres génériques). Les classeurs permettent aux auteurs d’obtenir la valeur par défaut de la zone de texte à partir d’une requête. Cela permet d’expérimenter des scénarios intéressants, tels que la définition du seuil par défaut en fonction du p95 de la métrique.

Les zones de texte sont couramment utilisées en tant que variables internes par d’autres contrôles de classeur. Cela s’effectue en tirant parti d’une requête pour les valeurs par défaut et en masquant le contrôle d’entrée en mode lecture. Par exemple, un utilisateur peut souhaiter qu’un seuil provienne d’une formule (et non d’un utilisateur), puis utiliser le seuil dans les requêtes suivantes.

## <a name="creating-a-text-parameter"></a>Création d’un paramètre de texte
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `SlowRequestThreshold`
    2. Type de paramètre : `Text`
    3. Obligatoire : `checked`
    4. Obtenir la valeur par défaut de la requête : `unchecked`
5. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

    ![Image représentant la création d’un paramètre de texte](./media/workbooks-text/text-create.png)

Voici à quoi ressemble le classeur en mode lecture.

![Image représentant un paramètre de texte en mode lecture](./media/workbooks-text/text-readmode.png)

## <a name="referencing-a-text-parameter"></a>Référencement d’un paramètre de texte
1. Ajoutez un contrôle de requête au classeur en sélectionnant le lien bleu `Add query`, puis sélectionnez une ressource Application Insights.
2. Dans la zone KQL, ajoutez l’extrait de code suivant :
    ```kusto
    requests
    | summarize AllRequests = count(), SlowRequests = countif(duration >= {SlowRequestThreshold}) by name
    | extend SlowRequestPercent = 100.0 * SlowRequests / AllRequests
    | order by SlowRequests desc
    ```
3. En utilisant le paramètre text avec la valeur 500 associée au contrôle de requête, vous exécutez la requête ci-dessous :
    ```kusto
    requests
    | summarize AllRequests = count(), SlowRequests = countif(duration >= 500) by name
    | extend SlowRequestPercent = 100.0 * SlowRequests / AllRequests
    | order by SlowRequests desc
    ```
4. Exécuter la requête pour afficher les résultats

    ![Image présentant un paramètre de texte référencé dans KQL](./media/workbooks-text/text-reference.png)

> [!NOTE]
> Dans l’exemple ci-dessus, `{SlowRequestThreshold}` représente une valeur entière. Si vous interrogez une chaîne comme `{ComputerName}`, vous devez modifier votre requête Kusto pour ajouter des guillemets `"{ComputerName}"` afin que le champ de paramètre accepte une entrée sans guillemets.

## <a name="setting-default-values"></a>Définir les valeurs par défaut
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `SlowRequestThreshold`
    2. Type de paramètre : `Text`
    3. Obligatoire : `checked`
    4. Obtenir la valeur par défaut de la requête : `checked`
5. Dans la zone KQL, ajoutez l’extrait de code suivant :
    ```kusto
    requests
    | summarize round(percentile(duration, 95), 2)
    ```
    Cette requête définit la valeur par défaut de la zone de texte sur la durée du 95e centile de toutes les requêtes dans l’application.
6. Exécuter la requête pour afficher le résultat
7. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

    ![Image représentant un paramètre de texte avec la valeur par défaut de KQL](./media/workbooks-text/text-default-value.png)

> [!NOTE]
> Bien que cet exemple interroge les données d’Application Insights, l’approche peut être utilisée pour n’importe quelle source de données basée sur les journaux - Log Analytics, Azure Resource Graph, etc.

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.