---
title: Configuration requise pour le réseau
description: Configuration réseau requise et meilleures pratiques en matière de réseau pour une expérience optimale
author: florianborn71
ms.author: flborn
ms.date: 02/13/2020
ms.topic: reference
ms.openlocfilehash: fe684d15e2ce9b8d302db4eb6bd31dd1416abf21
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83196544"
---
# <a name="network-requirements"></a>Configuration requise pour le réseau

Une connexion réseau stable et à faible latence à un centre de données Azure est essentielle pour une bonne expérience utilisateur dans Azure Remote Rendering. De mauvaises conditions de réseau peuvent entraîner l’interruption des connexions, des hologrammes instables, tremblants ou « sautillants », et un décalage notable lors de la mise à jour du graphique de scène côté serveur.

## <a name="guidelines-for-network-connectivity"></a>Instructions pour la connectivité réseau

La configuration réseau requise exacte dépend de votre utilisation spécifique, comme le nombre et la fréquence des modifications apportées au graphique de scène distant ainsi que la complexité de l’affichage rendu, mais il existe un certain nombre d’instructions pour vous assurer que votre expérience est la meilleure possible :

* Votre connectivité Internet doit prendre en charge au moins **40 Mbits/s en aval** et **5 Mbits/s en amont** de manière constante pour une seule session utilisateur d’Azure Remote Rendering, en supposant qu’il n’y ait aucun trafic en concurrence sur le réseau. Nous vous recommandons les niveaux tarifaires plus élevés pour de meilleures expériences. Avec davantage d’utilisateurs sur le même réseau, ces exigences font l’objet d’un scale-up en conséquence.
* En règle générale, l’utilisation de la **bande Wi-Fi de 5 GHz** produira de meilleurs résultats que la bande Wi-Fi de 2,4 GHz, bien que les deux fonctionnent.
* S’il existe d’autres réseaux Wi-Fi à proximité, évitez d’utiliser des canaux Wi-Fi en cours d’utilisation par ces autres réseaux. Vous pouvez utiliser les outils d’analyse réseau comme [WifiInfoView](https://www.nirsoft.net/utils/wifi_information_view.html) pour vérifier si les canaux utilisés par votre réseau Wi-Fi sont exempts de trafic en concurrence.
* **Évitez impérativement d’utiliser des répéteurs Wi-Fi** ou un transfert LAN par courants porteurs en ligne.
* **Évitez la mise en concurrence d’un trafic gourmand en bande passante**, tel que la diffusion en continu de vidéo ou de jeux, sur le même réseau Wi-Fi.
* Il est essentiel d’avoir une **bonne intensité du signal Wi-Fi**. Si possible, restez près du point d’accès Wi-Fi et évitez de placer des obstacles entre votre appareil client et les points d’accès.
* Assurez-vous de toujours vous connecter au **centre de données Azure le plus proche** de votre [région](regions.md). Plus le centre de données est proche, plus la latence du réseau est faible, ce qui a un effet considérable sur la stabilité de l’hologramme.

> [!NOTE]
> La bande passante dans le sens descendant est principalement utilisée par le flux vidéo, qui est à son tour divisé entre les informations de couleur et de profondeur (60 Hz, stéréo pour les deux).

## <a name="network-performance-tests"></a>Tests de performances réseau

Si vous souhaitez savoir si la qualité de la connectivité réseau est suffisante pour exécuter Azure Remote Rendering, vous pouvez utiliser les outils en ligne existants. Nous vous recommandons vivement d’exécuter ces outils en ligne à partir d’un ordinateur portable raisonnablement puissant connecté au même Wi-Fi que l’appareil sur lequel vous envisagez d’exécuter votre application cliente Azure Remote Rendering. Les résultats obtenus lors de l’exécution des tests sur un téléphone mobile ou HoloLens2 sont généralement moins utiles, car il a été prouvé qu’ils présentent des variations significatives sur les appareils de point de terminaison de faible puissance. L’endroit où vous placez l’ordinateur portable doit être à peu près le même que celui où vous prévoyez d’utiliser l’appareil qui exécute votre application cliente Azure Remote Rendering.

Voici quelques étapes simples pour un test rapide de votre connectivité réseau :

1. **Exécutez un outil de test de réseau comme www.speedtest.net pour obtenir des données sur la latence globale et la bande passante en amont/en aval de votre connexion réseau.**
Choisissez le serveur le plus proche de vous et exécutez le test. Bien que le serveur ne soit pas le centre de données Azure auquel se connectera Azure Remote Rendering, les données qui en résultent sont toujours utiles pour comprendre les performances de votre connexion Internet et du Wi-Fi.
   * **Configuration minimale requise** pour Azure Remote Rendering : Environ 40 Mbits/s en aval et 5 Mbits/s en amont.
   * **Configuration recommandée** pour Azure Remote Rendering : Environ 100 Mbits/s en aval et 10 Mbits/s en amont.
Nous vous recommandons d’exécuter le test plusieurs fois et de tenir compte du pire résultat.
1. **Utilisez un outil comme www.azurespeed.com qui mesure la latence vers les centres de données Azure**. Sélectionnez le centre de données Azure pris en charge par Azure Remote Rendering le plus proche de vous (cf. [régions prises en charge](regions.md)) et exécutez un **test de latence**. S’il existe des variations dans les nombres que vous observez, accordez aux résultats un peu de temps pour qu’ils se stabilisent.
   * **Configuration minimale requise** pour Azure Remote Rendering : La latence doit toujours être inférieure à 100 ms.
   * **Configuration recommandée** pour Azure Remote Rendering : La latence doit toujours être inférieure à 70 ms.

Bien qu’une faible latence ne garantisse pas le bon fonctionnement d’Azure Remote Rendering sur votre réseau, nous avons généralement constaté qu’il fonctionnait correctement dans les situations où ces tests ont réussi.
Si vous rencontrez des artefacts tels que des hologrammes instables, tremblants ou sautillants lors de l’exécution d’Azure Remote Rendering, reportez-vous au [guide de résolution des problèmes](../resources/troubleshoot.md).

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Afficher un modèle avec Unity](../quickstarts/render-model.md)
