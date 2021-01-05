---
title: 'Configurer l’authentification Azure AD pour la connexion VPN utilisateur : WAN virtuel'
description: Découvrez comment configurer l’authentification Azure Active Directory pour une connexion VPN utilisateur.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 10/14/2020
ms.author: alzam
ms.openlocfilehash: 9cc68eb60096c4431acfc988c87ca9bf99f1f045
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93043397"
---
# <a name="configure-azure-active-directory-authentication-for-user-vpn"></a>Configurer l’authentification Azure Active Directory pour une connexion VPN utilisateur

Cet article vous montre comment configurer l’authentification Azure AD pour un VPN utilisateur dans Virtual WAN afin de vous connecter à vos ressources dans Azure par le biais d’une connexion VPN OpenVPN. L’authentification Azure Active Directory est disponible uniquement pour les passerelles basées sur le protocole OpenVPN et les clients Windows.

Pour utiliser ce type de connexion, un client doit être configuré sur l’ordinateur client. Pour plus d’informations sur Azure Virtual WAN, consultez l’article [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).

Dans cet article, vous apprendrez comment :

* Créer un réseau étendu virtuel
* Créer un hub virtuel
* Créer une configuration VPN d’utilisateur
* Télécharger un profil VPN d’utilisateur de réseau étendu virtuel
* Appliquer la configuration VPN d’utilisateur à un hub virtuel
* Connecter un réseau virtuel à un hub virtuel
* Télécharger et appliquer la configuration du client VPN utilisateur
* Afficher votre réseau étendu virtuel

![Diagramme WAN virtuel](./media/virtual-wan-about/virtualwanp2s.png)

## <a name="before-you-begin"></a>Avant de commencer

Vérifiez que vous disposez des éléments ci-dessous avant de commencer votre configuration :

* Vous avez un réseau virtuel auquel vous souhaitez vous connecter. Vérifiez qu’aucun des sous-réseaux de votre réseau local ne chevauche les réseaux virtuels auxquels vous souhaitez vous connecter. Pour créer un réseau virtuel dans le portail Azure, consultez le [guide de démarrage rapide](../virtual-network/quick-create-portal.md).

* Votre réseau virtuel n’a pas de passerelle de réseau virtuel. Si votre réseau virtuel dispose d’une passerelle (VPN ou ExpressRoute), vous devez supprimer toutes les passerelles. Cette configuration nécessite que les réseaux virtuels soient connectés à la passerelle hub Virtual WAN.

* Obtenez une plage d’adresses IP pour la région de votre hub. Le hub est un réseau virtuel qui est créé et utilisé par Virtual WAN. La plage d’adresses que vous spécifiez pour le hub ne peut pas chevaucher les réseaux virtuels existants auxquels vous vous connectez. Elle ne peut pas non plus chevaucher vos plages d’adresses auxquelles vous vous connectez en local. Si vous ne maîtrisez pas les plages d’adresses IP situées dans votre configuration de réseau local, contactez une personne en mesure de vous aider.

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-virtual-wan"></a><a name="wan"></a>Créer un réseau étendu virtuel

Dans un navigateur, accédez au [portail Azure](https://portal.azure.com) et connectez-vous avec votre compte Azure.

1. Accédez à la page WAN Virtuel. Dans le portail, cliquez sur **+Créer une ressource**. Tapez **WAN virtuel** dans la zone de recherche et sélectionnez Entrée.
2. Sélectionnez **WAN virtuel** dans les résultats. Dans la page Virtual WAN, cliquez sur **Créer** pour ouvrir la page Créer un WAN.
3. Dans la page **Créer un WAN** , sous l’onglet **Fonctions de base** , renseignez les champs suivants :

   ![WAN virtuel](./media/virtual-wan-point-to-site-azure-ad/vwan.png)

   * **Abonnement**  : sélectionnez l’abonnement que vous voulez utiliser.
   * **Groupe de ressources**  : créez-en un ou utilisez-en un qui existe déjà.
   * **Emplacement du groupe de ressources**  : choisissez un emplacement de la ressource dans la liste déroulante. Un WAN est une ressource globale et ne réside pas dans une région particulière. Toutefois, vous devez sélectionner une région pour gérer et localiser plus facilement la ressource WAN que vous créez.
   * **Nom**  : tapez le nom que vous souhaitez donner à votre WAN.
   * **Type :** Standard. Si vous créez un WAN de base, vous ne pourrez créer qu’un hub de base. Les hubs de base offrent uniquement une connectivité de site VPN à site.
4. Quand vous avez fini de renseigner les champs, cliquez sur **Vérifier + Créer**.
5. Une fois la validation effectuée, sélectionnez **Créer** pour créer le WAN virtuel.

## <a name="create-an-empty-virtual-hub"></a><a name="site"></a>Créer un hub virtuel vide

1. Sous votre réseau étendu virtuel, sélectionnez Hubs, puis cliquez sur **+ Nouveau hub**.

   ![Capture d’écran affichant la boîte de dialogue Hubs configuration avec l’option New Hub sélectionnée.](media/virtual-wan-point-to-site-azure-ad/hub1.jpg)
2. Dans la page Créer un hub virtuel, renseignez les champs suivants.

   **Région**  : sélectionnez la région dans laquelle vous souhaitez déployer le hub virtuel.

   **Nom**  : entrez le nom que vous souhaitez donner à votre hub virtuel.

   **Espace d’adressage privé du hub**  : plage d’adresses du hub en notation CIDR.

   ![Capture d’écran montrant le volet Create virtual hub (Créer un hub virtuel) dans lequel vous pouvez entrer des valeurs.](media/virtual-wan-point-to-site-azure-ad/hub2.jpg)  
3. Cliquez sur **Vérifier + créer**.
4. Dans la page **Validation effectuée** , cliquez sur **Créer**.

## <a name="create-a-new-user-vpn-configuration"></a><a name="site"></a>Créer une configuration VPN d’utilisateur

Une configuration VPN d’utilisateur définit les paramètres permettant de connecter des clients distants.

1. Sous votre WAN virtuel, sélectionnez **Configurations de VPN utilisateur**.

   ![Capture d’écran représentant l’élément de menu de configurations User VPN (VPN utilisateur) sélectionné.](media/virtual-wan-point-to-site-azure-ad/aadportal1.jpg)

2. Cliquez sur **+ Créer une configuration de VPN utilisateur**.

   ![Capture d’écran représentant le lient de configuration Create user VPN (Créer le VPN utilisateur).](media/virtual-wan-point-to-site-azure-ad/aadportal2.jpg)

3. Entrez les informations et cliquez sur **Créer**.

   * **Nom de la configuration**  : entrez le nom que vous souhaitez utiliser pour votre configuration VPN d’utilisateur.
   * **Type de tunnel**  : sélectionnez OpenVPN.
   * **Méthode d’authentification**  : sélectionnez Azure Active Directory.
   * **Public ciblé**  : saisissez l’ID de l’application d’entreprise [Azure VPN](openvpn-azure-ad-tenant.md) inscrite dans votre locataire Azure AD. 
   * **Émetteur** - `https://sts.windows.net/<your Directory ID>/`
   * **Locataire AAD** - `https://login.microsoftonline.com/<your Directory ID>`
  
   ![Capture d’écran montrant le volet de configuration Create new User VPN (Créer un VPN d’utilisateur) dans lequel vous pouvez entrer les valeurs.](media/virtual-wan-point-to-site-azure-ad/aadportal3.jpg)

## <a name="edit-hub-assignment"></a><a name="hub"></a>Modifier l’affectation du hub

1. Accédez au panneau **Hubs** situé sous le WAN virtuel.
2. Sélectionnez le hub à associer à la configuration du serveur VPN, puis cliquez sur le bouton de sélection (...).

   ![Capture d’écran montrant l’option Modifier le hub virtuel sélectionnée dans le menu.](media/virtual-wan-point-to-site-azure-ad/p2s4.jpg)
3. Cliquez sur **Modifier le hub virtuel**.
4. Cochez la case **Inclure la passerelle point à site** et sélectionnez l’ **unité d’échelle de passerelle** souhaitée.

   ![Capture d’écran montrant la boîte de dialogue Modifier le hub virtuel dans laquelle vous pouvez sélectionner l’unité d’échelle de la passerelle.](media/virtual-wan-point-to-site-azure-ad/p2s2.jpg)
5. Entrez le **Pool d’adresses** à partir duquel les clients VPN se verront attribuer des adresses IP.
6. Cliquez sur **Confirmer**.
7. L’opération peut prendre jusqu’à 30 minutes.

## <a name="download-user-vpn-profile"></a><a name="device"></a>Télécharger un profil VPN d’utilisateur

Utilisez le profil VPN pour configurer vos clients.

1. Dans la page de votre réseau étendu virtuel, cliquez sur **Configurations de VPN utilisateur**.
2. En haut de la page, cliquez sur **Télécharger la configuration de VPN utilisateur**.
3. Une fois que le fichier a terminé la création, vous pouvez cliquer sur le lien pour le télécharger.
4. Utilisez le fichier de profil pour configurer les clients VPN.

## <a name="configure-user-vpn-clients"></a>Configurer les clients VPN utilisateur

Pour établir une connexion, vous devez télécharger Azure VPN Client et importer le profil de client VPN téléchargé au cours des étapes précédentes sur chaque ordinateur à connecter au réseau virtuel.

> [!NOTE]
> L’authentification Azure AD n’est prise en charge que pour les connexions de protocole OpenVPN&reg;.
>

#### <a name="to-download-the-azure-vpn-client"></a>Pour télécharger le client VPN Azure

Utilisez ce [lien](https://www.microsoft.com/p/azure-vpn-client-preview/9np355qt2sqb?rtc=1&activetab=pivot:overviewtab) pour télécharger Azure VPN Client.

#### <a name="to-import-a-client-profile"></a><a name="import"></a>Pour importer un profil client

1. Dans la page, sélectionnez **Importer**.

    ![Capture d’écran montrant l’option Importer sélectionnée dans le menu Plus.](./media/virtual-wan-point-to-site-azure-ad/import/import1.jpg)

2. Accédez au fichier XML de profil et sélectionnez-le. Une fois le fichier sélectionné, sélectionnez **Ouvrir**.

    ![Capture d’écran montrant une boîte de dialogue Ouvrir dans laquelle vous pouvez sélectionner un fichier.](./media/virtual-wan-point-to-site-azure-ad/import/import2.jpg)

3. Spécifiez le nom du profil et sélectionnez **Enregistrer**.

    ![Capture d’écran montrant le nom de la connexion ajouté et le bouton Enregistrer sélectionné.](./media/virtual-wan-point-to-site-azure-ad/import/import3.jpg)

4. Sélectionnez **Se connecter** pour vous connecter au VPN.

    ![Capture d’écran montrant le bouton Se connecter pour la connexion que vous venez de créer.](./media/virtual-wan-point-to-site-azure-ad/import/import4.jpg)

5. Une fois que vous êtes connecté, l’icône devient verte et indique **Connecté**.

    ![Capture d’écran montrant la connexion dans un état Connecté avec l’option de déconnexion.](./media/virtual-wan-point-to-site-azure-ad/import/import5.jpg)

#### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Pour supprimer un profil client

1. Sélectionnez le bouton de sélection (...) en regard du profil de client à supprimer. Sélectionnez ensuite **Supprimer**.

    ![Capture d’écran montrant l’option Supprimer sélectionnée dans le menu.](./media/virtual-wan-point-to-site-azure-ad/delete/delete1.jpg)

2. Sélectionnez **Supprimer** pour effectuer la suppression.

    ![Capture d’écran représentant une boîte de dialogue de confirmation avec l’option Supprimer ou Annuler.](./media/virtual-wan-point-to-site-azure-ad/delete/delete2.jpg)

#### <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnostiquer les problèmes de connexion

1. Pour diagnostiquer les problèmes de connexion, vous pouvez utiliser l’outil **Diagnostiquer**. Sélectionnez le bouton de sélection (...) en regard de la connexion VPN à diagnostiquer pour afficher le menu. Sélectionnez ensuite **Diagnostiquer**.

    ![Capture d’écran montrant l’option Diagnostiquer sélectionnée dans le menu.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose1.jpg)

2. Dans la page **Propriétés de connexion** , sélectionnez **Exécuter le diagnostic**.

    ![Capture d’écran montre le bouton Exécuter le diagnostic pour une connexion.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose2.jpg)

3. Connectez-vous avec vos informations d’identification.

    ![Capture d’écran montrant la boîte de dialogue de connexion pour cette action.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose3.jpg)

4. Examinez les résultats du diagnostic.

    ![Capture d’écran affichant les résultats du diagnostic.](./media/virtual-wan-point-to-site-azure-ad/diagnose/diagnose4.jpg)

## <a name="view-your-virtual-wan"></a><a name="viewwan"></a>Afficher votre WAN virtuel

1. Accédez au WAN virtuel.
2. Dans la page Vue d’ensemble, chaque point sur la carte représente un hub.
3. Dans la section des hubs et des connexions, vous pouvez voir l’état du hub, le site, la région, l’état de la connexion VPN et les octets entrés et sortis.

## <a name="clean-up-resources"></a><a name="cleanup"></a>Supprimer des ressources

Quand vous n’avez plus besoin de ces ressources, vous pouvez utiliser [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour supprimer le groupe de ressources et toutes les ressources qu’il contient. Remplacez « myResourceGroup » par le nom de votre groupe de ressources et exécutez la commande PowerShell suivante :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Virtual WAN, consultez la page [Vue d’ensemble de Virtual WAN](virtual-wan-about.md).
