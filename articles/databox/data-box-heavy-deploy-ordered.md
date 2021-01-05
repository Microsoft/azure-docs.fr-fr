---
title: Tutoriel - Commander Azure Data Box Heavy | Microsoft Docs
description: Dans ce tutoriel, découvrez Azure Data Box Heavy, une solution hybride qui vous permet d’importer des données locales dans Azure, et comment commander Data Box Heavy.
services: databox
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 9b2b97f11c1493deca9b79907e894efbb7b9c456
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/07/2020
ms.locfileid: "87920991"
---
# <a name="tutorial-order-azure-data-box-heavy"></a>Tutoriel : Commander Azure Data Box Heavy


Azure Data Box Heavy est une solution hybride qui vous permet d’importer vos données locales dans Azure de manière rapide, simple et fiable. Vous transférez vos données vers un dispositif de stockage de 770 To (capacité utilisable approximative) fourni par Microsoft, puis vous réexpédiez l’appareil. Ces données sont ensuite chargées dans Azure.

Ce tutoriel explique comment vous pouvez commander un Azure Data Box Heavy. Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
> * Prérequis pour le Data Box Heavy
> * Commander un Data Box Heavy
> * Suivre la commande
> * Annuler la commande

## <a name="prerequisites"></a>Prérequis

Effectuez les prérequis de configuration suivants pour l’appareil et le service Data Box avant de déployer l’appareil.

### <a name="for-installation-site"></a>Pour l’emplacement d’installation

Avant de commencer, assurez-vous que :

- L’appareil passe dans les portes et les entrées standard. Toutefois, vérifiez que l’appareil peut passer dans toutes vos entrées. Les dimensions de l’appareil sont les suivantes : largeur : 26 pouces (66,04 cm) longueur : 48 pouces (121,92 cm) hauteur : 28 pouces (71,12 cm).
- Si son installation est effectuée à un étage autre que le rez-de-chaussée, vous avez besoin d’un accès par le biais d’un ascenseur ou d’une rampe pour l’appareil. L’appareil pèse environ 500 livres (226,8 kg).
- Veillez à avoir une surface plane dans le centre de données à proximité d’une connexion réseau disponible et pouvant loger un appareil avec cet encombrement.

### <a name="for-service"></a>Pour le service

[!INCLUDE [Data Box service prerequisites](../../includes/data-box-supported-subscriptions.md)]

### <a name="for-device"></a>Pour l’appareil

Avant de commencer, assurez-vous que :
- Votre appareil est déballé.
- Vous disposez d’un ordinateur hôte connecté au réseau du centre de données. Le Data Box Heavy va copier les données de cet ordinateur. Votre ordinateur hôte doit exécuter un système d’exploitation pris en charge comme décrit dans [Configuration requise pour le système Azure Data Box Heavy](data-box-system-requirements.md).
- Vous devez disposer d’un ordinateur portable équipé d’un câble RJ-45 pour vous connecter à l’interface utilisateur locale et configurer l’appareil. Utilisez l’ordinateur portable pour configurer une fois chaque nœud de l’appareil.
- Votre centre de données doit avoir un réseau haut débit. Nous vous recommandons vivement d’utiliser au minimum une connexion 10 GbE.
- Vous devez disposer d’un câble 40 Gbits/s ou 10 Gbits/s par nœud d’appareil. Choisissez des câbles compatibles avec l’interface réseau [Mellanox MCX314A-BCCT](https://store.mellanox.com/products/mellanox-mcx314a-bcct-connectx-3-pro-en-network-interface-card-40-56gbe-dual-port-qsfp-pcie3-0-x8-8gt-s-rohs-r6.html) :

    - Concernant le câble 40 Gbits/s, l’extrémité du câble côté appareil doit être QSFP+.
    - Concernant le câble 10 Gbits/s, vous devez disposer d’un câble SFP+ dont une extrémité est branchée à un commutateur 10 G, avec une carte QSFP+ à SFP+ (ou une carte QSA) pour l’extrémité qui est branchée sur l’appareil.
    - Les câbles d’alimentation sont inclus avec l’appareil.

## <a name="order-data-box-heavy"></a>Commander un Data Box Heavy

Procédez comme suit dans le portail Azure pour commander un appareil.

1. Utilisez vos informations d’identification Microsoft Azure pour vous connecter à cette URL : [https://portal.azure.com](https://portal.azure.com).
2. Sélectionnez **+ Créer une ressource**, puis recherchez *Azure Data Box*. Sélectionnez **Azure Data Box**.
    
   [![Rechercher Azure Data Box 1](media/data-box-deploy-ordered/search-azure-data-box1.png)](media/data-box-deploy-ordered/search-azure-data-box1.png#lightbox)

3. Sélectionnez **Create** (Créer).

4. Vérifiez si le service Data Box est disponible dans votre région. Entrez ou sélectionnez les informations suivantes, puis sélectionnez **Appliquer**.

    |Paramètre  |Valeur  |
    |---------|---------|
    |Abonnement     | Sélectionnez un abonnement EA, CSP ou Azure pour le service Data Box. <br> L’abonnement est lié à votre compte de facturation.       |
    |Type de transfert     | Sélectionnez **Importer vers Azure**.        |
    |Pays/région source     | Sélectionnez le pays ou la région où vos données se trouvent actuellement.         |
    |Région Azure de destination     | Sélectionnez la région Azure où vous souhaitez transférer des données.        |

    [![Sélectionner la disponibilité de la famille d’appareils Data Box](media/data-box-deploy-ordered/select-data-box-option1.png)](media/data-box-deploy-ordered/select-data-box-option1.png#lightbox)

5. Sélectionnez **Data Box Heavy**. La capacité utilisable maximale pour une commande unique est de 770 To.

    [![Sélectionner Data Box Heavy](media/data-box-heavy-deploy-ordered/select-data-box-heavy.png)

6. Dans **Commande**, indiquez les **Détails de la commande**. Entrez ou sélectionnez les informations suivantes, puis sélectionnez **Suivant**.
    
    |Paramètre  |Valeur  |
    |---------|---------|
    |Nom     | Indiquez un nom convivial pour suivre la commande. <br> Le nom peut comporter entre 3 et 24 caractères qui peuvent être des lettres, des chiffres et des traits d’union. <br> Il doit commencer et se terminer par une lettre ou un chiffre.      |
    |Resource group     | Créez-en un nouveau ou utilisez un groupe existant. <br> Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble.         |
    |Région Azure de destination     | Sélectionnez une région pour votre compte de stockage. <br> Pour plus d’informations, consultez la [disponibilité des régions](https://azure.microsoft.com/global-infrastructure/services/?products=databox).        |
    |Destination de stockage     | Choisissez un compte de stockage, des disques managés ou les deux. <br> Selon la région Azure spécifiée, sélectionnez un ou plusieurs comptes de stockage dans la liste filtrée d’un compte de stockage existant. <br>Le Data Box Heavy peut être liée à 10 comptes de stockage maximum. <br> Vous pouvez également créer un **compte de stockage blob**, **Usage général v1** ou **Usage général v2**. <br>Consultez les [comptes de stockage pris en charge avec votre appareil](data-box-heavy-system-requirements.md#supported-storage-accounts). <br>Les comptes de stockage avec des réseaux virtuels sont pris en charge. Pour autoriser le service Data Box à travailler avec des comptes de stockage sécurisés, activez les services approuvés dans les paramètres de pare-feu réseau du compte de stockage. Pour plus d’informations, consultez le guide pratique pour [Ajouter le service Azure Data Box en tant que service approuvé](../storage/common/storage-network-security.md#exceptions).|

    Si vous utilisez le compte de stockage comme destination de stockage, vous voyez s’afficher l’écran suivant :

    ![Commande Data Box Heavy pour un compte de stockage](media/data-box-heavy-deploy-ordered/order-storage-account.png)

    Si, en plus d’un compte de stockage comme destination de stockage, vous utilisez également Data Box Heavy pour créer des disques managés à partir des disques durs virtuels (VHD) locaux, vous devez fournir les informations suivantes :

    |Paramètre  |Valeur  |
    |---------|---------|
    |Groupes de ressources     | Créez un groupe de ressources si vous envisagez de créer des disques managés à partir de disques durs virtuels locaux. Vous pouvez utiliser un groupe de ressources existant uniquement si celui-ci a été créé avant la création d’une commande Data Box Heavy pour un disque managé par le service Data Box. <br> Spécifiez plusieurs groupes de ressources séparés par des points-virgules. Un maximum de 10 groupes de ressources sont pris en charge.|

    ![Commande Data Box Heavy pour un disque managé](media/data-box-heavy-deploy-ordered/order-managed-disks.png)

    Le compte de stockage spécifié pour les disques managés est utilisé comme compte de stockage intermédiaire. Le service Data Box charge les disques durs virtuels en tant qu’objets blob de pages sur le compte de stockage intermédiaire avant de convertir celui-ci en disques managés et de le déplacer vers les groupes de ressources. Pour plus d’informations, voir [Vérifier le chargement des données dans Azure](data-box-deploy-picked-up.md#verify-data-upload-to-azure).

7. Dans la zone **Adresse d’expédition**, indiquez vos nom et prénom, le nom et l’adresse postale de la société, et un numéro de téléphone valide. Sélectionnez **Valider l’adresse**. 

    Le service valide l’adresse d’expédition de disponibilité du service. Si le service est disponible pour l’adresse de livraison indiquée, vous recevez une notification à cet effet. Sélectionnez **Suivant**.

8. Dans **Détails de la notification**, indiquez les adresses de messagerie. Le service envoie des notifications par courrier électronique concernant les mises à jour de l’état de la commande aux adresses de messagerie spécifiées.

    Nous vous recommandons d’utiliser un e-mail de groupe afin de continuer à recevoir des notifications si un administrateur du groupe quitte l’entreprise.

9. Consultez le **Résumé** des informations liées à la commande, au contact, à la notification et aux conditions de confidentialité. Cochez la case correspondant à l’acceptation des conditions de confidentialité.

10. Sélectionnez **Commander**. La création d’une commande peut prendre quelques minutes.


## <a name="track-the-order"></a>Suivre la commande

Une fois la commande passée, vous pouvez suivre son état à partir du portail Azure. Accédez à votre commande Data Box Heavy, puis à **Vue d’ensemble** pour voir l’état. Le portail affiche la commande avec l’état **Commandé**.

Si l’appareil n’est pas disponible, vous recevez une notification. Si l’appareil est disponible, Microsoft identifie l’appareil à expédier et le prépare. Pendant la préparation de l’appareil, les actions suivantes se produisent :

- Des partages SMB sont créés pour chaque compte de stockage associé à l’appareil.
- Pour chaque partage, des informations d’identification d’accès (nom d’utilisateur et mot de passe) sont générées.
- Un mot de passe qui permet de déverrouiller l’appareil est également généré.
- Le Data Box Heavy est verrouillé pour empêcher tout accès non autorisé à l’appareil à tout moment.

Une fois l’appareil préparé, le portail affiche la commande avec l’état **Traité**.

![Commande Data Box Heavy traitée](media/data-box-overview/data-box-order-status-processed.png)

Ensuite, Microsoft prépare et achemine l’appareil via un transporteur régional. Vous recevez un numéro de suivi une fois l’appareil expédié. Le portail affiche la commande dont l’état est **Distribué**.

![Commande Data Box Heavy expédiée](media/data-box-overview/data-box-order-status-dispatched.png)

## <a name="cancel-the-order"></a>Annuler la commande

Pour annuler cette commande, dans le portail Azure, accédez à **Vue d’ensemble** et cliquez sur **Annuler** dans la barre de commandes.

Après avoir passé commande, vous pouvez l’annuler à tout moment tant qu’elle n’a pas été traitée.
 
Pour supprimer une commande annulée, accédez à **Vue d’ensemble** et cliquez sur **Supprimer** à partir de la barre de commandes.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Data Box Heavy, notamment concernant les points suivants :

> [!div class="checklist"]
> * Prérequis
> * Commander un Data Box Heavy
> * Suivre la commande
> * Annuler la commande

Passez au tutoriel suivant pour apprendre à configurer votre Data Box Heavy.

> [!div class="nextstepaction"]
> [Configurer votre Azure Data Box Heavy](./data-box-heavy-deploy-set-up.md)
