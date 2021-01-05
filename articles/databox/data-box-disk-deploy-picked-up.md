---
title: Tutoriel sur le renvoi d’Azure Data Box Disk | Microsoft Docs
description: Dans ce tutoriel, découvrez comment retourner votre Azure Data Box Disk. Les instructions d’enlèvement varient selon la destination de retour de l’appareil.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: tutorial
ms.date: 12/10/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: d529c5900d851368d60099091bb5745660036dea
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029887"
---
::: zone target="docs"

# <a name="tutorial-return-azure-data-box-disk"></a>Tutoriel : Renvoyer un disque Azure Data Box

Ce tutoriel explique comment retourner votre Azure Data Box Disk. Les instructions d’enlèvement varient selon la destination de retour de l’appareil.

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
>
> * Expédier le disque Data Box à Microsoft

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous avez suivi le [tutoriel : Copier des données sur Azure Data Box Disk et procéder à une vérification](data-box-disk-deploy-copy-data.md).

## <a name="ship-data-box-disk-back"></a>Renvoyer un disque Data Box

::: zone-end

::: zone target="chromeless"

## <a name="return-azure-data-box-disk"></a>Renvoyer un disque Azure Data Box

::: zone-end

1. Une fois que la validation des données est terminée, déconnectez les disques. Débranchez les câbles de connexion.
2. Emballez l’ensemble des disques et câbles de connexion dans un emballage à bulles, et placez le tout dans la boîte d’expédition. Des frais peuvent s’appliquer si les accessoires sont manquants.
    - Réutilisez l’emballage de la première expédition.  
    - Nous vous recommandons d’emballer les disques avec du papier bulle pour les protéger.
    - Vérifiez qu’ils sont suffisamment serrés dans la boîte pour éviter qu’ils ne bougent.

Les étapes suivantes sont déterminées par l’endroit où vous retournez l’appareil. Les instructions sont différentes pour les États-Unis/le Canada, l’Union européenne (UE), l’Australie et les pays/régions d’Asie.

### <a name="us-or-canada"></a>[États-Unis ou Canada](#tab/in-us-or-canada)

Effectuez les étapes suivantes si le retour de l’appareil s’effectue aux États-Unis ou au Canada.

1. Utilisez l’étiquette d’expédition de retour figurant dans la pochette en plastique transparente apposée sur la boîte. Si l’étiquette est endommagée ou perdue :
    - Accédez à **Vue d’ensemble > Télécharger une étiquette d’expédition** et téléchargez une étiquette d’expédition de retour.
    - Collez l’étiquette sur l’appareil.

2. Scellez la boîte d’envoi et assurez-vous que l’étiquette de retour est visible.
3. Planifiez un enlèvement avec UPS. Pour planifier l’enlèvement :

    - Appelez l’UPS local (numéro d’appel gratuit propre à chaque pays/région).
    - Lors de votre appel, indiquez le numéro de suivi de retour mentionné sur votre étiquette imprimée.
    - Si le numéro de suivi n’est pas indiqué, UPS vous facturera des frais supplémentaires à l’enlèvement.
    - Au lieu de planifier l’enlèvement, vous pouvez également déposer le disque Data Box Disk à l’emplacement de dépôt le plus proche.


### <a name="europe-or-uk"></a>[Europe ou Royaume-Uni](#tab/in-europe-or-uk)

Effectuez les étapes suivantes si le retour de l’appareil s’effectue en Europe ou au Royaume-Uni.

1. Utilisez l’étiquette d’expédition de retour figurant dans la pochette en plastique transparente apposée sur la boîte. Si l’étiquette est endommagée ou perdue :
    - Accédez à **Vue d’ensemble > Télécharger une étiquette d’expédition** et téléchargez une étiquette d’expédition de retour.
    - Collez l’étiquette sur l’appareil.

2. Scellez la boîte d’envoi et assurez-vous que l’étiquette de retour est visible.
3. Accédez au site web DHL Express du pays ou de la région, puis sélectionnez **Planifier un enlèvement**. Sous **Avez-vous besoin de créer une étiquette d’expédition**, sélectionnez **Non** > **J’ai un numéro d’expédition DHL**.
4. Spécifiez le numéro d’expédition et cliquez sur **Planifier un enlèvement** pour choisir une date.

### <a name="australia"></a>[Australie](#tab/in-australia)

Les centres de données Azure de l’Australie ont une notification de sécurité supplémentaire. Toutes les expéditions entrantes doivent avoir une notification préalable. Effectuez les étapes suivantes pour l’enlèvement en Australie.

1. Utilisez l’étiquette d’expédition de retour et assurez-vous que le code TAU (numéro de référence) est écrit dessus. Si l’étiquette d’expédition fournie est manquante ou si vous rencontrez tout autre problème, envoyez un e-mail à [Data Box Asia Operations](mailto:adbo@microsoft.com). Indiquez le nom de la commande dans l’en-tête d’objet et les détails du problème auquel vous êtes confronté.
2. Collez l’étiquette sur le colis.
3. Réservez un enlèvement en ligne sur https://mydhl.express.dhl/au/en/schedule-pickup.html#/schedule-pickup#label-reference.

### <a name="japan"></a>[Japon](#tab/in-japan)

1. Écrivez le nom et l’adresse de votre entreprise dans la partie du bordereau de réexpédition réservée à l’expéditeur.
2. Envoyez un courrier à Quantium Solutions en utilisant le modèle d’e-mail suivant.

    ```
    To: Customerservice.JP@quantiumsolutions.com
    Subject: Pickup request for Microsoft Azure Data Box Disk｜Job Name： 
    Body: 
    - Japan Post Yu-Pack tracking number (reference number)：
    - Requested pickup date：mmdd (Select a requested time slot from below).
        a. 08：00-13：00
        b. 13：00-15：00
        c. 15：00-17：00
        d. 17：00-19：00
    ```
    - **Si l’enlèvement doit se faire à Osaka**, remplacez l’objet du modèle d’e-mail par celui-ci : `Pickup request for Microsoft Azure OSA`.
    - Si le bordereau de réexpédition de Japon Post Chakubarai était manquant, signalez-le dans cet e-mail. Quantium Solutions demandera à Japan Post d’apporter le bordereau de réexpédition à l’enlèvement.
    - Si vous avez plusieurs commandes, envoyez un e-mail pour garantir un enlèvement individuel.

3. Recevoir un e-mail de confirmation de Quantium Solutions près avoir réservé un enlèvement. L’e-mail de confirmation inclut également des informations sur le bordereau de réexpédition de Chakubarai.

Si nécessaire, vous pouvez contacter le support de Quantium Solutions (en langue japonaise) dont les coordonnées sont les suivantes : 

- E-mail : [Customerservice.JP@quantiumsolutions.com](mailto:Customerservice.JP@quantiumsolutions.com)
- Téléphone：+81 3 57 55 01 50 

### <a name="korea"></a>[Corée](#tab/in-korea)

1. Veillez à inclure le bordereau de réexpédition.
2. Pour demander un enlèvement lorsque le bordereau de réexpédition est présent :
    1. Appelez l’assistance téléphonique de *Quantium Solutions International* au 070-8231-1418 pendant les heures de bureau (du lundi au vendredi de 10 h à 17 h). Indiquez *Enlèvement Microsoft Azure* et le numéro de la demande de service pour organiser l’enlèvement.  
    2. Si l’assistance téléphonique est occupée, envoyez un e-mail à `microsoft@rocketparcel.com`, avec l’objet *Enlèvement Microsoft Azure* et le numéro de demande de service comme référence.
    3. Si le transporteur ne procède pas à l’enlèvement, appelez l’assistance téléphonique de *Quantium Solutions International* pour trouver une autre solution.
    4. Vous recevez un e-mail de confirmation pour la planification de l’enlèvement.
3. Effectuez cette étape uniquement si le bordereau de réexpédition n’est pas présent. Pour demander un enlèvement :
    1. Appelez l’assistance téléphonique de *Quantium Solutions International* au 070-8231-1418 pendant les heures de bureau (du lundi au vendredi de 10 h à 17 h). Indiquez *Enlèvement Microsoft Azure* et le numéro de la demande de service pour organiser l’enlèvement. Spécifiez que vous avez besoin d’un nouveau bordereau de réexpédition pour organiser un enlèvement. Fournissez à l’expéditeur (client) les informations du destinataire (centre de données Azure) et le numéro de référence (numéro de la demande de service). 
    2. Si l’assistance téléphonique est occupée, envoyez un e-mail à `microsoft@rocketparcel.com`, avec l’objet *Enlèvement Microsoft Azure* et le numéro de demande de service comme référence.
    3. Si le transporteur ne procède pas à l’enlèvement, appelez l’assistance téléphonique de *Quantium Solutions International* pour trouver une autre solution.
    4. Si la demande est effectuée par téléphone, vous recevez une confirmation verbale.

### <a name="singapore"></a>[Singapour](#tab/in-singapore)

1. Imprimez l’étiquette d’expédition et apposez-la sur la boîte. Si l’étiquette est endommagée ou perdue :
    - Accédez à **Vue d’ensemble > Télécharger une étiquette d’expédition** et récupérez une étiquette d’expédition de retour.
    - Collez l’étiquette sur l’appareil. Vérifiez que l’étiquette est visible.

2. Pour demander la collecte, envoyez un e-mail au service client SingPost en utilisant le modèle suivant, avec le numéro de suivi (le numéro de suivi se trouve sur l’étiquette de retour fournie dans le package remis).

    ```
    To: kadcustcare@singpost.com
    Subject: Microsoft Azure Pickup - XZ00001234567
    Body:
     a.    Requestor name
     b.    Requestor contact number
     c.    Requestor collection address
     d.    Preferred collection date
    ```

   > [!NOTE]
   > Pour les demandes de réservation reçues un jour ouvré :
   >
   > * Avant 15 h, la collecte sera effectuée le jour ouvré suivant entre 9 h et 13 h.
   > * Après 15 h, la collecte sera effectuée le jour ouvré suivant entre 14 h et 18 h.

   Si vous rencontrez des problèmes, contactez l’équipe Data Box Operations Asia à l’adresse suivante : adbo@microsoft.com. Indiquez le nom de la tâche dans l’objet du message, ainsi que le problème rencontré.

3. Remettez le colis au transporteur.

### <a name="south-africa"></a>[Afrique du Sud](#tab/in-sa)

Si vous retournez l’appareil en Afrique du Sud, procédez comme suit.

1. Apposez l’étiquette d’expédition fournie sur le colis. Cette étiquette mentionne le numéro de suivi. Si l’étiquette d’expédition est manquante, vous pouvez en télécharger une nouvelle à partir de **Vue d’ensemble > Télécharger l’étiquette d’expédition**.

2. Scellez la boîte d’envoi et assurez-vous que l’étiquette de retour est visible.

3. Demandez un code de retour à partir des opérations Azure Data Box. Un code de retour est nécessaire pour renvoyer le package au centre de données. Envoyez un e-mail à [adbops@microsoft.com](mailto:adbops@microsoft.com). Notez ce code sur l’étiquette d’expédition à côté de l’adresse de retour, afin qu’il soit clairement visible.

4. Prenez rendez-vous avec DHL pour un enlèvement en utilisant l’une des méthodes suivantes :
   * Prenez rendez-vous en ligne pour un enlèvement en accédant à [DHL Express South Africa, **Schedule a Pickup**](https://mydhl.express.dhl/za/en/schedule-pickup.html#/schedule-pickup#label-reference).
   * Envoyer un e-mail à l’adresse [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com) en utilisant le modèle suivant :

     ```output
     To: Priority.Support@dhl.com
     Subject: Pickup request for Microsoft Azure
     Body: Need pick up for the below shipment
       *  DHL tracking number: (reference number/waybill number)
       *  Requested pickup date: yyyy/mm/dd;time:HH MM
       *  Shipper contact: (company name)
       *  Contact person: 
       *  Phone number: 
       *  Full physical address: 
       *  Item to be collected: Azure Dt
     ```

    * Vous pouvez également déposer le colis au point de service DHL le plus proche.

5. Si vous rencontrez des problèmes, envoyez e-mail à l’adresse [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com), en détaillant les problèmes que vous avez rencontrés et en insérant le numéro de lettre de transport dans la ligne Objet. Vous pouvez également appeler le +27(0)119213902.

### <a name="china"></a>[Chine](#tab/in-china)

Si vous retournez l’appareil en Chine, procédez comme suit.

1. Joignez l’étiquette d’expédition fournie et collez-la sur la boîte. Cette étiquette mentionne le numéro de suivi. Si l’étiquette d’expédition est manquante, vous pouvez en télécharger une nouvelle à partir de **Vue d’ensemble > Télécharger l’étiquette d’expédition**.

2. Envoyez un e-mail au service FedEx Premier Customer Care, leur fournissant le numéro de suivi (numéro de référence de l’expédition) pour organiser l’enlèvement à l’aide du modèle de message électronique suivant :  

   ```output
   To: ying.bao@fedex.com;739951@fedex.com
   Subject: Pickup request for Microsoft Azure : Order Name
   Body: Need pick up for the below shipment
   * FedEx tracking number (reference number)
   * Requested pickup date：yyyy/mm/dd; time: HH MM
   ```

3. Recevez une confirmation par e-mail de FedEx une fois l’enlèvement réservé.  

4. Si vous rencontrez des problèmes, envoyez un e-mail à l’adresse [DL-DC-SHA@oe.21vianet.com](mailto:DL-DC-SHA@oe.21vianet.com), avec des détails sur le problème rencontré et le nom de la commande en objet.

#### <a name="premier-customer-care-contact-information"></a>Informations de contact du service Premier Customer Care

<ins>Primaire</ins>

| Informations de contact | Détails |
|---|---|
|Nom :       | Bao Ying|
|Désignation | Senior OneCall Representative |
|Téléphone :      | 400.889.6066 ext. 3693 |
|E-mail :     | [ying.bao@fedex.com](mailto:ying.bao@fedex.com) |

<ins>Sauvegarde</ins>

| Informations de contact | Détails |
|---|---|
|Nom :       | He Xun|
|Désignation | OneCall Representative |
|Téléphone :      | 400.889.6066 ext. 3603 |
|E-mail :     | [739951@fedex.com](mailto:739951@fedex.com) |

### <a name="self-managed"></a>[Autogérée](#tab/in-selfmanaged)

Si vous utilisez Data Box Disk au Japon, à Singapour, en Corée, en Europe de l'Ouest, en Afrique du Sud, en Inde ou dans le cadre d'une offre US Government et que vous avez sélectionné l'option d'expédition autogérée pendant la création de la commande, suivez ces instructions.

1. Accédez au panneau **Vue d’ensemble** de votre commande dans le portail Azure. Parcourez les instructions qui s’affichent lorsque vous sélectionnez **Planifier l’enlèvement**. Un code d’autorisation doit apparaître. Il est utilisé au moment de la remise de la commande.

2. Envoyez un e-mail à l’équipe des opérations Azure Data Box à l’aide du modèle suivant lorsque vous êtes prêt à renvoyer l’appareil.

    ```
    To: adbops@microsoft.com
    Subject: Request for Azure Data Box Disk drop-off for order: 'orderName'
    Body:
     a. Order name
     b. Contact name of the person dropping off. You will need to display a Government approved ID during the drop off.
    ```
3. L’équipe des opérations Azure Data Box vous accompagne pour organiser la remise au centre de données Azure.

---

::: zone target="docs"

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a apporté des connaissances sur les disques Azure Data Box, notamment concernant les points suivants :

> [!div class="checklist"]
>
> * Expédier le disque Data Box à Microsoft

Passez à la procédure suivante pour savoir comment vérifier le chargement de données à partir d’un disque Data Box vers un compte de stockage Azure.

> [!div class="nextstepaction"]
> [Vérifier le chargement de données à partir d’un disque Azure Data Box](./data-box-disk-deploy-upload-verify.md)

::: zone-end
