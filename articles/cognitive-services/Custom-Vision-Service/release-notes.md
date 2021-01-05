---
title: Notes de publication du service Vision personnalisée
titleSuffix: Azure Cognitive Services
description: Consultez les dernières informations sur les nouvelles versions fournies par l’équipe Custom Vision.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 04/03/2019
ms.author: pafarley
ms.openlocfilehash: ccf742e5ff612816cae1056ef7bf0e64fc9dbb5b
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616465"
---
# <a name="custom-vision-service-release-notes"></a>Notes de publication du service Vision personnalisée

## <a name="may-2-2019-and-may-10-2019"></a>2 mai 2019 et 10 mai 2019

- Résolution des bogues et améliorations de backend

## <a name="may-23-2019"></a>23 mai 2019

- Expérience UX du portail améliorée concernant les abonnements Azure, vous permettant de sélectionner vos annuaires Azure plus facilement.

## <a name="april-18-2019"></a>18 avril 2019 

- Exportation de détection d’objets ajoutée au Kit Vision AI Dev.
- Ajustements d’interface utilisateur, notamment la recherche de projet.

## <a name="april-3-2019"></a>3 avril 2019

- La limite du nombre de zones englobantes par image a été augmentée à 200. 
- Corrections de bogues, notamment la mise à jour importante des performances pour les modèles exportés vers TensorFlow. 

## <a name="march-26-2019"></a>26 mars 2019

- Le service Custom Vision est mis à la disponibilité générale sur Azure !
- Fonctionnalité Formation avancée ajoutée avec un nouveau système de Machine Learning améliorant les performances, en particulier pour les jeux de données complexes et la classification fine. Avec la formation avancée, vous pouvez spécifier un budget de temps de calcul pour la formation. Custom Vision identifiera de manière expérimentale les meilleurs paramètres de formation et d’augmentation. Pour les itérations rapides, vous pouvez continuer à utiliser la formation rapide existante.
- Lancement des API 3.0. Prochaine dépréciation annoncée des API antérieures à 3.0 le 1er octobre 2019. Consultez les Démarrages rapides dans la documentation pour [.Net](./quickstarts/image-classification.md), [Python](./quickstarts/image-classification.md), [Node](./quickstarts/image-classification.md), [Java](./quickstarts/image-classification.md) ou [Go](./quickstarts/image-classification.md) pour obtenir des exemples sur la prise en main.
- « Itérations par défaut » remplacées par Publier/Dépublier dans les API 3.0.
- De nouvelles cibles d’exportation de modèle ont été ajoutées. L’exportation Dockerfile a été mise à niveau pour prendre en charge ARM pour Raspberry Pi 3. Prise en charge de l’exportation ajoutée au [Kit Vision AI Dev](https://visionaidevkit.com/).
- Augmentation de la limite de balises par projet à 500 pour le niveau S0. Augmentation de la limite d’images par projet à 100 000 pour le niveau S0.
- Domaine Adult (Pour adultes) supprimé. Le domaine General (Général) est recommandé à la place.
- [Tarification](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) annoncée pour la disponibilité générale.  

## <a name="february-25-2019"></a>25 février 2019

- Fin annoncée des projets Version d’évaluation limitée (projets non associés à une ressource Azure). La migration de Custom Vision vers la préversion publique Azure est presque terminée. Depuis le 25 mars 2019, le site CustomVision.ai prendra uniquement en charge l’affichage de projets associés à une ressource Azure, comme la ressource Custom Vision gratuite. Jusqu’au 1er octobre 2019, vous pourrez toujours accéder à vos projets de version d’évaluation limitée existants via les API Custom Vision. Cela vous laisse le temps de mettre à jour les clés de l’API pour toutes les applications que vous avez écrites avec Custom Vision. Après le 1er octobre 2019, les projets de version d’évaluation limitée que vous n’avez pas déplacés vers Azure seront supprimés.

## <a name="january-22-2019"></a>22 janvier 2019

- Prise en charge ajoutée pour de nouvelles régions Azure : USA Ouest 2, USA Est, USA Est 2, Europe Ouest, Europe Nord, Asie Sud-Est, Australie Est, Inde Centre, Royaume-Uni Sud, Japon Est et USA Centre Nord. Prise en charge continue pour USA Centre Sud.

## <a name="december-12-2018"></a>12 décembre 2018

- Prise en charge de l’exportation pour les modèles de détection d’objets (introduction du domaine compact de détection d’objets).
- Correction de plusieurs problèmes d’accessibilité afin d’améliorer la prise en charge de la navigation au clavier et du lecteur d’écran.
- Mises à jour de l’expérience utilisateur de la visionneuse d’images et amélioration de l’expérience de détection des objets afin d’en accélérer l’identification.  
- Mise à jour du modèle de base pour le domaine de détection d’objets, afin d’améliorer la qualité de la détection des objets.
- Résolution des bogues.

## <a name="november-6-2018"></a>6 novembre 2018

- Ajout de la prise en charge du domaine de logo dans la détection d’objets.

## <a name="october-9-2018"></a>9 octobre 2018

- La détection d’objets passe en préversion payante. Vous pouvez désormais créer des projets de détection d’objets avec une ressource Azure.
- Ajout de la fonctionnalité « Déplacer vers Azure » au site web pour faciliter la mise à niveau d’un projet d’évaluation limitée à lier à un projet de ressources Azure lié (F0 ou S0). Vous pouvez la trouver sur la page Paramètres de votre produit.  
- Ajout de l’exportation vers ONNX 1.2 pour prendre en charge la mise à jour Windows d’octobre 2018 de ML Windows.
Résolution des bogues, dont l’exportation ONNX avec des caractères spéciaux.

## <a name="august-14-2018"></a>14 août 2018

- Ajout du widget « Mise en route » au site customvision.ai pour guider les utilisateurs dans l’entraînement du projet.
- Autres améliorations apportées au pipeline d’apprentissage automatique pour bénéficier des projets à plusieurs étiquettes (nouvelle couche de perte).

## <a name="june-28-2018"></a>28 juin 2018

- Résolution des bogues et améliorations de backend.
- Activation de la classification multiclasse pour les projets où les images ont exactement une étiquette. Dans les prédictions pour le mode multiclasse, les probabilités atteignent une somme égale à un (toutes les images sont classées parmi vos balises spécifiées).

## <a name="june-13-2018"></a>13 juin 2018

- Actualisation de l’expérience utilisateur, axée sur la facilité d’utilisation et l’accessibilité.
- Améliorations apportées au pipeline d’apprentissage automatique pour bénéficier des projets à plusieurs étiquettes avec un grand nombre de balises.
- Correction du bogue dans l’exportation TensorFlow. Activation de la gestion des versions des modèles exportés afin que les itérations puissent être exportées plusieurs fois.

## <a name="may-7-2018"></a>7 mai 2018

- Introduction de la fonctionnalité de détection d’objets en préversion pour les projets de la version d’essai limitée.
- Mise à niveau vers les API 2.0
- Le niveau S0 a été étendu jusqu’à 250 étiquettes et à 50 000 images.
- Améliorations significatives de backend pour le pipeline d’apprentissage automatique pour les projets de classification d’images. Les projets entraînés après le 27 avril 2018 bénéficient de ces mises à jour.
- Ajout de l’exportation de modèles vers ONNX, pour une utilisation avec Windows ML.
- Ajout de l’exportation de modèles vers Dockerfile. Ceci vous permet de télécharger les artefacts pour générer vos propres conteneurs Windows ou Linux, notamment un fichier DockerFile, un modèle TensorFlow et un code de service.
- Pour les modèles formés récemment et exportés vers TensorFlow dans les domaines Général (Compact) et Repères géographiques (Compact), [les valeurs moyennes sont maintenant (0,0,0)](https://github.com/azure-samples/cognitive-services-android-customvision-sample) pour des raisons de cohérence entre tous les projets.

## <a name="march-1-2018"></a>1er mars 2018

- Préversion payante entrée et intégrée sur le portail Azure. Les projets peuvent désormais être attachés à des ressources Azure avec un niveau F0 (Gratuit) ou S0 (Standard). Introduction des projets de niveau S0, qui autorisent jusqu’à 100 étiquettes et 25 000 images.
- Modification du backend concernant le paramètre de normalisation/le pipeline d’apprentissage automatique. Ceci donne aux clients un meilleur contrôle des compromis précision/rappel lors de l’ajustement du seuil de probabilité. Dans le cadre de ces modifications, le seuil de probabilité par défaut dans le portail CustomVision.ai a été défini sur 50 %.

## <a name="december-19-2017"></a>19 décembre 2017

- Ajout de l’exportation vers Android (TensorFlow), en plus de l’exportation déjà publiée vers iOS (CoreML.) Ceci permet d’exécuter hors connexion l’exportation d’un modèle compact entraîné dans une application.
- Ajout des domaines « compacts » Distribution et Repères géographiques pour permettre l’exportation de modèles pour ces domaines.
- Publication de [l’API d’apprentissage version 1.2](https://southcentralus.dev.cognitive.microsoft.com/docs/services/f2d62aa3b93843d79e948fe87fa89554/operations/5a3044ee08fa5e06b890f11f) et de [l’API de prédiction version 1.1](https://southcentralus.dev.cognitive.microsoft.com/docs/services/57982f59b5964e36841e22dfbfe78fc1/operations/5a3044f608fa5e06b890f164). Les API mises à jour prennent en charge l’exportation de modèles, un nouveau fonctionnement de la prédiction qui n’enregistre pas les images dans « Prédictions » et introduisent des opérations par lots pour l’API d’apprentissage.
- Ajustements de l’expérience utilisateur, notamment la possibilité de voir le domaine qui a été utilisé pour entraîner une itération.
- Mise à jour du [SDK et de l’exemple C#](https://github.com/Microsoft/Cognitive-CustomVision-Windows).