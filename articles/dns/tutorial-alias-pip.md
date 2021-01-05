---
title: 'Tutoriel : Créer un enregistrement d’alias Azure DNS pour référencer une adresse IP publique Azure'
description: Ce tutoriel vous montre comment configurer un enregistrement d’alias Azure DNS pour référencer une adresse IP publique Azure.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 9/25/2018
ms.author: rohink
ms.openlocfilehash: d3017d09e94040d16950598dad360fe32930c16b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80985437"
---
# <a name="tutorial-configure-an-alias-record-to-refer-to-an-azure-public-ip-address"></a>Tutoriel : Configurer un enregistrement d’alias pour faire référence à une adresse IP publique 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une infrastructure réseau.
> * Créer une machine virtuelle de serveur web avec une adresse IP publique.
> * Créer un enregistrement d’alias qui pointe vers l’adresse IP publique.
> * Tester l’enregistrement d’alias.


Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis
Vous devez disposer d’un nom de domaine disponible, que vous pouvez héberger dans Azure DNS pour le test. Vous devez disposer d’un contrôle total de ce domaine. Le contrôle total comprend notamment la possibilité de définir les enregistrements de serveur de noms pour le domaine.

Pour obtenir des instructions sur l’hébergement de votre domaine dans Azure DNS, consultez [Tutoriel : Héberger votre domaine dans Azure DNS](dns-delegate-domain-azure-dns.md).

L’exemple de domaine utilisé pour ce tutoriel est contoso.com, mais utilisez votre propre nom de domaine.

## <a name="create-the-network-infrastructure"></a>Créer l’infrastructure réseau
Créez d’abord un réseau virtuel et un sous-réseau pour y placer vos serveurs web.
1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans le portail en haut à gauche, sélectionnez **Créer une ressource**. Saisissez *groupe de ressources* dans la zone de recherche et créez un groupe de ressources nommé **RG-DNS-Alias-pip**.
3. Sélectionnez **Créer une ressource** > **Mise en réseau** > **Réseau virtuel**.
4. Créez un réseau virtuel nommé **VNet-Server**. Placez-le dans le groupe de ressources **RG-DNS-Alias-pip** et nommez le sous-réseau **SN-Web**.

## <a name="create-a-web-server-virtual-machine"></a>Créez une machine virtuelle de serveur web
1. Sélectionnez **Créer une ressource** > **Machine virtuelle Windows Server 2016**.
2. Saisissez **Web-01** pour le nom et placez la machine virtuelle dans le groupe de ressources **RG-DNS-Alias-TM**. Entrez un nom d’utilisateur et un mot de passe, puis sélectionnez **OK**.
3. Pour **Taille**, sélectionnez une référence SKU avec 8 Go de RAM.
4. Pour **Paramètres**, sélectionnez le réseau virtuel **VNet-Servers** et le sous-réseau **SN-Web**. Pour les ports d’entrée publics, sélectionnez **HTTP** > **HTTPS** > **RDP (3389)** , puis **OK**.
5. Dans la page **Résumé**, sélectionnez **Créer**.

Cette procédure s’achève après quelques minutes. La machine virtuelle est attachée à une carte réseau, qui a une adresse IP publique dynamique de base appelée Web-01-ip. L’adresse IP publique change à chaque redémarrage de la machine virtuelle.

### <a name="install-iis"></a>Installer IIS

Installez IIS sur **Web-01**.

1. Connectez-vous à **Web-01**.
2. Dans le tableau de bord **Gestionnaire de serveur**, sélectionnez **Ajouter des rôles et fonctionnalités**.
3. Sélectionnez **Suivant** trois fois de suite. Sur la page **Rôles de serveur**, cliquez sur **Serveur Web (IIS)** .
4. Sélectionnez **Ajouter des fonctionnalités**, puis sélectionnez **Suivant**.
5. Sélectionnez **Suivant** quatre fois de suite, puis **Installer**. Cette procédure s’achève après quelques minutes.
6. Une fois l’installation terminée, sélectionnez **Fermer**.
7. Ouvrez un navigateur web. Accédez à **localhost** pour vérifier que la page web IIS par défaut s’affiche.

## <a name="create-an-alias-record"></a>Créer un enregistrement d’alias

Créez un enregistrement d’alias qui pointe vers l’adresse IP publique.

1. Sélectionnez votre zone Azure DNS pour ouvrir la zone.
2. Sélectionnez **Jeu d’enregistrements**.
3. Dans la zone de texte **Nom**, sélectionnez **web01**.
4. Laissez le **Type** défini sur un enregistrement **A**.
5. Cochez la case **Alias du jeu d’enregistrements**.
6. Sélectionnez **Choisir un service Azure**, puis l’adresse IP publique **Web-01-ip**.

## <a name="test-the-alias-record"></a>Tester l’enregistrement d’alias

1. Dans le groupe de ressources **RG-DNS-Alias-pip**, sélectionnez la machine virtuelle **Web-01**. Notez l’adresse IP publique.
1. À partir d’un navigateur web, accédez au nom de domaine complet pour la machine virtuelle Web01-01. **web01.contoso.com** est un exemple. Vous voyez à présent la page web IIS par défaut.
2. Fermez le navigateur web.
3. Arrêtez la machine virtuelle **Web-01**, puis redémarrez-la.
4. Une fois la machine virtuelle redémarrée, notez la nouvelle adresse IP publique pour la machine virtuelle.
5. Ouvrez un nouveau navigateur web. Accédez de nouveau au nom de domaine complet pour la machine virtuelle Web01-01. **web01.contoso.com** est un exemple.

Cette procédure réussit, car vous avez utilisé un enregistrement d’alias pour pointer vers la ressource d’adresse IP publique, et non pas un enregistrement A standard.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’aurez plus besoin des ressources créées pour ce tutoriel, supprimez le groupe de ressources **RG-DNS-Alias-pip**.


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un enregistrement d’alias pour référencer une adresse IP publique Azure. Pour en savoir plus sur Azure DNS et les applications web, passez au tutoriel sur les applications web.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)
