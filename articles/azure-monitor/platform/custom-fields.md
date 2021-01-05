---
title: Champs personnalisés dans Azure Monitor (préversion) | Microsoft Docs
description: La fonction Champs personnalisés d’Azure Monitor vous permet de créer vos propres champs autorisant les recherches, à partir des enregistrements d’un espace de travail Log Analytics qui s’ajoutent aux propriétés d’un enregistrement collecté.  Cet article décrit la création d’un champ personnalisé et fournit une procédure détaillée avec un exemple d’événement.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/23/2019
ms.openlocfilehash: 496dab24f636c97e1c7b27b871e1fded9216277d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91448555"
---
# <a name="create-custom-fields-in-a-log-analytics-workspace-in-azure-monitor-preview"></a>Créer des champs personnalisés dans un espace de travail Log Analytics dans Azure Monitor (préversion)

> [!NOTE]
> Cet article décrit comment analyser les données texte dans un espace de travail Log Analytics au moment de leur collecte. Nous vous recommandons d’analyser les données texte dans un filtre de requête une fois qu’elles ont été collectées en suivant les instructions fournies dans [Analyser les données texte dans Azure Monitor](../log-query/parse-text.md). Cette méthode offre plusieurs avantages par rapport à l’utilisation de champs personnalisés.

> [!IMPORTANT]
> Les champs personnalisés augmentent la quantité de données collectées dans l’espace de travail Log Analytics, ce qui peut faire monter vos coûts. Pour plus d'informations, consultez [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](manage-cost-storage.md#pricing-model).

La fonction **Champs personnalisés** d’Azure Monitor vous permet de compléter les enregistrements existants dans votre espace de travail Log Analytics en leur ajoutant vos propres champs de recherche.  Les champs personnalisés sont renseignés automatiquement à partir des données extraites d’autres propriétés du même enregistrement.

![Le schéma montre un enregistrement d’origine associé à un enregistrement modifié dans un espace de travail Log Analytics avec des paires de valeurs de propriété ajoutées à la propriété d’origine dans l’enregistrement modifié.](media/custom-fields/overview.png)

Par exemple, l’enregistrement ci-dessous contient des données utiles dans la description de l’événement. L’extraction de ces données dans une propriété séparée les rend disponibles pour des opérations de tri et de filtrage.

![Extrait de l’échantillon](media/custom-fields/sample-extract.png)

> [!NOTE]
> Dans la version préliminaire, votre espace de travail est limité à 100 champs personnalisés.  Cette limite pourra être relevée lorsque cette fonction sera disponible dans le commerce.

## <a name="creating-a-custom-field"></a>Création d’un champ personnalisé
Lorsque vous créez un champ personnalisé, Log Analytics doit savoir quelles données utiliser pour le renseigner.  Il fait donc appel à une technologie de Microsoft Research, appelée FlashExtract, afin d’identifier rapidement ces données.  Au lieu de vous obliger à fournir des instructions explicites, Azure Monitor apprend à connaître les données que vous souhaitez extraire à partir des exemples que vous fournissez.

Les sections suivantes décrivent la procédure de création d’un champ personnalisé.  À la fin de cet article se trouve une procédure détaillée d’extraction.

> [!NOTE]
> Le champ personnalisé est renseigné lorsque des enregistrements correspondant aux critères spécifiés sont ajoutés à l’espace de travail Log Analytics. Il ne s’affiche donc que sur les enregistrements collectés après la création du champ personnalisé.  Le champ personnalisé n’est pas ajouté aux enregistrements déjà présents dans le magasin de données lors de sa création.
> 

### <a name="step-1--identify-records-that-will-have-the-custom-field"></a>Étape 1 : identifier les enregistrements qui contiendront le champ personnalisé
La première étape consiste à identifier les enregistrements qui recevront le champ personnalisé.  Commencez par une [requête de journal standard](../log-query/log-query-overview.md), puis vous sélectionnez l’enregistrement qui va servir de modèle à Azure Monitor.  Lorsque vous indiquez que vous allez extraire des données pour les placer dans un champ personnalisé, l’ **Assistant Extraction de champs** s’ouvre et vous permet d’affiner et de valider les critères.

1. Accédez **Journaux** et utilisez une [requête pour récupérer les enregistrements](../log-query/log-query-overview.md) qui contiendront le champ personnalisé.
2. Sélectionnez l’enregistrement que Log Analytics utilisera comme modèle pour extraire les données à afficher dans le champ personnalisé.  Vous allez identifier les données à extraire de cet enregistrement, données que Log Analytics va utiliser pour déterminer la logique permettant de renseigner le champ personnalisé de tous les enregistrements similaires.
3. Développez les propriétés de l’enregistrement, cliquez sur l’ellipse à gauche de la propriété en haut de l’enregistrement, puis sélectionnez **Extraire les champs de**.
4. L’**Assistant Extraction de champs s’ouvre** et l’enregistrement que vous avez sélectionné s’affiche dans la colonne **Exemple principal**.  Le champ personnalisé sera défini pour les enregistrements ayant les mêmes valeurs dans les propriétés sélectionnées.  
5. Si la sélection ne correspond pas exactement à ce que vous souhaitez, sélectionnez d’autres champs pour affiner les critères.  Pour modifier les valeurs des critères, vous devez annuler l’opération et sélectionner un autre enregistrement correspondant aux critères souhaités.

### <a name="step-2---perform-initial-extract"></a>Étape 2 : effectuer l’extraction initiale.
Après avoir identifié les enregistrements qui contiendront le champ personnalisé, vous identifiez les données à extraire.  Log Analytics utilisera ces informations pour identifier des données similaires dans des enregistrements similaires.  Dans l’étape suivante, vous allez valider les résultats et fournir d’autres informations que Log Analytics va exploiter dans son analyse.

1. Mettez en surbrillance le texte de l’enregistrement exemple dont vous souhaitez renseigner dans le champ personnalisé.  Dans la boîte de dialogue qui s’affiche, indiquez le nom et le type de données du champ et effectuez l’extraction initiale.  Les caractères **\_CF** sont automatiquement ajoutés à la fin.
2. Cliquez sur **Extraire** pour analyser les enregistrements collectés.  
3. Les sections **Résumé** et **Résultats** de la recherche affichent les résultats de l’extraction pour que vous puissiez vérifier qu’ils sont corrects.  **Résumé** affiche les critères utilisés pour identifier les enregistrements et le nombre de chacune des valeurs de données identifiées.  **Résultats de la recherche** fournit une liste détaillée des enregistrements correspondant aux critères.

### <a name="step-3--verify-accuracy-of-the-extract-and-create-custom-field"></a>Étape 3 : vérifier l’exactitude de l’extraction et créer le champ personnalisé
Une fois l’extraction initiale effectuée, Log Analytics affiche ses résultats en fonction des données déjà collectées.  Si les résultats semblent exacts, vous pouvez créer le champ personnalisé sans aucune opération supplémentaire.  Si tel n’est pas le cas, vous pouvez affiner les résultats afin d’améliorer la logique de Log Analytics.

1. Si des valeurs de l’extraction initiale sont incorrectes, cliquez sur l’icône **Modifier** en regard d’un enregistrement inexact, puis sélectionnez **Modifier cette mise en surbrillance** afin de modifier la sélection.
2. L’entrée est copiée dans la section **Exemples supplémentaires** sous **Exemple principal**.  Vous pouvez ajuster la mise en surbrillance pour aider Log Analytics à comprendre la sélection qu’il aurait dû faire.
3. Cliquez sur **Extraire** pour utiliser ces nouvelles informations et évaluer tous les enregistrements existants.  Les résultats peuvent être modifiés pour les enregistrements autres que celui dont vous venez de modifier la logique.
4. Continuez d’ajouter des corrections jusqu’à ce que tous les enregistrements de l’extraction identifient correctement les données à afficher dans le champ personnalisé.
5. Cliquez sur **Enregistrer l’extraction** lorsque vous êtes satisfait des résultats.  Le champ personnalisé est maintenant défini, mais il n’est encore ajouté à aucun enregistrement.
6. Attendez que des nouveaux enregistrements correspondant aux critères spécifiés soient collectés, puis relancez la recherche du journal. Les nouveaux enregistrements doivent contenir le champ personnalisé.
7. Utilisez le champ personnalisé comme n’importe quelle autre propriété d’enregistrement.  Il peut vous servir à agréger et grouper les données, et même à produire d’autres connaissances.

## <a name="viewing-custom-fields"></a>Affichage de champs personnalisés
Pour afficher une liste de l’ensemble des champs personnalisés de votre groupe de gestion, accédez au menu **Paramètres avancés** de votre espace de travail Log Analytics, dans le portail Azure.  Sélectionnez **Données** puis **Champs personnalisés** pour obtenir la liste de tous les champs personnalisés dans votre espace de travail.  

![Champs personnalisés](media/custom-fields/list.png)

## <a name="removing-a-custom-field"></a>Suppression d’un champ personnalisé
Il existe deux méthodes pour supprimer un champ personnalisé.  La première consiste à utiliser l’option **Supprimer** de chaque champ lorsque vous affichez la liste complète, comme indiqué ci-dessus.  L’autre consiste à extraire un enregistrement et à cliquer sur le bouton à gauche du champ.  Le menu affiche une option permettant de supprimer le champ personnalisé.

## <a name="sample-walkthrough"></a>Exemple de procédure pas à pas
La section suivante décrit la procédure complète de création d’un champ personnalisé.  Cet exemple extrait le nom du service dans les événements Windows indiquant un changement d’état de service.  Il s’appuie sur les événements créés par le Gestionnaire de contrôle des services lors du démarrage du système des ordinateurs Windows.  Si vous souhaitez suivre cet exemple, vous devez [collecter des événements d’information du journal système](data-sources-windows-events.md).

Nous spécifions la requête suivante pour renvoyer tous les événements du Gestionnaire de contrôle des services dont l’ID d’événement est 7036, c’est-à-dire l’événement indiquant le démarrage ou l’arrêt d’un service.

![La capture d’écran montre une requête pour une source et un ID d’événement.](media/custom-fields/query.png)

Ensuite, nous sélectionnons puis développons un enregistrement ayant l’ID d’événement 7036.

![Enregistrement source](media/custom-fields/source-record.png)

Nous définissons les champs personnalisés en cliquant sur les points de suspension en regard de la propriété du haut.

![Extraire des champs](media/custom-fields/extract-fields.png)

L’**Assistant Extraction de champs** s’ouvre. Les champs **EventLog** et **EventID** sont sélectionnés dans la colonne **Exemple principal**.  Cela indique que le champ personnalisé sera défini pour les événements du journal système ayant l’ID d’événement 7036.  Cela nous suffit, inutile de sélectionner d’autres champs.

![Exemple principal](media/custom-fields/main-example.png)

Nous mettons en surbrillance le nom du service dans la propriété **RenderedDescription** et utilisons **Service** pour identifier le nom du service.  Le nom du champ personnalisé sera **Service_CF**. Ici, le type de champ est une chaîne, donc nous pouvons le laisser tel quel.

![Titre du champ](media/custom-fields/field-title.png)

Nous constatons que le nom du service est identifié correctement pour certains enregistrements, mais pas pour d’autres.   Les **Résultats de la recherche** montrent que cette partie du nom de **Carte de performance WMI** n’est pas sélectionnée.  Le **Résumé** montre qu’un seul enregistrement identifié **Programme d’installation de modules** au lieu de **Programme d’installation de modules Windows**.  

![Capture d’écran montrant des parties du nom de service en surbrillance dans le volet des résultats de recherche et un nom de service incorrect en surbrillance dans le résumé.](media/custom-fields/search-results-01.png)

Commençons par l’enregistrement **Carte de performance WMI** .  Cliquez sur son icône de modification, puis sur **Modifier cette mise en surbrillance**.  

![Modifier la mise en surbrillance](media/custom-fields/modify-highlight.png)

Nous étendons la mise en surbrillance afin d’inclure le mot **WMI** , puis nous relançons l’extraction.  

![Exemple supplémentaire](media/custom-fields/additional-example-01.png)

Nous constatons que les entrées de **Carte de performance WMI** ont été corrigées, et que Log Analytics a utilisé ces informations pour corriger les enregistrements de **Programme d’installation pour les modules Windows**.

![Capture d’écran montrant le nom complet du service en surbrillance dans le volet des résultats de recherche et les noms de service corrects en surbrillance dans le résumé.](media/custom-fields/search-results-02.png)

Nous pouvons maintenant exécuter une requête qui vérifie que **Service_CF** est créé, mais qu’il n’est encore ajouté à aucun enregistrement. C’est parce que le champ personnalisé ne fonctionne pas sur les enregistrements existants. Nous devons attendre que de nouveaux enregistrements soient collectés.

![Nombre initial](media/custom-fields/initial-count.png)

Après la collecte de nouveaux événements, nous pouvons voir que le champ **Service_CF** est désormais ajouté aux enregistrements qui répondent à nos critères.

![Résultats finaux](media/custom-fields/final-results.png)

Nous pouvons maintenant utiliser le champ personnalisé comme n’importe quelle propriété d’enregistrement.  Pour illustrer ce propos, nous créons une requête qui classe les enregistrements par le nouveau champ **Service_CF** pour identifier les services les plus actifs.

![Regrouper par requête](media/custom-fields/query-group.png)

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [requêtes dans les journaux](../log-query/log-query-overview.md) pour générer des requêtes utilisant des champs personnalisés comme critères.
* Surveillez les [fichiers journaux personnalisés](data-sources-custom-logs.md) que vous analysez à l’aide de champs personnalisés.

