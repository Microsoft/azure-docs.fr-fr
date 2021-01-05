---
title: Téléchargement du Kit de développement logiciel (SDK) de suivi de corps d’Azure Kinect
description: Découvrez comment télécharger chaque version du Kit de développement logiciel (SDK) de capteur Azure Kinect sur Windows ou Linux.
author: qm13
ms.author: quentinm
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: azure, kinect, sdk, mise à jour de téléchargement, dernier, disponible, installer, corps, suivi
ms.openlocfilehash: 0ac0598d893617f341b9e1fd4d45c0c3e3f3c619
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359593"
---
# <a name="download-azure-kinect-body-tracking-sdk"></a>Télécharger le Kit de développement logiciel (SDK) de suivi de corps d’Azure Kinect

Ce document fournit des liens pour installer chaque version du Kit de développement logiciel (SDK) de suivi de corps d’Azure Kinect.

## <a name="azure-kinect-body-tracking-sdk-contents"></a>Contenu du Kit de développement logiciel (SDK) de suivi de corps d’Azure Kinect

- En-têtes et bibliothèques pour créer une application à l’aide de la solution Azure Kinect DK.
- DLL redistribuables requises par les applications de suivi de corps utilisant la solution Azure Kinect DK.
- Exemples d’applications de suivi de corps.

## <a name="windows-download-links"></a>Liens de téléchargement Windows

Version       | Téléchargement
--------------|----------
1.0.1 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100942) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/1.0.1)
1.0.0 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100848) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/1.0.0)
0.9.5 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100636) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.5)
0.9.4 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100415) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.4)
0.9.3 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100307) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.3)
0.9.2 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100128) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.2)
0.9.1 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=100063) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.1)
0.9.0 | [msi](https://www.microsoft.com/en-us/download/details.aspx?id=58402) [nuget](https://www.nuget.org/packages/Microsoft.Azure.Kinect.BodyTracking/0.9.0)

## <a name="linux-installation-instructions"></a>Instructions d’installation Linux

Actuellement, la seule distribution prise en charge est Ubuntu 18.04. Pour demander la prise en charge d’autres distributions, consultez [cette page](https://aka.ms/azurekinectfeedback).

Tout d’abord, vous devez configurer le [dépôt du package Microsoft](https://packages.microsoft.com/), en suivant les instructions reprises [ici](/windows-server/administration/linux-package-repository-for-microsoft-software).

Le package `libk4abt<major>.<minor>-dev` contient les en-têtes et les fichiers CMake à créer par rapport à `libk4abt`.
Le package `libk4abt<major>.<minor>` contient les objets partagés nécessaires pour exécuter les exécutables qui dépendent de `libk4abt`, ainsi que l’exemple de visionneuse.

Les didacticiels de base nécessitent le package `libk4abt<major>.<minor>-dev`. Pour l'installer, exécutez

`sudo apt install libk4abt1.0-dev`

Si la commande est réussie, le Kit de développement logiciel (SDK) est prêt à être utilisé.

> [!NOTE]
> Lorsque vous installez le Kit de développement logiciel (SDK), rappelez-vous le chemin d’installation. Par exemple, « C:\Program Files\Azure Kinect Body Tracking SDK 1.0.0 ». Les exemples référencés dans les articles sont accessibles via ce chemin.
> Les exemples de suivi de corps se trouvent dans le dossier [Body-Tracking-Samples](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples) dans le référentiel Azure-Kinect-Samples. Les exemples référencés dans les articles se trouvent ici.

## <a name="change-log"></a>Journal des modifications

### <a name="v101"></a>v1.0.1
* [Correctif de bogue] Résolution du problème de blocage du Kit de développement logiciel (SDK) lors du chargement d’onnxruntime. dll à partir du chemin d’accès sur Windows Build 19025 ou version ultérieure : [Lien](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/932)

### <a name="v100"></a>v1.0.0
* [Fonctionnalité] Ajout d’un wrapper C# au programme d’installation msi.
* [Correctif de bogue] Résolution du problème de non-détection correcte de la rotation la tête : [Lien](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/997)
* [Correctif de bogue] Résolution du problème d’utilisation de l’UC jusqu’à 100 % sur une machine Linux : [Lien](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/issues/1007)
* [Exemples] Ajout de deux exemples au référentiel. L’exemple 1 montre comment transformer les résultats de suivi de corps de l’espace de profondeur en espace de couleurs [Lien](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples/camera_space_transform_sample) ; l’exemple 2 montre comment détecter un plan de sol [Lien](https://github.com/microsoft/Azure-Kinect-Samples/tree/master/body-tracking-samples/floor_detector_sample)

### <a name="v095"></a>v0.9.5
* [Fonctionnalité] Prise en charge de C#. Un wrapper C# est empaqueté dans le package NuGet.
* [Fonctionnalité] Prise en charge de plusieurs dispositifs de suivi. La création de plusieurs dispositifs de suivi est autorisée. Désormais, l’utilisateur peut créer plusieurs dispositifs de suivi pour suivre les corps à partir de différents appareils Azure Kinect.
* [Fonctionnalité] Prise en charge du traitement multithread pour le mode UC. En mode UC, tous les cœurs sont utilisés pour maximiser la vitesse.
* [Fonctionnalité] Ajout de `gpu_device_id` à la structure `k4abt_tracker_configuration_t`. Autorisation accordée aux utilisateurs de spécifier un appareil GPU autre que celui par défaut pour exécuter l’algorithme de suivi de corps.
* [Correctif de bogue/changement cassant] Correction de faute de frappe dans un nom de jointure. Modification du nom de jointure de `K4ABT_JOINT_SPINE_NAVAL` en `K4ABT_JOINT_SPINE_NAVEL`.

### <a name="v094"></a>v0.9.4
* [Fonctionnalité] Ajout de la prise en charge des jointures de main. Le Kit de développement logiciel (SDK) fournit des informations sur trois jointures supplémentaires pour chaque main : MAIN, BOUTMAIN, POUCE.
* [Fonctionnalité] Ajout d’un niveau de confiance de prédiction pour chaque jointure détectée.
* [Fonctionnalité] Ajout de la prise en charge du mode UC. En modifiant la valeur `cpu_only_mode` en `k4abt_tracker_configuration_t`, le Kit de développement logiciel (SDK) peut désormais s’exécuter en mode UC qui ne nécessite pas que l’utilisateur dispose d’une carte graphique puissante.

### <a name="v093"></a>v0.9.3
* [Fonctionnalité] Publication d’un nouveau modèle DNN, dnn_model_2_0.onnx, qui améliore considérablement la robustesse du suivi de corps.
* [Fonctionnalité] Désactivation du lissage temporel par défaut. Les jointures suivies seront plus réactives.
* [Fonctionnalité] Amélioration de la précision du mappage d’index de corps.
* [Correctif de bogue] Correction du bogue qui avait pour effet que le paramètre d’orientation du capteur n’était pas efficace.
* [Correctif de bogue] Remplacement du type body_index_map de K4A_IMAGE_FORMAT_CUSTOM par K4A_IMAGE_FORMAT_CUSTOM8.
* [Problème connu] Deux corps proches peuvent fusionner en un segment d’instance unique.

### <a name="v092"></a>v0.9.2
* [Changement cassant] Mise à jour pour dépendre de la dernière version du Kit de développement logiciel (SDK) de capteur Kinect 1.2.0.
* [Modification d’API] La fonction `k4abt_tracker_create` commencera à prendre une entrée `k4abt_tracker_configuration_t`. 
* [Modification d’API] Modification d’API `k4abt_frame_get_timestamp_usec` en `k4abt_frame_get_device_timestamp_usec` pour qu’elle soit plus spécifique et cohérente avec le Kit de développement logiciel (SDK) de capteur 1.2.0.
* [Fonctionnalité] Autorisation accordée aux utilisateurs de spécifier l’orientation de montage du capteur lors de la création du dispositif de suivi pour obtenir des résultats de suivi de corps plus précis lors du montage sous différents angles.
* [Fonctionnalité] Fourniture de nouvelle API `k4abt_tracker_set_temporal_smoothing` pour modifier la quantité de lissage temporel que l’utilisateur souhaite appliquer.
* [Fonctionnalité] Ajout du wrapper C++ k4abt.hpp.
* [Fonctionnalité] Ajout de l’en-tête de définition de version k4abtversion.h.
* [Correctif de bogue] Correction du bogue qui a entraînait une utilisation très élevée du processeur.
* [Correctif de bogue] Correction du bogue qui entraînait un blocage de l’enregistreur d’événements.

### <a name="v091"></a>v0.9.1
* [Correctif de bogue] Correction de la fuite de mémoire lors de la destruction du dispositif de suivi.
* [Correctif de bogue] Meilleurs messages d’erreur pour les dépendances manquantes.
* [Correctif de bogue] Échec sans blocage lors de la création d’une deuxième instance de dispositif de suivi.
* [Correctif de bogue] Les variables d’environnement de l’enregistreur d’événements fonctionnent désormais correctement.
* Prise en charge Linux

### <a name="v090"></a>v0.9.0

* [Changement cassant] Retour à la version antérieure (CUDA 10.0 à partir de CUDA 10.1) de la dépendance du Kit de développement logiciel (SDK). Le runtime ONNX ne prend officiellement que CUDA 10.0.
* [Changement cassant] Passage au runtime ONNX au lieu du runtime Tensorflow. Réduit la temps de lancement de la première image et l’utilisation de la mémoire. Réduit également la taille du fichier binaire du Kit de développement logiciel (SDK).
* [Modification d’API] Changement du nom `k4abt_tracker_queue_capture()` en `k4abt_tracker_enqueue_capture()`.
* [Modification d’API] Décomposition de `k4abt_frame_get_body()` en deux fonctions distinctes : `k4abt_frame_get_body_skeleton()` et `k4abt_frame_get_body_id()`. Vous pouvez désormais interroger l’ID de corps sans toujours copier l’intégralité de la structure du squelette.
* [Modification d’API] Ajout de la fonction `k4abt_frame_get_timestamp_usec()` pour simplifier les étapes permettant aux utilisateurs d’interroger l’horodatage de l’ossature du corps.
* Amélioration de la précision de l’algorithme de suivi de corps et de la fiabilité du suivi.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure Kinect DK](about-azure-kinect-dk.md)

- [Configurer Azure Kinect DK](set-up-azure-kinect-dk.md)

- [Configurer le suivi de corps d’Azure Kinect](body-sdk-setup.md)