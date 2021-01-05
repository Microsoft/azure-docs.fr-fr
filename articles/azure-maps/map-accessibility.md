---
title: Créer une applicationde carte accessible avec Azure Maps | Microsoft Azure Maps
description: En savoir plus sur les considérations relatives à l’accessibilité dans Azure Maps. Découvrez les fonctionnalités disponibles pour rendre les applications cartographiques accessibles et affichez des conseils sur l’accessibilité.
services: azure-maps
author: rbrundritt
ms.author: richbrun
ms.date: 12/10/2019
ms.topic: conceptual
ms.service: azure-maps
manager: cpendleton
ms.openlocfilehash: ec88437a0fad3a6bd94a67a5ef5c75b3e506f9e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88006211"
---
# <a name="building-an-accessible-application"></a>Création d’une application accessible

Jusqu’à 20 % des utilisateurs Internet ont besoin d’applications web accessibles. C’est pourquoi il est important de vérifier que votre application est conçue pour que tout utilisateur puisse facilement s’en servir. Plutôt que de considérer l’accessibilité comme un ensemble de tâches à effectuer, envisagez-la dans le cadre de votre expérience utilisateur globale. Plus votre application est accessible, plus de personnes sont à même de l’utiliser. 

Pour tout contenu interactif riche comme les cartes, vous devez prendre en considération les points suivants sur l’accessibilité :
- Prendre en charge le lecteur d’écran pour les utilisateurs qui éprouvent des difficultés à voir l’application web.
- Mettre à disposition plusieurs méthodes pour interagir avec l’application web et y naviguer, comme la souris, l’interaction tactile et le clavier.
- S’assurer que le contraste de couleur empêche les couleurs de se mélanger et permet de bien les distinguer les unes des autres. 

Le SDK web Azure Maps comporte de nombreuses fonctionnalités d’accessibilité, telles que les suivantes :
- Descriptions émises par le lecteur d’écran quand la carte se déplace et que l’utilisateur place le focus sur un contrôle ou une fenêtre indépendante.
- Prise en charge de la souris, de l’interaction tactile et du clavier.
- Prise en charge du contraste de couleur accessible dans le style de la carte routière.
- Prise en charge du contraste élevé.

Vous trouverez [ici](https://cloudblogs.microsoft.com/industry-blog/government/2018/09/11/accessibility-conformance-reports/) des informations complètes sur la conformité de l’accessibilité pour tous les produits Microsoft. Recherchez « Azure Maps web » pour trouver le document spécial pour le SDK web Azure Maps. 

## <a name="navigating-the-map"></a>Navigation dans la carte

Il existe plusieurs façons d’effectuer des zooms, des panoramiques, des rotations et des tangages de la carte. Les informations suivantes détaillent les différentes façons de naviguer dans la carte.

**Faire un zoom sur la carte**

- À l’aide de la souris, double-cliquez sur la carte pour faire un zoom avant d’un niveau.
- Faites tourner la roulette de la souris pour faire un zoom sur la carte.
- À l’aide d’un écran tactile, touchez la carte avec deux doigts et rapprochez-les pour faire un zoom arrière ou écartez-les pour faire un zoom avant.
- À l’aide d’un écran tactile, appuyez deux fois sur la carte pour faire un zoom avant d’un niveau.
- Quand le focus est placé sur la carte, utilisez le signe plus (`+`) ou le signe égal (`=`) pour faire un zoom avant d’un niveau.
- Quand le focus est placé sur la carte, utilisez le signe moins, le trait d’union (`-`) ou le trait de soulignement (`_`) pour faire un zoom arrière d’un niveau.
- Utilisez le contrôle de zoom avec une souris, une interaction tactile ou les touches Tab/Entrée du clavier.
- Maintenez le bouton `Shift` enfoncé et appuyez sur la carte à l’aide du bouton gauche de la souris, puis faites glisser pour dessiner une zone dans laquelle faire un zoom sur la carte.
- En utilisant des pavés tactiles multipoints, faites glisser deux doigts vers le haut pour effectuer un zoom arrière, ou vers le bas pour effectuer un zoom avant.

**Effectuer un panoramique sur la carte**

- À l’aide de la souris, appuyez sur la carte avec le bouton gauche et faites-la glisser dans n’importe quelle direction.
- À l’aide d’un écran tactile, touchez la carte et faites-la glisser dans n’importe quelle direction.
- Quand le focus est placé sur la carte, utilisez les touches de direction pour la déplacer.

**Faire pivoter la carte**

- À l’aide de la souris, appuyez sur la carte avec le bouton droit et faites glisser le pointeur vers la gauche ou vers la droite. 
- À l’aide d’un écran tactile, touchez la carte avec deux doigts et effectuez une rotation.
- Quand le focus est placé sur la carte, utilisez la touche Maj et les touches de direction gauche ou droite.
- Utilisez le contrôle de rotation avec une souris, une interaction tactile ou les touches Tab/Entrée du clavier.

**Incliner la carte**

- À l’aide de la souris, appuyez sur la carte avec le bouton droit et faites glisser le pointeur vers le haut ou vers le bas. 
- À l’aide d’un écran tactile, touchez la carte avec deux doigts et faites-les glisser ensemble vers le haut ou vers le bas.
- Quand le focus est placé sur la carte, utilisez la touche Maj et les touches de direction haut ou bas. 
- Utilisez le contrôle de tangage avec une souris, une interaction tactile ou les touches Tab/Entrée du clavier.

## <a name="change-the-map-style"></a>Modifier le style de carte

Certains développeurs ne veulent pas nécessairement que tous les styles de carte possibles soient disponibles dans leur application. Si le développeur affiche le contrôle sélecteur de style de la carte, l’utilisateur peut changer le style de carte à l’aide de la souris, d’une interaction tactile ou du clavier avec la touches Tab ou Entrée. Le développeur peut spécifier les styles de carte qu’il souhaite rendre disponibles dans le contrôle sélecteur de style de carte. Le développeur peut également définir et modifier le style de carte par programmation.

**Utiliser un contraste élevé**

- Lorsque le contrôle de carte est chargé, il vérifie si le contraste élevé est activé et si le navigateur le prend en charge.
- Le contrôle de carte ne supervise pas le mode de contraste élevé de l’appareil. En cas de modification du mode de l’appareil, la carte n’est pas modifiée. Ainsi, l’utilisateur doit recharger la carte en actualisant la page.
- Lorsque le contraste élevé est détecté, le style de carte bascule automatiquement vers le contraste élevé, et tous les contrôles intégrés utilisent un style de contraste élevé. Par exemple, ZoomControl, PitchControl, CompassControl, StyleControl et d’autres contrôles intégrés utilisent un style de contraste élevé.
- Il existe deux types de contrastes élevés, clair et sombre. Si le type de contraste élevé est détectable par les contrôles de carte, le comportement de la carte est ajusté en conséquence. Si le type de contraste est clair, le style de carte grayscale_light est chargé. Si le type est indétectable ou sombre, le style high_contrast_dark est chargé.
- Si vous créez des contrôles personnalisés, il est utile de savoir si les contrôles intégrés utilisent un style de contraste élevé. Les développeurs peuvent ajouter une classe CSS sur la balise div de conteneur de carte à vérifier. Les classes CSS qui seraient ajoutées sont `high-contrast-dark` et `high-contrast-light`. Pour vérifier l’utilisation de JavaScript, utilisez :

```javascript
map.getMapContainer().classList.contains("high-contrast-dark")
```

ou utilisez :

```javascript
map.getMapContainer().classList.contains("high-contrast-light")
```

## <a name="keyboard-shortcuts"></a>Raccourcis clavier

La carte comporte un certain nombre de raccourcis clavier intégrés qui facilitent son utilisation. Ces raccourcis clavier fonctionnent quand le focus est placé sur la carte.

| Clé      | Action                            |
|----------|-----------------------------------|
| `Tab` | Naviguer dans les contrôles et les fenêtres indépendantes de la carte. |
| `ESC` | Déplacer le focus d’un élément de la carte vers l’élément cartographique de niveau supérieur. |
| `Ctrl` + `Shift` + `D` | Activer/désactiver le niveau de détail du lecteur d’écran.  |
| Touche de flèche gauche | Effectuer un panoramique de la carte de 100 pixels vers la gauche |
| Touche de direction droite | Effectuer un panoramique de la carte de 100 pixels vers la droite |
| Touche flèche bas | Effectuer un panoramique de la carte de 100 pixels vers le bas |
| Touche de direction Haut | Effectuer un panoramique de la carte de 100 pixels vers le haut |
| `Shift` + flèche haut | Augmenter le tangage de la carte de 10 degrés |
| `Shift` + flèche bas | Réduire le tangage de la carte de 10 degrés |
| `Shift` + flèche droite | Faire pivoter la carte de 15 degrés dans le sens horaire |
| `Shift` + flèche gauche | Faire pivoter la carte de 15 degrés dans le sens antihoraire |
| Signe plus (`+`) ou <sup>*</sup>signe égal (`=`) | Faire un zoom avant |
| Signe moins, trait d’union (`-`) ou <sup>*</sup>trait de soulignement (`_`) | Faire un zoom arrière | 
| `Shift` + glissement de la souris sur la carte pour dessiner la zone | Faire un zoom dans la zone |

<sup>*</sup> Ces raccourcis de touche partagent généralement la même touche sur un clavier. Ces raccourcis ont été ajoutés pour améliorer l’expérience utilisateur. Peu importe si l’utilisateur utilise la touche Maj ou non pour ces raccourcis.

## <a name="screen-reader-support"></a>Prise en charge du lecteur d’écran

Un utilisateur peut naviguer sur la carte à l’aide du clavier. Si un lecteur d’écran est utilisé, la carte informera l’utilisateur des modifications apportées à son état. Par exemple, les utilisateurs sont informés des modifications de la carte en cas de zoom ou de panoramique de celle-ci. Par défaut, la carte fournit des descriptions simplifiées qui excluent le niveau de zoom et les coordonnées du centre de la carte. L’utilisateur peut activer/désactiver le niveau de détail de ces descriptions à l’aide du raccourci clavier `Ctrl` + `Shift` + `D`.

Toutes les informations supplémentaires placées sur la carte de la base doivent avoir des informations textuelles correspondantes pour les utilisateurs d’un lecteur d’écran. Veillez à ajouter les attributs [ARIA (Accessible Rich Internet Applications)](https://www.w3.org/WAI/standards-guidelines/aria/), alt et title, le cas échéant. 

## <a name="make-popups-keyboard-accessible"></a>Rendre les fenêtres indépendantes accessibles par le biais du clavier

Un marqueur ou un symbole est souvent utilisé pour représenter une position sur la carte. Des informations supplémentaires sur la position s’affichent généralement dans une fenêtre indépendante quand l’utilisateur interagit avec le marqueur. Dans la plupart des applications, des fenêtres indépendantes s’affichent quand un utilisateur clique ou appuie sur un marqueur. Toutefois, pour cliquer et appuyer, l’utilisateur doit utiliser une souris et un écran tactile, respectivement. Une bonne pratique consiste à rendre les fenêtres indépendantes accessibles lors de l’utilisation d’un clavier. Cette fonctionnalité nécessite de créer une fenêtre contextuelle pour chaque point de données et de l’ajouter à la carte. 

L’exemple suivant charge des points d’intérêt sur la carte à l’aide d’une couche de symboles et ajoute une fenêtre indépendante à la carte pour chaque point d’intérêt. Une référence à chaque menu contextuel est stockée dans les propriétés de chaque point de données. Il est également possible de la récupérer pour un marqueur, par exemple, en cas de clic sur celui-ci. Quand le focus est placé sur la carte, l’utilisateur peut passer de l’une à l’autre des fenêtres indépendantes de la carte à l’aide de la touche Tab.

<br/>

<iframe height='500' scrolling='no' title='Créer une application accessible' src='//codepen.io/azuremaps/embed/ZoVyZQ/?height=504&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consultez la page <a href='https://codepen.io/azuremaps/pen/ZoVyZQ/'>Créer une application accessible</a> d’Azure Maps ( <a href='https://codepen.io/azuremaps'>@azuremaps</a> ) sur <a href='https://codepen.io'>CodePen</a>. </iframe>

<br/>

## <a name="additional-accessibility-tips"></a>Conseils supplémentaires sur l’accessibilité

Voici quelques conseils supplémentaires pour améliorer l’accessibilité de votre application de cartographie web.

- Si vous affichez une grande quantité de données de points interactifs sur la carte, envisagez de réduire l’encombrement et d’utiliser un clustering. 
- Assurez-vous que le rapport de contraste des couleurs entre le texte/les symboles et les couleurs d’arrière-plan est au moins de 4,5 pour 1.
- Veillez à ce que les messages de votre lecteur d’écran (attributs ARIA, alt et title) soient courts, descriptifs et explicites. Évitez les jargon et acronymes inutiles.
- Essayez d’optimiser les messages envoyés au lecteur d’écran pour fournir des informations brèves explicites, faciles à déchiffrer pour l’utilisateur. Par exemple, si vous souhaitez mettre à jour le lecteur d’écran à une fréquence élevée, par exemple lors du déplacement de la carte, envisagez d’effectuer les opérations suivantes :
    - Attendez la fin du déplacement de la carte pour mettre à jour le lecteur d’écran.
    - Limitez les mises à jour à une fois toutes les poignées de secondes. 
    - Combinez les messages de manière logique. 
- Évitez d’utiliser la couleur comme seul moyen de transmettre des informations. Utilisez du texte, des icônes ou des modèles pour compléter ou remplacer la couleur. Certaines considérations s’appliquent :
    - Si vous utilisez une couche de bulles pour afficher la valeur relative entre les points de données, envisagez de mettre à l’échelle le rayon de chaque bulle et/ou de définir la couleur des bulles. 
    - Envisagez d’utiliser une couche de symboles avec différentes icônes pour différentes catégories de métriques, telles que des triangles, des étoiles et des carrés. La couche de symboles prend également en charge la mise à l’échelle de la taille de l’icône. Une étiquette de texte peut également être affichée.
    - Si vous affichez des données de ligne, la largeur peut être utilisée pour représenter la pondération ou la taille. Un modèle de tableau de tirets peut être utilisé pour représenter différentes catégories de lignes. Une couche de symboles peut être utilisée en combinaison avec une ligne pour superposer les icônes le long de la ligne. L’utilisation d’une icône représentant une flèche est utile pour indiquer le flux ou la direction de la ligne.
    - Si vous affichez des données de polygone, vous pouvez utiliser un modèle, tel que des bandes, à la place de la couleur. 
- Certaines visualisations telles que les cartes thermiques, les couches de mosaïques et les couches d’images, ne sont pas accessibles aux utilisateurs atteints de troubles de la vision. Certaines considérations s’appliquent :
    - Faites en sorte que le lecteur d’écran décrive ce que la couche affiche quand elle est ajoutée à la carte. Par exemple, si une couche de mosaïques de radar météo s’affiche, faites en sorte que le lecteur d’écran indique « Les données de radar météo sont superposées sur la carte ».
- Limitez la quantité de fonctionnalités qui nécessitent un pointage avec la souris. Ces fonctionnalités sont inaccessibles aux utilisateurs qui utilisent un clavier ou un appareil tactile pour interagir avec votre application. Notez qu’il est toujours conseillé d’avoir un style de survol pour du contenu interactif, tel que des boutons, des liens et des icônes cliquables.
- Essayez de naviguer dans votre application à l’aide du clavier. Assurez-vous que l’ordre de tabulation est logique.
- Si vous créez des raccourcis clavier, essayez de les limiter à deux touches. 

## <a name="next-steps"></a>Étapes suivantes

Découvrez l’accessibilité dans les modules du SDK web.

> [!div class="nextstepaction"]
> [Accessibilité des outils de dessin](drawing-tools-interactions-keyboard-shortcuts.md)

Découvrez le développement d’applications accessibles avec Microsoft Learn :

> [!div class="nextstepaction"]
> [Parcours d’apprentissage de l’écusson numérique Accessibility in Action](https://ready.azurewebsites.net/learning/track/2940)

Jetez un coup d’œil à ces outils d’accessibilité utiles :
> [!div class="nextstepaction"]
> [Développement d’applications accessibles](https://developer.microsoft.com/windows/accessible-apps)

> [!div class="nextstepaction"]
> [Vue d’ensemble de WAI-ARIA](https://www.w3.org/WAI/standards-guidelines/aria/)

> [!div class="nextstepaction"]
> [Outil d’évaluation de l’accessibilité du web (WAVE)](https://wave.webaim.org/)

> [!div class="nextstepaction"]
> [Vérificateur de contraste de couleur WebAim](https://webaim.org/resources/contrastchecker/)

> [!div class="nextstepaction"]
> [Simulateur de vision NoCoffee](https://chrome.google.com/webstore/detail/nocoffee/jjeeggmbnhckmgdhmgdckeigabjfbddl?hl=en-US)
