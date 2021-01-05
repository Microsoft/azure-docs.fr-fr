---
title: Tutoriel - Se connecter à un serveur SQL Azure avec Azure Private Endpoint - Portail
description: Ce tutoriel explique comment créer un serveur SQL Azure avec un point de terminaison privé à l’aide du portail Azure.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 10/20/2020
ms.author: allensu
ms.openlocfilehash: a7eca29dc9390306ac6ad4e66eec75a25c2d33ff
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95522156"
---
# <a name="tutorial---connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-portal"></a>Tutoriel - Se connecter à un serveur SQL Azure avec Azure Private Endpoint - Portail Azure

Azure Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de façon privée avec des ressources Private Link.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un hôte Azure bastion.
> * Création d’une machine virtuelle
> * Créer un serveur SQL Azure et un point de terminaison privé
> * Tester la connectivité au point de terminaison privé du serveur SQL

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

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
    | Groupe de ressources   | Sélectionner **CreateSQLEndpointTutorial-rg** |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **USA Est**. |

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

    | Paramètre | Value                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **CreateSQLEndpointTutorial** |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **USA Est**. |
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

## <a name="create-an-azure-sql-server-and-private-endpoint"></a><a name ="create-a-private-endpoint"></a>Créer un serveur SQL Azure et un point de terminaison privé

Dans cette section, vous allez créer un serveur SQL dans Azure. 

1. En haut à gauche de l’écran du portail Azure, sélectionnez **Créer une ressource** > **Bases de données** > **Base de données SQL**.

1. Sous l’onglet **De base** de la page **Créer une base de données SQL**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **CreateSQLEndpointTutorial**. Vous avez créé ce groupe de ressources dans la section précédente.|
    | **Détails de la base de données** |  |
    | Nom de la base de données  | Entrez **mysqldatabase**. Si ce nom est utilisé, créez un nom unique. |
    | Serveur | Sélectionnez **Créer nouveau**. |

6. Dans **Nouveau serveur**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom du serveur  | Entrez **mysqlserver**. Si ce nom est utilisé, créez un nom unique.|
    | Connexion d’administrateur serveur | Entrez le nom d’un administrateur de votre choix. |
    | Mot de passe | Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 8 caractères et satisfaire aux exigences définies. |
    | Emplacement | Sélectionnez **USA Est**. |
    
7. Sélectionnez **OK**.

8. Sélectionnez l’onglet **Mise en réseau** ou sélectionnez **Suivant : Réseau**.

9. Sous l’onglet **Réseau**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Connectivité réseau** | |
    | Méthode de connexion | Sélectionnez **Point de terminaison privé**. |
   
10. Sélectionnez **+ Ajouter un point de terminaison privé** dans **Points de terminaison privés**.

11. Dans **Créer un point de terminaison privé**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Value |
    | ------- | ----- |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **CreateSQLEndpointTutorial**. |
    | Emplacement | Sélectionnez **USA Est**. |
    | Name | Entrez **myPrivateSQLendpoint**. |
    | Sous-ressource cible | Sélectionnez **SQLServer**. |
    | **Mise en réseau** |  |
    | Réseau virtuel | Sélectionnez **myVNet**. |
    | Subnet | Sélectionnez **mySubnet**. |
    | **Intégration à un DNS privé** | |
    | Intégrer à une zone DNS privée | Conservez la valeur par défaut **Oui**. |
    | Zone DNS privée | Laissez la valeur par défaut **(Nouveau) privatelink.database.windows.net**. |

12. Sélectionnez **OK**. 

13. Sélectionnez **Revoir + créer**.

14. Sélectionnez **Create** (Créer).

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter au serveur SQL via le point de terminaison privé.

1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **CreateSQLEndpointTutorial**.

3. Sélectionnez **myVM**.

4. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

5. Sélectionnez le bouton bleu **Utiliser le bastion**.

6. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

7. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

8. Entrez `nslookup <sqlserver-name>.database.windows.net`. Remplacez **\<sqlserver-name>** par le nom du serveur SQL que vous avez créé dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.1.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Une adresse IP privée **10.1.0.5** est retournée pour le nom du serveur SQL.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.


9. Installez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) sur **myVM**.

10. Ouvrez **SQL Server Management Studio**.

4. Dans **Se connecter au serveur**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Type de serveur | Sélectionnez **Moteur de base de données**.|
    | Nom du serveur | Entrez **\<sqlserver-name>.database.windows.net** |
    | Authentification | Sélectionnez **Authentification SQL Server**. |
    | Nom d’utilisateur | Entrez le nom d’utilisateur que vous avez entré lors de la création du serveur. |
    | Mot de passe | Entrez le mot de passe que vous avez entré lors de la création du serveur. |
    | Se souvenir du mot de passe | Sélectionnez **Oui**. |

1. Sélectionnez **Connecter**.
2. Parcourez les bases de données dans le menu de gauche.
3. (Facultatif) Créez ou interrogez des informations à partir de **mysqldatabase**.
4. Fermez la connexion Bureau à distance à **myVM**. 

## <a name="clean-up-resources"></a>Nettoyer les ressources 
Lorsque vous avez fini d’utiliser le point de terminaison privé, le serveur SQL et la machine virtuelle, supprimez le groupe de ressources et toutes les ressources qu’il contient : 
1. Entrez **CreateSQLEndpointTutorial** dans la zone **Recherche** située en haut du portail, puis sélectionnez **CreateSQLEndpointTutorial** dans les résultats de la recherche. 
2. Sélectionnez **Supprimer le groupe de ressources**. 
3. Dans **TAPER LE NOM DU GROUPE DE RESSOURCES**, entrez CreateSQLEndpointTutorial, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé les éléments suivants :

* Un réseau virtuel et un hôte bastion.
* Une machine virtuelle
* Un serveur SQL Azure avec un point de terminaison privé

Vous avez utilisé la machine virtuelle pour tester de façon sécurisée la connectivité au serveur SQL via le point de terminaison privé.

Découvrez comment créer un service Private Link :
> [!div class="nextstepaction"]
> [Créer un service Private Link](create-private-link-service-portal.md)