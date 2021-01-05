---
title: Démarrage rapide – Créer un point de terminaison privé en utilisant le portail Azure
description: Découvrez dans ce guide de démarrage rapide comment créer un point de terminaison privé avec le portail Azure.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: allensu
ms.openlocfilehash: 395912ae70c5a01bd7de9a80cf8a507dd516028e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018067"
---
# <a name="quickstart-create-a-private-endpoint-using-the-azure-portal"></a>Démarrage rapide : Créer un point de terminaison privé en utilisant le portail Azure

Démarrez avec Azure Private Link en utilisant un point de terminaison privé pour vous connecter en toute sécurité à une application web Azure.

Dans ce guide de démarrage rapide, vous allez créer un point de terminaison privé pour une application web Azure et déployer une machine virtuelle pour tester la connexion privée.  

Des points de terminaison privés peuvent être créés pour différents types de services Azure, tels qu’Azure SQL et Stockage Azure.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Une application web Azure avec un **niveau PremiumV2** ou un plan App Service supérieur déployé dans votre abonnement Azure.  
    * Pour plus d’informations et pour obtenir un exemple, consultez [Démarrage rapide : Créer une application web ASP.NET Core dans Azure](../app-service/quickstart-dotnetcore.md). 
    * Pour obtenir un tutoriel détaillé sur la création d’une application web et d’un point de terminaison, consultez [Tutoriel : Se connecter à une application web à l’aide d’Azure Private Endpoint](tutorial-private-endpoint-webapp-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-virtual-network-and-bastion-host"></a>Créer un réseau virtuel et un hôte bastion

Dans cette section, vous allez créer un réseau virtuel, un sous-réseau et un hôte bastion. 

L’hôte bastion sera utilisé pour se connecter de façon sécurisée à la machine virtuelle afin de tester le point de terminaison privé.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel**, ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionnez **CreatePrivateEndpointQS-rg** |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **Europe Ouest**.|

3. Sélectionnez l’onglet **Adresses IP**, ou sélectionnez le bouton **Suivant : Adresses IP** au bas de la page.

4. Sous l’onglet **Adresses IP**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Espace d’adressage IPv4 | Entrez **10.1.0.0/16** |

5. Sous **Nom de sous-réseau**, sélectionnez le mot **par défaut**.

6. Dans **Modifier le sous-réseau**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du sous-réseau | Entrez **mySubnet** |
    | Plage d’adresses de sous-réseau | Entrez **10.1.0.0/24** |

7. Sélectionnez **Enregistrer**.

8. Sélectionnez l'onglet **Sécurité** .

9. Sous **BastionHost**, sélectionnez **Activer**. Entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du bastion | Entrez **myBastionHost** |
    | Espace d’adressage AzureBastionSubnet | Entrez **10.1.1.0/24** |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. </br> Pour **Nom**, entrez **myBastionIP**. </br> Sélectionnez **OK**. |


8. Sélectionnez l’onglet **Vérifier + créer**, ou sélectionnez le bouton **Vérifier + créer**.

9. Sélectionnez **Create** (Créer).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Dans cette section, vous allez créer une machine virtuelle qui sera utilisée pour tester le point de terminaison privé.

1. En haut à gauche de l’écran du portail Azure, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**, ou recherchez **Machine virtuelle** dans la zone de recherche.
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **CreatePrivateEndpointQS-rg** |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **Europe Ouest**. |
    | Options de disponibilité | Sélectionnez **Aucune redondance d’infrastructure requise**. |
    | Image | Sélectionnez **Windows Server 2019 Datacenter - Gen1** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | **myVNet** |
    | Subnet | **mySubnet** |
    | Adresse IP publique | Sélectionnez **Aucun**. |
    | Groupe de sécurité réseau de la carte réseau | **De base**|
    | Aucun port d’entrée public | Sélectionnez **Aucun**. |
   
5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="create-a-private-endpoint"></a>Créer un point de terminaison privé

Dans cette section, vous allez créer un point de terminaison privé pour l’application web que vous avez créée dans la section des prérequis.

1. En haut à gauche de l’écran du portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Liaison privée** ou entrez **Liaison privée** dans la zone de recherche.

2. Sélectionnez **Create** (Créer).

3. Dans **Centre de liaisons privées**, sélectionnez **Points de terminaison privés** dans le menu de gauche.

4. Dans **Points de terminaison privés**, sélectionnez **+ Ajouter**.

5. Sous l’onglet **Général** de **Créer un point de terminaison privé**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **CreatePrivateEndpointQS-rg**. Vous avez créé ce groupe de ressources dans la section précédente.|
    | **Détails de l’instance** |  |
    | Nom  | Entrez **myPrivateEndpoint**. |
    | Région | Sélectionnez **Europe Ouest**. |

6. Sélectionnez l’onglet **Ressource** ou le bouton **Suivant : Ressource** en bas de la page.
    
7. Dans **Ressource**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Méthode de connexion | Sélectionnez **Se connecter à une ressource Azure dans mon répertoire**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Type de ressource | Sélectionnez **Microsoft.Web/sites**. |
    | Ressource | Sélectionnez **\<your-web-app-name>**. </br> Sélectionnez le nom de l’application web que vous avez créée dans les prérequis. |
    | Sous-ressource cible | Sélectionnez les **sites**. |

8. Sélectionnez l’onglet **Configuration** ou le bouton **Suivant : Configuration** en bas de l’écran.

9. Dans **Configuration**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Mise en réseau** |  |
    | Réseau virtuel | Sélectionnez **myVNet**. |
    | Subnet | Sélectionnez **mySubnet**. |
    | **Intégration à un DNS privé** |  |
    | Intégrer à une zone DNS privée | Conservez la valeur par défaut **Oui**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Zones DNS privées | Conservez la valeur par défaut de **(Nouveau) privatelink.azurewebsites.net**.
    

13. Sélectionnez **Revoir + créer**.

14. Sélectionnez **Create** (Créer).

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter à l’application web via le point de terminaison privé.

1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **CreatePrivateEndpointQS-rg**.

3. Sélectionnez **myVM**.

4. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

5. Sélectionnez le bouton bleu **Utiliser le bastion**.

6. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

7. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

8. Entrez `nslookup <your-webapp-name>.azurewebsites.net`. Remplacez **\<your-webapp-name>** par le nom de l’application web que vous avez créée dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mywebapp8675.privatelink.azurewebsites.net
    Address:  10.1.0.5
    Aliases:  mywebapp8675.azurewebsites.net
    ```

    Une adresse IP privée **10.1.0.5** est retournée pour le nom de l’application web.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.

11. Dans la connexion bastion à **myVM**, ouvrez Internet Explorer.

12. Entrez l’URL de votre application web : **https://\<your-webapp-name>.azurewebsites.net**.

13. Vous recevrez la page d’application web par défaut si votre application n’a pas été déployée :

    :::image type="content" source="./media/create-private-endpoint-portal/web-app-default-page.png" alt-text="Page d’application web par défaut." border="true":::

18. Fermez la connexion à **myVM**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le réseau virtuel, la machine virtuelle et l’application web en effectuant les étapes suivantes :

1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **CreatePrivateEndpointQS-rg**.

3. Sélectionnez **Supprimer le groupe de ressources**.

4. Entrez **CreatePrivateEndpointQS-rg** dans **ENTREZ LE NOM DU GROUPE DE RESSOURCES**.

5. Sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :

* Un réseau virtuel et un hôte bastion.
* Une machine virtuelle.
* Un point de terminaison privé pour une application web Azure.

Vous avez utilisé la machine virtuelle pour tester la connectivité de façon sécurisée à l’application web via le point de terminaison privé.



Pour plus d’informations sur les services qui prennent en charge un point de terminaison privé, consultez :
> [!div class="nextstepaction"]
> [Disponibilité des liaisons privées](private-link-overview.md#availability)
