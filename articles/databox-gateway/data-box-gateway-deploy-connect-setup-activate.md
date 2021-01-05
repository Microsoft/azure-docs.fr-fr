---
title: Se connecter à Azure Data Box Gateway, le configurer et l’activer dans le portail Azure
description: Ce troisième didacticiel sur le déploiement de Data Box Gateway vous apprend à connecter, configurer et activer votre appareil virtuel.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 03/18/2019
ms.author: alkohli
ms.openlocfilehash: cb6ef6ac25c4afa72160ba437e0ea3b5492cfd93
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580323"
---
# <a name="tutorial-connect-set-up-activate-azure-data-box-gateway"></a>Tutoriel : Se connecter à Azure Data Box Gateway, le configurer et l’activer

## <a name="introduction"></a>Introduction

Ce tutoriel explique comment vous connecter à votre appareil Data Box Gateway, le configurer et l’activer à l’aide de l’interface utilisateur web locale. 

Le processus de configuration et d’installation peut durer 10 minutes environ. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Vous connecter à un appareil virtuel
> * Configurer et activer l’appareil virtuel

## <a name="prerequisites"></a>Prérequis

Avant de configurer votre appareil Data Box Gateway, assurez-vous que :

* Vous avez approvisionné un appareil virtuel et obtenu une URL connectée à celui-ci comme indiqué dans [Tutorial: Provision Azure Data Box Gateway in Hyper-V (Preview)](data-box-gateway-deploy-provision-hyperv.md) (Didacticiel : Approvisionnement d’Azure Data Box Gateway dans Hyper-V (préversion)) ou [Tutorial: Provision Azure Data Box Gateway in VMware (Preview)](data-box-gateway-deploy-provision-vmware.md) (Didacticiel : Approvisionnement d’Azure Data Box Gateway dans VMware (préversion)).
* Vous disposez de la clé d’activation du service de Data Box Gateway que vous avez créée pour gérer les appareils Data Box Gateway. Pour plus d’informations, consultez [Tutorial: Prepare to deploy Azure Data Box Gateway (Preview)](data-box-gateway-deploy-prep.md) (Didacticiel : Préparation du déploiement d’Azure Data Box Gateway (préversion)).


## <a name="connect-to-the-local-web-ui-setup"></a>Se connecter à la configuration de l’interface utilisateur web locale 

1. Ouvrez une fenêtre de navigateur et accédez à l’interface utilisateur web locale de l’appareil à l’adresse :
   
   https:\//ip-address-of-network-interface
   
   Utilisez l’URL de connexion notée lors du didacticiel précédent. Une erreur ou un avertissement vous indique que le certificat de sécurité du site web présente un problème.

2. Sélectionnez **Poursuivre sur cette page web**. Ces étapes peuvent varier en fonction du navigateur que vous utilisez.
   
    ![Message erreur de certificat de sécurité du site web](./media/data-box-gateway-deploy-connect-setup-activate/image2.png)

3. Connectez-vous à l’interface utilisateur web de votre appareil virtuel. Le mot de passe par défaut est *Password1*. 
   
    ![Connectez-vous à l'interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image3.png)

4. À l’invite, modifiez le mot de passe de l’appareil. Le nouveau mot de passe doit contenir entre 8 et 16 caractères. Il doit contenir 3 caractères des types suivants : majuscules, minuscules, chiffres et caractères spéciaux.

    ![Modifier le mot de passe d’un appareil](./media/data-box-gateway-deploy-connect-setup-activate/image4.png)

Vous vous trouvez maintenant au niveau du **tableau de bord** de votre appareil.

## <a name="set-up-and-activate-the-virtual-device"></a>Configurer et activer l’appareil virtuel
 
Le tableau de bord présente les différents paramètres nécessaires à la configuration et à l’inscription de l’appareil virtuel auprès du service Data Box Gateway. Les options **Nom de l’appareil**, **Paramètres réseau**, **Paramètres de proxy web** et **Paramètres de l’heure** sont facultatives. Les seuls paramètres obligatoires sont les **Paramètres cloud**.
   
![Page « Tableau de bord » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image5.png)

1. Dans le volet gauche, sélectionnez **Nom de l’appareil**, puis entrez un nom convivial pour votre appareil. Ce nom convivial doit contenir entre 1 et 15 caractères et être composé de lettres, de chiffres et de traits d’union. 

    ![Page « Nom de l'appareil » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image6.png)

2. (Facultatif) Dans le volet gauche, sélectionnez **Paramètres réseau**, puis configurez les paramètres. Sur votre appareil virtuel, vous voyez une interface réseau minimum en fonction du nombre que vous avez configuré dans la machine virtuelle sous-jacente. La page **Paramètres réseau** d’un appareil virtuel avec une interface réseau activée est présentée ci-dessous.
    
    ![Page « Paramètres réseau » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image7.png)
   
    Lorsque vous configurez des paramètres réseau, gardez ce qui suit à l’esprit :

    - Si le protocole DHCP est activé dans votre environnement, les interfaces réseau sont configurées automatiquement. Une adresse IP, un sous-réseau, une passerelle et un DNS sont automatiquement attribués.
    - Si le protocole DHCP n’est pas activé, vous pouvez, le cas échéant, attribuer des adresses IP statiques.
    - Vous pouvez configurer votre interface réseau sur IPv4.

     >[!NOTE] 
     > Nous vous recommandons de ne pas replacer Statique par DHCP pour l'adresse IP locale de l'interface réseau, sauf si vous disposez d'une autre adresse IP pour vous connecter à l'appareil. Si vous utilisez une seule interface réseau et que vous passez à DHCP, il n’y a aucun moyen de déterminer l’adresse DHCP. Si vous voulez passer à une adresse DHCP, attendez que l’appareil se soit inscrit auprès du service, puis faites la modification. Vous pouvez ensuite voir les adresses IP de tous les adaptateurs dans les **propriétés de l’appareil** dans le portail Azure pour votre service.

3. (Facultatif) Configurez votre serveur proxy web. Bien que la configuration du proxy web soit facultative, si vous en utilisez un, vous pouvez le configurer uniquement sur cette page.
   
   ![Page « Paramètres du proxy web » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image8.png)
   
   À la page **Proxy web**, procédez comme suit :
   
   1. Dans la zone **URL de proxy Web**, entrez l’URL dans ce format : `http://&lt;host-IP address or FQDN&gt;:Port number`. Les URL HTTPS ne sont pas prises en charge.
   2. Sous **Authentification**, sélectionnez **Aucune** ou **NTLM**.
   3. Si vous utilisez une authentification, entrez un **nom d’utilisateur** et un **mot de passe**.
   4. Cliquez sur **Appliquer** pour valider et appliquer les paramètres du proxy web que vous avez configurés.

   > [!NOTE]
   > Les fichiers PAC ne sont pas pris en charge. Un fichier PAC définit la manière dont les navigateurs web et les autres agents utilisateurs peuvent choisir automatiquement le serveur proxy approprié (méthode d’accès) pour récupérer une URL donnée.
   > Les proxys qui tentent d’intercepter et de lire tout le trafic (puis de tout resigner avec leur propre certification) ne sont pas compatibles, car le certificat du proxy n’est pas approuvé.
   > En général, les proxys transparents fonctionnent bien avec Azure Data Box Gateway.

4. (Facultatif) Dans le volet de gauche, sélectionnez **Paramètres de l’heure**, puis configurez les paramètres de l’heure de votre appareil, notamment le fuseau horaire et les serveurs NTP principal et secondaire. 

    Les serveurs NTP sont requis. En effet, votre appareil doit synchroniser les heures pour pouvoir s’authentifier auprès de vos fournisseurs de services cloud.
    
    ![Page « Paramètres d’heure » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image9.png)
    
    À la page **Paramètres de l’heure**, procédez comme suit :
    
    1. Dans la liste déroulante **Fuseau horaire**, sélectionnez le fuseau horaire qui correspond à l’emplacement géographique de l’appareil déployé.
        Le fuseau horaire par défaut de votre appareil est PST. Votre appareil utilise ce fuseau horaire pour toutes les opérations planifiées.

    2. Spécifiez un **serveur NTP principal** pour votre appareil ou acceptez la valeur par défaut `time.windows.com`.   
        Vérifiez que votre réseau autorise le trafic NTP à passer de votre centre de données à Internet.

    3. Si vous le souhaitez, dans la zone **Serveur NTP secondaire**, entrez un serveur secondaire pour votre appareil.

    4. Cliquez sur **Appliquer** pour valider et appliquer les paramètres d’heure que vous avez configurés.

6. Dans le volet de gauche, sélectionnez **Paramètres cloud**, puis activez votre appareil avec le service Data Box Gateway dans le portail Azure.
    
    1. Dans la zone **Clé d’activation**, entrez la **Clé d’activation** que vous avez obtenue dans [Obtenir la clé d’activation](data-box-gateway-deploy-prep.md#get-the-activation-key) pour Data Box Gateway.

    2. Sélectionnez **Activer**.
       
         ![Page « Paramètres cloud » de l’interface utilisateur web locale](./media/data-box-gateway-deploy-connect-setup-activate/image10a.png)
    
    3. L’appareil est activé et, si elles sont disponibles, les mises à jour critiques sont automatiquement appliquées. Une notification correspondante s’affiche. Surveillez la progression de la mise à jour via le portail Azure.

        ![Page « Paramètres cloud » de l’interface utilisateur web locale 2](./media/data-box-gateway-deploy-connect-setup-activate/image12.png)
        
        **La boîte de dialogue contient également une clé de récupération que vous devriez copier et enregistrer à un emplacement sûr. Cette clé est utilisée pour récupérer vos données si l’appareil ne peut pas démarrer.**


    4. Vous devrez peut-être attendre plusieurs minutes pour que la mise à jour soit exécutée. Lorsque la mise à jour est terminée, connectez-vous à l’appareil. La page **Paramètres cloud** se met à jour pour indiquer que l’appareil a bien été activé.

        ![Page « Paramètres cloud » de l’interface utilisateur web locale mise à jour](./media/data-box-gateway-deploy-connect-setup-activate/image13.png)

La configuration de l’appareil est terminée. Vous pouvez maintenant ajouter des partages sur votre appareil.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Vous connecter à un appareil virtuel
> * Configurer et activer l’appareil virtuel

Pour découvrir comment transférer des données avec votre appareil Data Box Gateway, consultez :

> [!div class="nextstepaction"]
> [Transférer des données avec Data Box Gateway](./data-box-gateway-deploy-add-shares.md).
