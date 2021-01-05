---
title: 'Tutoriel : Router du trafic vers des points de terminaison pondérés - Azure Traffic Manager'
description: Ce didacticiel explique comment acheminer le trafic vers des points de terminaison pondérés à l’aide de Traffic Manager.
services: traffic-manager
author: duongau
Customer intent: As an IT Admin, I want to distribute traffic based on the weight assigned to a website endpoint so that I can control the user traffic to a given website.
ms.service: traffic-manager
ms.topic: tutorial
ms.date: 10/19/2020
ms.author: duau
ms.openlocfilehash: 55c316a370b9e44e906e48b4716201384567c9c2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003784"
---
# <a name="tutorial-control-traffic-routing-with-weighted-endpoints-by-using-traffic-manager"></a>Tutoriel : Contrôler le routage du trafic avec des points de terminaison pondérés à l’aide de Traffic Manager

Ce didacticiel explique comment utiliser Azure Traffic Manager pour contrôler l’acheminement du trafic utilisateur entre des points de terminaison à l’aide de la méthode de routage pondéré. Dans cette méthode de routage, vous affectez un poids à chaque point de terminaison dans la configuration du profil Traffic Manager. Le trafic utilisateur est alors acheminé en fonction du poids attribué à chaque point de terminaison. Le poids est un entier compris entre 1 et 1 000. Plus la valeur de pondération affectée à un point de terminaison est élevée, plus celui-ci est prioritaire.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créez deux machines virtuelles exécutant un site web de base sur IIS.
> * Créez deux machines virtuelles de test pour afficher Traffic Manager en action.
> * Configurez un nom DNS pour les machines virtuelles exécutant IIS.
> * Créer un profil Traffic Manager.
> * Ajoutez des points de terminaison de machine virtuelle au profil Traffic Manager.
> * Visualisez Traffic Manager en action.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour voir comment Traffic Manager fonctionne, déployez les éléments suivants pour ce didacticiel :

- Deux instances de sites web de base exécutées dans différentes régions Azure : USA Est et Europe Ouest.
- Deux machines virtuelles de test pour tester Traffic Manager : une machine virtuelle dans la région USA Est et la seconde dans la région Europe Ouest. Les machines virtuelles de test servent à illustrer la manière dont Traffic Manager achemine le trafic utilisateur vers un site web dont le point de terminaison s’est vu affecter la valeur de pondération la plus élevée.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

### <a name="create-websites"></a>Créer des sites web

Dans cette section, vous allez créer deux instances de site web qui fournissent les deux points de terminaison de service pour le profil Traffic Manager dans deux régions Azure. Pour créer les deux sites web, effectuez les étapes suivantes :

1. Créez deux machines virtuelles afin d’exécuter un site web de base : une est située dans la région USA Est et l’autre dans la région Europe Ouest.
2. Installez un serveur IIS sur chaque machine virtuelle. Mettez à jour la page web par défaut qui décrit le nom de la machine virtuelle à laquelle un utilisateur est connecté lorsqu’il visite le site web.

#### <a name="create-vms-for-running-websites"></a>Créer des machines virtuelles pour exécuter des sites web

Dans cette section, vous allez créer deux machines virtuelles (*myIISVMEastUS* et *myIISVMWestEurope*) dans les régions Azure USA Est et Europe Ouest.

1. En haut à gauche du portail Azure, sélectionnez **Créer une ressource** > **Calculer** > **Windows Server 2019 Datacenter**.
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base** :

   - **Abonnement** > **Groupe de ressources** : Sélectionnez **Créer un nouveau**, puis tapez **myResourceGroupTM1**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : Tapez *myIISVMEastUS*.
   - **Détails de l’instance** > **Région** :  Sélectionnez **USA Est**.
   - **Compte d’administrateur** > **Nom d’utilisateur** :  Entrez un nom d’utilisateur de votre choix.
   - **Compte d’administrateur** > **Mot de passe** :  Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).
   - **Règles des ports d’entrée** > **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Règles des ports d’entrée** > **Sélectionner des ports d’entrée** : Sélectionnez **RDP** et **HTTP** dans la zone déroulante.

3. Sélectionnez l’onglet **Gestion** ou sélectionnez **Suivant : Disques**, puis **Suivant : Mise en réseau**, puis **Suivant : Gestion**. Sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.
4. Sélectionnez **Revoir + créer**.
5. Vérifiez les paramètres, puis cliquez sur **Créer**.  
6. Suivez les étapes pour créer une seconde machine virtuelle nommée *myIISVMWestEurope*, avec un **groupe de ressources** nommé *myResourceGroupTM2*, une **localisation** dans la région *Europe Ouest* et tous les autres paramètres identiques à ceux de *myIISVMEastUS*.
7. La création des machines virtuelles peut prendre plusieurs minutes. Attendez que les deux machines virtuelles aient été créées avant de passer aux étapes restantes.

![Créer une machine virtuelle](./media/tutorial-traffic-manager-improve-website-response/createVM.png)

#### <a name="install-iis-and-customize-the-default-webpage"></a>Installer IIS et personnaliser la page web par défaut

Dans cette section, vous allez installer le serveur IIS sur les deux machines virtuelles, myIISVMEastUS et myIISVMWestEurope, puis mettre à jour la page web par défaut. La page web personnalisée affiche le nom de la machine virtuelle à laquelle vous êtes connecté lorsque vous visitez le site web à partir d’un navigateur web.

1. Sélectionnez **Toutes les ressources** dans le menu de gauche. Dans la liste des ressources, sélectionnez **myIISVMEastUS** dans le groupe de ressources **myResourceGroupTM1**.
2. Sur la page **Vue d’ensemble**, sélectionnez **Se connecter**. Dans **Se connecter à la machine virtuelle**, sélectionnez **Télécharger le fichier RDP**.
3. Ouvrez le fichier .rdp téléchargé. Si vous y êtes invité, sélectionnez **Se connecter**. Entrez le nom d’utilisateur et le mot de passe que vous avez indiqué lors de la création de la machine virtuelle. Vous devrez peut-être sélectionner **Plus de choix** > **Utiliser un autre compte**, pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle.
4. Sélectionnez **OK**.
5. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez l’avertissement, sélectionnez **Oui** ou **Continuer** pour poursuivre le processus de connexion.
6. Sur le bureau du serveur, accédez à **Outils d’administration Windows** > **Gestionnaire de serveur**.
7. Ouvrez Windows PowerShell sur VM1. Utilisez les commandes suivantes pour installer le serveur IIS et mettre à jour le fichier .htm par défaut.

    ```powershell-interactive
    # Install IIS
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default .htm file
    remove-item C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom .htm file
    Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    ```

    ![Installer IIS et personnaliser la page web](./media/tutorial-traffic-manager-improve-website-response/deployiis.png)

8. Fermez la connexion RDP établie avec la machine virtuelle **myIISVMEastUS**.
9. Répétez les étapes 1 à 8. Créez une connexion RDP avec la machine virtuelle **myIISVMWestEurope** au sein du groupe de ressources **myResourceGroupTM2** pour installer IIS et personnaliser sa page web par défaut.

#### <a name="configure-dns-names-for-the-vms-running-iis"></a>Configurer les noms DNS des machines virtuelles exécutant IIS

Traffic Manager achemine le trafic utilisateur en fonction du nom DNS des points de terminaison de service. Dans cette section, vous configurez les noms DNS des serveurs IIS myIISVMEastUS et myIISVMWestEurope.

1. Sélectionnez **Toutes les ressources** dans le menu de gauche. Dans la liste des ressources, sélectionnez **myIISVMEastUS** dans le groupe de ressources **myResourceGroupTM1**.
2. Dans la page **Vue d’ensemble**, sous **Nom DNS**, sélectionnez **Configurer**.
3. Dans la page **Configuration**, sous l’étiquette du nom DNS, ajoutez un nom unique. Ensuite, sélectionnez **Enregistrer**.
4. Répétez les étapes 1 à 3 pour la machine virtuelle nommée **myIISVMWestEurope** dans le groupe de ressources **myResourceGroupTM2**.

### <a name="create-a-test-vm"></a>Créer une machine virtuelle de test

Dans cette section, vous allez créer une machine virtuelle (*myVMEastUS* et *myVMWestEurope*) dans chaque région Azure (**USA Est** et **Europe Ouest**). Vous allez utiliser ces machines virtuelles pour tester la manière dont Traffic Manager route le trafic vers le point de terminaison du site web auquel est affectée la valeur de pondération la plus élevée.

1. En haut à gauche du portail Azure, sélectionnez **Créer une ressource** > **Calculer** > **Windows Server 2019 Datacenter**.
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base** :

   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupTM1**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : Tapez *myVMEastUS*.
   - **Détails de l’instance** > **Région** :  Sélectionnez **USA Est**.
   - **Compte d’administrateur** > **Nom d’utilisateur** :  Entrez un nom d’utilisateur de votre choix.
   - **Compte d’administrateur** > **Mot de passe** :  Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).
   - **Règles des ports d’entrée** > **Ports d’entrée publics** : Sélectionnez **Autoriser les ports sélectionnés**.
   - **Règles des ports d’entrée** > **Sélectionner des ports d’entrée** : Sélectionnez **RDP** dans la zone déroulante.

3. Sélectionnez l’onglet **Gestion** ou sélectionnez **Suivant : Disques**, puis **Suivant : Mise en réseau**, puis **Suivant : Gestion**. Sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.
4. Sélectionnez **Revoir + créer**.
5. Vérifiez les paramètres, puis cliquez sur **Créer**.  
6. Suivez les étapes pour créer une seconde machine virtuelle nommée *myVMWestEurope*, avec un **groupe de ressources** nommé *myResourceGroupTM2*, une **localisation** dans la région *Europe Ouest* et tous les autres paramètres identiques à ceux de *myVMEastUS*.
7. La création des machines virtuelles peut prendre plusieurs minutes. Attendez que les deux machines virtuelles aient été créées avant de passer aux étapes restantes.

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

Créez un profil Traffic Manager basé sur la méthode de routage **Pondéré**.

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Profil Traffic Manager** > **Créer**.
2. Dans **Créer un profil Traffic Manager**, entrez ou sélectionnez les informations de base suivantes. Acceptez les valeurs par défaut des autres paramètres, puis sélectionnez **Créer**.

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Nom                   | Entrez un nom unique dans la zone trafficmanager.net. Cela attribuera le nom DNS trafficmanager.net qui vous permettra d’accéder à votre profil Traffic Manager.                                   |
    | Méthode de routage          | Sélectionnez la méthode de routage **Pondéré**.                                       |
    | Abonnement            | Sélectionnez votre abonnement.                          |
    | Resource group          | Sélectionnez **Utiliser l’existant**, puis **myResourceGroupTM1**. |
    |        |   |

    ![Créer un profil Traffic Manager](./media/tutorial-traffic-manager-weighted-endpoint-routing/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Ajouter des points de terminaison Traffic Manager

Ajoutez les deux machines virtuelles qui exécutent les serveurs IIS myIISVMEastUS et myIISVMWestEurope afin d’y router le trafic utilisateur.

1. Dans la barre de recherche du portail, recherchez le nom du profil Traffic Manager que vous avez créé dans la section précédente. Sélectionnez le profil dans les résultats affichés.
2. Dans **Profil Traffic Manager**, à la section **Paramètres**, sélectionnez **Points de terminaison** > **Ajouter**.
3. Entrez ou sélectionnez les informations suivantes : Acceptez les valeurs par défaut des autres paramètres, puis sélectionnez **OK**.

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Type                    | Entrez le point de terminaison Azure.                                   |
    | Nom           | Entrez **myEastUSEndpoint**.                                        |
    | Type de ressource cible           | Sélectionnez **Adresse IP publique**.                          |
    | Ressource cible          | Choisissez une adresse IP publique pour afficher la liste des ressources pourvues d’adresses IP publiques dans le même abonnement. Dans **Ressource**, sélectionnez l’adresse IP publique nommée **myIISVMEastUS-ip**. Il s’agit de l’adresse IP publique de la machine virtuelle serveur IIS qui se trouve dans la région USA Est.|
    |  Poids      | Entrez **100**.        |
    |        |           |

4. Répétez les étapes 2 et 3 pour ajouter un autre point de terminaison nommé **myWestEuropeEndpoint** pour l’adresse IP publique **myIISVMWestEurope-ip**. Cette adresse est associée à la machine virtuelle du serveur IIS nommée myIISVMWestEurope. Pour **Poids**, entrez **25**.
5. Lorsque l’ajout des deux points de terminaison est terminé, ceux-ci s’affichent dans le profil Traffic Manager avec leur état de supervision comme étant **En ligne**.

## <a name="test-the-traffic-manager-profile"></a>Tester le profil Traffic Manager

Pour visualiser Traffic Manager en action, procédez comme suit :

1. Déterminez le nom DNS de votre profil Traffic Manager.
2. Visualisez Traffic Manager en action.

### <a name="determine-dns-name-of-traffic-manager-profile"></a>Déterminer le nom DNS du profil Traffic Manager

Dans ce didacticiel, par souci de simplicité, vous utilisez le nom DNS du profil Traffic Manager pour visiter les sites web.

Vous pouvez déterminer le nom DNS du profil Traffic Manager en procédant comme suit :

1. Dans la barre de recherche du portail, recherchez le nom du profil Traffic Manager que vous avez créé dans la section précédente. Dans les résultats affichés, sélectionnez le profil Traffic Manager.
2. Sélectionnez **Vue d’ensemble**.
3. Le profil Traffic Manager affiche son nom DNS. Dans les déploiements en environnements de production, vous configurez un nom de domaine personnalisé pour qu’il pointe vers le nom de domaine Traffic Manager à l’aide d’un enregistrement DNS CNAME.

   ![Nom DNS Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/traffic-manager-dns-name.png)

### <a name="view-traffic-manager-in-action"></a>Afficher Traffic Manager en action

Dans cette section, vous pouvez voir Traffic Manager en action.

1. Sélectionnez **Toutes les ressources** dans le menu de gauche. Dans la liste des ressources, sélectionnez **myVMEastUS** dans le groupe de ressources **myResourceGroupTM1**.
2. Sur la page **Vue d’ensemble**, sélectionnez **Se connecter**. Dans **Se connecter à la machine virtuelle**, sélectionnez **Télécharger le fichier RDP**.
3. Ouvrez le fichier .rdp téléchargé. Si vous y êtes invité, sélectionnez **Se connecter**. Entrez le nom d’utilisateur et le mot de passe que vous avez spécifiés lors de la création de la machine virtuelle. Vous devrez peut-être sélectionner **Plus de choix** > **Utiliser un autre compte**, pour spécifier les informations d’identification que vous avez entrées lors de la création de la machine virtuelle.
4. Sélectionnez **OK**.
5. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez l’avertissement, sélectionnez **Oui** ou **Continuer** pour poursuivre le processus de connexion.
6. Dans un navigateur web sur la machine virtuelle myVMEastUS, entrez le nom DNS de votre profil Traffic Manager pour afficher votre site web. Vous êtes redirigé vers le site web hébergé sur le serveur IIS myIISVMEastUS car un poids plus élevé de **100** lui a été assigné. Une valeur de pondération du point de terminaison inférieure s’élevant à **25** a été attribuée au serveur IIS myIISVMWestEurope.

   ![Tester le profil Traffic Manager](./media/tutorial-traffic-manager-improve-website-response/eastus-traffic-manager-test.png)

7. Répétez les étapes 1 à 6 sur la machine virtuelle myVMWestEurope pour voir la réponse du site web pondéré.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des groupes de ressources que vous avez créés dans ce didacticiel, vous pouvez les supprimer. Pour ce faire, sélectionnez le groupe de ressources (**ResourceGroupTM1** ou **ResourceGroupTM2**), puis **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les méthodes de routage, consultez :

> [!div class="nextstepaction"]
> [Méthode de routage du trafic](traffic-manager-routing-methods.md)
