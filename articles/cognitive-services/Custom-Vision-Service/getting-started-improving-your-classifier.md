---
title: 'Améliorer votre classifieur : Service Vision personnalisée'
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous allez découvrir comment la quantité, la qualité et la variété des données peuvent améliorer la qualité de votre classifieur dans le service Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/21/2019
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: a77d3d5c1225fdd85e27db20cdae23e0c77a5e28
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91271356"
---
# <a name="how-to-improve-your-classifier"></a>Comment améliorer votre classifieur

Dans ce guide, vous allez découvrir comment améliorer la qualité de votre classifieur Custom Vision. La qualité d’un classifieur dépend de la quantité, de la qualité et de la variété des données étiquetées fournies, ainsi que de l’équilibre du jeu de données global. Un bon classifieur comporte un jeu de données d’entraînement équilibré et représentatif de ce qui lui sera soumis. Le processus de création d’un tel classifieur est itératif ; il est normal de pratiquer plusieurs cycles d’entraînement pour atteindre les résultats escomptés.

Voici un modèle général pour vous aider à créer un classifieur plus précis :

1. Effectuer le premier cycle d’apprentissage.
1. Ajouter d’autres images et équilibrer les données ; réentraîner
1. Ajouter des images avec différents arrière-plans, éclairages, tailles d’objet, angles de vue et styles ; réentraîner
1. Utiliser de nouvelles images pour tester la prédiction
1. Modifier les données d’entraînement existantes en fonction des résultats de la prédiction

## <a name="prevent-overfitting"></a>Empêcher le surajustement

Un classifieur va parfois apprendre à faire des prédictions en fonction de caractéristiques arbitraires que vos images ont en commun. Par exemple, si vous créez un classifieur pommes/agrumes et que vous avez utilisé des images de pommes tenues dans des mains et des images d’agrumes posés dans des assiettes blanches, le classifieur risque de donner une trop grande importance à la distinction mains/assiettes, plutôt qu’à la distinction pommes/agrumes.

![Image de classification inattendue](./media/getting-started-improving-your-classifier/unexpected.png)

Pour corriger ce problème, suivez les instructions ci-après afin d’effectuer l’entraînement avec des images plus variées : fournissez des images prises selon différents angles, arrière-plans, tailles d’objet, groupes et autres variantes.

## <a name="data-quantity"></a>Quantité de données

Le nombre d’images d’entraînement est le facteur le plus important. Nous recommandons d’utiliser au moins 50 images par étiquette pour commencer. Avec moins d’images, il existe un risque plus élevé de surajustement. Et même si vos chiffres de performance suggèrent une bonne qualité, votre modèle aura peut-être du mal avec des données réelles. 

## <a name="data-balance"></a>Équilibre des données

Il est également essentiel de prendre en compte les quantités relatives de vos données d’entraînement. Par exemple, l’utilisation de 500 images pour une seule étiquette et de 50 images pour une autre déséquilibre le jeu de données d’entraînement. Le modèle sera donc plus précis pour prédire une étiquette plutôt qu’une autre. Les résultats seront très certainement meilleurs avec un ratio d’au moins 1:2 entre l’étiquette la moins bien dotée et l’étiquette la plus foisonnante. Si, par exemple, l’étiquette la plus fournie comporte 500 images, la moins fournie doit inclure au moins 250 images pour l’entraînement.

## <a name="data-variety"></a>Variété des données

Veillez à utiliser des images représentatives de ce qui va être soumis au classifieur en utilisation normale. Sinon, votre classifieur risque d’apprendre à faire des prédictions en fonction de caractéristiques arbitraires que vos images ont en commun. Par exemple, si vous créez un classifieur pommes/agrumes et que vous avez utilisé des images de pommes tenues dans des mains et des images d’agrumes posés dans des assiettes blanches, le classifieur risque de donner une trop grande importance à la distinction mains/assiettes, plutôt qu’à la distinction pommes/agrumes.

![Image de classification inattendue](./media/getting-started-improving-your-classifier/unexpected.png)

Pour corriger ce problème, incluez des images très diverses pour garantir une correcte généralisation par votre classifieur. Voici quelques moyens de rendre un jeu d’apprentissage plus varié :

* __Contexte__ : Fournissez des images de votre objet devant différents arrière-plans. Les photos prises dans des contextes naturels sont plus performantes que celles dont l’arrière-plan est neutre, car elles apportent davantage d’informations au classifieur.

    ![Exemples d’images (arrière-plan)](./media/getting-started-improving-your-classifier/background.png)

* __Éclairage :__ Fournissez des images présentant des éclairages variés (autrement dit, prises avec flash, avec exposition élevée, etc.), en particulier si les images utilisées pour la prédiction sont diversifiées de ce point de vue. Il est également conseillé d’utiliser dont la saturation, la teinte et la luminosité varient.

    ![Exemples d’images (éclairage)](./media/getting-started-improving-your-classifier/lighting.png)

* __Taille de l'objet :__ Fournissez des images dans lesquelles les objets varient en taille et en nombre (par exemple, une photo de régime de bananes et un gros plan sur une banane isolée). Les différences de taille permettent au classifieur de mieux généraliser.

    ![Exemples d’images (taille)](./media/getting-started-improving-your-classifier/size.png)

* __Angle de vue :__ fournissez des images prises avec différents angles de vue. Par ailleurs, si toutes vos photos doivent être prises par des caméras fixes (par exemple, des caméras de surveillance), veillez à attribuer une étiquette différente à chaque objet revenant régulièrement pour éviter tout surajustement (interprétation des objets non connexes comme étant la caractéristique).

    ![Exemples d’images (angle)](./media/getting-started-improving-your-classifier/angle.png)

* __Style :__ Fournissez des images de styles différents de la même classe (par exemple, des variétés différentes du même fruit). En revanche, si vous avez des objets de styles radicalement différents (par exemple, Mickey Mouse et une vraie souris), nous vous recommandons de les étiqueter comme des classes distinctes afin de mieux représenter leurs caractéristiques distinctes.

    ![Exemples d’images (style)](./media/getting-started-improving-your-classifier/style.png)

## <a name="negative-images"></a>Images négatives

À un moment donné dans votre projet, il peut être nécessaire d’ajouter des _exemples négatifs_ pour rendre votre classifieur plus précis. Les exemples négatifs sont ceux qui ne correspondent à aucune des autres balises. Quand vous chargez ces images, appliquez l’étiquette spéciale **Negative** (Négatif) à celles-ci.

> [!NOTE]
> Le service Vision personnalisée prend en charge un traitement automatique des images négatives. Par exemple, si vous générez un classifieur raisins/bananes et que vous soumettez une image de chaussure à la prédiction, il devra lui donner un score proche de 0 % aussi bien pour les raisins que pour les bananes.
> 
> À l’inverse, dans les cas où les images négatives représentent simplement une variante des images utilisées pour l’apprentissage, il est probable que le modèle les catégorise comme une classe étiquetée en raison des grandes similitudes qu’elles présentent. Par exemple, si vous avez un classifieur oranges/pamplemousses et que vous fournissez une image de clémentine, il est possible qu’il l’évalue comme étant une orange car la clémentine présente des caractéristiques similaires à l'orange. Si vos images négatives sont de cette nature, nous vous conseillons de créer une ou plusieurs balises supplémentaires (comme **Autre**) et d’étiqueter ainsi les images négatives pendant l’apprentissage pour permettre au modèle de mieux faire la distinction entre ces classes.

## <a name="use-prediction-images-for-further-training"></a>Utiliser des images de prédiction à des fins de réentraînement

Quand vous utilisez ou testez le classifieur d’images en soumettant des images au point de terminaison de prédiction, le service Custom Vision stocke ces images. Vous pouvez alors les utiliser pour améliorer le modèle.

1. Pour voir les images soumises au classifieur, ouvrez la [page web Custom Vision](https://customvision.ai), accédez à votre projet et sélectionnez l’onglet __Prédictions__. La vue par défaut montre des images à partir de l’itération active. Vous pouvez utiliser le menu déroulant __Iteration__ pour voir les images envoyées au cours des précédentes itérations.

    ![Capture d’écran de l’onglet des prédictions, avec des images dans la vue](./media/getting-started-improving-your-classifier/predictions.png)

2. Pointez sur une image pour lire les mots clés qui ont été prédits par le classifieur. Les images sont triées de sorte à ce que celles susceptibles d’apporter le plus d’améliorations au classifieur soient listées en premier. Pour utiliser une autre méthode de tri, effectuez une sélection dans la section __Sort__. 

    Pour ajouter une image à vos données d’entraînement existantes, sélectionnez-la, sélectionnez les étiquettes correspondantes et cliquez sur __Save and close__. L’image est supprimée de __Prédictions__ et ajoutée à l’ensemble d’images d’entraînement. Vous pouvez la voir en sélectionnant l’onglet __Training Images__ (Images d’entraînement).

    ![Image de la page d’étiquetage](./media/getting-started-improving-your-classifier/tag.png)

3. Utilisez ensuite le bouton __Train__ (Entraîner) pour réentraîner le classifieur.

## <a name="visually-inspect-predictions"></a>Inspecter visuellement les prédictions

Pour examiner les prédictions d’images, accédez à l’onglet __Training Images__, sélectionnez votre itération d’entraînement précédente dans le menu déroulant **Iteration** et cochez une ou plusieurs balises sous la section **Tags**. La vue doit à présent montrer une zone rouge autour de chacune des images pour lesquelles le modèle n’a pas réussi à prédire correctement la balise donnée.

![Image de l’historique des itérations](./media/getting-started-improving-your-classifier/iteration.png)

Parfois, un examen visuel permet d’identifier des modèles pouvant être rectifiés en ajoutant des données d’entraînement supplémentaires ou en modifiant les données existantes. Par exemple, un classifieur pommes/citrons verts risquerait d’étiqueter à tort toutes les pommes vertes comme des citrons verts. Vous pourriez alors corriger ce problème en ajoutant et en fournissant des données d’entraînement comportant des images étiquetées de pommes vertes.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez découvert plusieurs techniques permettant de rendre plus précis votre modèle de classification d’images personnalisé. Découvrez ensuite comment tester des images programmatiquement en les soumettant à l’API de prédiction.

> [!div class="nextstepaction"]
> [Utiliser l’API de prédiction](use-prediction-api.md)
