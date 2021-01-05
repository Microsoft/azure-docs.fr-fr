---
title: Didacticiel - Configurer une Azure Data Box | Microsoft Docs
description: Dans ce tutoriel, découvrez comment raccorder, connecter et mettre sous tension votre appareil Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 07/10/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: ac87d5040cd572635d81be51308f48a57ddd38e3
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94335466"
---
::: zone target="docs"

# <a name="tutorial-cable-and-connect-to-your-azure-data-box"></a>Tutoriel : Branchement des câbles et connexion à une Azure Data Box

::: zone-end

::: zone target="chromeless"

## <a name="cable-and-connect-to-your-device"></a>Brancher votre appareil et vous y connecter

::: zone-end

::: zone target="docs"

Ce didacticiel explique le raccordement, la connexion et le démarrage d’une Azure Data Box.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Raccordement de la Data Box
> * Connexion à la Data Box

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez passé la commande pour Azure Data Box.
    - Pour une commande d’importation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-ordered.md).
    - Pour une commande d’exportation, consultez [Tutoriel : Commander Azure Data Box](data-box-deploy-export-ordered.md)
1. Vous avez reçu votre Data Box et que l’état de la commande dans le portail est **Remis** 
    - Une étiquette d’expédition se trouve dans le sachet transparent fixé sur l’appareil sous l’étiquette actuelle. Gardez cette étiquette en lieu sûr car vous allez l’utiliser pour le retour.
    - Dans certaines régions d’Europe, l’appareil peut être envoyé par colis. Déballez l’appareil et conservez la boîte en cas de retour nécessaire.
1. Vous avez consulté les [consignes de sécurité de la Data Box](data-box-safety.md).
1. Vous avez reçu un cordon d’alimentation avec mise à la terre à utiliser avec votre périphérique de stockage de 100 To.
1. Vous disposez d’un ordinateur hôte utilisé pour copier des données sur votre Data Box (commande d’importation) ou copier des données à partir de votre Data Box (commande d’exportation). Votre ordinateur hôte doit
    - Exécuter un [système d’exploitation pris en charge](data-box-system-requirements.md)
    - Être connecté à un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE. Si vous ne disposez pas d’une connexion 10 GbE, vous pouvez utiliser une liaison de données 1 GbE. Cependant, cela a une incidence sur les vitesses de copie. 
1. Vous devez avoir accès à une surface plane sur laquelle placer la Data Box. Si vous souhaitez placer l’appareil sur un rack standard, vous avez besoin d’un emplacement 7U dans le rack de votre centre de données. Vous pouvez placer l’appareil à plat ou en position verticale dans le rack.
1. Vous disposez des câbles suivants pour connecter votre Data Box à l’ordinateur hôte.
    - Un ou plusieurs câbles Twinax 10 GbE SFP+ cuivre ou SFP+ fibre optique (à utiliser avec les interface réseau DATA 1, DATA 2). Étant donné que Data Box est équipé de cartes Mellanox ConnectX®-3 Pro EN 10GBASE-T Double port avec l’interface réseau PCI Express 3.0, les câbles qui sont compatibles avec cette interface doivent fonctionner. Par exemple, un câble CISCO SFP-H10GB-CU3M 10GBASE-CU TWINAX SFP +3M a été utilisé pour les tests internes. Pour plus d’informations, voir la [liste des câbles et commutateurs compatibles de Mellanox](https://www.mellanox.com/pdf/firmware/ConnectX3-FW-2_42_5000-release_notes.pdf).
    - Un câble réseau RJ-45 CAT 6 (à utiliser avec l’interface réseau MGMT)
    - Un câble réseau RJ-45 CAT 6A OU RJ-45 CAT 6 (à utiliser avec l’interface réseau DATA 3 et configurés en mode 10 Gbit/s ou 1 Gbit/s, respectivement)

## <a name="cable-your-device"></a>Raccordement de l'appareil

Procédez comme suit pour brancher les câbles de l’appareil.

1. Vérifiez que l’appareil n’a pas été endommagé. Si l’appareil est altéré ou gravement endommagé, ne continuez pas. Contactez immédiatement le Support Microsoft pour savoir si vous pouvez utiliser l’appareil ou si Microsoft doit vous envoyer un appareil de remplacement.
2. Transportez l’appareil à l’emplacement où vous souhaitez le mettre sous tension. Placez l’appareil sur une surface plane. L’appareil peut également être placé sur une étagère de rack standard.
3. Branchez les câbles d’alimentation et réseau. La vue arrière d’un appareil connecté avec une configuration commune est présentée ci-dessous. En fonction de votre environnement, vous pouvez choisir d’autres [options de câblage](data-box-cable-options.md).
    
    ![Vue arrière d’une Data Box raccordée](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)

    1. Branchez le câble d’alimentation à l’emplacement d’entrée étiqueté. L’autre extrémité du câble d’alimentation doit être branché sur une unité de distribution d’alimentation.
    2. Utilisez le câble RJ-45 CAT 6pour raccorder le port MGNT à un ordinateur portable.            
    3. Utilisez le câble réseau RJ-45 CAT 6A pour brancher le port DATA 3 à une extrémité. DATA 3 est configuré en tant que 10 GbE si vous utilisez un câble RJ-45 CAT 6A a et en tant que 1 GbE si vous utilisez un câble RJ-45 CAT 6.
    4. Selon les interfaces réseau que vous souhaitez connecter pour le transfert de données, utilisez jusqu’à deux câbles Twinax 10 GbE SFP+ cuivre ou SFP+ fibre optique afin de connecter les ports DATA 1 et DATA 2 respectivement. 
    5. Les autres extrémités des câbles des ports de données sont branchées sur l’ordinateur hôte via un commutateur 10 GbE.

4. Localisez le bouton d’alimentation sur le panneau de commande avant de l’appareil. Allumez l'appareil.

    ![Bouton d’alimentation de la Data Box](media/data-box-deploy-set-up/data-box-powered-door-open.png)

::: zone-end

::: zone target="chromeless"

Une fois que vous avez reçu l’appareil, vous devez le brancher et vous y connecter. 

## <a name="cable-your-device"></a>Raccordement de l'appareil

1. S’il existe des preuves que l’appareil a été trafiqué ou endommagé, arrêtez-vous. Contactez le support Microsoft pour demander un appareil de remplacement.
2. Avant de brancher votre appareil, assurez-vous d’avoir les câbles suivants :
    
    - Un cordon d’alimentation relié à la terre (inclus) de 10 A ou plus, muni d’un connecteur IEC60320 C-13 à une extrémité pour se connecter à l’appareil.
    - Un câble réseau RJ-45 CAT 6 (à utiliser avec l’interface réseau MGMT)
    - Deux câbles en cuivre Twinax SFP+ 10 GbE (à utiliser avec les interfaces réseau 10 Gbit/s DATA 1 et DATA 2)
    - Un câble réseau RJ-45 CAT 6A OU RJ-45 CAT 6 (à utiliser avec l’interface réseau DATA 3 et configurés en mode 10 Gbit/s ou 1 Gbit/s, respectivement)

3. Retirez et placez l’appareil sur une surface plane. 
    
4. Branchez l’appareil comme indiqué ci-dessous.  

    ![Vue arrière d’une Data Box raccordée](media/data-box-deploy-set-up/data-box-cabled-dhcp.png)  

    1. Connectez le câble d'alimentation à l'appareil.
    2. Utilisez le câble réseau RJ-45 CAT 6 pour connecter votre ordinateur hôte au port de gestion (MGMT) sur l'appareil. 
    3. Utilisez le câble en cuivre Twinax SFP+ pour connecter au moins une interface réseau de 10 Gbit/s (de préférence supérieure à 1 Gbit/s), DATA 1 ou DATA 2 pour les données. 
    4. Allumez l'appareil. Le bouton d’alimentation est situé sur le panneau avant de l’appareil.

::: zone-end

::: zone target="docs"


## <a name="connect-to-your-device"></a>Connexion à l’appareil

Procédez comme suit pour configurer votre appareil à l’aide de l’interface utilisateur web locale et de l’interface utilisateur du portail.

1. Configurez la carte Ethernet sur l'ordinateur portable que vous utilisez pour vous connecter à l’appareil avec l'adresse IP statique 192.168.100.5 et le sous-réseau 255.255.255.0. 
2. Connectez-vous au port MGMT de votre appareil et accédez à l’interface utilisateur web locale à l’adresse https\://192.168.100.10. Une fois l’appareil sous tension, cela peut prendre jusqu'à 5 minutes.
3. Cliquez sur **Détails** puis **accédez à la page web**.

   ![Connexion à l'interface utilisateur web locale](media/data-box-deploy-set-up/data-box-connect-local-web-ui.png)

4. Vous voyez une page de **connexion** pour l’interface utilisateur web locale. Assurez-vous que le numéro de série de l’appareil correspond dans l’interface utilisateur du portail et l’interface utilisateur web locale. L’appareil est verrouillé à ce stade.
5. Connectez-vous au [portail Azure](https://portal.azure.com).
6. Téléchargez les informations d’identification de l’appareil à partir du portail. Accédez à **Général > Détails de l’appareil**. Copiez le **mot de passe de l'appareil**. Le mot de passe est lié à une commande spécifique dans le portail. 

    ![Obtention des informations d’identification d’un appareil](media/data-box-deploy-set-up/data-box-device-credentials.png)
    
    
7. Indiquez le mot de passe que vous avez obtenu à partir du portail Azure à l’étape précédente pour vous connecter à l’interface utilisateur web locale de l’appareil. Cliquez sur **Se connecter**.
8. Sur le **tableau de bord** , assurez-vous que les interfaces réseau sont configurées. 
   - Si le protocole DHCP est activé dans votre environnement, les interfaces réseau sont configurées automatiquement. 
   - Si DHCP n’est pas activé, accédez à **Définir des interfaces réseau** et affectez des adresses IP statiques si nécessaire.

     ![Page du tableau de bord d’un appareil](media/data-box-deploy-set-up/data-box-dashboard-1.png)

Une fois que les interfaces réseau de données sont configurées, vous pouvez également utiliser l’adresse IP de n’importe quelle interface, de DATA 1 à DATA 3, pour accéder aux interfaces utilisateur locales web à `https://<IP address of a data network interface>`. 

Une fois que l’appareil est configuré, vous pouvez vous connecter aux partages de l’appareil et copier les données. 

::: zone-end

::: zone target="chromeless"

## <a name="connect-your-device"></a>Connexion de votre appareil

1. Pour obtenir le mot de passe du support, accédez à **Général > Détails du support** dans le [portail Azure](https://portal.azure.com).
2. Assignez une adresse IP statique 192.168.100.5 et le sous-réseau 255.255.255.0 à la carte Ethernet de l’ordinateur que vous utilisez pour vous connecter à Data Box. Accédez à l’interface utilisateur web locale de l’appareil à l’adresse `https://192.168.100.10`. Une fois l’appareil sous tension, la connexion peut prendre jusqu'à 5 minutes. 
3. Connectez-vous avec le mot de passe à partir du portail Azure. Une erreur vous signale un problème avec le certificat de sécurité du site web. Suivez les instructions spécifiques au navigateur pour afficher la page web.
4. Par défaut, les paramètres réseau de l’interface de données 10 Gbit/s (ou 1 Gbit/s) sont configurés en mode DHCP. Si nécessaire, vous pouvez configurer cette interface en mode statique et fournir une adresse IP. 

::: zone-end


::: zone target="docs"

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
> * Raccordement de la Data Box
> * Connexion à la Data Box

Passez au tutoriel suivant pour savoir comment copier les données.

> [!div class="nextstepaction"]
> [Copier des données sur une Azure Data Box pour une commande d’importation](./data-box-deploy-copy-data.md)

ou

> [!div class="nextstepaction"]
> [Copier des données à partir d’une Azure Data Box pour une commande d’exportation](./data-box-deploy-export-copy-data.md)

::: zone-end

