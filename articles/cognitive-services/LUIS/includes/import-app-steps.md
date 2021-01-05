---
title: Procédure d’importation d’application
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: include
ms.date: 10/16/2020
ms.openlocfilehash: b70a22e502de5441b5c91a8f33357375834f54e1
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130733"
---
1. Sur le [portail LUIS](https://www.luis.ai), dans la page **Mes applications** , sélectionnez **+Nouvelle application de conversation** , puis **Importer en tant que JSON**. Recherchez le fichier JSON enregistré à l’étape précédente. Vous n’avez pas besoin de modifier le nom de l’application. Sélectionnez **Terminé**

1. Dans la section **Gérer** , sous l’onglet **Versions** , sélectionnez la version `0.1`, puis choisissez **Cloner** pour cloner la version. Attribuez-lui ensuite un nouveau nom de `ml-entity`, puis sélectionnez **Terminer** pour terminer le processus de clonage. Étant donné que le nom de la version est utilisé dans le cadre de la route d’URL, il ne peut pas contenir de caractères qui ne sont pas valides dans une URL.

    > [!TIP]
    > Le clonage dans une nouvelle version est considéré comme une bonne pratique avant la modification de votre application. Quand vous en avez terminé avec la modification d’une version, exportez-la (sous forme de fichier .json ou .lu), puis archivez le fichier dans votre système de contrôle de code source.

1. Sélectionnez **Générer** en haut de l’écran, puis cliquez sur **Intentions** dans le menu de navigation de gauche. Vous verrez les intentions de l’application, qui sont les principaux modules d’une application LUIS.
