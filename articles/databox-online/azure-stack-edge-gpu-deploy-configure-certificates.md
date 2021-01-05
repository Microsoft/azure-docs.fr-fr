---
title: Tutoriel pour configurer des certificats pour l’appareil Azure Stack Edge Pro avec GPU dans le portail Azure | Microsoft Docs
description: Le tutoriel sur le déploiement d’Azure Stack Edge Pro avec GPU vous apprend à configurer des certificats pour votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure certificates for Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 7854aff0b4194efae7c4df653dee18e2676fdd41
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91446327"
---
# <a name="tutorial-configure-certificates-for-your-azure-stack-edge-pro-with-gpu"></a>Tutoriel : Configurer des certificats pour votre appareil Azure Stack Edge Pro avec GPU

Ce tutoriel explique comment configurer des certificats pour votre appareil Azure Stack Edge Pro en utilisant l’interface utilisateur web locale.

Le temps nécessaire pour cette étape peut varier en fonction de l’option spécifique que vous choisissez et de la façon dont le workflow de certificat est établi dans votre environnement.

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Pro avec GPU, vérifiez que :

* Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).
* Si vous envisagez d’apporter vos propres certificats :
    - Vos certificats doivent être prêts au format approprié, y compris le certificat de chaîne de signature. Pour plus d’informations sur les certificats, accédez à [Gérer des certificats](azure-stack-edge-j-series-manage-certificates.md).

<!--    - If your device is deployed in Azure Government or Azure Government Secret or Azure Government top secret cloud and not deployed in Azure public cloud, a signing chain certificate is required before you can activate your device. 
    For details on certificate, go to [Manage certificates](azure-stack-edge-j-series-manage-certificates.md).-->


## <a name="configure-certificates-for-device"></a>Configurer des certificats pour un appareil

1. Dans la page **Certificats** , vous allez configurer vos certificats. Selon que vous avez ou non modifié le nom de l’appareil ou le domaine DNS dans la page **Configuration de l’appareil**, vous pouvez choisir l’une des options suivantes pour vos certificats.

    - Si vous n’avez pas modifié le nom de l’appareil ou le domaine DNS à l’étape précédente et que vous ne souhaitez pas apporter vos propres certificats, vous pouvez ignorer cette étape et passer à la suivante. L’appareil a généré automatiquement des certificats auto-signés pour commencer. 

        ![Page « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-2.png)

    - Si vous avez modifié le nom de l’appareil ou le domaine DNS, vous noterez que l’état des certificats indique **Non valide**. 

        ![Page 2 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-1.png)    

        Sélectionnez un certificat pour afficher les détails relatifs à l’état.

        ![Page 3 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-1a.png)  

        Cela est dû au fait que les certificats ne reflètent pas le nom de l’appareil et le domaine DNS mis à jour (qui sont utilisés dans le nom de l’objet et l’autre nom de l’objet). Pour activer correctement votre appareil, choisissez l’une des options suivantes : 
    
        - **Générer tous les certificats d’appareil**. Ces certificats d’appareil doivent être utilisés à des fins de test uniquement, et pas avec des charges de travail de production. Pour plus d’informations, consultez [Générer des certificats d’appareil sur votre appareil Azure Stack Edge Pro](#generate-device-certificates).

        - **Apportez votre propre certificat**. Vous pouvez apporter vos propres certificats de point de terminaison signés et les chaînes de signature correspondantes. Vous ajoutez d’abord la chaîne de signature, puis vous chargez les certificats de point de terminaison. **Nous vous recommandons de toujours apporter vos propres certificats pour les charges de travail de production.** Pour plus d’informations, consultez [Apporter vos propres certificats sur votre appareil Azure Stack Edge Pro](#bring-your-own-certificates).
    
        - Vous pouvez apporter vos propres certificats et générer des certificats d’appareil. L’option **Générer des certificats** ne régénère que les certificats d’appareil.

    - Si vous avez modifié le nom de l’appareil ou le domaine DNS et que vous ne générez pas de certificats ou apportez vos propres certificats, **l’activation sera bloquée**.


### <a name="generate-device-certificates"></a>Générer des certificats d'appareil

Suivez ces étapes pour générer des certificats d’appareil.

Procédez comme suit pour régénérer et télécharger les certificats d’appareil Azure Stack Edge Pro :

1. Dans l’interface utilisateur locale de votre appareil, accédez à **Configuration > Certificats**. Sélectionnez **Générer des certificats**.

    ![Générer et télécharger le certificat 1](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-3.png)

2. Dans la **Générer des certificats d’appareil**, sélectionnez **Générer**. 

    ![Générer et télécharger le certificat 2](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-4.png)

    Les certificats des appareils sont désormais générés et appliqués. La génération et l’application des certificats prennent quelques minutes.
    
    > [!IMPORTANT]
    > Pendant que l’opération de génération de certificats est en cours, n’apportez pas vos propres certificats et essayez de les ajouter via l’option **+ Ajouter un certificat**.

    Vous êtes averti lorsque l’opération est terminée avec succès. **Pour éviter tout problème éventuel de cache, redémarrez votre navigateur.**
    
    ![Générer et télécharger le certificat 4](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-5.png)

3. Une fois les certificats générés : 

    - L’état de tous les certificats indique **Valide**. 

        ![Générer et télécharger le certificat 5](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-6.png)

    - Vous pouvez sélectionner un nom de certificat spécifique et afficher les détails qui s’y rapportent. 

        ![Générer et télécharger le certificat 6](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-6a.png)

    - La colonne **Télécharger** est maintenant renseignée. Cette colonne contient des liens permettant de télécharger les certificats regénérés. 

        ![Générer et télécharger le certificat 7](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-6b.png)


4. Sélectionnez le lien de téléchargement d’un certificat et, lorsque vous y êtes invité, enregistrez le certificat. 

    ![Générer et télécharger le certificat 8](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-7.png)

5. Répétez ce processus pour tous les certificats que vous souhaitez télécharger. 
    
    ![Générer et télécharger le certificat 9](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-8.png)

    Les certificats générés par l’appareil sont enregistrés sous forme de certificats DER avec le format de nom suivant : 

    `<Device name>_<Endpoint name>.cer`. Ces certificats contiennent la clé publique pour les certificats correspondants installés sur l’appareil. 

Vous devrez installer ces certificats sur le système client que vous utilisez pour accéder aux points de terminaison sur l’appareil ASE. Ces certificats établissent une relation de confiance entre le client et l’appareil.

Pour importer et installer ces certificats sur le client que vous utilisez pour accéder à l’appareil, suivez les étapes décrites dans [Importer des certificats sur les clients accédant à votre appareil Azure Stack Edge Pro](azure-stack-edge-j-series-manage-certificates.md#import-certificates-on-the-client-accessing-the-device). 

Si vous utilisez l’Explorateur Stockage Azure, vous devez installer des certificats sur votre client au format PEM et vous devrez convertir les certificats générés par l’appareil au format PEM. 

> [!IMPORTANT]
> - Le lien de téléchargement est uniquement disponible pour les certificats générés par l’appareil et non si vous apportez vos propres certificats.
> - Vous pouvez décider d’avoir une combinaison de certificats générés par l’appareil et de certificats que vous apportez tant que les autres exigences du certificat sont remplies. Pour plus d'informations, consultez [Exigences des certificats](azure-stack-edge-j-series-certificate-requirements.md).
    

### <a name="bring-your-own-certificates"></a>Apportez vos propres certificats

Procédez comme suit pour ajouter vos propres certificats, y compris la chaîne de signature.

1. Pour télécharger le certificat, sur la page **Certificat**, sélectionnez **+ Ajouter un certificat**.

    ![Page 4 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-1.png)

2. Chargez d’abord la chaîne de signature, puis sélectionnez **Valider et ajouter**.

    ![Page 5 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-2.png)

3. Vous pouvez maintenant télécharger d’autres certificats. Par exemple, vous pouvez télécharger les certificats de point de terminaison de stockage d’objets Blob et Azure Resource Manager.

    ![Page 6 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-3.png)

    Vous pouvez également télécharger le certificat de l’interface utilisateur web locale. Après avoir téléchargé ce certificat, vous devez démarrer votre navigateur et effacer le cache. Vous devrez ensuite vous connecter à l’interface utilisateur web locale de l’appareil.  

    ![Page 7 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-5.png)

    Vous pouvez également télécharger le certificat de nœud.

    ![Page 8 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-4.png)

    À tout moment, vous pouvez sélectionner un certificat et afficher les détails pour vous assurer qu’ils correspondent au certificat que vous avez chargé.

    ![Page 9 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-6.png)

    La page du certificat devrait s’actualiser pour refléter les nouveaux certificats ajoutés.

    ![Page 10 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-configure-certificates/add-certificate-7.png)  

    > [!NOTE]
    > À l’exception du cloud public Azure, les certificats de chaîne de signature doivent être apportés avant l’activation de toutes les configurations cloud (Azure Government ou Azure Stack).


Votre appareil est maintenant prêt à être activé. Sélectionnez **< Revenir à Démarrer**.


## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique

Pour savoir comment activer votre appareil Azure Stack Edge Pro, consultez :

> [!div class="nextstepaction"]
> [Activer un appareil Azure Stack Edge Pro](./azure-stack-edge-gpu-deploy-activate.md)
