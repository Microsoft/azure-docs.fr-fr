---
title: Fichier include
description: Fichier include
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 10/06/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 2f3e4bf640b8da31a7fa4d818b94b0372d3026b8
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96763417"
---
L’exemple d’application comprend deux appareils simulés et une passerelle IoT Edge. Les tutoriels suivants présentent deux approches permettant d’expérimenter et de comprendre les fonctionnalités de la passerelle :

* Créer la passerelle IoT Edge sur une machine virtuelle Azure et connecter une caméra simulée.
* Créer la passerelle IoT Edge sur un appareil réel, tel qu’un processeur Intel NUC, et connecter une caméra réelle.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Utiliser le modèle d’application d’analytique vidéo Azure IoT Central pour créer une application de magasin de vente au détail
> * Vérifier les paramètres d’application
> * Créer un modèle d’appareil pour un appareil de passerelle IoT Edge
> * Ajouter un appareil de passerelle à votre application IoT Central

## <a name="prerequisites"></a>Prérequis

Pour réaliser cette série de tutoriels, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un sur la [page d’inscription à Azure](https://aka.ms/createazuresubscription).
* Si vous utilisez une caméra réelle, il vous faut une connectivité entre l’appareil IoT Edge et la caméra, de même que le canal **Real Time Streaming Protocol**.

## <a name="initial-setup"></a>Configuration initiale

Dans ces tutoriels, vous mettez à jour et utilisez plusieurs fichiers de configuration. Les versions initiales de ces fichiers sont disponibles dans le référentiel GitHub [LVA-gateway](https://github.com/Azure/live-video-analytics/tree/master/ref-apps/lva-edge-iot-central-gateway). Ce dépôt comprend un fichier texte de [bloc-notes](https://github.com/Azure/live-video-analytics/blob/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt) que vous pouvez télécharger et utiliser pour enregistrer les valeurs de configuration des services que vous déployez. Ce fichier vous sera utile pour effectuer des étapes ultérieures dans les tutoriels.

Créez un dossier appelé *lva-configuration* sur votre ordinateur local pour enregistrer les copies de ces fichiers. Cliquez ensuite avec le bouton droit sur chacun des liens suivants, puis sélectionnez **Enregistrer sous** pour enregistrer le fichier dans le dossier *lva-configuration* :
