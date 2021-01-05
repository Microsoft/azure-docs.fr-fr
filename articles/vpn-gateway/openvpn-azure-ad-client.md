---
title: 'Passerelle VPN : client VPN pour les connexions P2S de protocole OpenVPN : Authentification Azure AD'
description: Apprenez à configurer un client VPN pour qu'il se connecte à un réseau virtuel à l'aide d'un VPN point à site et de l'authentification Azure Active Directory.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/15/2020
ms.author: alzam
ms.openlocfilehash: 7c2c1930b8f801db7f70baa5b713a641606be644
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96019773"
---
# <a name="azure-active-directory-authentication-configure-a-vpn-client-for-p2s-openvpn-protocol-connections"></a>Authentification Azure Active Directory : Configurer un client VPN pour les connexions P2S de protocole OpenVPN

Cet article vous aide à configurer un client VPN pour qu’il se connecte à un réseau virtuel à l’aide d’un VPN point à site et de l’authentification Azure Active Directory. Avant de pouvoir vous connecter et vous authentifier à l’aide d’Azure AD, vous devez d’abord configurer votre locataire Azure AD. Pour plus d’informations, consultez [Configurer un locataire Azure AD](openvpn-azure-ad-tenant.md).

[!INCLUDE [Windows 10 and OpenVPN note](../../includes/vpn-gateway-openvpn-auth-include.md)]

## <a name="working-with-client-profiles"></a><a name="profile"></a>Utilisation de profils clients

Pour vous connecter, vous devez télécharger Azure VPN Client et configurer un profil de client VPN sur chaque ordinateur qui veut se connecter au réseau virtuel. Vous pouvez créer un profil client sur un ordinateur, l’exporter, puis l’importer sur des ordinateurs supplémentaires.

### <a name="to-download-the-azure-vpn-client"></a>Pour télécharger le client VPN Azure

Utilisez ce [lien](https://go.microsoft.com/fwlink/?linkid=2117554) pour télécharger Azure VPN Client. Assurez-vous qu’Azure VPN Client est autorisé à s’exécuter en arrière-plan. Pour vérifier ou activer l’autorisation, suivez les étapes ci-dessous :

1. Accédez à Démarrer, puis sélectionnez Paramètres > Confidentialité > Applications en arrière-plan.
2. Sous Applications en arrière-plan, assurez-vous que l’option **Autoriser les applications à s’exécuter en arrière-plan** est activée.
3. Sous Choisir les applications qui peuvent s’exécuter en arrière-plan, configurez les paramètres d’Azure VPN Client sur **Activé**.

  ![autorisation](./media/openvpn-azure-ad-client/backgroundpermission.png)

### <a name="to-create-a-certificate-based-client-profile"></a><a name="cert"></a>Pour créer un profil client basé sur un certificat

Lorsque vous utilisez un profil basé sur un certificat, assurez-vous que les certificats appropriés sont installés sur l’ordinateur client. Pour plus d’informations sur les certificats, consultez [Installer un certificat client](point-to-site-how-to-vpn-client-install-azure-cert.md).

  ![cert](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="to-create-a-radius-client-profile"></a><a name="radius"></a>Pour créer un profil client RADIUS

  ![radius](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> Le secret du serveur peut être exporté dans le profil client VPN P2S.  Vous trouverez des instructions sur l’exportation d’un profil client [ici](about-vpn-profile-download.md).
>

### <a name="to-export-and-distribute-a-client-profile"></a><a name="export"></a>Pour exporter et distribuer un profil client

Une fois que vous disposez d’un profil fonctionnel et que vous devez le distribuer à d’autres utilisateurs, vous pouvez l’exporter en procédant comme suit :

1. Mettez en surbrillance le profil client VPN que vous souhaitez exporter, sélectionnez les points de sélection ( **...** ), puis sélectionnez **Exporter**.

    ![Capture d’écran montrant la page « Client VPN Azure », avec les points de suspension sélectionnés et « Exporter » mis en surbrillance.](./media/openvpn-azure-ad-client/export/export1.jpg)

2. Sélectionnez l’emplacement dans lequel vous souhaitez enregistrer ce profil, laissez le nom de fichier tel quel, puis sélectionnez **Enregistrer** pour enregistrer le fichier XML.

    ![export](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="to-import-a-client-profile"></a><a name="import"></a>Pour importer un profil client

1. Dans la page, sélectionnez **Importer**.

    ![Capture d’écran montrant le bouton « Ajouter » sélectionné et l’action « Importer » en surbrillance dans la partie inférieure gauche de la fenêtre.](./media/openvpn-azure-ad-client/import/import1.jpg)

2. Accédez au fichier XML de profil et sélectionnez-le. Une fois le fichier sélectionné, sélectionnez **Ouvrir**.

    ![Capture d’écran montrant un fichier de profil x m l sélectionné.](./media/openvpn-azure-ad-client/import/import2.jpg)

3. Spécifiez le nom du profil et sélectionnez **Enregistrer**.

    ![Capture d’écran montrant le « nom de la connexion » mis en surbrillance et le bouton « Enregistrer » sélectionné.](./media/openvpn-azure-ad-client/import/import3.jpg)

4. Sélectionnez **Se connecter** pour vous connecter au VPN.

    ![Capture d’écran montrant le VPN et le bouton « Connexion » sélectionné.](./media/openvpn-azure-ad-client/import/import4.jpg)

5. Une fois que vous êtes connecté, l’icône devient verte et indique **Connecté**.

    ![importer](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Pour supprimer un profil client

1. Sélectionnez les points de suspension en regard du profil client que vous souhaitez supprimer. Sélectionnez ensuite **Supprimer**.

    ![Capture d’écran montrant les ellipses et l’option « Supprimer » sélectionnée.](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. Sélectionnez **Supprimer** pour effectuer la suppression.

    ![supprimer](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="create-a-connection"></a><a name="connection"></a>Créer une connexion

1. Sur la page, sélectionnez **+** , puis **+ Ajouter**.

    ![Capture d’écran montrant le bouton « Ajouter » sélectionné.](./media/openvpn-azure-ad-client/create/create1.jpg)

2. Remplissez les informations de connexion. Si vous ne connaissez pas les valeurs, contactez votre administrateur. Quand vous avez rempli tous les champs, sélectionnez **Enregistrer**.

    ![Capture d’écran montrant les propriétés de la connexion VPN mises en surbrillance et le bouton « Enregistrer » sélectionné.](./media/openvpn-azure-ad-client/create/create2.jpg)

3. Sélectionnez **Se connecter** pour vous connecter au VPN.

    ![Capture d’écran montrant le VPN et le bouton « Connexion » sélectionné.](./media/openvpn-azure-ad-client/create/create3.jpg)

4. Sélectionnez les informations d’identification appropriées, puis sélectionnez **Continuer**.

    ![Capture d’écran montrant les informations d’identification de l’exemple en surbrillance et le bouton « Continuer » sélectionné.](./media/openvpn-azure-ad-client/create/create4.jpg)

5. Une fois que vous êtes connecté, l’icône devient verte et indique **Connecté**.

    ![connection](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="to-connect-automatically"></a><a name="autoconnect"></a>Pour se connecter automatiquement

Ces étapes vous aident à configurer votre connexion pour qu’elle se connecte automatiquement avec Always-on.

1. Sur la page d’accueil de votre client VPN, sélectionnez **Paramètres VPN**.

    ![Capture d’écran de la page d’accueil VPN avec l’option « Paramètres VPN » sélectionnée.](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. Sélectionnez **Oui** dans la boîte de dialogue de commutateur d’applications.

    ![Capture d’écran de la boîte de dialogue « Souhaitiez-vous changer d’application ? » avec le bouton « Oui » sélectionné.](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. Vérifiez que la connexion que vous souhaitez définir n’est pas déjà établie, puis mettez en surbrillance le profil et activez la case à cocher **Se connecter automatiquement**.

    ![Capture d’écran de la fenêtre « Paramètres », avec la case « Se connecter automatiquement » activée.](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. Sélectionnez **Se connecter** pour initier la connexion VPN.

    ![auto](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnostiquer les problèmes de connexion

1. Pour diagnostiquer les problèmes de connexion, vous pouvez utiliser l’outil **Diagnostiquer**. Sélectionnez les points de suspension ( **...** ) en regard de la connexion VPN que vous souhaitez diagnostiquer pour afficher le menu. Sélectionnez ensuite **Diagnostiquer**.

    ![Capture d’écran des ellipses et « Diagnostiquer » sélectionnés.](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. Dans la page **Propriétés de connexion**, sélectionnez **Exécuter le diagnostic**.

    ![Capture d’écran montrant la page « Propriétés de connexion » avec l’option « Exécuter le diagnostic » sélectionnée.](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. Connectez-vous avec vos informations d’identification.

    ![Capture d’écran montrant la boîte de dialogue « Connectez-vous » avec un « compte professionnel ou scolaire » sélectionné.](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. Examinez les résultats du diagnostic.

    ![diagnostiquer](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="is-the-azure-vpn-client-supported-with-windows-fips-mode"></a>Azure VPN Client est-il pris en charge avec le mode Windows FIPS ?

Oui, avec le correctif logiciel [KB4577063](https://support.microsoft.com/help/4577063/windows-10-update-kb4577063).

### <a name="how-do-i-add-dns-suffixes-to-the-vpn-client"></a>Comment faire pour ajouter des suffixes DNS au client VPN ?

Vous pouvez modifier le fichier XML de profil téléchargé et ajouter les balises **\<dnssuffixes>\<dnssufix> \</dnssufix>\</dnssuffixes>**

```
<azvpnprofile>
<clientconfig>

    <dnssuffixes>
          <dnssuffix>.mycorp.com</dnssuffix>
          <dnssuffix>.xyz.com</dnssuffix>
          <dnssuffix>.etc.net</dnssuffix>
    </dnssuffixes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-add-custom-dns-servers-to-the-vpn-client"></a>Comment ajouter des serveurs DNS personnalisés au client VPN ?

Vous pouvez modifier le fichier XML de profil téléchargé et ajouter les balises **\<dnsservers>\<dnsserver> \</dnsserver>\</dnsservers>**

```
<azvpnprofile>
<clientconfig>

    <dnsservers>
        <dnsserver>x.x.x.x</dnsserver>
        <dnsserver>y.y.y.y</dnsserver>
    </dnsservers>
    
</clientconfig>
</azvpnprofile>
```

> [!NOTE]
> Le client OpenVPN Azure AD utilise les entrées de la table de stratégie de résolution de noms DNS, ce qui signifie que les serveurs DNS ne seront pas listés sous la sortie de `ipconfig /all`. Pour confirmer vos paramètres DNS en cours d’utilisation, consultez [Get-DnsClientNrptPolicy](/powershell/module/dnsclient/get-dnsclientnrptpolicy?preserve-view=true&view=win10-ps) dans PowerShell.
>

### <a name="how-do-i-add-custom-routes-to-the-vpn-client"></a>Comment ajouter des routes personnalisées au client VPN ?

Vous pouvez modifier le fichier XML de profil téléchargé et ajouter les balises **\<includeroutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</includeroutes>**

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-block-exclude-routes-from-the-vpn-client"></a>Comment bloquer (exclure) des routes à partir du client VPN ?

Vous pouvez modifier le fichier XML de profil téléchargé et ajouter les balises **\<excluderoutes>\<route>\<destination>\<mask> \</destination>\</mask>\</route>\</excluderoutes>**

```
<azvpnprofile>
<clientconfig>

    <excluderoutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </excluderoutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="can-i-import-the-profile-from-a-command-line-prompt"></a>Puis-je importer le profil à partir d'une invite de ligne de commande ?

Vous pouvez importer le profil à partir d'une invite de ligne de commande en déplaçant le fichier **azurevpnconfig.xml** téléchargé vers le dossier **%userprofile%\AppData\Local\Packages\Microsoft.AzureVpn_8wekyb3d8bbwe\LocalState** et en exécutant la commande suivante :

```
azurevpn -i azurevpnconfig.xml 
```
Pour forcer l'importation, utilisez également le commutateur **-f**


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Créer un locataire Azure Active Directory qui utilise l’authentification Azure AD pour les connexions Open VPN P2S](openvpn-azure-ad-tenant.md).