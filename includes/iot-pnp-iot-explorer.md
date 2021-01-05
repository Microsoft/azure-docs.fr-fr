---
author: ericmitt
ms.author: ericmitt
ms.service: iot-pnp
ms.topic: include
ms.date: 07/13/2020
ms.openlocfilehash: 3ab9161c006a88a254b247a921512698ed46fa95
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87351740"
---
1. Ouvrez l’explorateur Azure IoT.

1. Dans la page **Hubs IoT**, si vous n’avez pas encore ajouté de connexion à votre hub IoT, sélectionnez **+ Ajouter une connexion**. Entrez la chaîne de connexion pour le hub IoT que vous avez créé précédemment et sélectionnez **Enregistrer**.

1. Dans la page **Paramètres IoT Plug-and-Play**, sélectionnez **+ Ajouter > Dossier local** et sélectionnez le dossier *models* local dans lequel vous avez enregistré vos fichiers de modèle.

1. Dans la page **Hubs IoT**, cliquez sur le nom du hub que vous souhaitez utiliser. Vous voyez la liste des appareils enregistrés sur le hub IoT.

1. Cliquez sur l’**ID d’appareil** de l’appareil que vous avez créé précédemment.

1. Le menu de gauche affiche les différents types d’informations disponibles pour l’appareil.

1. Sélectionnez **Composants IoT Plug-and-Play** pour afficher les informations de modèle de votre appareil.

1. Vous pouvez afficher les différents composants de l’appareil, le composant par défaut et tous les composants supplémentaires. Sélectionnez un composant à utiliser.

1. Sélectionnez la page **Télémétrie**, puis sélectionnez **Démarrer** pour afficher les données de télémétrie envoyées par l’appareil pour ce composant.

1. Sélectionnez la page **Propriétés (lecture seule)** pour afficher les propriétés en lecture seule signalées pour ce composant.

1. Sélectionnez la page **Propriétés (accessibles en écriture)** pour afficher les propriétés accessibles en écriture que vous pouvez mettre à jour pour ce composant.

1. Sélectionnez une propriété par son **nom**, entrez une nouvelle valeur, puis sélectionnez **Mettre à jour la valeur souhaitée**.

1. Pour afficher la nouvelle valeur, sélectionnez le bouton **Actualiser**.

1. Sélectionnez la page **Commandes** pour afficher toutes les commandes pour ce composant.

1. Sélectionnez la commande que vous souhaitez tester, le cas échéant. Sélectionnez **Envoyer la commande** pour appeler la commande sur l’appareil. Vous pouvez voir que votre appareil répond à la commande dans la fenêtre d’invite de commandes où l’exemple de code est en cours d’exécution.
