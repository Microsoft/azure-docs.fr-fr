---
title: Outil d’inspection ArrInspector
description: Manuel de l’utilisateur de l’outil ArrInspector
author: florianborn71
ms.author: flborn
ms.date: 03/09/2020
ms.topic: article
ms.openlocfilehash: 300e0ff26d643ae0263d21e604cb26da37a18841
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723823"
---
# <a name="the-arrinspector-inspection-tool"></a>Outil d’inspection ArrInspector

ArrInspector est un outil web utilisé pour inspecter une session Azure Remote Rendering en cours d’exécution. Il est destiné à être utilisé à des fins de débogage, pour inspecter la structure de la scène affichée, afficher les messages du journal et surveiller les performances en direct sur le serveur.

![ArrInspector](./media/arr-inspector.png)

## <a name="connecting-to-the-arrinspector"></a>Connexion à l’outil ArrInspector

Une fois que vous avez obtenu le nom d’hôte (se terminant par `mixedreality.azure.com`) de votre serveur ARR, connectez-vous à l’aide de [ConnectToArrInspectorAsync](../../how-tos/frontend-apis.md#connect-to-arr-inspector). Cette fonction crée un fichier `StartArrInspector.html` sur l’appareil sur lequel l’application s’exécute. Pour lancer ArrInspector, ouvrez ce fichier à l’aide d’un navigateur (Edge, Firefox ou Chrome) sur un PC. Le fichier n’est valide que pendant 24 heures.

Si l’application qui appelle `ConnectToArrInspectorAsync` est déjà en cours d’exécution sur un PC :

* Si vous utilisez l’intégration Unity, vous pouvez la lancer automatiquement.
* Dans le cas contraire, le fichier se trouve dans *User Folders\\LocalAppData\\[votre_application]\\AC\\Temp*.

Si l’application s’exécute sur un HoloLens :

1. Accédez à l’appareil HoloLens à l’aide du [portail d’appareil Windows](/windows/mixed-reality/using-the-windows-device-portal).
1. Accédez à *System > File Explorer*.
1. Accédez à *User Folders\\LocalAppData\\[votre_application]\\AC\\Temp*.
1. Enregistrez le fichier *StartArrInspector.html* sur votre PC.
1. Ouvrez le fichier *StartArrInspector.html* pour charger l’ArrInspector de la session.

## <a name="the-performance-panel"></a>Panneau Performances

![Panneau Performances](./media/performance-panel.png)

Ce panneau affiche les graphiques de toutes les valeurs de performances par image exposées par le serveur. Les valeurs incluent actuellement la durée de l’image, le FPS, l’utilisation de l’UC et de la mémoire, les statistiques de mémoire telles que l’utilisation globale de la RAM, le nombre d’objets, etc.

Pour visualiser l’un de ces paramètres, cliquez sur le bouton **Ajouter nouveau** et sélectionnez l’une des valeurs disponibles affichées dans la boîte de dialogue. Cette action ajoute un nouveau graphique défilant au panneau, en traçant les valeurs en temps réel. À droite, vous pouvez voir les valeurs *minimale*, *maximale* et *actuelle*.

Vous pouvez afficher un panoramique du graphique en faisant glisser son contenu à l’aide de la souris. Toutefois, le panoramique horizontal n’est possible que si ArrInspector est à l’état suspendu.

Maintenez la touche CTRL enfoncée en faisant glisser pour effectuer un zoom. Le zoom horizontal peut également être contrôlé à l’aide du curseur situé en bas.

La plage verticale est calculée par défaut en fonction des valeurs actuellement affichées, et les valeurs min. et max. sont affichées dans les zones de texte à droite. Lorsque les valeurs sont définies manuellement, soit en les saisissant directement dans la zone de texte, soit en à l’aide d’un panoramique/zoom, le graphique utilise ces valeurs. Pour restaurer le cadrage vertical automatique, cliquez sur l’icône dans le coin supérieur droit.

![plage verticale](./media/vertical-range.png)

## <a name="the-log-panel"></a>Panneau Journal

![Panneau Journal](./media/log-panel.png)

Le panneau Journal affiche une liste des messages de journal générés côté serveur. Lors de la connexion, il affiche jusqu’à 200 messages de journal précédents et en imprime de nouveaux à mesure de leur génération.

Vous pouvez filtrer la liste en fonction du type de journal `[Error/Warning/Info/Debug]` à l’aide des boutons situés en haut.
![Log Filter Buttons](./media/log-filter.png)

## <a name="the-timing-data-capture-panel"></a>Panneau de capture des données de temporisation

![Capture de données de temporisation](./media/timing-data-capture.png)

Ce panneau est utilisé pour capturer les informations de temporisation sur le serveur et les télécharger. Le fichier utilise le [format JSON Chrome Tracing](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/edit). Pour inspecter les données, ouvrez Chrome sur l’URL `Chrome://tracing` et glissez-déplacez le fichier téléchargé dans la page. Les données de temporisation sont collectées en continu dans une mémoire tampon circulaire de taille fixe. En cas d’écriture, la capture comprend uniquement des informations sur le passé immédiat, ce qui signifie de quelques secondes à quelques minutes.

## <a name="the-scene-inspection-panel"></a>Panneau Inspection de scène

![Panneau Inspection de scène](./media/scene-inspection-panel.png)

Ce panneau affiche la structure de la scène affichée. La hiérarchie d’objets se trouve à gauche, et le contenu de l’objet sélectionné se trouve à droite. Le panneau est en lecture seule et est mis à jour en temps réel.

## <a name="the-vm-debug-information-panel"></a>Panneau Informations de débogage de machine virtuelle

![Panneau Informations de débogage de machine virtuelle](./media/state-debugger-panel.png)

Ce panneau offre des fonctionnalités de débogage.

### <a name="restart-service"></a>Redémarrer le service

Le bouton **Redémarrer le service** redémarre le runtime sur la machine virtuelle à laquelle ArrInspector est connecté. Tout client attaché sera déconnecté et la page ArrInspector doit être rechargée pour se connecter au service redémarré.

### <a name="collect-debug-information"></a>Collecter des informations de débogage

Le bouton **Collecter des informations de débogage pour la machine virtuelle** ouvre une boîte de dialogue qui vous permet de déclencher l’instance ARR pour collecter des informations de débogage sur la machine virtuelle :

![Boîte de dialogue Informations de débogage de machine virtuelle](./media/state-debugger-dialog.png)

Les informations de débogage aident l’équipe Azure Remote Rendering à analyser les problèmes qui se produisent dans une instance ARR en cours d’exécution. La boîte de dialogue contient un champ de texte pour fournir des détails supplémentaires, par exemple des étapes permettant de reproduire un problème.

Après avoir cliqué sur le bouton **Commencer la collecte**, la boîte de dialogue se ferme et le processus de collecte commence. La collecte des informations sur la machine virtuelle peut prendre quelques minutes.

![Collecte des informations de débogage de la machine virtuelle en cours](./media/state-debugger-panel-in-progress.png)

Une fois la collecte terminée, vous recevrez une notification dans la fenêtre ArrInspector. Cette notification contient un ID qui identifie cette collecte particulière. Veillez à enregistrer cet ID pour le transmettre à l’équipe Azure Remote Rendering.

![Collecte des informations de débogage de la machine virtuelle réussie](./media/state-debugger-snackbar-success.png)

> [!IMPORTANT]
> Vous ne pouvez pas télécharger les informations de débogage d’une machine virtuelle ni y accéder. Seule l’équipe Azure Remote Rendering a accès aux données collectées. Vous devez nous contacter et nous envoyer l’ID de la collecte pour que nous puissions examiner le problème que vous rencontrez.

## <a name="pause-mode"></a>Mode pause

Dans l’angle supérieur droit, un commutateur vous permet de suspendre les mises à jour automatiques des panneaux. Ce mode peut être utile pour inspecter minutieusement un état spécifique.

![Mode pause](./media/pause-mode.png)

Lors de la réactivation des mises à jour automatiques, tous les panneaux sont réinitialisés.