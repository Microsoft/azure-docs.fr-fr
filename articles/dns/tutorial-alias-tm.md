---
title: 'Tutoriel : Créer un enregistrement d’alias pour prendre en charge les noms Apex de domaine – Traffic Manager'
titleSuffix: Azure DNS
description: Ce tutoriel vous montre comment configurer un enregistrement d’alias Azure DNS pour prendre en charge l’utilisation du nom d’apex de votre domaine avec Traffic Manager.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 9/25/2018
ms.author: rohink
ms.openlocfilehash: 4bdfc950cc1277809811dc2c548a57cc2138a8e4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77149947"
---
# <a name="tutorial-configure-an-alias-record-to-support-apex-domain-names-with-traffic-manager"></a>Tutoriel : Configurer un enregistrement d’alias pour prendre en charge des noms de domaine d’apex avec Traffic Manager 

Vous pouvez créer un enregistrement d’alias pour l’apex de votre nom de domaine pour référencer un profil Azure Traffic Manager. contoso.com en est un exemple. Au lieu d’utiliser un service de redirection, vous pouvez configurer Azure DNS pour référencer un profil Traffic Manager directement à partir de votre zone. 


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une infrastructure de machines virtuelles hôtes et de réseau.
> * Créer un profil Traffic Manager.
> * Créer un enregistrement d’alias.
> * Tester l’enregistrement d’alias.


Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis
Vous devez disposer d’un nom de domaine disponible, que vous pouvez héberger dans Azure DNS pour le test. Vous devez disposer d’un contrôle total de ce domaine. Le contrôle total comprend notamment la possibilité de définir les enregistrements de serveur de noms pour le domaine.

Pour obtenir des instructions sur l’hébergement de votre domaine dans Azure DNS, consultez [Tutoriel : Héberger votre domaine dans Azure DNS](dns-delegate-domain-azure-dns.md).

L’exemple de domaine utilisé pour ce tutoriel est contoso.com, mais utilisez votre propre nom de domaine.

## <a name="create-the-network-infrastructure"></a>Créer l’infrastructure réseau
Créez d’abord un réseau virtuel et un sous-réseau pour y placer vos serveurs web.
1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans le portail en haut à gauche, sélectionnez **Créer une ressource**. Saisissez *groupe de ressources* dans la zone de recherche et créez un groupe de ressources nommé **RG-DNS-Alias-TM**.
3. Sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**.
4. Créez un réseau virtuel nommé **VNet-Servers**. Placez-le dans le groupe de ressources **RG-DNS-Alias-TM** et nommez le sous-réseau **SN-Web**.

## <a name="create-two-web-server-virtual-machines"></a>Créer deux machines virtuelles de serveur web
1. Sélectionnez **Créer une ressource** > **Machine virtuelle Windows Server 2016**.
2. Saisissez **Web-01** pour le nom et placez la machine virtuelle dans le groupe de ressources **RG-DNS-Alias-TM**. Entrez un nom d’utilisateur et un mot de passe, puis sélectionnez **OK**.
3. Pour **Taille**, sélectionnez une référence SKU avec 8 Go de RAM.
4. Pour **Paramètres**, sélectionnez le réseau virtuel **VNet-Servers** et le sous-réseau **SN-Web**.
5. Sélectionnez **Adresse IP publique**. Sous **Affectation**, sélectionnez **Statique**, puis **OK**.
6. Pour les ports d’entrée publics, sélectionnez **HTTP** > **HTTPS** > **RDP (3389)** , puis **OK**.
7. Dans la page **Résumé**, sélectionnez **Créer**. Cette procédure s’achève après quelques minutes.

Répétez cette procédure pour créer une autre machine virtuelle nommée **Web-02**.

### <a name="add-a-dns-label"></a>Ajouter une étiquette DNS
Les adresses IP publiques nécessitent une étiquette DNS pour fonctionner avec Traffic Manager.
1. Dans le groupe de ressources **RG-DNS-Alias-TM**, sélectionnez l’adresse IP publique **Web-01-ip**.
2. Sous **Paramètres**, sélectionnez **Configuration**.
3. Dans la zone de texte Étiquette du nom DNS, saisissez **web01pip**.
4. Sélectionnez **Enregistrer**.

Répétez cette procédure pour l’adresse IP publique **Web-02-ip** en utilisant **web02pip** pour l’étiquette du nom DNS.

### <a name="install-iis"></a>Installer IIS

Installez IIS sur **Web-01** et sur **Web-02**.

1. Connectez-vous à **Web-01**.
2. Dans le tableau de bord **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et fonctionnalités**.
3. Sélectionnez **Suivant** trois fois de suite. Sur la page **Rôles de serveur**, cliquez sur **Serveur Web (IIS)** .
4. Sélectionnez **Ajouter des fonctionnalités**, puis **Suivant**.
5. Sélectionnez **Suivant** quatre fois de suite. Sélectionnez **Installer**. Cette procédure s’achève après quelques minutes.
6. Une fois l’installation terminée, sélectionnez **Fermer**.
7. Ouvrez un navigateur web. Accédez à **localhost** pour vérifier que la page web IIS par défaut s’affiche.

Répétez cette procédure pour installer IIS sur **Web-02**.


## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

1. Ouvrez le groupe de ressources **RG-DNS-Alias-TM**, puis sélectionnez l’adresse IP publique **Web-01-ip**. Notez l’adresse IP pour une utilisation ultérieure. Répétez cette étape pour l’adresse IP publique **Web-02-ip**.
1. Sélectionnez **Créer une ressource** > **Mise en réseau** > **Profil Traffic Manager**.
2. Pour le nom, entrez **TM-alias-test**. Placez-le dans le groupe de ressources **RG-DNS-Alias-TM**.
3. Sélectionnez **Create** (Créer).
4. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.
5. Dans la page Profil Traffic Manager, sous **Paramètres**, sélectionnez **Points de terminaison**.
6. Sélectionnez **Ajouter**.
7. Pour **Type**, sélectionnez **Point de terminaison externe**, et pour **Nom** saisissez **EP-Web01**.
8. Dans la zone de texte **Nom de domaine complet (FQDN) ou adresse IP**, saisissez l’adresse IP de **Web-01-ip** que vous avez notée précédemment.
9. Sélectionnez le même **emplacement** que vos autres ressources, puis **OK**.

Répétez cette procédure pour ajouter le point de terminaison **Web-02** en utilisant l’adresse IP que vous avez notée précédemment pour **Web-02-ip**.

## <a name="create-an-alias-record"></a>Créer un enregistrement d’alias

Créez un enregistrement d’alias qui pointe vers le profil Traffic Manager.

1. Sélectionnez votre zone Azure DNS pour ouvrir la zone.
2. Sélectionnez **Jeu d’enregistrements**.
3. Laissez la zone de texte **Nom** vide pour représenter l’apex du nom de domaine. contoso.com en est un exemple.
4. Laissez le **Type** défini sur un enregistrement **A**.
5. Cochez la case **Alias du jeu d’enregistrements**.
6. Cliquez sur **Choisir un service Azure**, puis sélectionnez le profil Traffic Manager **TM-alias-test**.

## <a name="test-the-alias-record"></a>Tester l’enregistrement d’alias

1. À partir d’un navigateur web, accédez à l’apex de votre nom de domaine. contoso.com en est un exemple. Vous voyez la page web IIS par défaut. Fermez le navigateur web.
2. Arrêtez la machine virtuelle **Web-01**. Patientez quelques minutes jusqu’à ce qu’elle s’arrête complètement.
3. Ouvrez un nouveau navigateur web et accédez à nouveau à l’apex de votre nom de domaine.
4. Vous voyez à nouveau la page web par défaut d’IIS, car Traffic Manager a géré la situation et dirigé le trafic vers **Web-02**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’aurez plus besoin des ressources créées pour ce tutoriel, supprimez le groupe de ressources **RG-DNS-Alias-TM**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un enregistrement d’alias afin d’utiliser l’apex de votre nom de domaine pour référencer un profil Traffic Manager. Pour en savoir plus sur Azure DNS et les applications web, passez au tutoriel sur les applications web.

> [!div class="nextstepaction"]
> [Héberger des applications web à charge équilibrée au niveau de l’apex de zone](./dns-alias-appservice.md)
