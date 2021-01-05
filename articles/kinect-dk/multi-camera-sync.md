---
title: Synchroniser plusieurs appareils Azure Kinect DK
description: Cet article présente les avantages de la synchronisation de plusieurs appareils, ainsi que la façon de configurer les appareils à synchroniser.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 02/20/2020
ms.topic: article
keywords: azure, kinect, spécifications, matériel, DK, fonctionnalités, profondeur, couleur, RVB, capteur de mouvement, contrôleur, synchronisation
ms.openlocfilehash: 7c79101de5e5455ae2ff9fd8b5d8369a3832631c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91361158"
---
# <a name="synchronize-multiple-azure-kinect-dk-devices"></a>Synchroniser plusieurs appareils Azure Kinect DK

Chaque appareil Azure Kinect DK comprend des ports de synchronisation 3,5 mm (**Sync in** et **Sync out**) que vous pouvez utiliser pour lier plusieurs appareils. Une fois les appareils connectés, votre logiciel peut coordonner la synchronisation de leurs déclencheurs. 

Cet article explique comment connecter et synchroniser les appareils.

## <a name="benefits-of-using-multiple-azure-kinect-dk-devices"></a>Avantages de l’utilisation de plusieurs appareils Azure Kinect DK

Il existe de nombreuses raisons d’utiliser plusieurs appareils Azure Kinect DK, notamment les suivantes :

- Remplir les occlusions. Bien que les transformations de données d’Azure Kinect DK produisent une image unique, les deux caméras (profondeur et RVB) sont en fait légèrement écartées l’une de l’autre. Cet écart peut entraîner des occlusions. Une occlusion se produit quand un objet au premier plan bloque la vue d’une partie d’un objet à l’arrière-plan pour l’une des deux caméras d’un appareil. Dans l’image en couleurs obtenue, l’objet au premier plan semble projeter une ombre sur l’objet à l’arrière-plan.  
   Par exemple, dans le diagramme suivant, la caméra de gauche voit le pixel gris « P2 ». Toutefois, l’objet blanc au premier plan bloque le faisceau IR de la caméra de droite. La caméra de droite ne reçoit pas de données pour « P2 ».  
   ![Diagramme montrant deux caméras focalisées sur vers le même point, l’une d’elles étant bloquée.](./media/occlusion.png)  
   Des appareils synchronisés supplémentaires peuvent fournir les données occluses.
- Analyser des objets en trois dimensions.
- Augmenter la fréquence d’images effective en la réglant sur une valeur supérieure à 30 images par seconde (FPS).
- Capturer plusieurs images en couleurs de 4 Ko de la même scène, toutes alignées dans les 100 microsecondes (&mu;s) par rapport au centre d’exposition.
- Augmenter la couverture de la caméra dans l’espace.

## <a name="plan-your-multi-device-configuration"></a>Planifier votre configuration de plusieurs appareils

Avant de commencer, veillez à consulter [Spécifications matérielles Azure Kinect DK](hardware-specification.md) et [Caméra de profondeur DK Azure Kinect](depth-camera.md).

### <a name="select-a-device-configuration"></a>Sélectionner une configuration d’appareil

Pour la configuration de votre appareil, vous pouvez adopter l’une des approches suivantes :

- **Configuration en guirlande**. Permet de synchroniser un appareil maître avec jusqu’à huit appareils subordonnés.  
   ![Diagramme montrant comment connecter des appareils Azure Kinect DK dans une configuration en guirlande.](./media/multicam-sync-daisychain.png)
- **Configuration en étoile**. Permet de synchroniser un appareil maître avec jusqu’à deux appareils subordonnés.  
   ![Diagramme montrant comment connecter des appareils Azure DK dans une configuration en étoile.](./media/multicam-sync-star.png)

#### <a name="using-an-external-sync-trigger"></a>Utilisation d’un déclencheur de synchronisation externe

Dans les deux configurations, l’appareil maître fournit le signal de déclenchement pour les appareils subordonnés. Toutefois, vous pouvez utiliser une source externe personnalisée pour le déclencheur de synchronisation. Par exemple, vous pouvez utiliser cette option pour synchroniser des captures d’images avec d’autres équipements. Dans les configurations de connexion tant en guirlande qu’en étoile, la source du déclencheur externe se connecte à l’appareil maître.

Votre source de déclencheur externe doit fonctionner de la même façon que l’appareil maître. Elle doit fournir un signal de synchronisation présentant les caractéristiques suivantes :

- Actif élevé
- Largeur d’impulsion : supérieure à 8&mu;s
- TTL/CMOS 5V
- Capacité motrice maximale : pas inférieure à 8 milliampères (mA)
- Prise en charge de fréquence : Exactement 30 FPS, 15 FPS et 5 FPS (fréquence du signal VSYNC de la caméra couleur maître)

La source du déclencheur doit envoyer le signal au port **Sync in** de l’appareil maître en utilisant un câble audio 3,5 mm. Vous pouvez utiliser un câble stéréo ou mono. L’appareil Azure Kinect DK met à la terre l’ensemble des gaines et anneaux du connecteur du câble audio. Comme illustré dans le diagramme suivant, l’appareil reçoit le signal de synchronisation uniquement de l’embout du connecteur.

![Configurations de câble pour un signal de déclenchement externe](./media/resources/camera-trigger-signal.jpg)

Pour plus d’informations sur l’utilisation de l’équipement externe, consultez [Utiliser un enregistreur Azure Kinect avec des appareils synchronisés externes](record-external-synchronized-units.md)

### <a name="plan-your-camera-settings-and-software-configuration"></a>Planifier les paramètres et la configuration logicielle de vos caméras

Pour plus d’informations sur la configuration de votre logiciel pour contrôler les caméras et utiliser les données d’image, consultez [SDK du capteur Kinect Azure](about-sensor-sdk.md).

Cette section aborde plusieurs facteurs qui affectent les appareils synchronisés (pas les appareils isolés). Votre logiciel doit tenir compte de ces facteurs.

#### <a name="exposure-considerations"></a>Considérations relatives à l’exposition
Si vous souhaitez contrôler la synchronisation précise de chaque appareil, nous vous recommandons d’utiliser un paramétrage d’exposition manuelle. Avec le paramétrage d’exposition automatique, chaque caméra couleur peut modifier de façon dynamique l’exposition réelle. Étant donné que l’exposition affecte la synchronisation, ces modifications ont pour effet de désynchroniser rapidement les caméras.

Dans la boucle de capture d’images, évitez de définir plusieurs fois le même paramètre d’exposition. N’appelez l’API qu’une seule fois si nécessaire.

#### <a name="avoiding-interference-between-multiple-depth-cameras"></a>Éviter les interférences entre les plusieurs caméras de profondeur

Quand plusieurs caméras de profondeur acquièrent des images dont les champs de vision se chevauchent, chacune d’elles doit acquérir l’image associée à son propre laser. Pour éviter que les lasers interfèrent entre eux, les captures des caméras doivent être décalées entre elles d’au minimum 160 μs.

Pour chaque capture de caméra de profondeur, le laser s’active neuf fois et est actif pendant seulement 125&mu;s à chaque fois. Le laser reste ensuite inactif pendant 14 505&mu;s ou 23 905&mu;s, selon le mode de fonctionnement. Ce comportement signifie que le point de départ pour le calcul du décalage est de 125&mu;s.

De plus, les différences entre l’horloge de la caméra et celle du microprogramme de l’appareil augmentent le décalage minimal pour le porter à 160&mu;s. Pour calculer un décalage plus précis pour votre configuration, notez le mode de profondeur que vous utilisez et consultez le [tableau de synchronisation brute du capteur de profondeur](hardware-specification.md#depth-sensor-raw-timing). Les données de ce tableau vous permettent de calculer le décalage minimal (temps d’exposition de chaque caméra) à l’aide de l’équation suivante :

> *Temps d’exposition* = (*Impulsions IR* &times; *Largeur d’impulsion*) + (*Périodes d’inactivité* &times; *Temps d’inactivité*)

En utilisant un décalage de 160&mu;s, vous pouvez configurer jusqu’à neuf caméras de profondeur supplémentaires de façon à ce que chaque laser s’allume alors que les autres sont inactifs.

Dans votre logiciel, utilisez ```depth_delay_off_color_usec``` ou ```subordinate_delay_off_master_usec``` pour vous assurer que chaque laser IR s’exécute dans sa fenêtre de 160&mu;s ou ait un champ de vision différent.

## <a name="prepare-your-devices-and-other-hardware"></a>Préparer vos appareils et autres composants matériels

En plus de plusieurs appareils Azure Kinect DK, il se peut que vous deviez obtenir des ordinateurs hôtes et d’autres composants matériels supplémentaires afin de prendre en charge la configuration que vous souhaitez créer. Utilisez les informations de cette section pour vous assurer que tous les appareils et composants matériels sont prêts avant de commencer la configuration.

### <a name="azure-kinect-dk-devices"></a>Appareils Azure Kinect DK

Pour chaque appareil Azure Kinect DK à synchroniser, procédez comme suit :

- Assurez-vous que le microprogramme le plus récent est installé sur l’appareil. Pour plus d’informations sur la mise à jour de vos appareils, accédez à [Mettre à jour le microprogramme Azure Kinect DK](update-device-firmware.md). 
- Retirez le cache de l’appareil pour voir les ports de synchronisation.
- Notez le numéro de série de chaque appareil. Vous l’utiliserez plus tard dans le processus d’installation.

### <a name="host-computers"></a>Ordinateurs hôtes

En règle générale, chaque appareil Azure Kinect DK utilise son propre ordinateur hôte. Vous pouvez utiliser un contrôleur d’hôte dédié, en fonction de la façon dont vous utilisez l’appareil et de la quantité de données transférées via la connexion USB. 

Assurez-vous que le Kit de développement logiciel (SDK) du capteur Azure Kinect est installé sur chaque ordinateur hôte. Pour plus d’informations sur l’installation du Kit de développement logiciel (SDK) du capteur, consultez le [Guide de démarrage rapide : Configurer Azure Kinect DK](set-up-azure-kinect-dk.md). 

#### <a name="linux-computers-usb-memory-on-ubuntu"></a>Ordinateurs Linux : mémoire USB sur Ubuntu

Par défaut, les ordinateurs hôtes basés sur Linux n’allouent au contrôleur USB que 16 Mo de mémoire du noyau pour gérer les transferts USB. Cette quantité est généralement suffisante pour prendre en charge un appareil Azure Kinect DK. En revanche, pour prendre en charge plusieurs appareils, le contrôleur USB doit avoir plus de mémoire. Pour augmenter la mémoire, procédez comme suit :

1. Modifiez /**etc/default/grub**.
1. Recherchez la ligne suivante :
   ```cmd
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
   ```
   Remplacez-la par cette ligne :
   ```cmd
   GRUB_CMDLINE_LINUX_DEFAULT="quiet splash usbcore.usbfs_memory_mb=32"
   ```
   > [!NOTE]  
   > Ces commandes définissent la mémoire USB sur 32 Mo. Voici un exemple de paramétrage sur deux fois la valeur par défaut. Vous pouvez définir une valeur sensiblement plus élevée en fonction de votre solution.
1. Exécutez **sudo update-grub**.
1. Redémarrez l'ordinateur.

### <a name="cables"></a>Câbles

Pour connecter les appareils entre eux et aux ordinateurs hôtes, vous devez utiliser des câbles 3,5 mm mâle-mâle (également appelés câbles audio 3,5 mm). Les câbles doivent avoir une longueur inférieure à 10 mètres, et peuvent être stéréo ou mono.

Le nombre de câbles que vous devez avoir dépend du nombre d’appareils que vous utilisez, ainsi que de la configuration d’appareil spécifique. Le boîtier Azure Kinect DK n’inclut pas de câbles. Vous devez les acheter séparément.

Si vous connectez les appareils dans une configuration en étoile, vous devez également disposer d’un répartiteur de casque.

## <a name="connect-your-devices"></a>Connecter vos appareils

**Pour connecter des appareils Azure Kinect DK dans une configuration de connexion en guirlande**

1. Connectez chaque Azure Kinect DK à l’alimentation.
1. Connectez chaque appareil à son PC hôte. 
1. Sélectionnez un appareil maître et branchez un câble audio 3,5 mm à son port **Sync out**.
1. Branchez l’autre extrémité du câble au port **Sync in** du premier appareil subordonné.
1. Pour connecter un autre appareil, branchez un autre câble au port **Sync out** du premier appareil subordonné, puis au port **Sync out** de l’appareil suivant.
1. Répétez l’étape précédente jusqu’à ce que tous les appareils soient connectés. Le dernier appareil ne doit avoir qu’une seule connexion câblée. Son port **Sync out** doit être libre.

**Pour connecter des appareils Azure Kinect DK dans une configuration en étoile**

1. Connectez chaque Azure Kinect DK à l’alimentation.
1. Connectez chaque appareil à son PC hôte. 
1. Sélectionnez un appareil comme maître et branchez l’extrémité du répartiteur de casque à son port **Sync out**.
1. Connectez les câbles audio 3,5 mm aux extrémités « Split » du répartiteur de casque.
1. Branchez l’autre extrémité de chaque câble au port **Synch in** de l’un des appareils subordonnés.

## <a name="verify-that-the-devices-are-connected-and-communicating"></a>Vérifier que les appareils sont connectés et communiquent

Pour vérifier que les appareils sont connectés correctement, utilisez la [Visionneuse Kinect Azure](azure-kinect-viewer.md). Répétez cette procédure si nécessaire pour tester chaque appareil subordonné combiné avec l’appareil maître.

> [!IMPORTANT]  
> Pour cette procédure, vous devez connaître le numéro de série de chaque Azure Kinect DK.

1. Ouvrez deux instances de la visionneuse Kinect Azure.
1. Sous **Ouvrir l’appareil**, sélectionnez le numéro de série de l’appareil subordonné à tester.  
   ![Ouvrir l’appareil](./media/open-devices.png)
   > [!IMPORTANT]  
   > Pour obtenir un alignement précis de la capture d’images entre tous les appareils, vous devez démarrer l’appareil maître en dernier.  
1. Sous **Synchronisation externe**, sélectionnez **Sous**.  
   ![Démarrage de caméra subordonnée](./media/sub-device-start.png)
1.  Sélectionnez **Démarrer**.  
    > [!NOTE]  
    > Étant donné qu’il s’agit d’un appareil subordonné, Azure Kinect Viewer n’affiche pas d’image après le démarrage de l’appareil. Aucune image n’est affichée tant que l’appareil subordonné n’a pas reçu de signal de synchronisation de l’appareil maître.
1. Une fois l’appareil subordonné démarré, utilisez l’autre instance de la Visionneuse Azure Kinect pour ouvrir l’appareil maître.
1. Sous **Synchronisation externe**, sélectionnez **Maître**.
1. Sélectionnez **Démarrer**.

Lors du démarrage de l’appareil Azure Kinect maître, les deux instances de la Visionneuse Azure Kinect doivent afficher des images.

## <a name="calibrate-the-devices-as-a-synchronized-set"></a>Étalonner les appareils en tant qu’ensemble synchronisé

Une fois que vous avez vérifié que les appareils communiquent correctement, vous êtes prêt à les étalonner pour produire des images dans un domaine unique.

Dans un appareil, les caméras de profondeur et RVB sont calibrées en usine pour fonctionner ensemble. Cependant, lorsque plusieurs appareils doivent fonctionner ensemble, ils doivent être étalonnés pour déterminer comment transformer une image du domaine de la caméra qui l’a capturée dans le domaine de la caméra que vous souhaitez utiliser pour traiter les images.

Il existe plusieurs options pour l’étalonnage croisé d’appareils. Microsoft fournit l’[exemple de code GitHub green screen](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/green_screen) qui utilise la méthode OpenCV. Le fichier LisezMoi de cet exemple de code fournit des détails et instructions supplémentaires pour l’étalonnage des appareils.

Pour plus d’informations sur l’étalonnage, consultez [Utiliser les fonctions d’étalonnage d’Azure Kinect](use-calibration-functions.md).

## <a name="next-steps"></a>Étapes suivantes

Après avoir configuré les appareils synchronisés, vous pouvez également apprendre à utiliser une
> [!div class="nextstepaction"]
> [API d’enregistrement et de lecture du Kit de développement logiciel (SDK) du capteur Azure Kinect](record-playback-api.md)

## <a name="related-topics"></a>Rubriques connexes

- [À propos du Kit de développement logiciel (SDK) du capteur Azure Kinect](about-sensor-sdk.md)
- [Spécifications matérielles pour Azure Kinect DK](hardware-specification.md) 
- [Démarrage rapide : Configurer Azure Kinect DK](set-up-azure-kinect-dk.md) 
- [Mettre à jour le microprogramme Azure Kinect DK](update-device-firmware.md) 
- [Réinitialiser Azure Kinect DK](reset-azure-kinect-dk.md) 
- [Visionneuse Azure Kinect](azure-kinect-viewer.md) 
