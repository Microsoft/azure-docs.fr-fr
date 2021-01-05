---
title: Galerie de plug-ins du Lecteur multimédia Azure
description: Cet article contient la liste des plug-ins disponibles pour le Lecteur multimédia Azure.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.openlocfilehash: 6d592064729bbdd64e485e0bd0373b2d471597e0
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87282481"
---
# <a name="azure-media-player-plugin-gallery"></a>Galerie de plug-ins du Lecteur multimédia Azure #

## <a name="plugins"></a>Plug-ins ##

| Nom du plug-in                         | URL de démonstration                    | Code source                | Description    |
|-------------------------------------|-----------------------------|----------------------------|----------------|
| Fonctionnalités supplémentaires                 | | | |
| **Nouveau !** AMP360Video                | [Démonstration](http://www.babylonjs.com/demos/amp360video/)                        | [GitHub](https://github.com/BabylonJS/Extensions/tree/master/Amp360Video)                     | Le plug-in vous permet de visualiser 360 video dans Amp, sur votre ordinateur ou sur des appareils compatibles avec la réalité virtuelle. La documentation complète est disponible [ici](https://doc.babylonjs.com/extensions/amp360video) : |
|  Info-bulle Sprite                         | [Démonstration](http://www.skymedia.tv/asset/sprite)                        | [GitHub](https://github.com/RickShahid/SpriteTip)                    | Plug-in du Lecteur multimédia Azure (AMP) pour le rendu chronologique d’un sprite d’images miniatures de vidéo, généré depuis Media Encoder Standard (MES) d’Azure Media Services (AMS). |
| Superposition de diagnostics                 | [Démonstration](https://openidconnectweb.azurewebsites.net/Diagnoverlay.html)                        | [GitHub](https://github.com/willzhan/diagnoverlay)                     | Ce plug-in affiche les éléments suivants : tous les paramètres clés, les statistiques de la vidéo, tous les événements du cycle de vie de la lecture vidéo et les informations de protection DRM, telles que l’ID de clé, les URL d’acquisition de licence, s’ils sont protégés.                                                                                                                                                                      |
| Calculatrice de fréquence d’images et de code horaire | Aucune démonstration disponible | [GitHub](https://github.com/mconverti/media-services-javascript-azure-media-player-framerate-timecode-calculator-plugin)                     | Ce plug-in calcule la fréquence d’images d’une vidéo en fonction des zones MP4 `tfhd`/`trun` du premier fragment vidéo MPEG-DASH, il analyse la valeur de l’échelle de temps à partir du manifeste du client MPEG-DASH et fournit également un moyen de générer le code horaire d’un temps absolu donné à partir du Lecteur (ainsi que le temps absolu du Lecteur en fonction du code horaire). |
| <strike>Vitesse de lecture</strike>                      | [Démonstration](https://azure-samples.github.io/media-services-javascript-Azure-Media-Player-playback-rate-plugin/)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-time-tip-plugin)                     | Ce plug-in permet aux viewers de contrôler la vitesse de la vidéo. *Notez que cette fonctionnalité est automatiquement disponible dans la version AMP v 2.0.0+, mais qu’elle est désactivée par défaut.* Pour savoir comment l’activer, consultez nos exemples [ici](https://github.com/Azure-Samples/azure-media-player-samples) |
| Info-bulle de temps par pointage                      | [Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/timetip/example.html)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-time-tip-plugin)                     | Affiche une info-bulle de temps sur la barre de progression, lors du pointage de la souris pour une recherche précise d’un temps. *Remarque : Ce plug-in est déjà intégré à AMP*, mais si vous souhaitez voir comment il est programmé, n’hésitez pas à l’examiner de plus près.                                                                                                                 |
| Superposition de titre                       | [Démonstration](https://azure-samples.github.io/media-services-javascript-azure-media-player-title-overlay-plugin)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-title-overlay-plugin)                     | Superpose le titre de la vidéo configurable sur l’écran |
| Marqueurs chronologiques                    | [Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/timelinemarkers/example.html)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-timeline-markers-plugin)                     | Ce plug-in accepte un tableau de marqueurs minuscules de temps et de superpositions, sur la barre de progression aux moments voulus. |
| Analytics                           | | | |
| Application Insights                | [Billet de blog](https://azure.microsoft.com/blog/player-analytics-azure-media-player-plugin/)                   | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-application-insights-plugin)                     | Plug-in qui effectue le suivi des métriques de votre Lecteur et les achemine vers Power BI, pour une représentation graphique intuitive de l’expérience utilisateur du Lecteur par vos viewers. |
| Google Analytics                    | N/A                         | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-google-analytics-plugin)                     | Plug-in Google Analytics pour le Lecteur multimédia Azure |
| Diagnostics                         | | | |
| Sortie de diagnostics                  | [Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/diagnosticslogger/example.html)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-diagnostic-logger-plugin)                     | Ce plug-in génère un tableau de diagnostics depuis votre Lecteur ; pour le voir fonctionner, accédez au lien de démonstration et ouvrez votre console JavaScript. |
| Options d’ergonomie                      | | | |
| Zoom avant                             | [Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/zoom/example.html)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-zoom-plugin)                     | Ce plug-in affiche une échelle de zoom qu’il est possible de faire glisser sur l’écran des Lecteurs, afin que les utilisateurs puissent zoomer sur votre contenu |
| Sous-titres en direct                       | [Billet de blog Azure](https://azure.microsoft.com/blog/live-real-time-captions-with-azure-media-services-and-player/),[Billet SubPly](http://www.subply.com/en/Products/AzureLiveCaptions.htm) | N/A | *Pour plus d’informations, consultez les billets.* Workflow de bout en bout, conçu pour le plug-in intégré de sous-titrage en direct, destiné au Lecteur multimédia Azure ; cliquez sur le lien le plus à gauche pour accéder au site de SubPly et en savoir plus sur la solution |
| Touches d’accès rapide                            | <strike>[Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/hotkeys/example.html)</strike>                        | <strike>[GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-hot-keys-plugin)</strike>                     | Le plug-in de touches d’accès rapide permet aux viewers de contrôler divers aspects du Lecteur à l’aide de combinaisons de plug-ins génériques, comme F pour le plein écran, M pour muet et les touches de direction pour le contrôle de la barre de progression. *Remarque : Ce plug-in a déjà été intégré à AMP, mais n’hésitez pas à l’utiliser en tant que ressource* |
| Réseaux sociaux                              | | | |
| Partager                               | [Démonstration](http://sr-test.azurewebsites.net/Tests/Plugin%20Gallery/plugins/share/example.html)                        | [GitHub](https://github.com/Azure-Samples/media-services-javascript-azure-media-player-social-share-plugin)                     | Ce plug-in ajoute un bouton Partager à la barre de contrôle du Lecteur, pour que vos utilisateurs puissent partager la vidéo qu’ils regardent avec leurs amis via Facebook, Twitter ou LinkedIn. |

## <a name="next-steps"></a>Étapes suivantes ##

- [Démarrage rapide du Lecteur multimédia Azure](azure-media-player-quickstart.md)
