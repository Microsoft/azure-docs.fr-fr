---
title: Tutoriel sur le renvoi d’Azure Data Box dans une commande d’exportation | Microsoft Docs
description: Découvrez comment expédier votre Azure Data Box à Microsoft une fois la commande d’exportation terminée
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 12/11/2020
ms.author: alkohli
ms.openlocfilehash: 1035f7fb34ed5b2b11e3ef472ccda7cccc30f652
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359028"
---
# <a name="tutorial-return-azure-data-box-preview"></a>Tutoriel : Retourner Azure Data Box (préversion)

Ce tutoriel explique comment retourner Azure Data Box et comment les données sont effacées une fois que l’appareil est réceptionné au centre de données Azure.

Ce didacticiel fournit des informations sur les sujets suivants :

> [!div class="checklist"]
>
> * Prérequis
> * Préparer l’expédition
> * Expédier la Data Box à Microsoft
> * Effacer les données de la Data Box

[!INCLUDE [Data Box feature is in preview](../../includes/data-box-feature-is-preview-info.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez les points suivants :

* Vous avez suivi le [Tutoriel : Copier des données à partir d’Azure Data Box](data-box-deploy-export-copy-data.md).
* Les tâches de copie sont terminées. La préparation de l’expédition ne peut pas s’effectuer si les tâches de copie sont en cours d’exécution.

## <a name="prepare-to-ship"></a>Préparer l’expédition

[!INCLUDE [data-box-export-prepare-to-ship](../../includes/data-box-export-prepare-to-ship.md)]

Les étapes suivantes sont déterminées par l’endroit où vous retournez l’appareil.

## <a name="ship-data-box-back"></a>Renvoyer la Data Box

Vérifiez que la copie des données à partir de l’appareil a été effectuée et que l’exécution de la **Préparation de l’expédition** a réussi. La procédure varie en fonction de la région où vous expédiez l’appareil.

## <a name="us-canada-europe"></a>[États-Unis, Canada, Europe](#tab/in-us-canada-europe)

Effectuez les étapes suivantes si vous retournez l’appareil aux États-Unis, au Canada ou en Europe.

1. Vérifiez que l’appareil est hors tension et que les câbles sont débranchés. 
2. Enroulez le cordon d’alimentation fourni avec la Data Box et fixez-le à l’arrière de l’appareil.
3. Vérifiez que l’étiquette d’expédition est affichée sur l’écran E-ink et planifiez l’enlèvement avec votre transporteur. Si l’étiquette est endommagée ou perdue, ou si elle ne s’affiche pas sur l’écran E-ink, contactez le support Microsoft. Si le support technique le suggère, vous pouvez accéder à **Vue d’ensemble > Télécharger une étiquette d’expédition** dans le portail Azure. Téléchargez l’étiquette d’expédition et collez-la sur l’appareil. 
4. Planifiez une date d’enlèvement avec UPS en cas de renvoi de l’appareil. Pour planifier l’enlèvement :

   - Appelez l’UPS local (numéro d’appel gratuit propre à chaque pays/région).
   - Lors de l’appel, indiquez le numéro de suivi du retour, qui est indiqué dans l’écran E-ink ou sur l’étiquette de retour imprimée. Si vous n’indiquez pas le numéro de suivi, UPS vous facturera des frais supplémentaires à l’enlèvement.
   - Si des problèmes surviennent pendant la planification d’un enlèvement, ou si vous êtes invité à payer des frais supplémentaires, contactez le service des opérations Azure Data Box. Envoyez un e-mail à [adbops@microsoft.com](mailto:adbops@microsoft.com).

    Au lieu de planifier l’enlèvement, vous pouvez déposer la Data Box dans le point de dépôt le plus proche.
4. Une fois que la Data Box a été scannée et récupérée par le transporteur, l’état de la commande dans le portail affiche **Récupérée**. Un ID de suivi s’affiche également.

## <a name="australia"></a>[Australie](#tab/in-australia)

Les centres de données Azure de l’Australie ont une notification de sécurité supplémentaire. Toutes les expéditions entrantes doivent avoir une notification préalable. Effectuez les étapes suivantes pour expédier en Australie.

1. Conservez la boîte d’origine utilisée pour expédier l’appareil, en cas de retour.
2. Vérifiez que la copie des données sur l’appareil a été effectuée et que la **préparation de l’expédition** a réussi.
3. Mettez l’appareil hors tension et débranchez les câbles.
4. Enroulez le cordon d’alimentation fourni avec l’appareil, et fixez-le à l’arrière de ce dernier.
5. Réservez un enlèvement en ligne via le [Lien DHL](https://mydhl.express.dhl/au/en/schedule-pickup.html#/schedule-pickup#label-reference).

## <a name="japan"></a>[Japon](#tab/in-japan)

1. Conservez la boîte d’origine utilisée pour expédier l’appareil, en cas de retour.
2. Mettez l’appareil hors tension et débranchez les câbles.
3. Enroulez le cordon d’alimentation fourni avec l’appareil, et fixez-le à l’arrière de ce dernier.
4. Écrivez le nom et l’adresse de votre entreprise dans la partie du bordereau de réexpédition réservée à l’expéditeur.
5. Envoyez un courrier à Quantium Solutions en utilisant le modèle d’e-mail suivant.

    * Si le bordereau de réexpédition de Japon Post Chakubarai était manquant, consignez ce fait que dans cet e-mail. Quantium Solutions demandera à Japan Post d’apporter le bordereau de réexpédition à l’enlèvement.
    * Si vous avez plusieurs commandes, envoyez un e-mail pour garantir un enlèvement individuel.

    ```
    To: Customerservice.JP@quantiumsolutions.com
    Subject: Pickup request for Azure Data Box｜Job name： 
    Body: 
    - Japan Post Yu-Pack tracking number (reference number)：
    - Requested pickup date：mmdd (Select a requested time slot from below).
    a. 08：00-13：00 
    b. 13：00-15：00 
    c. 15：00-17：00 
    d. 17：00-19：00 
    ```

6. Recevoir un e-mail de confirmation de Quantium Solutions près avoir réservé un enlèvement. L’e-mail de confirmation inclut également des informations sur le bordereau de réexpédition de Chakubarai.

Si nécessaire, vous pouvez contacter le support de Quantium Solutions (en langue japonaise) dont les coordonnées sont les suivantes : 

* E-mail ：Customerservice.JP@quantiumsolutions.com
* Téléphone：+81 3 57 55 01 50

## <a name="singapore"></a>[Singapour](#tab/in-singapore)

1. Conservez la boîte d’origine utilisée pour expédier l’appareil, en cas de retour.
2. Notez le numéro de suivi (indiqué comme numéro de référence dans la page Préparation de l’expédition de l’interface utilisateur web locale de la Data Box). Il est disponible une fois l’étape de préparation de l’expédition terminée. Téléchargez l’étiquette d’expédition à partir de cette page, puis collez-la sur le carton d’emballage.
3. Mettez l’appareil hors tension et débranchez les câbles.
4. Enroulez le cordon d’alimentation fourni avec l’appareil, et fixez-le à l’arrière de ce dernier. 
5. Envoyez un e-mail au service client SingPost en utilisant le modèle d’e-mail suivant avec le numéro de suivi.

    ```
    To: kadcustcare@singpost.com
    Subject: Microsoft Azure Pickup - OrderName 
    Body: 
        1. Requestor name  
        2. Requestor contact number
        3. Requestor collection address
        4. Preferred collection date
    ```

   > [!NOTE]
   > Pour les demandes de réservation reçues un jour ouvré :
   >
   > * Avant 15 h, la collecte sera effectuée le jour ouvré suivant entre 9 h et 13 h.
   > * Après 15 h, la collecte sera effectuée le jour ouvré suivant entre 14 h et 18 h.  

## <a name="south-africa"></a>[Afrique du Sud](#tab/in-sa)

1. Conservez la boîte d’origine utilisée pour expédier l’appareil, en cas de retour.
2. Notez le numéro de référence (numéro de lettre de transport) indiqué dans l’interface utilisateur web locale de l’appareil. Ce numéro s’affiche une fois que la **préparation de l’expédition** a abouti.
3. Téléchargez et imprimez l’étiquette d’expédition disponible dans l’interface utilisateur web locale de l’appareil et apposez-la sur le colis d’expédition.
4. Pour réserver un enlèvement avec DHL, vous avez plusieurs options :

    * Appeler le centre de contact du service clientèle avant 14h00 en composant le **+27(0)119213600**, en sélectionnant l’option 1, puis en spécifiant le numéro de lettre de transport.
    * Envoyer un e-mail à l’adresse [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com) en utilisant le modèle suivant :

    ```output
    To: Priority.Support@dhl.com
    Subject: Pickup request for Microsoft Azure
    Body: Need pick up for the below shipment
      *  DHL tracking number (reference number/waybill number)
      *  Requested pickup date: yyyy/mm/dd;time:HH MM
    ```

    * Vous pouvez également déposer le colis au point de service DHL le plus proche.

5. En cas de problème, envoyez e-mail à l’adresse [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com), en détaillant le ou les problèmes rencontrés et en insérant le numéro de lettre de transport dans la ligne Objet. Vous pouvez également appeler le +27(0)119213902.

## <a name="hong-kong"></a>[Hong Kong (R.A.S.)](#tab/in-hk)

1. Emballez l’appareil pour réexpédition dans sa boîte d’origine.
2. Notez le numéro de référence (numéro de suivi de retour) affiché sur l’interface utilisateur web locale de l’appareil. Ce numéro s’affiche une fois que la **préparation de l’expédition** a abouti.
3. Téléchargez et imprimez l’étiquette d’expédition disponible dans l’interface utilisateur web locale de l’appareil et apposez-la sur le colis d’expédition.
4. Enroulez le cordon d’alimentation fourni avec l’appareil, et fixez-le à l’arrière de ce dernier.
5. Appelez l’assistance téléphonique de **Quantium Solutions** au **(852) 2318 1213** pendant les heures de bureau (de 9h00 à 18h00, du lundi au vendredi).  
6. Indiquez Enlèvement Microsoft Azure, le numéro de référence et le numéro de suivi (code-barres en haut) sur l’étiquette de réexpédition pour organiser l’enlèvement.
7. Vous recevrez une confirmation verbale pour la planification de l’enlèvement. Si le transporteur ne procède pas à l’enlèvement, appelez l’assistance téléphonique de Quantium Solutions pour trouver une autre solution.
8. Lors de la réservation d’un enlèvement auprès de Quantium, partagez la confirmation avec [Microsoft Data Box Operations Asia](mailto:adbo@microsoft.com) à l’aide du modèle suivant :

    ```output
    To: adbo@microsoft.com
    Subject: Microsoft Data Box Job: [order name] has completed copy
    Body:
    We have confirmed the pickup details with Quantium.

       * Requestor name:
       * Requestor contact number:
       * Pickup Date:  
       * Pickup time:
    ```

En cas de problème, envoyez un e-mail à Data Box Operations Asia à l’adresse [adbo@microsoft.com](mailto:adbo@microsoft.com), en spécifiant le nom du travail dans l’en-tête d’objet et le problème rencontré.

## <a name="self-managed"></a>[Autogérée](#tab/in-selfmanaged)

Si vous utilisez la Data Box au Japon, à Singapour, en Corée, en Inde, en Afrique du Sud ou en Europe de l’Ouest et que vous avez sélectionné l’option d’expédition autogérée pendant la création de la commande, suivez ces instructions.

1. Notez le code d’autorisation indiqué dans la page Préparation de l’expédition de l’interface utilisateur web locale de la Data Box une fois cette étape terminée.
2. Mettez l’appareil hors tension et débranchez les câbles. Enroulez le cordon d’alimentation fourni avec l’appareil, et fixez-le à l’arrière de ce dernier.
3. Envoyez un e-mail à l’équipe des opérations Azure Data Box en utilisant le modèle ci-dessous lorsque vous êtes prêt à renvoyer l’appareil.
    
    ```
    To: adbops@microsoft.com 
    Subject: Request for Azure Data Box drop-off for order: ‘orderName’ 
    Body: 
        1. Order name  
        2. Authorization code available after Prepare to Ship has completed [Yes/No]  
        3. Contact name of the person dropping off. You will need to display a Government approved ID during the drop off.
    ```

---

## <a name="erasure-of-data-from-data-box"></a>Effacer les données de la Data Box

Une fois que l’appareil a atteint le centre de données Azure, la Data Box efface les données de ses disques, conformément aux [instructions du standard NIST SP 800-88 Revision 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous a apporté des connaissances concernant les sujets suivants :

> [!div class="checklist"]
> * Prérequis
> * Préparer l’expédition
> * Expédier la Data Box à Microsoft
> * Effacer les données de la Data Box

Passez à l’article suivant pour découvrir comment gérer votre Data Box.

> [!div class="nextstepaction"]
> [Gérer Data Box par le biais du portail Azure](./data-box-portal-admin.md)


