---
title: Installer un appareil Microsoft Azure StorSimple 8600
description: Explique comment déballer, monter en rack et câbler votre appareil StorSimple 8600 avant de déployer et de configurer le logiciel.
author: alkohli
ms.assetid: 3d82ba5f-3e34-40dc-9c33-50f952bc6be8
ms.service: storsimple
ms.topic: conceptual
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: 8d118e71b29b54b034788c37916091ad8f8b996f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871811"
---
# <a name="unpack-rack-mount-and-cable-your-storsimple-8600-device"></a>Déballer, monter en rack et câbler votre appareil StorSimple 8600

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble
Microsoft Azure StorSimple 8600 est un appareil composé d’un boîtier principal et d’un boîtier EBOD. Ce didacticiel explique comment déballer, monter en rack et brancher les câbles de votre appareil StorSimple 8600 avant de configurer son logiciel.

## <a name="unpack-your-storsimple-8600-device"></a>Déballage de votre appareil StorSimple 8600
La procédure suivante explique de façon claire et détaillée comment déballer votre appareil de stockage StorSimple 8600. Cet appareil est livré dans deux emballages, l’un pour le boîtier principal et l’autre pour le boîtier EBOD. Ces deux emballages sont ensuite placés dans une seule boîte.

### <a name="prepare-to-unpack-your-device"></a>Préparation du déballage de votre appareil
Avant de déballer l’appareil, lisez les informations suivantes.

![Icône d’avertissement](./media/storsimple-safety/IC740879.png)![Icône objet lourd](./media/storsimple-8600-hardware-installation/HCS_HeavyWeight_Icon.png) **AVERTISSEMENT !**

1. Compte tenu du poids de l’appareil, deux personnes doivent être disponibles pour vous aider à le porter et à le manipuler. En effet, un boîtier complet peut peser jusqu’à 32 kg.
2. Placez le carton sur une surface plane et droite.

Ensuite, procédez comme suit pour déballer votre appareil.

#### <a name="to-unpack-your-device"></a>Pour déballer votre appareil
1. Vérifiez que le carton et le polystyrène ne comportent pas de trace d’impacts, de coupures, d’infiltrations d’eau ou tout autre type de dégâts. Si le carton ou le reste de l’emballage vous semble trop endommagé, ne l’ouvrez pas. [Contactez le support technique Microsoft](storsimple-8000-contact-microsoft-support.md) pour savoir si l’appareil est en état de marche.
2. Ouvrez la boîte externe, puis sortez les deux emballages correspondant aux boîtiers principal et EBOD. Vous pouvez maintenant déballer le boîtier principal et le boîtier EBOD. L’illustration suivante représente la vue de l’un des boîtiers déballé.
   
    ![Déballage de votre appareil de stockage](./media/storsimple-8600-hardware-installation/HCSUnpackyour4Udevice.png)
   
    **Vue de votre appareil de stockage déballé**
   
   | Étiquette | Description |
   | --- | --- |
   |   1 |Carton d’emballage |
   |   2 |Câbles SAS (dans le carton des accessoires et des câbles) |
   |   3 |Polystyrène de protection inférieur |
   |   4 |Appareil |
   |   5 |Polystyrène de protection supérieur |
   |   6 |Carton contenant les accessoires |
3. Une fois les cartons déballés, vérifiez que vous disposez des éléments suivants :
   
   * 1 boîtier principal (le boîtier principal et le boîtier EBOD sont dans deux cartons distincts)
   * 1 boîtier EBOD
   * 4 câbles d’alimentation, 2 dans chaque carton
   * 2 câbles SAS (pour raccorder le boîtier principal au boîtier EBOD)
   * 1 câble Ethernet croisé
   * 2 câbles de console série
   * 1 convertisseur de série USB pour l’accès en série
   * 4 adaptateurs QSFP-SFP+ à utiliser avec les interfaces réseau 10 Gigabit Ethernet
   * 2 kits de montage en rack (4 rails latéraux avec matériel de montage, 2 pour chaque boîtier), 1 dans chaque carton
   * Documentation de mise en route
     
     Si vous n’avez pas reçu l’un des éléments ci-dessus, [contactez le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).  

L’étape suivante consiste à monter votre appareil en rack.

## <a name="rack-mount-your-storsimple-8600-device"></a>Montage en rack de votre appareil StorSimple 8600
Respectez les étapes suivantes pour installer votre appareil de stockage StorSimple 8600 en un rack standard de 19 pouces avec poteaux avant et arrière. Cet appareil est fourni avec deux boîtiers : un boîtier principal et un boîtier EBOD. Tous deux doivent être montés en rack.

L’installation se déroule en plusieurs étapes, chacune étant décrite dans les procédures suivantes.

> [!IMPORTANT]
> Les appareils StorSimple doivent être montés en rack pour fonctionner correctement.
> 
> 

### <a name="site-preparation"></a>Préparation du site
Les boîtiers doivent être installés dans un rack standard de 19 pouces équipé de poteaux avant et arrière. Procédez comme suit pour préparer l’installation en rack.

#### <a name="to-prepare-the-site-for-rack-installation"></a>Pour préparer le site pour le montage en rack
1. Vérifiez que le boîtier principal et le boîtier EBOD sont posés en toute sécurité sur une surface de travail plate, stable et de niveau (ou similaire).
2. Vérifiez que le site où vous envisagez de monter l’appareil dispose d’une alimentation secteur standard provenant d’une source indépendante ou d’une unité de distribution de l’alimentation (PDU) en rack avec un onduleur (UPS).
3. Assurez-vous qu’un emplacement de 4U (2 x 2U) est disponible sur le rack dans lequel vous avez l’intention de monter les boîtiers.

![Icône d’avertissement](./media/storsimple-safety/IC740879.png)![Icône objet lourd](./media/storsimple-8600-hardware-installation/HCS_HeavyWeight_Icon.png) **AVERTISSEMENT !**

 Compte tenu du poids de l’appareil, deux personnes doivent être disponibles pour vous aider à le porter et à le manipuler. En effet, un boîtier complet peut peser jusqu’à 32 kg.

### <a name="rack-prerequisites"></a>Configuration requise pour le rack
Les boîtiers sont conçus pour une installation dans une armoire à rack standard de 19 pouces présentant les caractéristiques suivantes :

* Profondeur minimale de 27,84 pouces entre les poteaux du rack
* Poids maximal de 32 kg pour l’appareil
* Contre-pression maximale de 5 pascals (colonne d’eau de 0,5 mm)

### <a name="rack-mounting-rail-kit"></a>Kit du rail de montage en rack
Un ensemble de rails de montage compatible avec l’armoire à rack de 19 pouces est fourni. Les rails ont été testés pour supporter le poids maximal d’un boîtier. Ces rails permettent également d’installer plusieurs boîtiers sans perte d’espace dans le rack. Installez d’abord le boîtier EBOD.

#### <a name="to-install-the-ebod-enclosure-on-the-rails"></a>Pour installer le boîtier EBOD sur les rails
1. Effectuez cette étape uniquement si les rails internes ne sont pas installés sur votre appareil. En général, les rails internes sont installés en usine. S’ils ne le sont pas, installez les glissières du rail gauche et celles du rail droit sur les côtés du châssis du boîtier. Six vis métriques permettent de les fixer de chaque côté. Pour faciliter l’orientation, les mentions **LH – Front** (avant gauche) et **RH – Front** (avant droit) sont indiquées sur les glissières et l’extrémité qui est apposée à l’arrière du boîtier est effilée.
   
    ![Fixation des glissières de rail sur le châssis du boîtier](./media/storsimple-8600-hardware-installation/HCSAttachingRailSlidestoEnclosureChassis.png)
   
    **Fixation des glissières de rail sur les côtés du boîtier**
   
   | Étiquette | Description |
   | --- | --- |
   |  1 |M 3 x 4 vis à tête ronde |
   |  2 |Glissières du châssis |
2. Fixez le rail gauche et le rail droit sur les éléments verticaux de l’armoire à rack. Les crochets sont marqués **LH** (gauche), **RH** (droite) et **This side up** (Ce côté vers le haut) afin de vous aider à orienter correctement les éléments.
3. Recherchez les ergots à l’avant et à l’arrière du rail. Développez le rail afin qu’il soit suffisamment long pour relier les poteaux du rack, puis insérez les ergots dans les trous des éléments verticaux du poteau arrière du rack. Vérifiez que le rail est de niveau.
4. Sécurisez le rail sur l’élément vertical du rack à l’aide des deux vis métriques fournies. Utilisez l’une des vis à l’avant et l’autre à l’arrière.
5. Répétez ces étapes pour l’autre rail.
   
     ![Fixation des glissières de rail sur l’armoire à rack](./media/storsimple-8600-hardware-installation/HCSAttachingRailSlidestoRackCabinet.png)
   
    **Fixation des rails au rack**
   
   | Étiquette | Description |
   | --- | --- |
   |   1 |Vis de serrage |
   |   2 |Ouverture carrée pour vis du poteau avant du rack |
   |   3 |Ergots de positionnement du rail avant gauche |
   |   4 |Vis de serrage |
   |   5 |Ergots de positionnement du rail arrière gauche |

### <a name="mounting-the-ebod-enclosure-in-the-rack"></a>Montage du boîtier EBOD dans le rack
À l’aide des rails du rack qui viennent d’être installés, procédez comme suit pour monter le boîtier EBOD dans le rack.

#### <a name="to-mount-the-ebod-enclosure"></a>Montage du boîtier EBOD
1. Avec l’aide d’un assistant, soulevez le boîtier et alignez-le avec les rails du rack.
2. Insérez le boîtier avec précaution dans les rails, puis poussez-le complètement dans l’armoire à rack.
   
    ![Insertion de l’appareil dans le rack](./media/storsimple-8600-hardware-installation/HCSInsertingDeviceintheRack.png)
   
    **Montage du boîtier dans le rack**
3. Retirez les capuchons d’embase avant gauche et droit. Les capuchons sont simplement enfoncés sur les embases.
4. Sécurisez le boîtier dans le rack en installant une vis cruciforme fournie sur chaque embase, à gauche et à droite.
5. Placez les capuchons sur les embases et appuyez dessus pour les mettre en place.
   
     ![Installation des capuchons d’embase](./media/storsimple-8600-hardware-installation/HCSInstallingFlangeCaps.png)
   
    **Installation des capuchons d’embase**
   
   | Étiquette | Description |
   | --- | --- |
   |   1 |Vis de fixation du boîtier |

### <a name="mounting-the-primary-enclosure-in-the-rack"></a>Montage du boîtier principal dans le rack
Une fois que vous avez terminé le montage du boîtier EBOD, respectez les mêmes étapes pour monter le boîtier principal.

> [!NOTE]
> * Vous pouvez laisser quelques emplacements vides dans le rack entre le boîtier principal et le boîtier EBOD.
> * Utilisez le câble SAS de 2 m fourni pour connecter le boîtier principal au boîtier EBOD.
> * Il n’y a pas de contraintes de positionnement relatif de l’unité principale par rapport à l’unité EBOD. Par conséquent, le boîtier principal peut être placé dans l’emplacement supérieur et le boîtier EBOD dans l’emplacement inférieur ou vice versa.
> 
> 

L’étape suivante consiste à brancher l’alimentation, le réseau et l’accès en série à votre appareil.

## <a name="cable-your-storsimple-8600-device"></a>Branchement des câbles de votre appareil StorSimple 8600
Les procédures suivantes expliquent comment brancher les câbles d’alimentation, de réseau et d’accès en série sur votre appareil StorSimple 8600.

### <a name="prerequisites"></a>Conditions préalables requises
Avant de commencer à raccorder votre appareil, vous devez disposer des éléments suivants :

* Votre boîtier principal et votre boîtier EBOD, totalement déballés
* 4 câbles d’alimentation (2 par boîtier) fournis avec votre appareil
* 2 câbles SAS fournis avec l’appareil pour connecter le boîtier EBOD au boîtier principal
* Accès à 2 unités de distribution de l’alimentation (PDU) (recommandé)
* Câbles réseau
* Câbles série fournis
* Convertisseur de série USB avec le pilote approprié installé sur votre ordinateur (si nécessaire)
* 4 adaptateurs QSFP-SFP+ fournis à utiliser avec les interfaces réseau 10 Gigabit Ethernet
* [Matériel pris en charge pour les interfaces réseau 10 GbE sur votre appareil StorSimple](storsimple-supported-hardware-for-10-gbe-network-interfaces.md)

### <a name="sas-and-power-cabling"></a>Branchement des câbles d’alimentation et SAS
Votre appareil possède un boîtier principal et un boîtier EBOD. Les unités doivent donc être reliées entre elles pour l'alimentation et la connectivité SAS (Serial Attached SCSI).

Lorsque vous configurez cet appareil pour la première fois, commencez par les étapes du câblage SAS, puis effectuez les étapes pour le câblage d'alimentation.

[!INCLUDE [storsimple-cable-8600-for-SAS](../../includes/storsimple-sas-cable-8600.md)]

[!INCLUDE [storsimple-cable-8600-for-power](../../includes/storsimple-cable-8600-for-power.md)]

### <a name="network-cabling"></a>Branchement des câbles réseau
Votre appareil est dans une configuration de veille active : à un moment donné, un module de contrôleur est actif et procède au traitement de toutes les opérations de disque et du réseau pendant que l’autre module de contrôleur est en veille. En cas de défaillance du contrôleur actif, celui qui est en veille s’active immédiatement et poursuit le traitement de toutes les opérations de disque et de mise en réseau.

Pour prendre en charge ce basculement de contrôleur redondant, vous devez brancher les câbles réseau de votre appareil comme indiqué dans les étapes suivantes.

#### <a name="to-cable-for-network-connection"></a>Pour brancher les câbles de connexion réseau
1. Votre appareil possède six interfaces réseau sur chaque contrôleur : quatre ports Ethernet de 1 Gbit/s et deux de 10 Gbit/s. Reportez-vous au schéma suivant pour identifier les ports de données sur le fond de panier de votre appareil.
   
     ![Fond de panier de l’appareil 8600](./media/storsimple-8600-hardware-installation/HCSBackplaneof2UDevicewithPortsLabeled.jpg)
   
    **Dos de votre appareil avec ports de données**
   
   | Étiquette | Description |
   | --- | --- |
   |   0/1/4/5 |Interfaces réseau de 1 Gigabit Ethernet |
   |   2/3 |Interfaces réseau de 10 Gigabit Ethernet |
   |   6 |Ports série |
2. Consultez le schéma suivant pour le branchement des câbles réseau. (La configuration réseau minimale est indiquée par des lignes bleues pleines. Pour une haute disponibilité et des performances, la configuration supplémentaire requise est représentée par des lignes en pointillés.)

![Câble réseau de votre appareil 4U](./media/storsimple-8600-hardware-installation/HCSCableYour4UDeviceforNetwork.png)

**Branchement des câbles réseau de votre appareil**

| Étiquette | Description |
| --- | --- |
| Un |LAN avec accès à Internet |
| B |Contrôleur 0 |
| C |PCM 0 |
| D |Contrôleur 1 |
| E |PCM 1 |
| F |Contrôleur 0 du boîtier EBOD |
| G |Contrôleur 1 du boîtier EBOD |
| H/I |Hôtes (par exemple, les serveurs de fichiers) |
| 0-5 |Interfaces réseau |
| 6 |Boîtier principal |
| 7 |Boîtier EBOD |

Lors du câblage de l’appareil, la configuration minimale requiert les éléments suivants :

* Au moins deux interfaces réseau connectées sur chaque contrôleur : l’une pour l’accès au cloud et l’autre pour iSCSI. Le port de données DATA 0 est automatiquement activé et configuré via la console série de l’appareil. Hormis DATA 0, un autre port de données doit également être configuré via le portail Azure Classic. Dans ce cas, connectez le port DATA 0 au LAN principal (réseau avec accès à Internet). Les autres ports de données peuvent être connectés au segment SAN/iSCSI LAN (VLAN) du réseau, en fonction du rôle prévu.
* Interfaces identiques sur chaque contrôleur avec une connexion au même réseau pour garantir la disponibilité en cas de basculement d’un contrôleur. Par exemple, si vous choisissez de connecter DATA 0 et DATA 3 pour l’un des contrôleurs, vous devez connecter les ports de données correspondants DATA 0 et DATA 3 sur l’autre contrôleur.

Gardez à l’esprit la disponibilité et les performances élevées :

* Dans la mesure du possible, configurez une paire d’interface réseau pour l’accès au cloud (1 GbE) et une autre paire pour iSCSI (10 GbE recommandé) sur chaque contrôleur.
* Dans la mesure du possible, connectez les interfaces réseau de chaque contrôleur à deux commutateurs différents afin de garantir la disponibilité en cas de défaillance d’un commutateur. La figure illustre les deux interfaces réseau 10 GbE, DATA 2 et DATA 3, de chaque contrôleur connecté à deux commutateurs différents. Pour plus d’informations, reportez-vous à la section **Interfaces réseau** sous [Configuration requise pour la haute disponibilité de votre appareil StorSimple](storsimple-8000-system-requirements.md#high-availability-requirements-for-storsimple).

> [!NOTE]
> Si vous utilisez des transmetteurs SFP+ avec vos interfaces réseau de 10 GbE, utilisez les adaptateurs QSFP-SFP+ fournis. Pour plus d’informations, consultez [Matériel pris en charge pour les interfaces réseau 10 GbE sur votre appareil StorSimple](storsimple-supported-hardware-for-10-gbe-network-interfaces.md).
> 
> 

### <a name="serial-port-cabling"></a>Branchement des câbles de port série
Procédez comme suit pour brancher les câbles de port série de votre appareil.

#### <a name="to-cable-for-serial-connection"></a>Pour brancher les câbles de connexion en série
1. Votre appareil possède un port série sur chaque contrôleur, qui est identifié par une icône en forme de clé. Pour localiser les ports série, reportez-vous à l'illustration qui affiche les ports de données à l'arrière de votre appareil.
2. Identifiez le contrôleur actif sur le fond de panier de votre appareil. Un voyant bleu clignotant indique que le contrôleur est actif.
3. Utilisez le câble série fourni (si nécessaire, le convertisseur USB de série pour votre ordinateur portable) et connectez votre console ou votre ordinateur (avec émulation de terminal vers l’appareil) au port série du contrôleur actif.
4. Installez les pilotes USB de série (fournis avec l’appareil) sur votre ordinateur.
5. Configurez la connexion série comme suit :
   
   * 115 200 bauds
   * 8 bits de données
   * 1 bit d’arrêt
   * Aucune parité
   * Valeur de contrôle du flux réglée sur **Aucun**
6. Vérifiez que la connexion fonctionne en appuyant sur Entrée sur la console. Un menu de la console série doit apparaître.

> [!NOTE]
> **Gestion en service réduit :** lorsque l’appareil est installé dans un centre de données distant ou dans une salle informatique avec un accès limité, assurez-vous que les connexions série sur les deux contrôleurs sont toujours connectées à un commutateur de console série ou à un équipement similaire. Cela permet de commander à distance hors bande et de prendre en charge les opérations en cas d’interruption du réseau ou en cas de défaillance inattendue.
> 
> 

Vous avez terminé le branchement des câbles d’alimentation, d’accès réseau et de connexion en série de votre appareil. L’étape suivante consiste à configurer le logiciel sur votre appareil.

## <a name="next-steps"></a>Étapes suivantes
Vous êtes maintenant prêt à procéder au [déploiement et à la configuration de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md).

