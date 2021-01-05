---
title: Azure VMware Solution by CloudSimple - Configurer une passerelle VPN
description: Décrit comment configurer une passerelle VPN de point à site et une passerelle VPN de site à site et créer des connexions entre votre réseau local et votre cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: a8b7e238333196381524d189904871fe5933c906
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91871998"
---
# <a name="set-up-vpn-gateways-on-cloudsimple-network"></a>Configurer des passerelles VPN sur le réseau CloudSimple

Les passerelles VPN vous permettent de vous connecter à un réseau CloudSimple à partir de votre réseau local et d’un ordinateur client à distance. Une connexion VPN entre votre réseau local et votre réseau CloudSimple permet d’accéder à vCenter et aux charges de travail depuis votre cloud privé. CloudSimple prend en charge les passerelles VPN de site à site et de point à site.

## <a name="vpn-gateway-types"></a>Types de passerelle VPN

* La connexion **VPN de site à site** vous permet de configurer vos charges de travail de cloud privé pour accéder aux services locaux. Vous pouvez également utiliser Active Directory localement comme source d’identité pour l’authentification auprès de votre vCenter de cloud privé.  Actuellement, seul le type **VPN basé sur une stratégie** est pris en charge.
* Une connexion **VPN de point à site** est la façon la plus simple pour vous connecter à votre cloud privé à partir de votre ordinateur. Utilisez la connectivité VPN de point à site si pour vous connecter au cloud privé à distance. Pour plus d’informations sur l’installation d’un client pour une connexion VPN de point à site, consultez [Configurer une connexion VPN à votre cloud privé](set-up-vpn.md).

Dans une région, vous pouvez créer une passerelle VPN de point à site et une passerelle VPN de site à site.

## <a name="automatic-addition-of-vlansubnets"></a>Ajout automatique de réseaux/sous-réseaux virtuels locaux

Les passerelles VPN CloudSimple fournissent des stratégies pour l’ajout de réseaux/sous-réseaux locaux virtuels aux passerelles VPN.  Les stratégies vous permettent de spécifier des règles différentes pour les réseaux/sous-réseaux locaux virtuels de gestion et ceux définis par l’utilisateur.  Les règles des réseaux/sous-réseaux locaux virtuels de gestion s’appliquent à tous les nouveaux clouds privés que vous créez.  Les règles pour les réseaux/sous-réseaux locaux virtuels définis par l’utilisateur vous permettent d’ajouter automatiquement tout nouveau réseau/sous-réseau local virtuel à des clouds privés existants ou nouveaux. Pour une passerelle VPN de site à site, vous devez définir la stratégie pour chaque connexion.

Les stratégies relatives à l’ajout de réseaux/sous-réseaux locaux virtuels à des passerelles VPN s’appliquent aux passerelles VPN de site à site et de point à site.

## <a name="automatic-addition-of-users"></a>Ajout automatique d’utilisateurs

Une passerelle VPN de point à site vous permet de définir une stratégie d’ajout automatique pour les nouveaux utilisateurs. Par défaut, tous les propriétaires et contributeurs de l’abonnement ont accès au portail CloudSimple.  Les utilisateurs sont créés uniquement lorsque le portail CloudSimple est lancé pour la première fois.  La sélection de règles **Ajouter automatiquement** permet à tout nouvel utilisateur d’accéder au réseau CloudSimple à l’aide d’une connexion VPN de point à site.

## <a name="set-up-a-site-to-site-vpn-gateway"></a>Configurer une passerelle VPN de site à site

1. [Accédez au portail CloudSimple](access-cloudsimple-portal.md) et sélectionnez **Réseau**.
2. Sélectionnez **Passerelle VPN**.
3. Cliquez sur **Nouvelle passerelle VPN**.

    ![Créer la passerelle VPN](media/create-vpn-gateway.png)

4. Pour **Configuration de la passerelle**, spécifiez les paramètres suivants et cliquez sur **Suivant**.

    * Sélectionnez **VPN de site à site** comme type de passerelle.
    * Entrez un nom pour identifier la passerelle.
    * Sélectionnez l’emplacement Azure où votre service CloudSimple est déployé.
    * Si vous le souhaitez, activez la haute disponibilité.

    ![Créer une passerelle VPN de site à site](media/create-vpn-gateway-s2s.png)

    > [!WARNING]
    > L’activation de la haute disponibilité nécessite que votre périphérique VPN local prenne en charge la connexion à deux adresses IP. Cette option ne peut pas être désactivée une fois la passerelle VPN déployée.

5. Créez la première connexion à partir de votre réseau local, puis cliquez sur **Suivant**.

    * Entrez un nom pour identifier la connexion.
    * Pour l’adresse IP de l’homologue, entrez l’adresse IP publique de votre passerelle VPN locale.
    * Entrez l’identificateur d’homologue de votre passerelle VPN locale.  L’identificateur d’homologue est généralement l’adresse IP publique de votre passerelle VPN locale.  Si vous avez configuré un identificateur spécifique sur votre passerelle, entrez l’identificateur.
    * Copiez la clé partagée à utiliser pour la connexion à partir de votre passerelle VPN locale.  Pour modifier la clé partagée par défaut et en spécifier une nouvelle, cliquez sur l’icône Modifier.
    * Pour les **préfixes locaux**, entrez les préfixes CIDR locaux qui accéderont au réseau CloudSimple.  Vous pouvez ajouter plusieurs préfixes CIDR lorsque vous créez la connexion.

    ![Créer une connexion à la passerelle VPN de site à site](media/create-vpn-gateway-s2s-connection.png)

6. Activez les réseaux/sous-réseaux locaux virtuels de votre réseau de cloud privé qui seront accessibles à partir du réseau local, puis cliquez sur **Suivant**.

    * Pour ajouter un réseau/sous-réseau local virtuel de gestion, activez **Ajouter les réseaux/sous-réseaux locaux virtuels de gestion des clouds privés**.  Le sous-réseau de gestion est requis pour les sous-réseaux vMotion et vSAN.
    * Pour ajouter des sous-réseaux vMotion, activez **Ajouter le réseau vMotion des clouds privés**.
    * Pour ajouter des sous-réseaux vSAN, activez **Ajouter le sous-réseau vSAN des clouds privés**.
    * Sélectionnez ou désélectionnez des réseaux locaux virtuels spécifiques.

    ![Créer une connexion](media/create-vpn-gateway-s2s-connection-vlans.png)

7. Vérifiez les paramètres, puis cliquez sur **Envoyer**.

    ![Vérification et création de la passerelle VPN de site à site](media/create-vpn-gateway-s2s-review.png)

## <a name="create-point-to-site-vpn-gateway"></a>Créer une passerelle VPN de point à site

1. [Accédez au portail CloudSimple](access-cloudsimple-portal.md) et sélectionnez **Réseau**.
2. Sélectionnez **Passerelle VPN**.
3. Cliquez sur **Nouvelle passerelle VPN**.

    ![Créer la passerelle VPN](media/create-vpn-gateway.png)

4. Pour **Configuration de la passerelle**, spécifiez les paramètres suivants et cliquez sur **Suivant**.

    * Sélectionnez **VPN de point à site** comme type de passerelle.
    * Entrez un nom pour identifier la passerelle.
    * Sélectionnez l’emplacement Azure où votre service CloudSimple est déployé.
    * Spécifiez le sous-réseau client de la passerelle point à site.  Les adresses DHCP seront accordées à partir du sous-réseau client lorsque vous vous connectez.

5. Pour **Connexions/Utilisateur**, spécifiez les paramètres suivants et cliquez sur **Suivant**.

    * Pour autoriser automatiquement tous les utilisateurs actuels et futurs à accéder au cloud privé via cette passerelle point à site, sélectionnez **Ajouter automatiquement tous les utilisateurs**. Lorsque vous sélectionnez l’option, tous les utilisateurs dans la liste des utilisateurs sont automatiquement sélectionnés. Vous pouvez remplacer l’option automatique en désélectionnant des utilisateurs individuels dans la liste.
    * Pour sélectionner des utilisateurs individuels, cliquez sur les cases dans la liste des utilisateurs.

6. La section Réseaux locaux virtuels/Sous-réseaux vous permet de spécifier la gestion et les utilisateur des réseaux locaux virtuels/sous-réseaux pour la passerelle et les connexions.

    * Les options **Ajouter automatiquement** définissent la stratégie globale pour la passerelle. Les paramètres s’appliquent à la passerelle en cours. Les paramètres peuvent être remplacés dans la zone **Sélectionner**.
    * Sélectionnez **Ajouter la gestion des réseaux locaux virtuels/sous-réseaux de clouds privés**. 
    * Pour ajouter tous les réseaux locaux virtuels/sous-réseaux définis par l’utilisateur, cliquez sur **Ajouter des réseaux locaux virtuels/sous-réseaux définis par l’utilisateur**.
    * Les paramètres **Sélectionner** remplacent les paramètres généraux sous **Ajouter automatiquement**.

7. Cliquez sur **Suivant** pour passer en revue les paramètres. Cliquez sur les icônes Modifier pour apporter des modifications.
8. Cliquez sur **Créer** pour créer la passerelle VPN.

### <a name="client-subnet-and-protocols-for-point-to-site-vpn-gateway"></a>Sous-réseau client et protocoles pour la passerelle VPN de point à site

La passerelle VPN de point à site autorise les connexions TCP et UDP.  Choisissez le protocole à utiliser lorsque vous vous connectez à partir de votre ordinateur en sélectionnant la configuration TCP ou UDP.

Le sous-réseau client configuré est utilisé pour les clients TCP et UDP.  Le préfixe CIDR est divisé en deux sous-réseaux, un pour TCP et un pour les clients UDP. Choisissez le masque de préfixe en fonction du nombre d’utilisateurs VPN qui se connecteront simultanément.  

Le tableau suivant répertorie le nombre de connexions clientes simultanées pour le masque de préfixe.

| Masque de préfixe | /24 | /25 | /26 | /27 | /28 |
|-------------|-----|-----|-----|-----|-----|
| Nombre de connexions TCP simultanées | 124 | 60 | 28 | 12 | 4 |
| Nombre de connexions UDP simultanées | 124 | 60 | 28 | 12 | 4 |

Pour vous connecter à l’aide d’un VPN de point à site, consultez [Se connecter à CloudSimple à l’aide du VPN de point à site](set-up-vpn.md#connect-to-cloudsimple-using-point-to-site-vpn).
