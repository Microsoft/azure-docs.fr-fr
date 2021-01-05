---
title: Exporter votre modèle sur votre mobile - Service Vision personnalisée
titleSuffix: Azure Cognitive Services
description: Cet article explique comment exporter votre modèle pour l'utiliser lors de la création d'applications mobiles ou pour l'exécuter localement à des fins de classification en temps réel.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/21/2019
ms.author: pafarley
ms.openlocfilehash: be28a10f8a96d21d64d08cfacee4543c065168ae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85391755"
---
# <a name="export-your-model-for-use-with-mobile-devices"></a>Exporter votre modèle pour l’utiliser avec des appareils mobiles

Le service Vision personnalisée permet d’exporter des classifieurs pour les exécuter en mode hors connexion. Vous pouvez incorporer un classifieur exporté dans une application et l’exécuter localement sur un appareil pour la classification en temps réel.

## <a name="export-options"></a>Options d'exportation

Le service Vision personnalisée prend en charge les exportations suivantes :

* __Tensorflow__ pour __Android__.
* __CoreML__ pour __iOS11__.
* __ONNX__ pour __Windows ML__.
* __[Kit de développement Vision AI](https://azure.github.io/Vision-AI-DevKit-Pages/)__ .
* Un __conteneur Docker__ pour l'architecture Windows, Linux ou ARM. Le conteneur inclut un modèle Tensorflow et un code de service permettant d'utiliser l'API Custom Vision.

> [!IMPORTANT]
> Le service Vision personnalisée exporte uniquement des domaines __compacts__. Les modèles générés par des domaines compacts sont optimisés en fonction des contraintes de la classification en temps réel sur les appareils mobiles. Les classifieurs conçus avec un domaine compact sont parfois légèrement moins précis qu’avec un domaine standard, pour une même quantité de données d’apprentissage.
>
> Pour plus d’informations sur l’amélioration des classifieurs, consultez l’article [Améliorer votre classifieur](getting-started-improving-your-classifier.md).

## <a name="convert-to-a-compact-domain"></a>Convertir un domaine en domaine compact

> [!NOTE]
> Les étapes décrites dans cette section ne s'appliquent que si vous utilisez un modèle qui n'est pas défini avec un domaine compact.

Pour convertir le domaine d'un modèle existant, procédez comme suit :

1. Sur le [site web de Custom Vision](https://customvision.ai), sélectionnez l'icône __Accueil__ pour afficher la liste de vos projets.

    ![Image de l’icône Accueil et de la liste des projets](./media/export-your-model/projects-list.png)

1. Sélectionnez un projet, puis sélectionnez l’icône __d’engrenage__ en haut à droite de la page.

    ![Image de l’icône d’engrenage](./media/export-your-model/gear-icon.png)

1. Dans la section __Domaines__, sélectionnez un des domaines __compacts__. Sélectionnez __Enregistrer les modifications__. 

    > [!NOTE]
    > Pour le kit de développement Vision AI, le projet doit être créé avec le domaine __Général (Compact)__ , et vous devez spécifier l'option **Kit de développement Vision AI** dans la section **Fonctionnalités d'exportation**.

    ![Image de la sélection des domaines](./media/export-your-model/domains.png)

1. En haut de la page, sélectionnez __Entraîner__ pour recommencer l’entraînement avec le nouveau domaine.

## <a name="export-your-model"></a>Exporter votre modèle

Pour exporter le modèle après le nouvel entraînement, effectuez les étapes suivantes :

1. Accédez à l’onglet **Performances** et sélectionnez __Exporter__. 

    ![Image de l’icône Exporter](./media/export-your-model/export.png)

    > [!TIP]
    > Si l’option __Exporter__ n’est pas disponible, cela signifie que l’itération sélectionnée n’utilise pas un domaine compact. Dans la section __Itérations__ de cette page, sélectionnez une itération qui utilise un domaine compact, puis sélectionnez __Exporter__.

1. Sélectionnez le format d'exportation de votre choix, puis sélectionnez __Exporter__ pour télécharger le modèle.

## <a name="next-steps"></a>Étapes suivantes

Intégrer votre modèle exporté dans une application en explorant un des articles ou exemples suivants :

* [Utiliser votre modèle Tensorflow avec Python](export-model-python.md)
* [Utiliser votre modèle ONNX avec Windows Machine Learning](custom-vision-onnx-windows-ml.md)
* Voir l’exemple de [modèle CoreML dans une application iOS](https://go.microsoft.com/fwlink/?linkid=857726) pour une classification des images en temps réel avec Swift.
* Voir l’exemple de [modèle Tensorflow dans une application Android](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample) pour une classification des images en temps réel sur Android.
* Voir l’exemple de [modèle CoreML avec Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel) pour une classification des images en temps réel dans une application Xamarin iOS.
