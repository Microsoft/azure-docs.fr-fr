---
title: 'Tutoriel : Prérequis pour un groupe de disponibilité'
description: Ce tutoriel montre comment configurer les prérequis pour la création d’un groupe de disponibilité SQL Server AlwaysOn sur des machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: c492db4c-3faa-4645-849f-5a1a663be55a
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 03/29/2018
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 1a9a2f9d999a51f2b4600e8379d4a8913675b338
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97360014"
---
# <a name="tutorial-prerequisites-for-creating-availability-groups-on-sql-server-on-azure-virtual-machines"></a>Tutoriel : Prérequis de la création de groupes de disponibilité sur SQL Server sur des machines virtuelles Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce tutoriel montre comment remplir les prérequis pour la création d’un [groupe de disponibilité SQL Server AlwaysOn sur des machines virtuelles Azure](availability-group-manually-configure-tutorial.md). Une fois les prérequis remplis, vous disposez d’un contrôleur de domaine, de deux machines virtuelles SQL Server et d’un serveur témoin dans un même groupe de ressources.

Bien que cet article configure manuellement l’environnement du groupe de disponibilité, il est également possible de le faire à l’aide du [portail Azure](availability-group-azure-portal-configure.md), [de PowerShell ou d’Azure CLI](availability-group-az-commandline-configure.md) ou [de modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md). 

**Durée estimée** : remplir les conditions préalables peut prendre plusieurs heures. La majeure partie de ce temps est consacré à la création de machines virtuelles.

Le schéma suivant illustre ce que vous allez créer dans ce didacticiel.

![Groupe de disponibilité](./media/availability-group-manually-configure-prerequisites-tutorial-/00-EndstateSampleNoELB.png)

## <a name="review-availability-group-documentation"></a>Consulter la documentation sur le groupe de disponibilité

Ce didacticiel suppose que vous avez des notions de base sur les groupes de disponibilité AlwaysOn SQL Server. Si vous n’êtes pas familiarisé avec cette technologie, consultez [Vue d’ensemble des groupes de disponibilité AlwaysOn (SQL Server)](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server).


## <a name="create-an-azure-account"></a>Création d'un compte Azure

Vous avez besoin d’un compte Azure. Vous pouvez [ouvrir un compte Azure gratuit](https://signup.azure.com/signup?offer=ms-azr-0044p&appId=102&ref=azureplat-generic) ou [activer les avantages de l’abonnement à Visual Studio](/visualstudio/subscriptions/subscriber-benefits).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez **+** pour créer un objet dans le portail.

   ![Nouvel objet](./media/availability-group-manually-configure-prerequisites-tutorial-/01-portalplus.png)

3. Tapez **Groupe de ressources** dans la fenêtre de recherche **Marketplace**.

   ![Resource group](./media/availability-group-manually-configure-prerequisites-tutorial-/01-resourcegroupsymbol.png)

4. Sélectionnez **Groupe de ressources**.
5. Sélectionnez **Create** (Créer).
6. Sous **Nom du groupe de ressources**, saisissez un nom pour le groupe de ressources. Par exemple, tapez **sql-ha-rg**.
7. Si vous avez plusieurs abonnements Azure, vérifiez qu’il s’agit bien de celui dans lequel vous souhaitez créer le groupe de disponibilité.
8. Sélectionnez un emplacement. L’emplacement est la région Azure où vous souhaitez créer le groupe de disponibilité. Cet article génère toutes les ressources à un emplacement Azure.
9. Assurez-vous que l’option **Épingler au tableau de bord** est activée. Ce paramètre facultatif place un raccourci vers le groupe de ressources sur le tableau de bord du Portail Azure.

   ![Raccourci vers le groupe de ressources pour le portail Azure](./media/availability-group-manually-configure-prerequisites-tutorial-/01-resourcegroup.png)

10. Sélectionnez **Créer** pour créer le groupe de ressources.

Azure crée le groupe de ressources et épingle un raccourci vers celui-ci dans le portail.

## <a name="create-the-network-and-subnets"></a>Créer un réseau et des sous-réseaux

L’étape suivante consiste à créer les réseaux et les sous-réseaux dans le groupe de ressources Azure.

La solution utilise un réseau virtuel avec deux sous-réseaux. Pour plus d’informations sur les réseaux dans Azure, consultez la page [Présentation du réseau virtuel](../../../virtual-network/virtual-networks-overview.md).

Pour créer le réseau virtuel dans le portail Azure :

1. Dans votre groupe de ressources, sélectionnez **+ Ajouter**. 

   ![Nouvel élément](./media/availability-group-manually-configure-prerequisites-tutorial-/02-newiteminrg.png)
2. Recherchez **Réseau virtuel**.

     ![Rechercher un réseau virtuel](./media/availability-group-manually-configure-prerequisites-tutorial-/04-findvirtualnetwork.png)
3. Sélectionnez **Réseau virtuel**.
4. Sur le **Réseau virtuel**, sélectionnez le modèle de déploiement **Resource Manager**, puis **Créer**.

    Le tableau suivant indique les paramètres relatifs au réseau virtuel :

   | **Champ** | Valeur |
   | --- | --- |
   | **Nom** |autoHAVNET |
   | **Espace d’adressage** |10.33.0.0/24 |
   | **Nom du sous-réseau** |Admin |
   | **Plage d’adresses de sous-réseau** |10.33.0.0/29 |
   | **Abonnement** |Spécifiez l’abonnement à utiliser. Le champ **Abonnement** est vide si vous n'avez qu'un seul abonnement. |
   | **Groupe de ressources** |Choisissez **Utiliser existant** et sélectionnez le nom du groupe de ressources. |
   | **Lieu** |Spécifiez l’emplacement Azure. |

   Votre espace d’adressage et votre plage d’adresses de sous-réseau peuvent être différents de ceux indiqués dans ce tableau. En fonction de votre abonnement, le portail suggère un espace d’adressage disponible et la plage d’adresses de sous-réseau correspondante. Si vous ne disposez pas d’un espace d’adressage suffisant, utilisez un autre abonnement.

   L’exemple utilise le nom de sous-réseau **Admin**. Ce sous-réseau est destiné aux contrôleurs de domaine.

5. Sélectionnez **Create** (Créer).

   ![Configurer le réseau virtuel](./media/availability-group-manually-configure-prerequisites-tutorial-/06-configurevirtualnetwork.png)

Vous revenez au tableau de bord du portail et Azure vous avertit lorsque le réseau est créé.

### <a name="create-a-second-subnet"></a>Créer un deuxième sous-réseau

Le nouveau réseau virtuel dispose d'un sous-réseau, nommé **Admin**. Les contrôleurs de domaine utilisent ce sous-réseau. Les machines virtuelles SQL Server utilisent un deuxième sous-réseau nommé **SQL**. Pour configurer ce sous-réseau :

1. Dans votre tableau de bord, sélectionnez le groupe de ressources que vous avez créé, **SQL-HA-RG**. Recherchez le réseau dans le groupe de ressources, sous **Ressources**.

    Si **SQL-HA-RG** n’est pas visible, sélectionnez **Groupes de ressources** et filtrez les données en fonction du nom du groupe de ressources.

2. Sélectionnez **autoHAVNET** dans la liste des ressources. 
3. Sur le réseau virtuel **autoHAVNET**, sous **Paramètres**, sélectionnez **Sous-réseaux**.

    Vous pouvez voir le sous-réseau que vous avez déjà créé.

   ![Voir le sous-réseau déjà créé](./media/availability-group-manually-configure-prerequisites-tutorial-/07-addsubnet.png)

5. Pour créer un deuxième sous-réseau, sélectionnez **+ Sous-réseau**.
6. Sur **Ajouter un sous-réseau**, configurez le sous-réseau en saisissant **sqlsubnet** sous **Nom**. Azure spécifie automatiquement une **plage d’adresses** valide. Vérifiez que cette plage d’adresses comporte au moins 10 adresses. Dans un environnement de production, vous pouvez avoir besoin de davantage d’adresses.
7. Sélectionnez **OK**.

    ![Configurer le sous-réseau](./media/availability-group-manually-configure-prerequisites-tutorial-/08-configuresubnet.png)

Le tableau suivant récapitule les paramètres de configuration du réseau :

| **Champ** | Valeur |
| --- | --- |
| **Nom** |**autoHAVNET** |
| **Espace d’adressage** |Cette valeur dépend des espaces d’adressage disponibles dans votre abonnement. 10.0.0.0/16 est une valeur courante. |
| **Nom du sous-réseau** |**admin** |
| **Plage d’adresses de sous-réseau** |Cette valeur dépend des plages d’adresses disponibles dans votre abonnement. 10.0.0.0/24 est une valeur courante. |
| **Nom du sous-réseau** |**sqlsubnet** |
| **Plage d’adresses de sous-réseau** |Cette valeur dépend des plages d’adresses disponibles dans votre abonnement. 10.0.1.0/24 est une valeur courante. |
| **Abonnement** |Spécifiez l’abonnement à utiliser. |
| **Groupe de ressources** |**SQL-HA-RG** |
| **Lieu** |Spécifiez le même emplacement que celui choisi pour le groupe de ressources. |

## <a name="create-availability-sets"></a>Créer des groupes à haute disponibilité

Avant de créer des machines virtuelles, vous devez créer des groupes à haute disponibilité. Les groupes à haute disponibilité réduisent le temps d’arrêt lors des événements de maintenance planifiée et non planifiée. Un groupe à haute disponibilité Azure est un groupe de ressources logique qu’Azure place dans des domaines de mise à jour et d’erreur physiques. Un domaine d’erreur garantit que les membres du groupe à haute disponibilité disposent de ressources réseau et d’une alimentation distinctes. Un domaine de mise à jour permet de s’assurer que les membres du groupe à haute disponibilité ne sont pas arrêtés simultanément pour maintenance. Pour plus d’informations, consultez [Gestion de la disponibilité des machines virtuelles](../../../virtual-machines/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Vous avez besoin de deux groupes à haute disponibilité. L’un pour les contrôleurs de domaine. L’autre pour les machines virtuelles SQL Server.

Pour créer un groupe à haute disponibilité, accédez au groupe de ressources, puis sélectionnez **Ajouter**. Filtrez les résultats en tapant **groupe à haute disponibilité**. Sélectionnez **Groupe à haute disponibilité** dans les résultats, puis **Créer**.

Configurez deux groupes à haute disponibilité en vous basant sur les paramètres du tableau suivant :

| **Champ** | Groupe à haute disponibilité du contrôleur de domaine | Groupe à haute disponibilité SQL Server |
| --- | --- | --- |
| **Nom** |adavailabilityset |sqlavailabilityset |
| **Groupe de ressources** |SQL-HA-RG |SQL-HA-RG |
| **Domaines d'erreur** |3 |3 |
| **Domaines de mise à jour** |5 |3 |

Après avoir créé les groupes à haute disponibilité, revenez au groupe de ressources dans le Portail Azure.

## <a name="create-domain-controllers"></a>Création de contrôleurs de domaine

Une fois que vous avez créé le réseau, les sous-réseaux et les groupes à haute disponibilité, vous êtes prêt à créer les machines virtuelles associées aux contrôleurs de domaine.

### <a name="create-virtual-machines-for-the-domain-controllers"></a>Créer les machines virtuelles associées aux contrôleurs de domaine

Pour créer et configurer les contrôleurs de domaine, revenez au groupe de ressources **SQL-HA-RG** .

1. Sélectionnez **Ajouter**. 
2. Tapez **Windows Server 2016 Datacenter**.
3. Sélectionnez **Windows Server 2016 Datacenter**. Dans **Windows Server 2016 Datacenter**, vérifiez que le modèle de déploiement est défini sur **Gestionnaires des ressources**, puis sélectionnez **Créer**. 

Répétez les étapes précédentes pour créer deux machines virtuelles. Nommez les deux machines virtuelles comme suit :

* ad-primary-dc
* ad-secondary-dc

  > [!NOTE]
  > La machine virtuelle **ad-secondary-dc** est un composant facultatif utilisé pour fournir une haute disponibilité à Active Directory Domain Services.
  >

Le tableau suivant indique les paramètres relatifs à ces deux machines :

| **Champ** | Valeur |
| --- | --- |
| **Nom** |Premier contrôleur de domaine : *ad-primary-dc*.</br>Second contrôleur de domaine *ad-secondary-dc*. |
| **Type de disque de machine virtuelle** |SSD |
| **Nom d'utilisateur** |DomainAdmin |
| **Mot de passe** |Contoso!0000 |
| **Abonnement** |*Votre abonnement* |
| **Groupe de ressources** |SQL-HA-RG |
| **Lieu** |*Votre emplacement* |
| **Taille** |DS1_V2 |
| **Stockage** | **Utiliser des disques managés** - **Oui** |
| **Réseau virtuel** |autoHAVNET |
| **Sous-réseau** |admin |
| **Adresse IP publique** |*Même nom que la machine virtuelle* |
| **Groupe de sécurité réseau** |*Même nom que la machine virtuelle* |
| **Groupe à haute disponibilité** |adavailabilityset </br>**Domaines d'erreur** : 2 </br>**Domaines de mise à jour** : 2|
| **Diagnostics** |activé |
| **Compte de stockage de diagnostics** |*Créé automatiquement* |

   >[!IMPORTANT]
   >Vous pouvez uniquement placer une machine virtuelle dans un groupe de disponibilité lors de sa création. Vous ne pouvez pas modifier le groupe à haute disponibilité après la création d’une machine virtuelle. Consultez la rubrique [Gérer la disponibilité des machines virtuelles](../../../virtual-machines/manage-availability.md).

Azure crée les machines virtuelles.

Une fois les machines virtuelles créées, configurez le contrôleur de domaine.

### <a name="configure-the-domain-controller"></a>Configurer le contrôleur de domaine

Dans les étapes suivantes, vous allez configurer la machine **ad-primary-dc** comme contrôleur de domaine pour corp.contoso.com.

1. Dans le portail, ouvrez le groupe de ressources **SQL-HA-RG**, puis sélectionnez la machine **ad-primary-dc**. Sur **ad-primary-dc**, sélectionnez **Connecter** pour ouvrir un fichier RDP pour l’accès au Bureau à distance.

    ![Connexion à une machine virtuelle](./media/availability-group-manually-configure-prerequisites-tutorial-/20-connectrdp.png)

2. Connectez-vous avec votre compte Administrateur configuré ( **\DomainAdmin**) et votre mot de passe (**Contoso!0000**).
3. Par défaut, le tableau de bord **Gestionnaire de serveur** doit être affiché.
4. Sélectionnez le lien du tableau de bord **Ajouter des rôles et fonctionnalités**.

    ![Gestionnaire de serveur - Ajouter des rôles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)

5. Sélectionnez **Suivant** jusqu’à ce que vous atteigniez la section **Rôles de serveur**.
6. Sélectionnez les rôles **Services de domaine Active Directory** et **Serveur DNS**. Lorsque vous y êtes invité, ajoutez les fonctionnalités supplémentaires requises par ces rôles.

   > [!NOTE]
   > Windows vous avertit qu’il n’y a aucune adresse IP statique. Si vous testez la configuration, sélectionnez **Continuer**. Pour les scénarios de production, définissez l’adresse IP comme statique dans le portail Azure ou [utilisez PowerShell pour définir l’adresse IP statique de la machine de contrôleur de domaine](/previous-versions/azure/virtual-network/virtual-networks-reserved-private-ip).
   >

    ![Boîte de dialogue Ajouter des rôles](./media/availability-group-manually-configure-prerequisites-tutorial-/23-addroles.png)

7. Sélectionnez **Suivant** jusqu’à ce que vous atteigniez la section **Confirmation**. Cochez la case **Redémarrer automatiquement le serveur de destination, si nécessaire**.
8. Sélectionnez **Installer**.
9. Une fois les fonctionnalités installées, retournez dans le tableau de bord **Gestionnaire de serveur** .
10. Sélectionnez la nouvelle option **AD DS** dans le volet gauche.
11. Sélectionnez le lien **Plus** dans la barre d’avertissement jaune.

    ![Boîte de dialogue AD DS sur la machine virtuelle du serveur DNS](./media/availability-group-manually-configure-prerequisites-tutorial-/24-addsmore.png)
    
12. Dans la colonne **Action** de la boîte de dialogue **Tous les détails de la tâche serveur**, sélectionnez **Promouvoir ce serveur en contrôleur de domaine**.
13. Dans l' **Assistant de configuration des services de domaine Active Directory**, utilisez les valeurs suivantes :

    | **Page** | Paramètre |
    | --- | --- |
    | **Configuration du déploiement** |**Ajouter une nouvelle forêt**<br/> **Nom de domaine racine** = corp.contoso.com |
    | **Options de contrôleur de domaine :** |**Mot de passe DSRM** = Contoso!0000<br/>**Confirmer le mot de passe** = Contoso!0000 |

14. Sélectionnez **Suivant** pour parcourir les autres pages de l’Assistant. Sur la page **Vérification de la configuration requise**, vérifiez que vous voyez le message suivant : **Toutes les vérifications de la configuration requise ont donné satisfaction**. Vous pouvez examiner les messages d’avertissement applicables, mais il est possible de poursuivre l’installation.
15. Sélectionnez **Installer**. La machine virtuelle **ad-primary-dc** redémarre automatiquement.

### <a name="note-the-ip-address-of-the-primary-domain-controller"></a>Notez l’adresse IP du contrôleur de domaine principal

Utilisez le contrôleur de domaine principal pour DNS. Notez l’adresse IP du contrôleur de domaine principal.

Vous pouvez obtenir l’adresse IP du contrôleur de domaine principal via le portail Azure.

1. Sur le portail Azure, ouvrez le groupe de ressources.

2. Sélectionnez le contrôleur de domaine principal.

3. Sur le contrôleur de domaine principal, sélectionnez **Interfaces réseau**.

![Interfaces réseau](./media/availability-group-manually-configure-prerequisites-tutorial-/25-primarydcip.png)

Notez l’adresse IP privée de ce serveur.

### <a name="configure-the-virtual-network-dns"></a>Configurer le DNS du réseau virtuel

Après avoir créé le premier contrôleur de domaine et activé DNS sur le premier serveur, configurez le réseau virtuel pour utiliser ce serveur pour le DNS.

1. Dans le portail Azure, sélectionnez le réseau virtuel.

2. Sous **Paramètres**, sélectionnez **Serveur DNS**.

3. Sélectionnez **Personnalisé** et tapez l’adresse IP privée du contrôleur de domaine principal.

4. Sélectionnez **Enregistrer**.

### <a name="configure-the-second-domain-controller"></a>Configuration du second contrôleur de domaine

Une fois le contrôleur de domaine principal redémarré, vous pouvez configurer le second contrôleur de domaine. Cette étape facultative intervient pour les scénarios de haute disponibilité. Suivez ces étapes pour configurer le second contrôleur de domaine :

1. Dans le portail, ouvrez le groupe de ressources **SQL-HA-RG**, puis sélectionnez la machine **ad-secondary-dc**. Sur **ad-secondary-dc**, sélectionnez **Connecter** pour ouvrir un fichier RDP pour l’accès au Bureau à distance.
2. Connectez-vous à la machine virtuelle avec votre compte Administrateur configuré (**BUILTIN\DomainAdmin**) et votre mot de passe (**Contoso!0000**).
3. Remplacez l’adresse du serveur DNS préféré par celle du contrôleur de domaine.
4. Dans **Centre Réseau et partage**, sélectionnez l’interface réseau.

   ![interface réseau](./media/availability-group-manually-configure-prerequisites-tutorial-/26-networkinterface.png)

5. Sélectionner **Propriétés**.
6. Sélectionnez **Internet Protocol Version 4 (TCP/IPv4)** , puis **Propriétés**.
7. Sélectionnez **Utiliser l’adresse de serveur DNS suivante** et spécifiez l’adresse du contrôleur de domaine principal sous **Serveur DNS préféré**.
8. Sélectionnez **OK**, puis **Fermer** pour valider les modifications. Vous pouvez maintenant joindre la machine virtuelle à **corp.contoso.com**.

   >[!IMPORTANT]
   >Si vous perdez la connexion à votre Bureau à distance après avoir modifié le paramètre DNS, accédez au portail et redémarrez la machine virtuelle.

9. À partir du Bureau à distance au contrôleur de domaine secondaire, ouvrez le **tableau de bord du Gestionnaire de serveur**.
10. Sélectionnez le lien du tableau de bord **Ajouter des rôles et fonctionnalités**.

    ![Gestionnaire de serveur - Ajouter des rôles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)
11. Sélectionnez **Suivant** jusqu’à ce que vous atteigniez la section **Rôles de serveur**.
12. Sélectionnez les rôles **Services de domaine Active Directory** et **Serveur DNS**. Lorsque vous y êtes invité, ajoutez les fonctionnalités supplémentaires requises par ces rôles.
13. Une fois les fonctionnalités installées, retournez dans le tableau de bord **Gestionnaire de serveur** .
14. Sélectionnez la nouvelle option **AD DS** dans le volet gauche.
15. Sélectionnez le lien **Plus** dans la barre d’avertissement jaune.
16. Dans la colonne **Action** de la boîte de dialogue **Tous les détails de la tâche serveur**, sélectionnez **Promouvoir ce serveur en contrôleur de domaine**.
17. Sous **Configuration du déploiement**, sélectionnez **Ajouter un contrôleur de domaine à un domaine existant**.

    ![Configuration du déploiement](./media/availability-group-manually-configure-prerequisites-tutorial-/28-deploymentconfig.png)

18. Cliquez sur **Sélectionner**.
19. Connectez-vous à l’aide du compte Administrateur (**CORP.CONTOSO.COM\domainadmin**) et du mot de passe (**Contoso!0000**).
20. Dans **Sélectionner un domaine dans la forêt**, choisissez votre domaine, puis sélectionnez **OK**.
21. Dans **Options du contrôleur de domaine**, utilisez les valeurs par défaut et définissez un mot de passe DSRM.

    >[!NOTE]
    >La page **Options DNS** peut vous informer de l’impossibilité de créer une délégation pour ce serveur DNS. Vous pouvez ignorer cet avertissement dans les environnements hors production.
    >

22. Sélectionnez **Suivant** jusqu’à ce que la boîte de dialogue affiche la page de vérification **Conditions préalables**. Sélectionnez **Installer**.

Une fois que le serveur a terminé de modifier la configuration, redémarrez-le.

### <a name="add-the-private-ip-address-to-the-second-domain-controller-to-the-vpn-dns-server"></a>Ajouter l’adresse IP privée au second contrôleur de domaine pour le serveur DNS VPN

Dans le portail Azure, sous réseau virtuel, modifiez le serveur DNS pour inclure l’adresse IP du contrôleur de domaine secondaire. Ce paramètre permet la redondance du service DNS.

### <a name="configure-the-domain-accounts"></a><a name="DomainAccounts"></a> Configurer les comptes de domaine

Lors des étapes suivantes, vous configurez les comptes Active Directory. La table qui suit affiche les comptes :

| |Compte d’installation<br/> |sqlserver-0 <br/>SQL Server et compte SQL Agent Service |sqlserver-1<br/>SQL Server et compte SQL Agent Service
| --- | --- | --- | ---
|**Prénom** |Installer |SQLSvc1 | SQLSvc2
|**SamAccountName de l'utilisateur** |Installer |SQLSvc1 | SQLSvc2

Procédez comme suit pour créer chaque compte.

1. Connectez-vous à la machine **ad-primary-dc**.
2. Dans **Gestionnaire de serveur**, sélectionnez **Outils**, puis sélectionnez **Centre d’administration Active Directory**.   
3. Sélectionnez **corp (local)** dans le volet gauche.
4. Dans le volet droit **Tâches**, sélectionnez **Nouveau**, puis sélectionnez **Utilisateur**.

   ![Centre d'administration Active Directory](./media/availability-group-manually-configure-prerequisites-tutorial-/29-addcnewuser.png)

   >[!TIP]
   >Définissez un mot de passe complexe pour chaque compte.<br/> Pour les environnements hors production, définissez le compte d’utilisateur afin qu'il n’expire jamais.
   >

5. Sélectionnez **OK** pour créer l’utilisateur.
6. Répétez les étapes précédentes pour chacun des trois comptes.

### <a name="grant-the-required-permissions-to-the-installation-account"></a>Accorder les autorisations requises pour le compte d’installation

1. Dans le **Centre d’administration Active Directory**, sélectionnez **corp (local)** dans le volet gauche. Ensuite, dans le volet droit **Tâches**, sélectionnez **Propriétés**.

    ![Propriétés de l’utilisateur CORP](./media/availability-group-manually-configure-prerequisites-tutorial-/31-addcproperties.png)

2. Sélectionnez **Extensions**, puis sélectionnez le bouton **Avancé** de l’onglet **Sécurité**.
3. Dans la boîte de dialogue **Paramètres de sécurité avancés pour corp**, sélectionnez **Ajouter**.
4. Cliquez sur **Sélectionnez un principal**, recherchez **CORP\Install**, puis sélectionnez **OK**.
5. Cochez la case **Lire toutes les propriétés**.

6. Cochez la case **Créer des objets ordinateur**.

     ![Autorisations de l’utilisateur Corp](./media/availability-group-manually-configure-prerequisites-tutorial-/33-addpermissions.png)

7. Sélectionnez **OK**, puis de nouveau **OK**. Fermez la fenêtre des propriétés **corp**.

Maintenant que vous avez fini de configurer Active Directory et les objets utilisateur, créez deux machines virtuelles SQL Server et une machine virtuelle de serveur témoin. Joignez les trois machines virtuelles au domaine.

## <a name="create-sql-server-vms"></a>Créer des machines virtuelles SQL Server

Créez trois machines virtuelles supplémentaires. Cette solution nécessite deux machines virtuelles avec des instances de SQL Server. Une troisième machine virtuelle fonctionnera comme témoin. Windows Server 2016 peut utiliser un [témoin cloud](/windows-server/failover-clustering/deploy-cloud-witness). Toutefois, pour la cohérence avec les systèmes d’exploitation précédents, cet article utilise une machine virtuelle pour témoin.  

Avant de continuer, envisagez les décisions de conception suivantes.

* **Stockage - Azure Disques managés**

   Pour le stockage de la machine virtuelle, utilisez Azure Disques managés. Microsoft recommande des machines virtuelles de la fonctionnalité Disques managés pour SQL Server. La fonctionnalité Disques managés gère le stockage en arrière-plan. En outre, lorsque les machines virtuelles avec la fonctionnalité Disques managés sont dans le même groupe à haute disponibilité, Azure distribue les ressources de stockage pour fournir une redondance appropriée. Pour plus d’informations, voir la page [Azure Managed Disks overview](../../../virtual-machines/managed-disks-overview.md) (Vue d’ensemble d’Azure Disques managés). Pour plus de détails sur la fonctionnalité Disques managés dans un groupe à haute disponibilité, consultez [Utilisation de la fonctionnalité Disques managés pour les machines virtuelles dans le groupe à haute disponibilité](../../../virtual-machines/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

* **Réseau - Adresses IP privées en production**

   Pour les machines virtuelles, ce didacticiel utilise des adresses IP publiques. Une adresse IP publique assure une connexion à distance directe à la machine virtuelle via Interne et simplifie les étapes de configuration. Dans les environnements de production, Microsoft recommande d’utiliser uniquement des adresses IP privées afin de réduire l’empreinte vulnérable de la ressource de machine virtuelle de l’instance SQL Server.

* **Réseau – Recommander une seule carte réseau par serveur** 

Utilisez une seule carte réseau (NIC) par serveur (nœud de cluster) et un seul sous-réseau. Les réseaux Azure intègrent une redondance physique, ce qui rend inutiles les cartes réseau et les sous-réseaux supplémentaires sur un cluster invité de machine virtuelle Azure. Le rapport de validation du cluster vous avertit que les nœuds sont accessibles uniquement sur un seul réseau. Vous pouvez ignorer cet avertissement sur les clusters de basculement invités de machines virtuelles Azure.

### <a name="create-and-configure-the-sql-server-vms"></a>Créer et configurer les machines virtuelles SQL Server

Créez ensuite trois machines virtuelles : deux machines virtuelles SQL Server et une machine virtuelle pour un nœud de cluster supplémentaire. Pour créer chacune des machines virtuelles, revenez au groupe de ressources **SQL-HA-RG**, puis sélectionnez **Ajouter**. Recherchez l’élément de galerie approprié, sélectionnez **Machine virtuelle**, puis sélectionnez **À partir de la galerie**. Utilisez les informations du tableau suivant pour vous aider à créer les machines virtuelles :


| Page | MV1 | MV2 | MV3 |
| --- | --- | --- | --- |
| Sélectionnez l’élément de la galerie approprié. |**Windows Server 2016 Datacenter** |**SQL Server 2016 SP1 Enterprise sur Windows Server 2016** |**SQL Server 2016 SP1 Enterprise sur Windows Server 2016** |
| **Notions** |**Nom** = cluster-fsw<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre localisation Azure |**Nom** = sqlserver-0<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre localisation Azure |**Nom** = sqlserver-1<br/>**Nom d’utilisateur** = DomainAdmin<br/>**Mot de passe** = Contoso!0000<br/>**Abonnement** = votre abonnement<br/>**Groupe de ressources** = SQL-HA-RG<br/>**Emplacement** = Votre localisation Azure |
| Configuration de la machine virtuelle - **Taille** |**TAILLE** = DS1\_V2 (1 processeur virtuel, 3,5 Go) |**TAILLE** = DS2\_V2 (2 processeurs virtuels, 7 Go)</br>La taille doit prendre en charge un stockage SSD (prise en charge des disques Premium. )) |**TAILLE** = DS2\_V2 (2 processeurs virtuels, 7 Go) |
| Configuration de la machine virtuelle - **Paramètres** |**Stockage** : Utiliser des disques managés.<br/>**Réseau virtuel** = autoHAVNET<br/>**Sous-réseau** = sqlsubnet(10.1.1.0/24)<br/>**Adresse IP publique** générée automatiquement.<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**Groupe à haute disponibilité** = sqlAvailabilitySet<br/> |**Stockage** : Utiliser des disques managés.<br/>**Réseau virtuel** = autoHAVNET<br/>**Sous-réseau** = sqlsubnet(10.1.1.0/24)<br/>**Adresse IP publique** générée automatiquement.<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**Groupe à haute disponibilité** = sqlAvailabilitySet<br/> |**Stockage** : Utiliser des disques managés.<br/>**Réseau virtuel** = autoHAVNET<br/>**Sous-réseau** = sqlsubnet(10.1.1.0/24)<br/>**Adresse IP publique** générée automatiquement.<br/>**Groupe de sécurité réseau** = aucun<br/>**Diagnostics de surveillance** = activés<br/>**Compte de stockage de diagnostics** = utilisez un compte de stockage généré automatiquement<br/>**Groupe à haute disponibilité** = sqlAvailabilitySet<br/> |
| Configuration de la machine virtuelle - **Paramètres SQL Server** |Non applicable |**Connectivité SQL** = privée (dans le réseau virtuel)<br/>**Port** = 1433<br/>**Authentification SQL** = désactivée<br/>**Configuration du stockage** = général<br/>**Mise à jour corrective automatisée** : dimanche à 2h00<br/>**Sauvegarde automatisée** = désactivée</br>**Intégration Azure Key Vault** = Désactivée |**Connectivité SQL** = privée (dans le réseau virtuel)<br/>**Port** = 1433<br/>**Authentification SQL** = désactivée<br/>**Configuration du stockage** = général<br/>**Mise à jour corrective automatisée** : dimanche à 2h00<br/>**Sauvegarde automatisée** = désactivée</br>**Intégration Azure Key Vault** = Désactivée |

<br/>

> [!NOTE]
> Les tailles de machines suggérées ici sont destinées au test des groupes de disponibilité dans les machines virtuelles Azure. Pour optimiser les performances des charges de travail de production, consultez les recommandations relatives à la taille des ordinateurs SQL Server et à la configuration dans [Meilleures pratiques relatives aux performances de SQL Server dans Azure Virtual Machines](performance-guidelines-best-practices.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
>

Une fois les trois machines virtuelles entièrement configurées, vous devez les attacher au domaine **corp.contoso.com** et accorder à CORP\Install des droits d’administration sur les machines.

### <a name="join-the-servers-to-the-domain"></a><a name="joinDomain"></a>Joindre les serveurs au domaine

Vous pouvez maintenant joindre les machines virtuelles à **corp.contoso.com**. Procédez comme suit pour les machines virtuelles SQL Server et pour le serveur témoin de partage :

1. Connectez-vous à distance à la machine virtuelle avec **BUILTIN\DomainAdmin**.
2. Dans le **Gestionnaire de serveur**, sélectionnez **Serveur Local**.
3. Sélectionnez le lien **GROUPE DE TRAVAIL**.
4. Dans la section **Nom de l’ordinateur**, sélectionnez **Modifier**.
5. Cochez la case **Domaine** et saisissez **corp.contoso.com** dans la zone de texte. Sélectionnez **OK**.
6. Dans la boîte de dialogue contextuelle **Sécurité Windows**, spécifiez les informations d’identification du compte d’administrateur de domaine par défaut (**CORP\DomainAdmin**) et le mot de passe (**Contoso!0000**).
7. Quand le message « Bienvenue dans le domaine corp.contoso.com » s’affiche, sélectionnez **OK**.
8. Sélectionnez **Fermer**, puis sélectionnez **Redémarrer maintenant** dans la boîte de dialogue contextuelle.

## <a name="add-accounts"></a>Ajouter des comptes

Ajoutez le compte d’installation en tant qu’administrateur sur chaque machine virtuelle, accordez l’autorisation au compte d’installation et aux comptes locaux dans SQL Server et mettez à jour le compte de service SQL Server. 

### <a name="add-the-corpinstall-user-as-an-administrator-on-each-cluster-vm"></a>Ajoutez l’utilisateur Corp\Install en tant qu’administrateur sur chaque machine virtuelle du cluster

Après le redémarrage de chaque machine virtuelle en tant que membre du domaine, ajoutez **CORP\Install** comme membre du groupe Administrateurs local.

1. Patientez jusqu’à ce que la machine virtuelle ait redémarré, puis lancez de nouveau le fichier RDP à partir du contrôleur de domaine principal pour vous connecter à **sqlserver-0** à l’aide du compte **CORP\DomainAdmin**.

   >[!TIP]
   >Assurez-vous d’ouvrir une session avec le compte Administrateur de domaine. Dans les étapes précédentes, vous utilisiez le compte Administrateur BUILT IN. Maintenant que le serveur figure dans le domaine, utilisez le compte de domaine. Dans votre session RDP, spécifiez *DOMAINE*\\*nom d’utilisateur*.
   >

2. Dans **Gestionnaire de serveur**, sélectionnez **Outils**, puis sélectionnez **Gestion de l’ordinateur**.
3. Dans la fenêtre **Gestion de l’ordinateur**, développez **Utilisateurs et groupes locaux**, puis sélectionnez **Groupes**.
4. Double-cliquez sur le groupe **Administrateurs** .
5. Dans la boîte de dialogue **Propriétés de Administrateurs**, sélectionnez **Ajouter**.
6. Entrez l’utilisateur **CORP\Install**, puis sélectionnez **OK**.
7. Sélectionnez **OK** pour fermer la boîte de dialogue **Propriétés de Administrateurs**.
8. Répétez les étapes précédentes avec **sqlserver-1** et **cluster-fsw**.


### <a name="create-a-sign-in-on-each-sql-server-vm-for-the-installation-account"></a>Créer une connexion sur chaque machine virtuelle SQL Server pour le compte d’installation

Utilisez le compte d’installation (CORP\install) pour configurer le groupe de disponibilité. Ce compte doit être membre du rôle serveur fixe **sysadmin** pour chaque machine virtuelle SQL Server. Les étapes suivantes créent une connexion pour le compte d’installation :

1. Connectez-vous au serveur par le biais du protocole RDP (Remote Desktop Protocol) à l'aide du compte *\<MachineName\>\DomainAdmin*.

1. Ouvrez SQL Server Management Studio et connectez-vous à l'instance locale de SQL Server.

1. Dans l’**Explorateur d’objets**, sélectionnez **Sécurité**.

1. Cliquez avec le bouton droit sur **Connexions**. Sélectionnez **Nouvelle connexion**.

1. Dans **Nouvelle connexion**, sélectionnez **Rechercher**.

1. Sélectionner **Emplacements**.

1. Entrez les informations d’identification du réseau de l’administrateur de domaine.

1. Utilisez le compte d’installation (CORP\install).

1. Définissez la connexion à un membre du rôle serveur fixe **sysadmin**.

1. Sélectionnez **OK**.

Répétez les étapes précédentes sur l’autre machine virtuelle SQL Server.

### <a name="configure-system-account-permissions"></a>Configurer les autorisations du compte système

Pour créer un compte pour le compte système et accorder les autorisations appropriées, procédez comme suit sur chaque instance de SQL Server :

1. Créer un compte pour `[NT AUTHORITY\SYSTEM]` sur chaque instance de SQL Server. Le script suivant crée ce compte :

   ```sql
   USE [master]
   GO
   CREATE LOGIN [NT AUTHORITY\SYSTEM] FROM WINDOWS WITH DEFAULT_DATABASE=[master]
   GO 
   ```

1. Accordez les autorisations suivantes à `[NT AUTHORITY\SYSTEM]` sur chaque instance de SQL Server :

   - `ALTER ANY AVAILABILITY GROUP`
   - `CONNECT SQL`
   - `VIEW SERVER STATE`

   Le script suivant accorde ces autorisations :

   ```sql
   GRANT ALTER ANY AVAILABILITY GROUP TO [NT AUTHORITY\SYSTEM]
   GO
   GRANT CONNECT SQL TO [NT AUTHORITY\SYSTEM]
   GO
   GRANT VIEW SERVER STATE TO [NT AUTHORITY\SYSTEM]
   GO 
   ```

### <a name="set-the-sql-server-service-accounts"></a><a name="setServiceAccount"></a>Définir les comptes de service SQL Server

Sur chaque machine virtuelle SQL Server, définissez le compte de service SQL Server. Utilisez les comptes que vous avez créés lors de la configuration des comptes de domaine.

1. Ouvrez le **Gestionnaire de configuration SQL Server**.
2. Cliquez avec le bouton droit sur le service SQL Server, puis sélectionnez **Propriétés**.
3. Définissez le compte et le mot de passe.
4. Répétez ces étapes sur l’autre machine virtuelle SQL Server.  

Pour les groupes de disponibilité SQL Server, chaque machine virtuelle SQL Server doit s’exécuter en tant que compte de domaine.

## <a name="add-failover-clustering-features-to-both-sql-server-vms"></a>Ajouter les fonctionnalités de clustering de basculement aux deux machines virtuelles SQL Server

Pour ajouter les fonctionnalités de clustering de basculement, procédez comme suit sur les deux machines virtuelles SQL Server :

1. Connectez-vous à la machine virtuelle SQL Server par le biais du protocole RDP (Remote Desktop Protocol) à l’aide du compte *CORP\install*. Ouvrez le **Tableau de bord de gestionnaire de serveur**.
2. Sélectionnez le lien du tableau de bord **Ajouter des rôles et fonctionnalités**.

    ![Gestionnaire de serveur - Ajouter des rôles](./media/availability-group-manually-configure-prerequisites-tutorial-/22-addfeatures.png)

3. Sélectionnez **Suivant** jusqu’à ce que vous atteigniez la section **Fonctionnalités de serveur**.
4. Dans **Fonctionnalités**, sélectionnez **Clustering de basculement**.
5. Ajoutez les fonctionnalités supplémentaires requises.
6. Sélectionnez **Installer** pour ajouter les fonctionnalités.

Répétez les étapes sur l’autre machine virtuelle SQL Server.

  >[!NOTE]
  > Cette étape, ainsi que la jonction des machines virtuelles au cluster de basculement, peut désormais être automatisée à l'aide d'une [interface en ligne de commande de machine virtuelle Azure SQL](./availability-group-az-commandline-configure.md) et de [modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md).
  >

### <a name="tuning-failover-cluster-network-thresholds"></a>Réglage des seuils réseau de cluster de basculement

Quand vous exécutez des nœuds de cluster de basculement Windows dans des machines virtuelles Azure avec SQL Server AlwaysOn, il est recommandé de remplacer le paramètre de cluster par un état d’analyse plus souple.  Cela rend le cluster bien plus stable et fiable.  Pour plus de détails, consultez [IaaS avec SQL AlwaysOn – Réglage des seuils réseau de cluster de basculement](/windows-server/troubleshoot/iaas-sql-failover-cluster).


## <a name="configure-the-firewall-on-each-sql-server-vm"></a><a name="endpoint-firewall"></a> Configurer le pare-feu sur chaque machine virtuelle SQL Server

La solution nécessite l'ouverture des ports TCP suivants sur le pare-feu :

- **Machine virtuelle SQL Server** : le port 1433 pour une instance SQL Server par défaut.
- **Sonde Azure Load Balancer :** N’importe quel port disponible. Les exemples utilisent souvent le port 59999.
- **Point de terminaison de mise en miroir de bases de données :** N’importe quel port disponible. Les exemples utilisent souvent le port 5022.

Les ports du pare-feu doivent être ouverts sur les deux machines virtuelles SQL Server.

La méthode d’ouverture des ports dépend de la solution de pare-feu que vous utilisez. La section suivante explique comment ouvrir les ports sur le pare-feu Windows. Ouvrez les ports requis pour chacune de vos machines virtuelles SQL Server.

### <a name="open-a-tcp-port-in-the-firewall"></a>Ouvrir un Port TCP sur le pare-feu

1. Sur l'écran de **démarrage** du premier serveur SQL, lancez le **Pare-feu Windows avec fonctions de sécurité avancées**.
2. Dans le volet gauche, sélectionnez **Règles de trafic entrant**. Dans le volet droit, sélectionnez **Nouvelle règle**.
3. Dans **Type de règle**, choisissez **Port**.
4. Pour le port, spécifiez **TCP** et entrez les numéros de port correspondants. Voir l’exemple suivant :

   ![Pare-feu SQL](./media/availability-group-manually-configure-prerequisites-tutorial-/35-tcpports.png)

5. Sélectionnez **Suivant**.
6. Dans la page **Action**, sélectionnez **Autoriser la connexion**, puis sélectionnez **Suivant**.
7. Dans la page **Profil**, acceptez les paramètres par défaut puis sélectionnez **Suivant**.
8. Dans la page **Nom**, spécifiez un nom pour la règle (par exemple **Sondage Azure LB**) au sein de la zone de texte **Nom**, puis sélectionnez **Terminer**.

Répétez ces étapes sur la seconde machine virtuelle SQL Server.


## <a name="next-steps"></a>Étapes suivantes

* [Créer un groupe de disponibilité AlwaysOn sur des machines virtuelles Azure](availability-group-manually-configure-tutorial.md)
