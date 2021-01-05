---
title: Tutoriel expliquant comment connecter, configurer et activer l'appareil Azure Stack Edge Pro sur le portail Azure | Microsoft Docs
description: Le tutoriel consacré au déploiement d'Azure Stack Edge Pro avec GPU vous apprend à connecter, configurer et activer votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to connect and activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: 8a143dadffb3f89ef67dc20a2038bb3c9bf5a0e4
ms.sourcegitcommit: a07a01afc9bffa0582519b57aa4967d27adcf91a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91743333"
---
# <a name="tutorial-connect-set-up-and-activate-azure-stack-edge-pro"></a>Tutoriel : Connecter, configurer et activer Azure Stack Edge Pro 

Ce tutoriel vous explique comment connecter, configurer et activer votre appareil Azure Stack Edge Pro à l'aide de l'interface utilisateur web locale.

Le processus de configuration et d’activation peut prendre 20 minutes environ.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Se connecter à un appareil physique
> * Configurer et activer l’appareil physique

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d'activer votre appareil Azure Stack Edge Pro, vérifiez que :

* Vous avez installé l'appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro](azure-stack-edge-deploy-install.md).
* Vous disposez de la clé d'activation du service Azure Stack Edge que vous avez créé pour gérer l'appareil Azure Stack Edge Pro. Pour plus d'informations, consultez [Préparer le déploiement d'Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md).

## <a name="connect-to-the-local-web-ui-setup"></a>Se connecter à la configuration de l’interface utilisateur web locale

1. Configurez l'adaptateur Ethernet sur votre ordinateur pour vous connecter à l'appareil Azure Stack Edge Pro avec l'adresse IP statique 192.168.100.5 et le sous-réseau 255.255.255.0.

2. Connectez l’ordinateur au PORT 1 de votre appareil. Utilisez l’illustration ci-dessous pour identifier le PORT 1 sur votre appareil.

    ![Fond de panier d’un appareil câblé](./media/azure-stack-edge-deploy-install/backplane-cabled.png)

3. Ouvrez une fenêtre de navigateur et accédez à l’interface utilisateur web locale de l’appareil à l’adresse `https://192.168.100.10`.  
    Cette action peut prendre quelques minutes après la mise sous tension de l’appareil.

    Une erreur ou un avertissement vous indique que le certificat de sécurité du site web présente un problème.
   
    ![Message erreur de certificat de sécurité du site web](./media/azure-stack-edge-deploy-connect-setup-activate/image2.png)

4. Sélectionnez **Poursuivre sur cette page web**.  
    Ces étapes peuvent varier en fonction du navigateur que vous utilisez.

5. Connectez-vous à l’interface utilisateur web de votre appareil physique. Le mot de passe par défaut est *Password1*. 
   
    ![Page de connexion de l'appareil Azure Stack Edge Pro](./media/azure-stack-edge-deploy-connect-setup-activate/image3.png)

6. À l’invite, modifiez le mot de passe d’administrateur de l’appareil.  
    Le nouveau mot de passe doit contenir entre 8 et 16 caractères. Il doit contenir trois des types de caractères suivants : majuscule, minuscule, chiffre et caractère spécial.

Vous vous trouvez maintenant au niveau du tableau de bord de votre appareil.

## <a name="set-up-and-activate-the-physical-device"></a>Configurer et activer l’appareil physique
 
Le tableau de bord comprend les différents paramètres nécessaires à la configuration et à l’inscription de l’appareil physique auprès du service Azure Stack Edge. Les options **Nom de l’appareil**, **Paramètres réseau**, **Paramètres de proxy web** et **Paramètres de l’heure** sont facultatives. Les seuls paramètres obligatoires sont les **Paramètres cloud**.
   
![Page « Tableau de bord » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-1.png)

1. Dans le volet gauche, sélectionnez **Nom de l’appareil**, puis entrez un nom convivial pour votre appareil.  
    Ce nom convivial doit contenir entre 1 et 15 caractères et être composé de lettres, de chiffres et de traits d’union.

    ![Page « Nom de l'appareil » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-2.png)

2. (Facultatif) Dans le volet gauche, sélectionnez **Paramètres réseau**, puis configurez les paramètres.  
    Votre appareil physique comporte six interfaces réseau. Le PORT 1 et le PORT 2 sont des interfaces réseau de 1 Gbit/s. Les PORTS 3, 4, 5 et 6 sont tous des interfaces réseau de 25 Gbits/s qui peuvent également servir d’interfaces réseau de 10 Gbits/s. Le PORT 1 est automatiquement configuré comme port réservé à la gestion, et les PORTS 2 à 6 sont tous des ports de données. La page **Paramètres réseau** est illustrée ci-dessous.
    
    ![Page « Paramètres réseau » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-3.png)
   
    Lorsque vous configurez les paramètres réseau, gardez à l’esprit :

   - Si le protocole DHCP est activé dans votre environnement, les interfaces réseau sont configurées automatiquement. Une adresse IP, un sous-réseau, une passerelle et un DNS sont automatiquement attribués.
   - Si le protocole DHCP n’est pas activé, vous pouvez, le cas échéant, attribuer des adresses IP statiques.
   - Vous pouvez configurer votre interface réseau sur IPv4.

     >[!NOTE] 
     > Nous vous recommandons de ne pas remplacer Statique par DHCP pour l’adresse IP locale de l’interface réseau, sauf si vous disposez d’une autre adresse IP pour vous connecter à l’appareil. Si vous utilisez une seule interface réseau et que vous passez à DHCP, il n’y a aucun moyen de déterminer l’adresse DHCP. Si vous voulez passer à une adresse DHCP, attendez que l’appareil se soit inscrit auprès du service, puis faites la modification. Vous pouvez ensuite voir les adresses IP de tous les adaptateurs dans les **propriétés de l’appareil** dans le portail Azure pour votre service.

3. (Facultatif) Dans le volet de gauche, sélectionnez **Paramètres du proxy Web**, puis configurez votre serveur proxy web. Bien que la configuration du proxy web soit facultative, si vous en utilisez un, vous pouvez uniquement le configurer sur cette page.
   
   ![Page « Paramètres du proxy web » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-4.png)
   
   Dans la page **Paramètres du proxy web**, procédez comme suit :
   
   a. Dans la zone **URL de proxy Web**, entrez l’URL dans ce format : `http://host-IP address or FQDN:Port number`. Les URL HTTPS ne sont pas prises en charge.

   b. Sous **Authentification**, sélectionnez **Aucune** ou **NTLM**. Si vous activez le calcul et utilisez le module IoT Edge sur votre appareil Azure Stack Edge Pro, nous vous recommandons de définir l'authentification du proxy web sur **Aucune**. **NTLM** n’est pas pris en charge.

   c. Si vous utilisez une authentification, entrez un nom d’utilisateur et un mot de passe.

   d. Cliquez sur **Appliquer les paramètres** pour valider et appliquer les paramètres du proxy web que vous avez configurés.

   > [!NOTE]
   > Les fichiers PAC ne sont pas pris en charge. Un fichier PAC définit la manière dont les navigateurs web et les autres agents utilisateurs peuvent choisir automatiquement le serveur proxy approprié (méthode d’accès) pour récupérer une URL donnée.
   > Les proxys qui tentent d’intercepter et de lire tout le trafic (puis de tout resigner avec leur propre certification) ne sont pas compatibles, car le certificat du proxy n’est pas approuvé.
   > En général, les proxys transparents fonctionnent bien avec Azure Stack Edge Pro.

4. (Facultatif) Dans le volet de gauche, sélectionnez **Paramètres de l’heure**, puis configurez les paramètres de l’heure de votre appareil, notamment le fuseau horaire et les serveurs NTP principal et secondaire.  
    Les serveurs NTP sont requis. En effet, votre appareil doit synchroniser les heures pour pouvoir s’authentifier auprès de vos fournisseurs de services cloud.
       
    Dans la page **Paramètres de l’heure**, procédez comme suit :
    
    1. Dans la liste déroulante **Fuseau horaire**, sélectionnez le fuseau horaire qui correspond à l’emplacement géographique de l’appareil déployé.
        Le fuseau horaire par défaut de votre appareil est PST. Votre appareil utilise ce fuseau horaire pour toutes les opérations planifiées.

    2. Dans la zone **Serveur NTP principal**, entrez le serveur principal de votre appareil ou acceptez la valeur par défaut time.windows.com.  
        Vérifiez que votre réseau autorise le trafic NTP à passer de votre centre de données à Internet.

    3. Si vous le souhaitez, dans la zone **Serveur NTP secondaire**, entrez un serveur secondaire pour votre appareil.

    4. Cliquez sur **Appliquer les paramètres** pour valider et appliquer les paramètres d’heure que vous avez configurés.

        ![Page « Paramètres d’heure » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-5.png)

5. (Facultatif) Dans le volet gauche, sélectionnez **Paramètres de stockage** pour configurer la résilience de stockage sur votre appareil. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Par défaut, le stockage sur l’appareil n’est pas résilient, et il y a perte de données en cas de défaillance d’un disque de données sur l’appareil. Lorsque vous activez l’option Résilient, le stockage sur l’appareil est reconfiguré et l’appareil peut résister à la défaillance d’un disque de données sans perte de données. La configuration du stockage en tant que résilient réduit la capacité utilisable de votre appareil.

    > [!IMPORTANT] 
    > La résilience ne peut être configurée qu’avant l’activation de l’appareil. 

    ![Page « Paramètres de stockage » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/storage-settings.png)

6. Dans le volet de gauche, sélectionnez **Paramètres cloud**, puis activez votre appareil auprès du service Azure Stack Edge sur le portail Azure.
    
    1. Dans la zone **Clé d'activation**, entrez la clé d'activation que vous avez obtenue dans [Obtenir la clé d'activation](azure-stack-edge-deploy-prep.md#get-the-activation-key) pour Azure Stack Edge Pro.
    2. Sélectionnez **Appliquer**.
       
        ![Page « Paramètres cloud » de l’interface utilisateur web locale](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-6.png)

    3. Tout d’abord, l’appareil est activé. L’appareil est ensuite analysé pour rechercher des mises à jour critiques et, le cas échéant, ces mises à jour sont automatiquement appliquées. Une notification correspondante s’affiche.

        La boîte de dialogue contient également une clé de récupération que vous devriez copier et enregistrer à un emplacement sûr. Cette clé est utilisée pour récupérer vos données si l’appareil ne peut pas démarrer.

        ![Page « Paramètres cloud » de l’interface utilisateur web locale mise à jour](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-7.png)

    4. Vous devrez peut-être attendre plusieurs minutes après l’exécution de la mise à jour. La page se met à jour pour indiquer que l’appareil a bien été activé.

        ![Page « Paramètres cloud » de l’interface utilisateur web locale mise à jour 2](./media/azure-stack-edge-deploy-connect-setup-activate/set-up-activate-8.png)

La configuration de l’appareil est terminée. Vous pouvez maintenant ajouter des partages sur votre appareil.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Se connecter à un appareil physique
> * Configurer et activer l’appareil physique

Pour savoir comment transférer des données avec votre appareil Azure Stack Edge Pro, consultez :

> [!div class="nextstepaction"]
> [Transférer des données avec Azure Stack Edge Pro](./azure-stack-edge-deploy-add-shares.md)
