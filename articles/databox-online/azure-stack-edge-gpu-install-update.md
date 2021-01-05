---
title: Installer une mise à jour sur un appareil Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment utiliser le portail Azure et l’interface utilisateur web locale pour appliquer des mises à jour sur un appareil Azure Stack Edge Pro avec GPU et le cluster Kubernetes associé sur l’appareil
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 12/11/2020
ms.author: alkohli
ms.openlocfilehash: 004c84387c51233542b1bbbf11d33cdff42259af
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359147"
---
# <a name="update-your-azure-stack-edge-pro-gpu"></a>Mettre à jour votre GPU Azure Stack Edge Pro 

Cet article décrit les étapes à effectuer pour installer des mises à jour sur votre appareil Azure Stack Edge Pro avec GPU en utilisant le portail Azure et l’interface utilisateur web locale. Vous appliquez les mises à jour logicielles ou les correctifs logiciels pour maintenir à jour votre appareil Azure Stack Edge Pro et le cluster Kubernetes associé sur l’appareil. 

La procédure décrite dans cet article a été effectuée à l’aide d’une version différente du logiciel, mais le processus reste le même pour la version actuelle du logiciel.

> [!IMPORTANT]
> - La mise à jour **2012** est la mise à jour actuelle et correspond à :
>   - Version du logiciel de l’appareil – **2.2.1438.2470**
>   - Version du serveur Kubernetes – **v1.17.3**
>   - Version d’IoT Edge : **0.1.0-beta10**
>    
>    Pour plus d’informations sur les nouveautés de cette mise à jour, consultez [Notes de publication](azure-stack-edge-gpu-2012-release-notes.md).
> - Pour appliquer la mise à jour 2012, votre appareil doit exécuter la version 2010.
> - N’oubliez pas que l’installation d’une mise à jour ou d’un correctif logiciel nécessite le redémarrage de votre appareil. Cette mise à jour contient les mises à jour logicielles de l'appareil et celles de Kubernetes. Étant donné que l’appareil Azure Stack Edge Pro est un appareil à nœud unique, les E/S en cours sont interrompues et votre appareil subit un temps d’arrêt qui peut durer jusqu’à 1,5 heure pour la mise à jour.

Pour installer les mises à jour sur votre appareil, vous devez d’abord configurer l’emplacement du serveur de mise à jour. Une fois le serveur de mise à jour configuré, vous pouvez appliquer les mises à jour par le biais de l’interface utilisateur du portail Azure ou de l’interface utilisateur web locale.

Chacune de ces étapes est décrite dans les sections suivantes.

## <a name="configure-update-server"></a>Configurer le serveur de mise à jour

1. Dans l’interface utilisateur web locale, accédez à **Configuration** > **Serveur de mise à jour**. 
   
    ![Configurer les mises à jour 1](./media/azure-stack-edge-gpu-install-update/configure-update-server-1.png)

2. Dans **Sélectionner le type de serveur de mise à jour**, dans la liste déroulante, choisissez entre Serveur Microsoft Update (par défaut) et Windows Server Update Services.  
   
    Si vous mettez à jour à partir de Windows Server Update Services, spécifiez l’URI du serveur. Le serveur de cet URI déploiera les mises à jour sur tous les appareils connectés à ce serveur.

    ![Configurer les mises à jour 2](./media/azure-stack-edge-gpu-install-update/configure-update-server-2.png)
    
    Le serveur WSUS est utilisé pour gérer et distribuer les mises à jour par le biais d’une console de gestion. Un serveur WSUS peut également être la source de mises à jour d’autres serveurs WSUS de l’organisation. Le serveur WSUS qui agit en tant que source des mises à jour est appelé serveur en amont. Dans une implémentation WSUS, au moins un serveur WSUS de votre réseau doit se connecter à Microsoft Update pour obtenir les informations sur les mises à jour disponibles. En tant qu’administrateur, vous pouvez déterminer le nombre d’autres serveurs WSUS se connectant directement à Microsoft Update en fonction de la configuration et de la sécurité du réseau.
    
    Pour plus d’informations, consultez [Windows Server Update Services (WSUS)](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus)

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Nous vous recommandons d’installer les mises à jour par le biais du portail Azure. L’appareil recherche automatiquement les mises à jour une fois par jour. Une fois les mises à jour disponibles, vous verrez une notification dans le portail. Vous pouvez alors télécharger et installer les mises à jour. 

> [!NOTE]
> Avant de procéder à l’installation des mises à jour, vérifiez que l’appareil est sain et que l’état indique **En ligne**.

1. Lorsque des mises à jour sont disponibles pour votre appareil, une notification s’affiche. Vous pouvez sélectionner la notification ou cliquer sur **Mettre à jour l’appareil** dans la barre de commandes supérieure. Cela vous permettra d’appliquer les mises à jour logicielles de l’appareil.

    ![Version logicielle après la mise à jour](./media/azure-stack-edge-gpu-install-update/portal-update-1.png)

2. Dans le panneau **Mises à jour de l’appareil**, consultez les termes du contrat de licence associés aux nouvelles fonctionnalités dans les notes de publication.

    Vous pouvez choisir de **Télécharger et installer** les mises à jour ou simplement de les **Télécharger**. Vous pourrez ensuite installer ces mises à jour ultérieurement.

    ![Version logicielle après la mise à jour 2](./media/azure-stack-edge-gpu-install-update/portal-update-2a.png)    

    Si vous souhaitez télécharger et installer les mises à jour, sélectionnez l’option qui met à jour l’installation automatiquement une fois le téléchargement terminé.

    ![Version logicielle après la mise à jour 3](./media/azure-stack-edge-gpu-install-update/portal-update-2b.png)

3. Le téléchargement des mises à jour démarre. Un message de notification indique que le téléchargement est en cours.

    ![Version logicielle après la mise à jour 4](./media/azure-stack-edge-gpu-install-update/portal-update-3.png)

    Une bannière de notification s’affiche également dans le portail Azure. Celle-ci indique la progression du téléchargement. 

    ![Version logicielle après la mise à jour 5](./media/azure-stack-edge-gpu-install-update/portal-update-4.png)

    Vous pouvez sélectionner cette notification ou sélectionner **Mettre à jour l’appareil** pour afficher le détail de la mise à jour.

    ![Version logicielle après la mise à jour 6](./media/azure-stack-edge-gpu-install-update/portal-update-5.png)   


4. Une fois le téléchargement terminé, la bannière de notification est mise à jour pour indiquer la fin de l’opération. Si vous avez choisi de télécharger et d’installer les mises à jour, l’installation démarre automatiquement.

    ![Version logicielle après la mise à jour 7](./media/azure-stack-edge-gpu-install-update/portal-update-6.png)

    Si vous avez choisi de télécharger uniquement les mises à jour, sélectionnez la notification pour ouvrir le panneau **Mises à jour de l’appareil**. Sélectionnez **Installer**.
  
    ![Version logicielle après la mise à jour 8](./media/azure-stack-edge-gpu-install-update/portal-update-7.png)

5. Un message de notification indique que l’installation est en cours. 

    ![Version logicielle après la mise à jour 9](./media/azure-stack-edge-gpu-install-update/portal-update-8.png)
 
    Le portail affiche également une alerte d’information pour indiquer que l’installation est en cours. L’appareil est mis hors connexion et est en mode maintenance.
   
    ![Version logicielle après la mise à jour 10](./media/azure-stack-edge-gpu-install-update/portal-update-9.png)

6. Comme il s'agit d'un appareil à nœud unique, il redémarre après l'installation des mises à jour. L'alerte critique qui s'affiche pendant le redémarrage indique que la pulsation de l'appareil a été perdue.

    ![Version logicielle après la mise à jour 11](./media/azure-stack-edge-gpu-install-update/portal-update-10.png)

    Sélectionnez l’alerte pour afficher l’événement d’appareil correspondant.
    
    ![Version logicielle après la mise à jour 12](./media/azure-stack-edge-gpu-install-update/portal-update-11.png)


7. Après le redémarrage, l’appareil est de nouveau mis en mode maintenance et une alerte d’information s’affiche pour indiquer cela.

    Si vous sélectionnez **Mettre à jour l’appareil** à partir de la barre de commandes supérieure, vous pouvez voir la progression des mises à jour.   

8. Après l’installation des mises à jour, l’état de l’appareil passe à **En ligne**. 

    ![Version logicielle après la mise à jour 13](./media/azure-stack-edge-gpu-install-update/portal-update-14.png)

    Dans la barre de commandes supérieure, sélectionnez **Mises à jour de l’appareil**. Vérifiez que la mise à jour a bien été installée et que la version du logiciel de l’appareil reflète la mise à jour.

    ![Version logicielle après la mise à jour 14](./media/azure-stack-edge-gpu-install-update/portal-update-15.png)

<!--9. You will again see a notification that updates are available. These are the Kubernetes updates. Select the notification or select **Update device** from the top command bar.

    ![Software version after update 15](./media/azure-stack-edge-gpu-install-update/portal-update-16.png)

10. Download the Kubernetes updates. You can see that the package size is different when compared to the previous update package.

    ![Software version after update 16](./media/azure-stack-edge-gpu-install-update/portal-update-17.png)

    The process of installation is identical to that of device updates. First the updates are downloaded.

    ![Software version after update 17](./media/azure-stack-edge-gpu-install-update/portal-update-18.png)    
    
11. Once the updates are downloaded, you can then install the updates. 

    ![Software version after update 18](./media/azure-stack-edge-gpu-install-update/portal-update-19.png)

    As the updates are installed, the device is put into maintenance mode. The device does not restart for the Kubernetes updates. -->

Une fois le logiciel de l'appareil et les mises à jour de Kubernetes correctement installés, la notification de bannière disparaît. Votre appareil dispose à présent de la dernière version du logiciel de l’appareil et de Kubernetes.


## <a name="use-the-local-web-ui"></a>Utilisation de l’interface utilisateur web locale

La procédure de mise à jour à l’aide de l’interface utilisateur web locale se déroule en deux étapes :

* Téléchargement de la mise à jour ou du correctif
* Installation de la mise à jour ou du correctif

Chacune de ces étapes est décrite en détail dans les sections suivantes.


### <a name="download-the-update-or-the-hotfix"></a>Téléchargement de la mise à jour ou du correctif

Effectuez les étapes suivantes pour télécharger la mise à jour logicielle. Vous pouvez télécharger la mise à jour à partir de l’emplacement fourni par Microsoft ou à partir du catalogue Microsoft Update.


Effectuez les étapes suivantes pour télécharger la mise à jour à partir du catalogue Microsoft Update.

1. Lancez le navigateur et accédez à [https://catalog.update.microsoft.com](https://catalog.update.microsoft.com).

    ![Rechercher dans le catalogue](./media/azure-stack-edge-gpu-install-update/download-update-1.png)

2. Dans la zone de recherche du catalogue Microsoft Update, entrez le numéro KB (Base de connaissances) du correctif ou des mots clés relatifs à la mise à jour que vous souhaitez télécharger. Par exemple, entrez **Azure Stack Edge Pro**, puis cliquez sur **Rechercher**.
   
    La liste des mises à jour se présente sous le nom **Azure Stack Edge Update 2012**.
   
    <!--![Search catalog 2](./media/azure-stack-edge-gpu-install-update/download-update-2b.png)-->

4. Sélectionnez **Télécharger**. Il y a deux fichiers à télécharger avec des suffixes *SoftwareUpdatePackage.exe* et *Kubernetes_Package.exe* qui correspondent respectivement aux mises à jour logicielles des appareils et à celles de Kubernetes. Téléchargez les fichiers dans un dossier du système local. Vous pouvez également copier le dossier sur un partage réseau accessible à partir de l’appareil.

### <a name="install-the-update-or-the-hotfix"></a>Installation de la mise à jour ou du correctif

Avant d’installer la mise à jour ou le correctif logiciel, vérifiez les points suivants :

 - La mise à jour ou le correctif logiciel est téléchargé localement sur votre hôte ou accessible via un partage réseau.
 - L’état de votre appareil est sain, comme l’indique la page **Vue d’ensemble** de l’interface utilisateur web locale.

   ![mettre à jour l'appareil](./media/azure-stack-edge-gpu-install-update/local-ui-update-1.png) 

Cette procédure prend environ 20 minutes. Effectuez les opérations suivantes pour installer la mise à jour ou le correctif logiciel.

1. Dans l’interface utilisateur web locale, accédez à **Maintenance** > **Mise à jour logicielle**. Prenez note de la version du logiciel que vous exécutez. 
   
   ![mettre à jour l’appareil 2](./media/azure-stack-edge-gpu-install-update/local-ui-update-2.png)

2. Indiquez le chemin du fichier de mise à jour. Vous pouvez également accéder au fichier d'installation de la mise à jour si celui-ci a été placé sur un partage réseau. Sélectionnez le fichier de mise à jour logicielle dont le nom se termine par *SoftwareUpdatePackage.exe*.

   ![mettre à jour l’appareil 3](./media/azure-stack-edge-gpu-install-update/local-ui-update-3a.png)

3. Sélectionnez **Appliquer**. 

   ![mettre à jour l’appareil 4](./media/azure-stack-edge-gpu-install-update/local-ui-update-4.png)

4. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui** pour continuer. Puisqu’il s’agit d’un appareil à nœud unique, l’application de la mise à jour entraîne le redémarrage de l’appareil et provoque un temps d’arrêt. 
   
   ![mettre à jour l’appareil 5](./media/azure-stack-edge-gpu-install-update/local-ui-update-5.png)

5. La mise à jour démarre. Une fois l’appareil correctement mis à jour, il est redémarré. L’interface utilisateur locale n’est pas accessible pendant cet intervalle.
   
6. Une fois le redémarrage effectué, vous êtes redirigé vers la page **Se connecter** . Pour vérifier que le logiciel de l’appareil a été mis à jour, accédez à **Maintenance** > **Mise à jour logicielle** dans l’interface utilisateur web locale. Pour la version actuelle, la version logicielle affichée doit être **Azure Stack Edge 2012**.

   <!--![update device 6](./media/azure-stack-edge-gpu-install-update/local-ui-update-6.png)--> 

7. Vous allez maintenant mettre à jour la version logicielle de Kubernetes. Répétez les étapes ci-dessus. Fournissez un chemin d’accès au fichier de mise à jour Kubernetes avec le suffixe *Kubernetes_Package.exe*.  

   <!--![update device](./media/azure-stack-edge-gpu-install-update/local-ui-update-7.png)--> 

8. Sélectionnez **Appliquer**. 

   ![mettre à jour l’appareil 7](./media/azure-stack-edge-gpu-install-update/local-ui-update-8.png)

9. Quand vous êtes invité à confirmer l’opération, sélectionnez **Oui** pour continuer. 

10. Une fois la mise à jour Kubernetes installée, aucune modification n’est apportée au logiciel affiché dans **Maintenance** > **Mise à jour logicielle**. 


## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’[administration des appareils Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).