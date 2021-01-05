---
title: Guide de déploiement FortiGate | Microsoft Docs
description: Configurez et utilisez le produit de pare-feu nouvelle génération Fortinet FortiGate.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/30/2020
ms.author: jeedes
ms.openlocfilehash: cdaa6a9601452100ab90ef8b0f2191002f256b74
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025468"
---
# <a name="fortigate-azure-virtual-machine-deployment-guide"></a>Guide de déploiement de machine virtuelle Azure FortiGate

Utilisez ce guide de déploiement pour apprendre à configurer et à utiliser le produit de pare-feu nouvelle génération Fortinet FortiGate déployé en tant que machine virtuelle Azure. Vous allez également configurer l’application de la galerie Azure AD FortiGate SSL VPN pour fournir une authentification VPN via Azure Active Directory.

## <a name="redeem-the-fortigate-license"></a>Échanger la licence FortiGate

Le produit de pare-feu nouvelle génération Fortinet FortiGate est disponible en tant que machine virtuelle dans Azure IaaS (Infrastructure as a Service). Il existe deux modes de licence pour cette machine virtuelle : le paiement à l’utilisation et BYOL (apportez votre propre licence).

Si vous avez acheté une licence FortiGate auprès de Fortinet pour vous en servir avec l’option de déploiement de machine virtuelle BYOL, échangez-la dans la page d’activation des produits de Fortinet – https://support.fortinet.com. Le fichier de licence obtenu comportera une extension de fichier. lic.

## <a name="download-firmware"></a>Télécharger le microprogramme

Au moment de la rédaction de cet article, la version du microprogramme requise pour l’authentification SAML n’est pas fournie avec la machine virtuelle Azure Fortinet FortiGate. La version la plus récente doit être obtenue auprès de Fortinet.

1. Connectez-vous à https://support.fortinet.com/.
2. Accédez à **Download** > **Firmware Images** (Télécharger - Images du microprogramme).
3. Sélectionnez **Download** (Télécharger) à droite de **Release Notes** (Notes de publication).
4. Sélectionnez **v6.00** > **6.4** > **6.4.2**.
5. Téléchargez **FGT_VM64_AZURE-v6-build1723-FORTINET.out** en sélectionnant le lien **HTTPS** sur la même ligne.
6. Enregistrez le fichier pour une utilisation ultérieure.

## <a name="deploy-the-fortigate-vm"></a>Déployer la machine virtuelle FortiGate

1. Accédez au portail Azure et connectez-vous à l’abonnement dans lequel vous voulez déployer la machine virtuelle FortiGate.
2. Créez un groupe de ressources ou ouvrez le groupe de ressources dans lequel vous allez déployer la machine virtuelle FortiGate.
3. Sélectionnez **Ajouter**.
4. Dans la zone **Search the Marketplace** (Recherche sur le marketplace), entrez *Forti*. Sélectionnez **Fortinet FortiGate Next-Generation Firewall** (Pare-feu Fortigate nouvelle génération).
5. Sélectionnez le plan logiciel : BYOL (apportez votre propre licence) si vous disposez d’une licence ou Paiement à l’utilisation si ce n’est pas le cas. Sélectionnez **Create** (Créer).
6. Complétez la configuration de la machine virtuelle.

    ![Capture d’écran Créer une machine virtuelle.](virtual-machine.png)

7. Définissez **Authentication type** (Type d’authentification) sur **Password** (Mot de passe) et fournissez des informations d’identification d’administration pour la machine virtuelle.
8. Sélectionnez **Vérifier + créer** > **Créer**.
9. Attendez la fin du déploiement de la machine virtuelle.


### <a name="set-a-static-public-ip-address-and-assign-a-fully-qualified-domain-name"></a>Définir une adresse IP publique statique et affecter un nom de domaine complet

Pour bénéficier d’une expérience utilisateur cohérente, configurez l’adresse IP publique affectée à la machine virtuelle FortiGate comme affectée de manière statique. En outre, mappez-la à un nom de domaine complet (FQDN).

1. Accédez au portail Azure et ouvrez les paramètres de la machine virtuelle FortiGate.
2. Dans l’écran **Vue d’ensemble**, sélectionnez l’adresse IP publique.

    ![Capture d’écran du VPN SSL FortiGate.](public-ip-address.png)

3. Sélectionnez **Statique** > **Enregistrer**.

Si vous possédez un nom de domaine routable publiquement pour l’environnement dans lequel la machine virtuelle FortiGate est en cours de déploiement, créez un enregistrement d’hôte (A) pour la machine virtuelle. Cet enregistrement opère un mappage à l’adresse IP publique précédente qui est affectée de manière statique.

### <a name="create-a-new-inbound-network-security-group-rule-for-tcp-port-8443"></a>Créer une règle de groupe de sécurité réseau de trafic entrant pour un port TCP 8443

1. Accédez au portail Azure et ouvrez les paramètres de la machine virtuelle FortiGate.
2. Dans le menu de gauche, sélectionnez **Networking** (Mise en réseau). L’interface réseau est listée et les règles de port entrant s’affichent.
3. Sélectionnez **Ajouter une règle de port d’entrée**.
4. Créez une nouvelle règle de port entrant pour TCP 8443.

    ![Capture d’écran Ajouter une règle de sécurité de trafic entrant.](port-rule.png)

5. Sélectionnez **Ajouter**.

## <a name="create-a-second-virtual-nic-for-the-vm"></a>Créer une deuxième carte réseau virtuelle pour la machine virtuelle

Pour mettre les ressources internes à la disposition des utilisateurs, une deuxième carte réseau virtuelle doit être ajoutée à la machine virtuelle FortiGate. Le service Réseau virtuel dans Azure, sur lequel réside la carte réseau virtuelle, doit disposer d’une connexion routable vers ces ressources internes.

1. Accédez au portail Azure et ouvrez les paramètres de la machine virtuelle FortiGate.
2. Si la machine virtuelle FortiGate n’est pas déjà stoppée, sélectionnez **Arrêter** et attendez l’arrêt de la machine virtuelle.
3. Dans le menu de gauche, sélectionnez **Networking** (Mise en réseau).
4. Sélectionnez **Attacher l’interface réseau**.
5. Sélectionnez **Créer et attacher une interface réseau**.
6. Configurez les propriétés de la nouvelle interface réseau, puis sélectionnez **Créer**.

    ![Capture d’écran de Créer une interface réseau.](new-network-interface.png)

7. Déployez la machine virtuelle FortiGate.


## <a name="configure-the-fortigate-vm"></a>Configurer la machine virtuelle FortiGate

Les sections suivantes vous expliquent comment configurer la machine virtuelle FortiGate.

### <a name="install-the-license"></a>Installer la licence

1. Accédez à `https://<address>`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Continuez malgré toute erreur de certificat éventuelle.
3. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
4. Si le déploiement utilise le modèle BYOL (apportez votre propre licence), vous serez invité à charger une licence. Sélectionnez le fichier de licence créé précédemment et chargez-le. Sélectionnez **OK** et redémarrez la machine virtuelle FortiGate.

    ![Capture d’écran de la licence de la machine virtuelle FortiGate.](license.png)

5. Après le redémarrage, connectez-vous à nouveau avec les informations d’identification d’administrateur pour valider la licence.

### <a name="update-firmware"></a>Mettre à jour le microprogramme

1. Accédez à `https://<address>`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Continuez malgré toute erreur de certificat éventuelle.
3. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
4. Dans le menu de gauche, sélectionnez **System** > **Firmware** (Système - Microprogramme).
5. Dans **Firmware Management** (Gestion du microprogramme), sélectionnez **Browse** (Parcourir) et choisissez le fichier de microprogramme téléchargé précédemment.
6. Ignorez l’avertissement et sélectionnez **Backup config and upgrade** (Configuration et mise à niveau de la sauvegarde).

    ![Capture d’écran de Firmware Management.](backup-configure-upgrade.png)

7. Sélectionnez **Continuer**.
8. Lorsque vous êtes invité à enregistrer la configuration FortiGate (sous la forme d’un fichier .conf), sélectionnez **Save** (Enregistrer).
9. Attendez que le microprogramme soit téléchargé et appliqué. Attendez que la machine virtuelle FortiGate redémarre.
10. Après le redémarrage de la machine virtuelle FortiGate, connectez-vous à nouveau avec les informations d’identification d’administrateur.
11. Lorsque vous êtes invité à configurer le tableau de bord, sélectionnez **Later** (Ultérieurement).
12. Au début de la vidéo du tutoriel, sélectionnez **OK**

### <a name="change-the-management-port-to-tcp-8443"></a>Changer le port de gestion en port TCP 8443

1. Accédez à `https://<address>`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Continuez malgré toute erreur de certificat éventuelle.
3. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
4. Dans le menu de gauche, sélectionnez **System** (Système).
5. Sous **Administration Settings** (Paramètres d’administration), changez le port HTTPS en spécifiant **8443** et sélectionnez **Apply** (Appliquer).
6. Une fois la modification appliquée, le navigateur tente de recharger la page d’administration, mais l’opération échoue. Désormais, l’adresse de la page d’administration est `https://<address>:8443`.

    ![Capture d’écran de Upload Remote Certificate (Charger un certificat distant).](certificate.png)

### <a name="upload-the-azure-ad-saml-signing-certificate"></a>Charger le certificat de signature SAML Azure AD

1. Accédez à `https://<address>:8443`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Continuez malgré toute erreur de certificat éventuelle.
3. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
4. Dans le menu de gauche, sélectionnez **System** > **Certificates** (Système - Certificats).
5. Sélectionnez **Importer** > **Certificat distant**.
6. Accédez au certificat téléchargé à partir du déploiement de l’application personnalisée FortiGate dans le locataire Azure. Sélectionnez-le, puis cliquez sur **OK**.

### <a name="upload-and-configure-a-custom-ssl-certificate"></a>Charger et configurer un certificat SSL personnalisé

Vous pouvez configurer la machine virtuelle FortiGate avec votre propre certificat SSL qui prend en charge le nom de domaine complet que vous utilisez. Si vous avez accès à un certificat SSL empaqueté avec la clé privée au format .PFX, il peut être utilisé à cet effet.

1. Accédez à `https://<address>:8443`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Continuez malgré toute erreur de certificat éventuelle.
3. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
4. Dans le menu de gauche, sélectionnez **System** > **Certificates** (Système - Certificats).
5. Sélectionnez **Import** > **Local Certificate** > **PKCS #12 Certificate** (Importer - Certificat local - Certificat PKCS #12).
6. Accédez au fichier .PFX contenant le certificat SSL et la clé privée.
7. Spécifiez le mot de passe .PFX et un nom explicite pour le certificat. Sélectionnez ensuite **OK**.
8. Dans le menu de gauche, sélectionnez **System** > **Settings** (Système - Paramètres).
9. Sous **Administration Settings** (Paramètres d’administration), développez la liste en regard de **HTTPS server certificate** (Certificat de serveur HTTPS) et sélectionnez le certificat SSL importé précédemment.
10. Sélectionnez **Appliquer**.
11. Fermez la fenêtre du navigateur et accédez à `https://<address>:8443`.
12. Connectez-vous avec les informations d'identification d'administrateur FortiGate. Vous devez maintenant voir le certificat SSL correct en cours d’utilisation.

### <a name="configure-authentication-timeout"></a>Configurer le délai d’expiration de l’authentification

1. Accédez au portail Azure et ouvrez les paramètres de la machine virtuelle FortiGate.
2. Dans le menu de gauche, sélectionnez **Serial Console** (Console série).
3. Connectez-vous à la console série avec les informations d’identification d’administrateur de la machine virtuelle FortiGate.
4. Sur la console série, exécutez les commandes suivantes :

    ```
    config system global
    set remoteauthtimeout 60
    end
    ```

### <a name="ensure-network-interfaces-are-obtaining-ip-addresses"></a>Vérifier que les interfaces réseau obtiennent des adresses IP

1. Accédez à `https://<address>:8443`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
3. Dans le menu de gauche, sélectionnez **Networking** (Mise en réseau).
4. Sous Network (Réseau), sélectionnez **Interfaces**.
5. Examinez port1 (interface externe) et port2 (interface interne) pour vous assurer qu’ils obtiennent une adresse IP du sous-réseau Azure approprié.
    a. Si l’un des ports ne récupère pas d’adresse IP du sous-réseau (via DHCP), cliquez avec le bouton droit sur le port et sélectionnez **Edit** (Modifier).
    b. En regard de la zone du mode d’adressage, vérifiez que le protocole **DHCP** est sélectionné.
    c. Sélectionnez **OK**.

    ![Capture d’écran de l’adressage de l’interface réseau.](addressing.png)

### <a name="ensure-fortigate-vm-has-correct-route-to-on-premises-corporate-resources"></a>Vérifier que la machine virtuelle FortiGate dispose d’une route correcte vers les ressources d’entreprise locales

Les machines virtuelles Azure multirésidentes possèdent toutes les interfaces réseau sur le même réseau virtuel (mais peut-être des sous-réseaux distincts). Cela signifie souvent que les deux interfaces réseau sont dotées d’une connexion aux ressources d’entreprise locales publiées via FortiGate. C’est pourquoi il est nécessaire de créer des entrées de routage personnalisées, qui garantissent la sortie du trafic à partir de l’interface appropriée, lorsque des demandes de ressources d’entreprise locales sont effectuées.

1. Accédez à `https://<address>:8443`. Où `<address>` est le nom de domaine complet ou l’adresse IP publique affecté(e) à la machine virtuelle FortiGate.

2. Connectez-vous à l’aide des informations d’identification d’administrateur fournies lors du déploiement de la machine virtuelle FortiGate.
3. Dans le menu de gauche, sélectionnez **Networking** (Mise en réseau).
4. Sous Network (Réseau), sélectionnez **Static Routes** (Routes statiques).
5. Sélectionnez **Créer**.
6. En regard de la zone de destination, sélectionnez **Subnet** (Sous-réseau).
7. Sous Subnet, spécifiez les informations du sous-réseau dans lequel résident les ressources d’entreprise locales (par exemple, 10.1.0.0/255.255.255.0)
8. En regard de la zone d’adresse de la passerelle, spécifiez la passerelle sur le sous-réseau Azure où port2 est connecté (elle se termine généralement par 1, par exemple 10.6.1.1)
9. En regard de la zone d’interface, sélectionnez l’interface réseau interne, port2
10. Sélectionnez **OK**.

    ![Capture d’écran de la configuration d’une route.](route.png)

## <a name="configure-fortigate-ssl-vpn"></a>Configurer FortiGate SSL VPN

Suivre les étapes présentées dans https://docs.microsoft.com/azure/active-directory/saas-apps/fortigate-ssl-vpn-tutorial
