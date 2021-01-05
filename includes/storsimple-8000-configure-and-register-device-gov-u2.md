---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 338332eb90fda9afe7655f9f2ad19ed726419b6c
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95557267"
---
### <a name="to-configure-and-register-the-device"></a>Configuration et inscription de l’appareil
1. Accédez à l’interface Windows PowerShell sur la console série de votre appareil StorSimple. Consultez la page [Utilisation de PuTTY pour se connecter à la console en série de l’appareil](../articles/storsimple/storsimple-8000-deployment-walkthrough-gov-u2.md#use-putty-to-connect-to-the-device-serial-console) pour obtenir des instructions. **Veillez à suivre la procédure pas à pas, sans quoi vous ne pourrez pas accéder à la console.**
2. Dans la session qui s’ouvre, appuyez une fois sur **Entrée** pour lancer une invite de commandes.
3. Vous devez choisir la langue que vous souhaitez définir pour votre appareil. Spécifiez la langue, puis appuyez sur **Entrée**.
   
    ![Configuration et inscription de l’appareil StorSimple 1](./media/storsimple-configure-and-register-device-gov-u2/HCS_RegisterYourDevice1-gov-include.png)
4. Dans le menu de console en série qui s’affiche, choisissez l’option 1 : **Se connecter avec accès total**.
   
    ![Inscription de l’appareil StorSimple 2](./media/storsimple-configure-and-register-device-gov-u2/HCS_RegisterYourDevice2-gov-include.png)
5. Procédez comme suit pour configurer les paramètres réseau minimum requis pour votre appareil.
   
   > [!IMPORTANT]
   > Ces étapes de configuration doivent être effectuées sur le contrôleur actif de l’appareil. Le menu de la console en série indique l’état du contrôleur dans le message de bannière. Si vous n’êtes pas connecté au contrôleur actif, déconnectez-vous, puis connectez-vous à ce contrôleur.
   
   1. À l’invite de commandes, tapez votre mot de passe. Le mot de passe par défaut de l’appareil est **Password1**.
   2. Tapez la commande suivante :
      
        `Invoke-HcsSetupWizard`
   3. Un Assistant Installation s’affiche pour vous aider à configurer les paramètres réseau de l’appareil. Fournissez les informations suivantes :
      
      * Adresse IP de l’interface réseau DATA 0
      * Masque de sous-réseau
      * Passerelle
      * Adresse IP du serveur DNS principal
      * Adresse IP du serveur NTP principal
      
      > [!NOTE]
      > Vous devrez peut-être attendre quelques minutes que les paramètres de masque de sous-réseau et de DNS soient appliqués.
    
   4. Configurez éventuellement votre serveur proxy web.
      
      > [!IMPORTANT]
      > Bien que la configuration du proxy web soit facultative, si vous en utilisez un, vous pouvez uniquement le configurer ici. Pour plus d’informations, consultez la section [Configuration du proxy web pour votre appareil](../articles/storsimple/storsimple-8000-configure-web-proxy.md).
     
6. Appuyez sur Ctrl+C pour quitter l’Assistant Installation.
8. Exécutez l’applet de commande suivante pour faire pointer l’appareil vers le portail Microsoft Azure Government (il pointe en effet vers le portail Azure Classic par défaut). Les deux contrôleurs redémarrent. Nous vous recommandons d’utiliser deux sessions PuTTY pour vous connecter simultanément aux deux contrôleurs afin de voir quand chacun d’eux est redémarré.
   
    `Set-CloudPlatform -AzureGovt_US`
   
   Un message de confirmation s’affiche. Acceptez la valeur par défaut (**Y**).
9. Exécutez l’applet de commande suivante pour reprendre l’installation :
   
    `Invoke-HcsSetupWizard`
   
    ![Reprise de l’Assistant Installation](./media/storsimple-configure-and-register-device-gov-u2/HCS_ResumeSetup-gov-include.png)
   
10. Acceptez les paramètres réseau. Un message de validation apparaît lorsque vous acceptez un paramètre.
11. Pour des raisons de sécurité, le mot de passe administrateur de l’appareil expire après la première session, et vous devez le modifier maintenant. Lorsque vous y êtes invité, fournissez un mot de passe administrateur de l’appareil. Un mot de passe administrateur d’appareil valide doit comprendre entre 8 et 15 caractères. Le mot de passe doit contenir trois des éléments suivants : caractères en minuscules, en majuscules, numériques et spéciaux.
    
    <br/>![Inscription de l’appareil StorSimple 5](./media/storsimple-configure-and-register-device-gov-u2/HCS_RegisterYourDevice5_gov-include.png)
12. La dernière étape de l’Assistant Installation inscrit votre appareil auprès du service StorSimple Device Manager. Pour cela, vous avez besoin de la clé d’inscription de service que vous avez obtenue à [l’étape 2 : obtention de la clé d’inscription](../articles/storsimple/storsimple-8000-deployment-walkthrough-gov-u2.md#step-2-get-the-service-registration-key). Après avoir entré la clé d’enregistrement, vous devrez peut-être attendre 2 à 3 minutes avant que l’appareil ne soit inscrit.
    
    > [!NOTE]
    > Vous pouvez appuyer sur Ctrl + C à tout moment pour quitter l’Assistant Installation. Si vous avez entré tous les paramètres réseau (adresse IP pour Data 0, masque de sous-réseau et passerelle), vos entrées sont conservées.
    
    ![Progression de l’inscription de StorSimple](./media/storsimple-configure-and-register-device-gov-u2/HCS_RegistrationProgress-gov-include.png)
13. Une fois l’appareil inscrit, une clé de chiffrement de données de service s’affiche. Copiez-la et enregistrez-la en lieu sûr. **Cette clé et la clé d’inscription du service sont requises pour l’inscription d’appareils supplémentaires auprès du service StorSimple Device Manager.** Reportez-vous à la section [Sécurité StorSimple](../articles/storsimple/storsimple-8000-security.md) pour plus d’informations sur cette clé.
    
    ![Inscription de l’appareil StorSimple 7](./media/storsimple-configure-and-register-device-gov-u2/HCS_RegisterYourDevice7_gov-include.png)
    > [!IMPORTANT]
    > Pour copier le texte à partir de la fenêtre de console de série, sélectionnez-le simplement. Vous devez ensuite pouvoir le coller dans le Presse-papiers ou dans un éditeur de texte.
    > 
    > N’utilisez PAS **Ctrl + C** pour copier la clé de chiffrement des données de service. L’utilisation de **Ctrl + C** ferme l’Assistant Installation. Par conséquent, le mot de passe administrateur de l’appareil n’est pas modifié et l’appareil rétablit le mot de passe par défaut.
    
14. Quittez la console en série.
15. Revenez au portail Azure Government et procédez comme suit :
    
    1. Accédez à votre service StorSimple Device Manager.
    2. Cliquez sur **Appareils**. Dans la liste des appareils, identifiez l'appareil que vous déployez. Vérifiez que l’appareil s’est bien connecté au service en vérifiant son état. L’état de l’appareil doit être **En ligne**.
            
        Si l’état de l’appareil est **Hors ligne**, attendez quelques minutes qu’il soit en ligne.
       
        Si l’appareil est toujours déconnecté après quelques minutes, vous devez vous assurer que votre réseau de pare-feu a été configuré comme décrit dans [Configuration réseau requise pour votre appareil StorSimple](../articles/storsimple/storsimple-8000-system-requirements.md).
       
        Vérifiez que le port 9354 est ouvert pour la communication sortante, car le Service Bus l’utilise pour la communication entre le service StorSimple Device Manager et l’appareil.