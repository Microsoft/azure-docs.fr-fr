---
title: Installer une mise à jour sur un appareil Azure Data Box Gateway | Microsoft Docs
description: Explique comment appliquer des mises à jour aux appareils Azure Data Box Gateway à l’aide du portail Azure et de l’interface utilisateur web locale.
services: databox
author: alkohli
ms.service: databox
ms.topic: article
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: 7fda10310a4bac085b248248d80d708dfa7f3ff7
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580923"
---
# <a name="update-your-azure-data-box-gateway"></a>Mettre à jour Azure Data Box Gateway

Cet article décrit les étapes à effectuer pour installer des mises à jour sur votre instance Azure Data Box Gateway à l’aide de l’interface utilisateur web locale et du portail Azure. Les mises à jour et les correctifs logiciels permettent à votre appareil Data Box Gateway de rester à jour.

> [!IMPORTANT]
>
> - La mise à jour **1911** correspond à la version logicielle **1.6.1049.786** de votre appareil. Pour plus d’informations sur cette mise à jour, consultez les [notes de publication](data-box-gateway-1911-release-notes.md).
>
> - N’oubliez pas que l’installation d’une mise à jour ou d’un correctif logiciel nécessite le redémarrage de votre appareil. Étant donné que l’appareil Data Box Gateway est un appareil à nœud unique, les E/S en cours sont interrompues et votre appareil subit un temps d’arrêt pouvant atteindre 30 minutes pour la mise à jour de son logiciel.

Chacune de ces étapes est décrite dans les sections suivantes.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Nous vous recommandons d’installer les mises à jour par le biais du portail Azure. L’appareil recherche automatiquement les mises à jour une fois par jour. Une fois les mises à jour disponibles, vous verrez une notification dans le portail. Vous pouvez alors télécharger et installer les mises à jour.

> [!NOTE]
> Avant de procéder à l’installation des mises à jour, vérifiez que l’appareil est sain et que l’état indique **En ligne**.

1. Lorsque des mises à jour sont disponibles pour votre appareil, une notification s’affiche. Vous pouvez sélectionner la notification ou cliquer sur **Mettre à jour l’appareil** dans la barre de commandes supérieure. Cela vous permettra d’appliquer les mises à jour logicielles de l’appareil.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec les options Vue d’ensemble et Mettre à jour l’appareil en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-01a.png)

2. Dans le panneau **Mises à jour de l’appareil**, consultez les termes du contrat de licence associés aux nouvelles fonctionnalités dans les notes de publication.

    Vous pouvez choisir de **Télécharger et installer** les mises à jour ou simplement de les **Télécharger**. Vous pourrez ensuite installer ces mises à jour ultérieurement.

    ![Capture d’écran de la boîte de dialogue Mises à jour de l’appareil, avec les options « J’accepte... » et « Je comprends que... », ainsi que l’option « Télécharger et installer » mises en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-02.png)

    Si vous souhaitez télécharger et installer les mises à jour, sélectionnez l’option qui met à jour l’installation automatiquement une fois le téléchargement terminé.

    ![Capture d’écran de la boîte de dialogue Mises à jour de l’appareil, avec les options « J’accepte... » et « Télécharger » mises en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-03.png)

3. Le téléchargement des mises à jour démarre. Un message de notification indique que le téléchargement est en cours.

    ![Capture d’écran de la notification « Téléchargement et installation des mises à jour en cours ».](./media/data-box-gateway-apply-updates/portal-apply-update-05.png)

    Une bannière de notification s’affiche également dans le portail Azure. Celle-ci indique la progression du téléchargement.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec l’option Vue d’ensemble et la bannière de notification Téléchargement des mises à jour en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-08a.png)

    Vous pouvez sélectionner cette notification ou sélectionner **Mettre à jour l’appareil** pour afficher le détail de la mise à jour.

    ![Capture d’écran de la boîte de dialogue Télécharger et installer des mises à jour avec le message État : En cours et le message Télécharger les mises à jour mis en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-09.png)

4. Une fois le téléchargement terminé, la bannière de notification est mise à jour pour indiquer la fin de l’opération. Si vous avez choisi de télécharger et d’installer les mises à jour, l’installation démarre automatiquement.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec l’option Vue d’ensemble et la bannière de notification Mises à jour téléchargées en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-10a.png)

    Si vous avez choisi de télécharger uniquement les mises à jour, sélectionnez la notification pour ouvrir le panneau **Mises à jour de l’appareil**. Sélectionnez **Installer**.
  
    ![Capture d’écran de la boîte de dialogue Mises à jour de l’appareil, avec l’option « Installer » mise en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-11a.png)

5. Un message de notification indique que l’installation est en cours.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec l’option Vue d’ensemble et le message de progression du téléchargement et de l’installation en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-12a.png)

    Le portail affiche également une alerte d’information pour indiquer que l’installation est en cours. <!-- The device goes offline and is in maintenance mode.-->

    <!-- ![Software version after update](./media/data-box-gateway-apply-updates/update-13.png)-->

6. Comme il s’agit d’un appareil à nœud unique, celui-ci redémarre après l’installation des mises à jour. L’alerte critique qui s’affiche pendant le redémarrage indique que la pulsation de l’appareil a été perdue.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec l’option Vue d’ensemble et la bannière de notification Alerte critique en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-19a.png)

    Sélectionnez l’alerte pour afficher l’événement d’appareil correspondant.

    ![Capture d’écran de la section Événements de l’appareil avec l’événement Perte de la pulsation de votre appareil mis en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-20a.png)

7. Après l’installation des mises à jour, l’état de l’appareil passe à **En ligne**.

    ![Capture d’écran de la page d’accueil d’Azure Data Box Gateway, avec l’option Vue d’ensemble et l’état En ligne en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-23a.png)

    Dans la barre de commandes supérieure, sélectionnez **Mises à jour de l’appareil**. Vérifiez que la mise à jour a bien été installée et que la version du logiciel de l’appareil reflète la mise à jour.

    ![Capture d’écran de la boîte de dialogue Mises à jour de l’appareil, avec la section Version du logiciel installée mise en évidence.](./media/data-box-gateway-apply-updates/portal-apply-update-24.png)

## <a name="use-the-local-web-ui"></a>Utilisation de l’interface utilisateur web locale

La procédure de mise à jour à l’aide de l’interface utilisateur web locale se déroule en deux étapes :

- Téléchargement de la mise à jour ou du correctif
- Installation de la mise à jour ou du correctif

Chacune de ces étapes est décrite en détail dans les sections suivantes.

### <a name="download-the-update-or-the-hotfix"></a>Téléchargement de la mise à jour ou du correctif

Effectuez les étapes suivantes pour télécharger la mise à jour logicielle. Vous pouvez télécharger la mise à jour à partir de l’emplacement fourni par Microsoft ou à partir du catalogue Microsoft Update.

Effectuez les étapes suivantes pour télécharger la mise à jour à partir du catalogue Microsoft Update.

1. Lancez le navigateur et accédez à [https://catalog.update.microsoft.com](https://catalog.update.microsoft.com).

   ![Capture d’écran du catalogue Microsoft Update dans une fenêtre de navigateur, avec Data Box Gateway tapé dans la zone de recherche. La barre d’adresse et la zone de recherche du navigateur sont mises en évidence.](./media/data-box-gateway-apply-updates/download-update-1.png)

2. Dans la zone de recherche du catalogue Microsoft Update, entrez le numéro KB (Base de connaissances) du correctif ou des mots clés relatifs à la mise à jour que vous souhaitez télécharger. Par exemple, entrez **Azure Data Box Gateway**, puis cliquez sur **Rechercher**.

   La liste des mises à jour s’affiche : **Azure Data Box Gateway 1911**.

   ![Capture d’écran du catalogue Microsoft Update dans une fenêtre de navigateur, avec les résultats de recherche pour Data Box Gateway mis en évidence.](./media/data-box-gateway-apply-updates/download-update-2.png)

3. Sélectionnez **Télécharger**. Un seul fichier correspondant à la mise à jour logicielle de l’appareil est disponible au téléchargement : *SoftwareUpdatePackage.exe*. Téléchargez le fichier dans un dossier du système local. Vous pouvez également copier le dossier sur un partage réseau accessible à partir de l’appareil.

   ![Capture d’écran de la boîte de dialogue Télécharger avec le lien vers le fichier SoftwareUpdatePackage.exe mis en évidence](./media/data-box-gateway-apply-updates/download-update-3.png)

### <a name="install-the-update-or-the-hotfix"></a>Installation de la mise à jour ou du correctif

Avant d’installer la mise à jour ou le correctif logiciel, vérifiez les points suivants :

- La mise à jour ou le correctif logiciel est téléchargé localement sur votre hôte ou accessible via un partage réseau.
- L’état de votre appareil est sain, comme l’indique la page **Vue d’ensemble** de l’interface utilisateur web locale.

   ![Capture d’écran de l’interface utilisateur web locale de Data Box Gateway, avec l’option Tableau de bord et le message État du logiciel : Sain mis en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-1.png)

Cette procédure prend environ 20 minutes. Effectuez les opérations suivantes pour installer la mise à jour ou le correctif logiciel.

1. Dans l’interface utilisateur web locale, accédez à **Maintenance** > **Mise à jour logicielle**. Prenez note de la version du logiciel que vous exécutez.

   ![Capture d’écran de l’interface utilisateur web locale de Data Box Gateway, avec l’option Mise à jour logicielle et le message Version actuelle du logiciel mis en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-2.png)

2. Indiquez le chemin du fichier de mise à jour. Vous pouvez également accéder au fichier d'installation de la mise à jour si celui-ci a été placé sur un partage réseau. Sélectionnez le fichier de mise à jour logicielle dont le nom se termine par *SoftwareUpdatePackage.exe*.

   ![Capture d’écran de l’Explorateur de fichiers avec le lien vers le fichier SoftwareUpdatePackage.exe mis en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-3.png)

3. Sélectionnez **Appliquer**.

   ![Capture d’écran de l’interface utilisateur web locale de Data Box Gateway, avec l’option Mise à jour logicielle, la zone de texte Chemin du fichier de mise à jour et l’option Appliquer la mise à jour mises en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-4.png)

4. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui** pour continuer. Puisqu’il s’agit d’un appareil à nœud unique, l’application de la mise à jour entraîne le redémarrage de l’appareil et provoque un temps d’arrêt.

   ![Capture d’écran de la boîte de dialogue Mise à jour logicielle, avec l’option Oui mise en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-5.png)

5. La mise à jour démarre. Une fois l’appareil correctement mis à jour, il est redémarré. L’interface utilisateur locale n’est pas accessible pendant cet intervalle.

6. Une fois le redémarrage effectué, vous êtes redirigé vers la page **Se connecter** . Pour vérifier que le logiciel de l’appareil a été mis à jour, accédez à **Maintenance** > **Mise à jour logicielle** dans l’interface utilisateur web locale. La version logicielle de cet exemple est **1.6.1049.786**.

   ![Capture d’écran de l’interface utilisateur web locale de Data Box Gateway, avec l’option Mise à jour logicielle et le message Version actuelle du logiciel (après mise à jour) mis en évidence.](./media/data-box-gateway-apply-updates/local-ui-update-6.png)

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’[administration des appareils Azure Data Box Gateway](data-box-gateway-manage-users.md).
