---
title: 'Tutoriel : Se connecter à une application web avec Azure Private Endpoint'
titleSuffix: Azure Private Link
description: Suivez ce tutoriel pour vous connecter à une application web à l’aide d’Azure Private Endpoint.
author: asudbring
ms.author: allensu
ms.service: private-link
ms.topic: tutorial
ms.date: 10/19/2020
ms.openlocfilehash: 502c48a92f5b41c4434d03139335a0ce05fa451f
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896971"
---
# <a name="tutorial-connect-to-a-web-app-using-an-azure-private-endpoint"></a>Tutoriel : Se connecter à une application web avec Azure Private Endpoint

Azure Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de façon privée avec des ressources Private Link.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un hôte Azure bastion.
> * Création d’une machine virtuelle
> * Créer une application web
> * Créer un point de terminaison privé
> * Tester la connectivité au point de terminaison privé de l’application web

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

> [!Note]
> Private Endpoint est disponible dans les régions publiques pour les applications web Windows de niveau Premium V2, Premium V3, les applications Linux et le plan Azure Functions Premium (parfois appelé plan Élastique Premium). 

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-virtual-network-and-bastion-host"></a>Créer un réseau virtuel et un hôte bastion

Dans cette section, vous allez créer un réseau virtuel, un sous-réseau et un hôte bastion. 

L’hôte bastion sera utilisé pour se connecter de façon sécurisée à la machine virtuelle afin de tester le point de terminaison privé.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel** , ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel** , entrez ou sélectionnez les informations suivantes sous l’onglet **Général**  :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionner **myResourceGroup** |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **Europe Ouest** |

3. Sélectionnez l’onglet **Adresses IP** , ou sélectionnez le bouton **Suivant : Adresses IP** au bas de la page.

4. Sous l’onglet **Adresses IP** , entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Espace d’adressage IPv4 | Entrez **10.1.0.0/16** |

5. Sous **Nom de sous-réseau** , sélectionnez le mot **par défaut**.

6. Dans **Modifier le sous-réseau** , entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du sous-réseau | Entrez **mySubnet** |
    | Plage d’adresses de sous-réseau | Entrez **10.1.0.0/24** |

7. Sélectionnez **Enregistrer**.

8. Sélectionnez l'onglet **Sécurité** .

9. Sous **BastionHost** , sélectionnez **Activer**. Entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du bastion | Entrez **myBastionHost** |
    | Espace d’adressage AzureBastionSubnet | Entrez **10.1.1.0/24** |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. </br> Pour **Nom** , entrez **myBastionIP**. </br> Sélectionnez **OK**. |


8. Sélectionnez l’onglet **Vérifier + créer** , ou sélectionnez le bouton **Vérifier + créer**.

9. Sélectionnez **Create** (Créer).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Dans cette section, vous allez créer une machine virtuelle qui sera utilisée pour tester le point de terminaison privé.


1. En haut à gauche de l’écran du portail Azure, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle** , ou recherchez **Machine virtuelle** dans la zone de recherche.
   
2. Dans **Créer une machine virtuelle** , tapez ou sélectionnez les valeurs sous l’onglet **De base**  :

    | Paramètre | Value                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **Europe Ouest** |
    | Options de disponibilité | Sélectionnez **Aucune redondance d’infrastructure requise**. |
    | Image | Sélectionnez **Windows Server 2019 Datacenter - Gen1** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques** , puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Value |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | **myVNet** |
    | Subnet | **mySubnet** |
    | Adresse IP publique | Sélectionnez **Aucun**. |
    | Groupe de sécurité réseau de la carte réseau | **De base**|
    | Aucun port d’entrée public | Sélectionnez **Aucun**. |
   
5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="create-web-app"></a>Créer une application web

Dans cette section, vous allez créer une application web.

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Stockage** > **Application web** , ou recherchez **Application web** dans la zone de recherche.

2. Sous l’onglet **Général** de la page **Application web** , entrez ou sélectionnez les informations suivantes.

    | Paramètre | Value                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
    | **Détails de l’instance** |  |
    | Nom | Entrez **mywebapp**. Si le nom n’est pas disponible, entrez un nom unique. |
    | Publish | Sélectionnez **Code**. |
    | Pile d’exécution | Sélectionnez **.NET Core 3.1 (LTS)** . |
    | Système d'exploitation | Sélectionnez **Windows**. |
    | Région | Sélectionnez **Europe Ouest** |
    | **Plan App Service** |  |
    | Plan Windows (Europe Ouest) | Sélectionnez **Créer nouveau**. </br> Entrez **myServicePlan** dans le champ **Nom**. |
    | SKU et taille | Sélectionnez **Changer la taille**. </br> Sélectionnez **P2V2** dans l’écran **Sélecteur de spécifications**. </br> Sélectionnez **Appliquer**. |
   
3. Sélectionnez **Revoir + créer**.

4. Sélectionnez **Create** (Créer).

    :::image type="content" source="./media/tutorial-private-endpoint-webapp-portal/create-web-app.png" alt-text="Onglet De base dans la page Créer une application web du portail Azure." border="true":::

## <a name="create-private-endpoint"></a>Créer un point de terminaison privé

1. Dans le menu de gauche, sélectionnez **Toutes les ressources** > **mywebapp** , ou le nom que vous avez choisi lors de la création.

2. Dans la vue d’ensemble de l’application web, sélectionnez **Paramètres** > **Réseau**.

3. Dans **Réseau** , sélectionnez **Configurer vos connexions de point de terminaison privé**.

4. Dans l’écran **Connexions de points de terminaison privé** , sélectionnez **+ Ajouter**.

5. Dans l’écran **Ajouter un point de terminaison privé** , entrez ou sélectionnez les informations suivantes :

    | Paramètre | Value |
    | ------- | ----- |
    | Name | Entrez **mywebappendpoint**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Réseau virtuel | Sélectionnez **myVNet**. |
    | Subnet | Sélectionnez **mySubnet**. |
    | Intégrer à une zone DNS privée | Sélectionnez **Oui**. |

6. Sélectionnez **OK**.
    

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter à l’application web via le point de terminaison privé.

1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **myVM**.

4. Dans la page de vue d’ensemble pour **myVM** , sélectionnez **Se connecter** , puis **Bastion**.

5. Sélectionnez le bouton bleu **Utiliser le bastion**.

6. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

7. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

8. Entrez `nslookup <webapp-name>.azurewebsites.net`. Remplacez **\<webapp-name>** par le nom de l’application web que vous avez créée dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mywebapp8675.privatelink.azurewebsites.net
    Address:  10.1.0.5
    Aliases:  mywebapp8675.azurewebsites.net
    ```

    Une adresse IP privée **10.1.0.5** est retournée pour le nom de l’application web.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.

9. Ouvrez un navigateur web sur votre ordinateur local, puis entrez l’URL externe de votre application web : **https://\<webapp-name>.azurewebsites.net**.

10. Vérifiez qu’une page **403** s’affiche. Cette page indique que l’application web n’est pas accessible en externe.

    :::image type="content" source="./media/tutorial-private-endpoint-webapp-portal/web-app-ext-403.png" alt-text="Page 403 concernant l’adresse externe de l’application web." border="true":::

11. Dans la connexion bastion à **myVM** , ouvrez Internet Explorer.

12. Entrez l’URL de votre application web : **https://\<webapp-name>.azurewebsites.net**.

13. Vérifiez que la page de l’application web par défaut s’affiche.

    :::image type="content" source="./media/tutorial-private-endpoint-webapp-portal/web-app-default-page.png" alt-text="Page de l’application web par défaut." border="true":::

18. Fermez la connexion à **myVM**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le réseau virtuel, la machine virtuelle et l’application web en effectuant les étapes suivantes :

1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **Supprimer le groupe de ressources**.

4. Entrez **myResourceGroup** dans **TAPEZ LE NOM DU GROUPE DE RESSOURCES**.

5. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer un service Private Link :
> [!div class="nextstepaction"]
> [Créer un service Private Link](create-private-link-service-portal.md)
