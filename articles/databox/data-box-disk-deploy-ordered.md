---
title: Tutoriel - Commander Azure Data Box Disk | Microsoft Docs
description: Ce tutoriel vous apprend à vous inscrire et à commander un disque Azure Data Box Disk pour importer des données dans Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 07/03/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to be able to order Data Box Disk to upload on-premises data from my server onto Azure.
ms.openlocfilehash: c83e46932e8d2a28d8503fe3fb6514988c364d97
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92125267"
---
# <a name="tutorial-order-an-azure-data-box-disk"></a>Tutoriel : Commander une solution Azure Data Box Disk

Azure Data Box Disk est une solution cloud hybride qui vous permet d’importer vos données locales dans Azure de manière rapide, simple et fiable. Vous transférez vos données vers des disques SSD fournis par Microsoft et renvoyez les disques. Ces données sont ensuite chargées dans Azure.

Ce tutoriel explique comment commander un Azure Data Box Disk. Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Commander un Data Box Disk
> * Suivre la commande
> * Annuler la commande

## <a name="prerequisites"></a>Prérequis

Avant le déploiement, effectuez les prérequis de configuration suivants pour le service Data Box et Data Box Disk.

### <a name="for-service"></a>Pour le service

[!INCLUDE [Data Box service prerequisites](../../includes/data-box-supported-subscriptions.md)]

### <a name="for-device"></a>Pour l’appareil

Avant de commencer, assurez-vous que :

* Vous avez un ordinateur client disponible à partir duquel vous pouvez copier les données. Votre ordinateur client doit :
  * Exécuter un [système d’exploitation pris en charge](data-box-disk-system-requirements.md#supported-operating-systems-for-clients)
  * Être équipé des autres [logiciels requis](data-box-disk-system-requirements.md#other-required-software-for-windows-clients) s’il s’agit d’un client Windows  

## <a name="order-data-box-disk"></a>Commander un Data Box Disk

Connectez-vous à :

* Le portail Azure à cette URL : https://portal.azure.com pour commander Data Box Disk.
* Ou bien, au portail Azure Government à cette URL : https://portal.azure.us. Pour plus d’informations, accédez à [Se connecter à Azure Government à l’aide du portail](../azure-government/documentation-government-get-started-connect-with-portal.md).

Effectuez les étapes suivantes pour commander Data Box Disk.

1. Dans le coin supérieur gauche du portail, cliquez sur **+Créer une ressource**, puis recherchez *Azure Data Box*. Cliquez sur **Azure Data Box**.

   ![Recherchez Azure Data Box 1](media/data-box-disk-deploy-ordered/search-data-box11.png)

2. Cliquez sur **Créer**.

3. Vérifiez si le service Data Box est disponible dans votre région. Saisissez ou sélectionnez les informations suivantes, puis sélectionnez **Appliquer**.

    ![Sélectionnez l’option Data Box Disk](media/data-box-disk-deploy-ordered/select-data-box-sku-1.png)

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Sélectionnez l’abonnement pour lequel le service Data Box est activé.<br> L’abonnement est lié à votre compte de facturation. |
    |Type de transfert| Importer vers Azure|
    |Pays/région source | Sélectionnez le pays ou la région où vos données se trouvent actuellement.|
    |Région Azure de destination|Sélectionnez la région Azure où vous souhaitez transférer des données.|
  
4. Sélectionnez **Data Box Disk**. La capacité maximale de la solution pour une seule commande de 5 disques est de 35 To. Vous pouvez créer plusieurs commandes pour des tailles de données supérieures.

     ![Sélectionner l’option Data Box Disk 2](media/data-box-disk-deploy-ordered/select-data-box-sku-zoom.png)

5. Dans **Commande**, indiquez les **Détails de la commande**. Entrez ou sélectionnez les informations suivantes :

    |Paramètre|Valeur|
    |---|---|
    |Nom|Indiquez un nom convivial pour suivre la commande.<br> Le nom peut comporter entre 3 et 24 caractères qui peuvent être des lettres, des chiffres et des traits d’union. <br> Il doit commencer et se terminer par une lettre ou un chiffre. |
    |Resource group| Créez-en un nouveau ou utilisez un groupe existant. <br> Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. |
    |Région Azure de destination| Sélectionnez une région pour votre compte de stockage.<br> Actuellement, les comptes de stockage de toutes les régions des États-Unis, d’Europe du Nord et de l’Ouest, du Canada et de l’Australie sont pris en charge. |
    |Taille de données estimée en To| Entrez une estimation en To. <br>Selon la taille des données, Microsoft vous envoie un nombre de disques SSD de 8 To (7 To de capacité utilisable) approprié. <br>La capacité utilisable maximale de 5 disques peut atteindre 35 To. |
    |Clé d’accès de disque| Fournissez la clé d’accès de disque si vous cochez l’option **Utiliser une clé personnalisée à la place de la clé d’accès générée par Azure**. <br> Fournissez une clé alphanumérique comprise entre 12 et 32 caractères, et contenant au moins une valeur numérique et un caractère spécial. Seuls les caractères spéciaux `@?_+` sont autorisés. <br> Vous pouvez choisir d’ignorer cette option et d’utiliser la clé d’accès générée par Azure pour déverrouiller vos disques.|
    |Destination de stockage     | Choisissez un compte de stockage, des disques managés ou les deux. <br> Selon la région Azure spécifiée, sélectionnez un compte de stockage dans la liste filtrée d’un compte de stockage existant. Data Box Disk peut être lié à 1 compte de stockage uniquement. <br> Vous pouvez également créer un **compte de stockage blob**, **Usage général v1** ou **Usage général v2**. <br>Les comptes de stockage avec des réseaux virtuels sont pris en charge. Pour autoriser le service Data Box à travailler avec des comptes de stockage sécurisés, activez les services approuvés dans les paramètres de pare-feu réseau du compte de stockage. Pour plus d’informations, découvrez comment [Ajouter Azure Data Box en tant que service approuvé](../storage/common/storage-network-security.md#exceptions).|

    Si vous utilisez le compte de stockage comme destination de stockage, vous voyez s’afficher l’écran suivant :

    ![Commande Data Box Disk pour un compte de stockage](media/data-box-disk-deploy-ordered/order-storage-account.png)

    Si vous utilisez Azure Data Box Disk pour créer des disques managés à partir de disques durs virtuels locaux, vous devez également fournir les informations suivantes :

    |Paramètre  |Valeur  |
    |---------|---------|
    |Resource group     | Créez un nouveau groupe de ressources si vous envisagez de créer des disques managés à partir de disques durs virtuels locaux. N’utilisez un groupe de ressources existant que s’il a été créé pour la commande Data Box Disk de disque managé par le service Data Box. <br> Seul un groupe de ressources est pris en charge.|

    ![Commande Data Box Disk pour un disque managé](media/data-box-disk-deploy-ordered/order-managed-disks.png)

    Le compte de stockage spécifié pour les disques managés est utilisé comme compte de stockage intermédiaire. Le service Data Box charge les disques durs virtuels sur le compte de stockage intermédiaire, les convertit en disques managés, puis passe aux groupes de ressources. Pour plus d’informations, voir [Vérifier le chargement des données dans Azure](data-box-disk-deploy-upload-verify.md#verify-data-upload-to-azure).

6. Cliquez sur **Suivant**.

    ![Fournir les détails de la commande](media/data-box-disk-deploy-ordered/data-box-order-details.png)

7. Dans l’onglet **Adresse d’expédition**, indiquez vos nom et prénom, le nom et l’adresse postale de la société et un numéro de téléphone valide. Cliquez sur **Valider l’adresse**. Le service valide l’adresse d’expédition de disponibilité du service. Si le service est disponible pour l’adresse de livraison indiquée, vous recevez une notification à cet effet.

   Une fois la commande traitée, vous recevez une notification par e-mail. Si vous avez choisi l’expédition autogérée, consultez [Utiliser l’expédition autogérée](data-box-disk-portal-customer-managed-shipping.md).

    ![Indiquer l’adresse de livraison](media/data-box-disk-deploy-ordered/data-box-shipping-address.png)
8. Dans **Détails de la notification**, indiquez les adresses de messagerie. Le service envoie des notifications par courrier électronique concernant les mises à jour de l’état de la commande aux adresses de messagerie spécifiées.

    Nous vous recommandons d’utiliser un e-mail de groupe afin de continuer à recevoir des notifications si un administrateur du groupe quitte l’entreprise.

9. Consultez le **Résumé** des informations liées à la commande, au contact, à la notification et aux conditions de confidentialité. Cochez la case correspondant à l’acceptation des conditions de confidentialité.

10. Cliquez sur **Commande**. La création d’une commande peut prendre quelques minutes.

## <a name="track-the-order"></a>Suivre la commande

Une fois la commande passée, vous pouvez suivre son état à partir du portail Azure. Accédez à votre commande, puis à **Vue d’ensemble** pour afficher l’état. Le portail affiche le travail dont l’état est **Commandé**.

![Data Box Disk - État Commandé](media/data-box-disk-deploy-ordered/data-box-portal-ordered.png)

Si les disques ne sont pas disponibles, vous recevez une notification. Si les disques sont disponibles, Microsoft identifie les disques à expédier et les prépare. Pendant la préparation du disque, les actions suivantes se produisent :

* Les disques sont chiffrés à l’aide du chiffrement AES-128 BitLocker.  
* Les disques sont verrouillés pour empêcher tout accès non autorisé aux disques.
* La clé de sécurité permettant de déverrouiller les disques est générée au cours de ce processus.

Une fois la préparation du disque terminée, le portail affiche la commande dont l’état est **Traité**.

Ensuite, Microsoft prépare et achemine les disques via un transporteur régional. Vous recevez un numéro de suivi une fois les disques expédiés. Le portail affiche la commande dont l’état est **Distribué**.

## <a name="cancel-the-order"></a>Annuler la commande

Pour annuler cette commande, dans le portail Azure, accédez à **Vue d’ensemble** et cliquez sur **Annuler** dans la barre de commandes.

L’annulation est possible uniquement lorsque les disques sont commandés, et la commande en cours de traitement en vue de l’expédition. Une fois la commande traitée, vous ne pouvez plus l’annuler.

![Annuler la commande](media/data-box-disk-deploy-ordered/cancel-order1.png)

Pour supprimer une commande annulée, accédez à **Vue d’ensemble** et cliquez sur **Supprimer** à partir de la barre de commandes.

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
>
> * Commander un Data Box Disk
> * Suivre la commande
> * Annuler la commande

Passez au tutoriel suivant pour apprendre à configurer votre Data Box Disk.

> [!div class="nextstepaction"]
> [Configurer votre Azure Data Box Disk](./data-box-disk-deploy-set-up.md)