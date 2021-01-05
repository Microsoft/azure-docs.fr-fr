---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: ee92e145acd981bb8c84b43986b90306b994c262
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559838"
---
#### <a name="to-configure-and-register-the-device"></a>Configuration et inscription de l’appareil

1. Accédez à l’interface Windows PowerShell sur la console série de votre appareil StorSimple. Consultez la page [Utilisation de PuTTY pour se connecter à la console en série de l’appareil](#use-putty-to-connect-to-the-device-serial-console) pour obtenir des instructions. **Veillez à suivre la procédure pas à pas, sans quoi vous ne pourrez pas accéder à la console.**

2. Dans la session qui s’ouvre, appuyez une fois sur **Entrée** pour lancer une invite de commandes.

3. Vous devez choisir la langue que vous souhaitez définir pour votre appareil. Spécifiez la langue, puis appuyez sur **Entrée**.

4. Dans le menu de console en série qui s’affiche, choisissez l’option 1 : **Se connecter avec accès total**.
     Suivez les étapes 5 à 12 pour configurer les paramètres réseau minimum requis pour votre appareil. **Ces étapes de configuration doivent être effectuées sur le contrôleur actif de l’appareil.** Le menu de la console en série indique l’état du contrôleur dans le message de bannière. Si vous n’êtes pas connecté au contrôleur actif, déconnectez-vous, puis connectez-vous à ce contrôleur.

5. À l’invite de commandes, tapez votre mot de passe. Le mot de passe par défaut de l’appareil est **Password1**.

6. Tapez la commande suivante : `Invoke-HcsSetupWizard`.

7. Un Assistant Installation s’affiche pour vous aider à configurer les paramètres réseau de l’appareil. Fournissez les informations suivantes :
   
   * Adresse IP de l’interface réseau DATA 0
   * Masque de sous-réseau
   * Passerelle
   * Adresse IP du serveur DNS principal

   Un exemple de sortie est présenté ci-dessous.

    ```
        ---------------------------------------------------------------
        Microsoft Azure StorSimple Appliance Model 8100
        Name: 8100-SHX0991003G44MT
        Software Version: 6.3.9600.17759
        Copyright (C) 2014 Microsoft Corporation. All rights reserved.
        You are connected to Controller0 - Active
        ---------------------------------------------------------------

        Your device needs to be registered with the Microsoft Azure StorSimple Manager service. Please run 'Invoke-HcsSetupWizard' to set up your device.

        Controller0>Invoke-HcsSetupWizard

        Which IP address family would you like to configure on interface Data0?
        [4] IPv4 [6] IPv6 [B] Both (Default is "4"): 4

        Data0 IPv4 address:10.111.111.00
        Data0 IPv4 subnet: 255.255.252.0
        Data0 IPv4 gateway: 10.111.111.11

        IPv4 primary DNS server [10.222.118.154]:10.222.222.111
    ```

    <br>
    Dans l’exemple de sortie précédent, vous pouvez observer que le système valide les paramètres réseau après chaque étape du processus.

     > [!NOTE]
     > Vous devrez peut-être attendre quelques minutes que les paramètres de masque de sous-réseau et de DNS soient appliqués. Si vous obtenez le message d’erreur « Vérifiez la connectivité du réseau sur Data 0 », vérifiez la connexion réseau physique sur l’interface réseau DATA 0 de votre contrôleur actif.

8. (Facultatif) Configurez votre serveur proxy web. Bien que la configuration du proxy web soit facultative, **si vous en utilisez un, vous pouvez uniquement le configurer ici**. Pour plus d’informations, consultez la section [Configuration du proxy web pour votre appareil](../articles/storsimple/storsimple-8000-configure-web-proxy.md).
9. Configurez un serveur NTP principal pour votre appareil. Les serveurs NTP sont requis. En effet, votre appareil doit synchroniser les heures pour pouvoir s’authentifier auprès de vos fournisseurs de services cloud. Vérifiez que votre réseau autorise le trafic NTP à passer de votre centre de données à Internet. Si ce n’est pas possible, spécifiez un serveur NTP interne.

    Voici un exemple de sortie obtenue.

    ```
        Would you like to configure a web proxy?
        [Y] Yes [N] No (Default is "N"):N

        Primary NTP server [time.windows.com]:time.windows.com

    ```

10. Pour des raisons de sécurité, le mot de passe administrateur de l’appareil expire après la première session, et vous devez le modifier maintenant. Lorsque vous y êtes invité, fournissez un mot de passe administrateur de l’appareil. Un mot de passe administrateur d’appareil valide doit comprendre entre 8 et 15 caractères. Le mot de passe doit contenir trois des éléments suivants : caractères en minuscules, en majuscules, numériques et spéciaux.

    ```
        The device administrator password must be between 8 and 15 characters. The password must contain a combination of uppercase letters, lowercase letters, numbers and special characters.
        Administrator Password:********
        Confirm Administrator Password:********
    ```
11. La dernière étape de l’Assistant Installation inscrit votre appareil auprès du service StorSimple Device Manager. Pour cela, vous avez besoin de la clé d’inscription de service que vous avez obtenue à l’étape 2. Après avoir entré la clé d’enregistrement, vous devrez peut-être attendre 2 à 3 minutes avant que l’appareil ne soit inscrit.
    
    > [!NOTE]
    > Vous pouvez appuyer sur Ctrl + C à tout moment pour quitter l’Assistant Installation. Si vous avez entré tous les paramètres réseau (adresse IP pour Data 0, masque de sous-réseau et passerelle), vos entrées sont conservées.
    
    Voici un exemple de sortie obtenue.

    ```
        The service registration key is available in the StorSimple Manager service.
        Enter service registration key:**************************************
        Device registration is in progress. Please wait.

    ```

12. Une fois l’appareil inscrit, une clé de chiffrement de données de service s’affiche. Copiez-la et enregistrez-la en lieu sûr. **Cette clé et la clé d’inscription au service sont requises pour inscrire des appareils supplémentaires auprès du service StorSimple Device Manager.** Reportez-vous à la section [Sécurité StorSimple](../articles/storsimple/storsimple-8000-security.md) pour plus d’informations sur cette clé.
    
    ![Inscription de l’appareil StorSimple 7](./media/storsimple-8000-configure-and-register-device-u2/step3pssetup1.png)
    
    > [!NOTE]
    > Pour copier le texte à partir de la fenêtre de console de série, sélectionnez-le simplement. Vous devez ensuite pouvoir le coller dans le Presse-papiers ou dans un éditeur de texte. N’utilisez PAS Ctrl + C pour copier la clé de chiffrement de données de service. Cette combinaison de touches ferme l’Assistant Installation. Par conséquent, le mot de passe administrateur de l’appareil n’est pas modifié et l’appareil rétablit le mot de passe par défaut.
    
13. Quittez la console en série.
14. Revenez au portail Azure et procédez comme suit :
    
    1. Accédez à votre service StorSimple Device Manager.
    2. Cliquez sur **Appareils**.
    3. Dans la liste tabulaire des appareils, vérifiez que l’appareil s’est bien connecté au service en vérifiant son état. L’état de l’appareil doit être **Prêt pour la configuration**.
       
        ![Page Appareils StorSimple](./media/storsimple-8000-configure-and-register-device-u2/step3pssetup2.png)
       
        Vous devrez peut-être attendre quelques minutes pour que l’état de l’appareil s’affiche comme **Prêt pour la configuration**.
       
        Si l’appareil n’apparaît pas dans la liste, vous devez vous assurer que votre réseau de pare-feu a été configuré comme décrit dans [la configuration réseau requise pour votre appareil StorSimple](../articles/storsimple/storsimple-8000-system-requirements.md). Vérifiez que le port 9354 est ouvert pour la communication sortante, car le Service Bus l’utilise pour la communication entre le service StorSimple Device Manager et l’appareil.