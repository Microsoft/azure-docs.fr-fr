---
title: Créer un profil Intune pour les clients VPN Azure
titleSuffix: Azure VPN Gateway
description: Découvrez comment créer un profil personnalisé Intune pour déployer des profils de client VPN Azure
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/15/2020
ms.author: cherylmc
ms.openlocfilehash: 63b1d7f95711017feba52cad97f05128d83734da
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94655168"
---
# <a name="create-an-intune-profile-to-deploy-vpn-client-profiles"></a>Créer un profil Intune pour déployer des profils de client VPN

Vous pouvez déployer des profils pour les clients VPN Azure (Windows 10) à l’aide de Microsoft Intune. Cet article vous aide à créer un profil Intune avec des paramètres personnalisés.

> [!NOTE]
> Cette méthode ne fonctionne que pour le déploiement de profils qui utilisent Azure Active Directory ou un certificat commun pour l’authentification du client. Si des certificats client uniques sont utilisés, chaque utilisateur doit sélectionner manuellement le certificat approprié dans Azure VPN Client.
>

## <a name="prerequisites"></a>Prérequis

* Les appareils sont déjà inscrits auprès de la gestion des appareils mobiles d’Intune.
* Le client VPN Azure pour Windows 10 est déjà déployé sur l’ordinateur client.
* Seul Windows version 19H2 ou ultérieure est pris en charge.

## <a name="modify-xml"></a><a name="xml"></a>Modifier le XML

Dans les étapes suivantes, nous utilisons un exemple de code XML pour un profil OMA-URI personnalisé pour Intune avec les paramètres suivants :

* Connexion automatique activée
* Détection des réseaux approuvés activée.

Pour d’autres options prises en charge, consultez l’article [CSP VPNv2](/windows/client-management/mdm/vpnv2-csp).

1. Téléchargez le profil VPN à partir du Portail Azure et extrayez le fichier *azurevpnconfig.xml* du package.
1. Copiez et collez le texte ci-dessous dans un nouveau fichier d’éditeur de texte.

   ```xml-interactive
    <VPNProfile>
      <!--<EdpModeId>corp.contoso.com</EdpModeId>-->
      <RememberCredentials>true</RememberCredentials>
      <AlwaysOn>true</AlwaysOn>
      <TrustedNetworkDetection>contoso.com,test.corp.contoso.com</TrustedNetworkDetection>
      <DeviceTunnel>false</DeviceTunnel>
      <RegisterDNS>false</RegisterDNS>
      <PluginProfile>
        <ServerUrlList>azuregateway-7cee0077-d553-4323-87df-069c331f58cb-053dd0f6af02.vpn.azure.com</ServerUrlList> 
        <CustomConfiguration>

        </CustomConfiguration>
        <PluginPackageFamilyName>Microsoft.AzureVpn_8wekyb3d8bbwe</PluginPackageFamilyName>
      </PluginProfile>
    </VPNProfile>
   ```
1. Modifiez l’entrée entre ```<ServerUrlList>``` et ```</ServerUrlList>``` par l’entrée de votre profil téléchargé (azurevpnconfig.xml). Modifiez le nom de domaine complet « TrustedNetworkDetection » pour l’adapter à votre environnement.
1. Ouvrez le profil Azure téléchargé (azurevpnconfig.xml) et copiez l’intégralité du contenu dans le presse-papiers en mettant en surbrillance le texte et en appuyant sur Ctrl + C. 
1. Collez le texte copié de l’étape précédente dans le fichier que vous avez créé à l’étape 2 entre les balises ```<CustomConfiguration>  </CustomConfiguration>```. Enregistrez le fichier avec l'extension .xml.
1. Notez la valeur dans les balises ```<name>  </name>```. Le nom du profil. Vous aurez besoin de ce nom lorsque vous créerez le profil dans Intune. Fermez le fichier et mémorisez l’emplacement où il est enregistré.

## <a name="create-intune-profile"></a>Créer un profil Intune

Dans cette section, vous allez créer un profil Microsoft Intune avec des paramètres personnalisés.

1. Connectez-vous à Intune et accédez à **Appareils-> Profils de configuration**. Sélectionnez **+ Créer un profil**.

   :::image type="content" source="./media/create-profile-intune/configuration-profile.png" alt-text="Profils de configuration":::
1. Pour **Plateforme**, sélectionnez **Windows 10 et ultérieur**. Pour **Profil**, sélectionnez **Personnalisé**. Sélectionnez ensuite **Create** (Créer).
1. Donnez un nom et une description au profil, puis sélectionnez **Suivant**.
1. Sous l’onglet **Paramètres de configuration**, sélectionnez **Ajouter**.

    * **Nom :** Entrez un nom pour la configuration.
    * **Description :** Description facultative.
    * **OMA-URI :** ```./User/Vendor/MSFT/VPNv2/<name of your connection>/ProfileXML``` (ces informations se trouvent dans le fichier azurevpnconfig.xml dans la balise <name></name>).
    * **Type de données :** Chaîne (fichier XML).

   Sélectionnez l’icône de dossier et choisissez le fichier que vous avez enregistré à l’étape 6 dans les étapes de [XML](#xml). Sélectionnez **Ajouter**.

   :::image type="content" source="./media/create-profile-intune/configuration-settings.png" alt-text="Paramètres de configuration" lightbox="./media/create-profile-intune/configuration-settings.png":::
1. Sélectionnez **Suivant**.
1. Sous **Affectations**, sélectionnez le groupe vers lequel vous souhaitez transmettre la configuration. Ensuite, sélectionnez **Suivant**.
1. Les règles d’applicabilité sont facultatives. Définissez des règles si nécessaire, puis sélectionnez **Suivant**.
1. Dans la page **Vérifier + créer**, sélectionnez **Créer**.

    :::image type="content" source="./media/create-profile-intune/create-profile.png" alt-text="Création d’un profil":::
1. Votre profil personnalisé est maintenant créé. Pour les étapes Microsoft Intune de déploiement de ce profil, consultez [Attribuer des profils d’utilisateur et d’appareil](/mem/intune/configuration/device-profile-assign).
 
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la fonctionnalité point à site, consultez l’article [À propos de la fonctionnalité point à site](point-to-site-about.md).