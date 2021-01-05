---
title: 'Tutoriel : Se connecter à un compte Azure Cosmos à l’aide d’un point de terminaison privé Azure'
titleSuffix: Azure Private Link
description: Suivez ce tutoriel pour établir une connexion privée avec un compte Azure Cosmos à l’aide d’Azure Private Endpoint.
author: asudbring
ms.author: allensu
ms.service: private-link
ms.topic: tutorial
ms.date: 9/25/2020
ms.openlocfilehash: c9866ac2c14e4deb29326ffdd1a93a7d8eae1ea3
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484802"
---
# <a name="tutorial-connect-to-an-azure-cosmos-account-using-an-azure-private-endpoint"></a>Tutoriel : Se connecter à un compte Azure Cosmos à l’aide d’un point de terminaison privé Azure

Azure Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de façon privée avec des ressources Private Link.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un hôte Azure bastion.
> * Création d’une machine virtuelle
> * Créez un compte Cosmos DB avec un point de terminaison privé.
> * Testez la connectivité à un point de terminaison privé d’un compte Cosmos DB.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-virtual-network-and-bastion-host"></a>Créer un réseau virtuel et un hôte bastion

Dans cette section, vous allez créer un réseau virtuel, un sous-réseau et un hôte bastion. 

L’hôte bastion sera utilisé pour se connecter de façon sécurisée à la machine virtuelle afin de tester le point de terminaison privé.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel**, ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionner **myResourceGroup** |
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

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
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

## <a name="create-a-cosmos-db-account-with-a-private-endpoint"></a>Créer un compte Cosmos DB avec un point de terminaison privé

Dans cette section, vous allez créer un compte Cosmos DB et configurer le point de terminaison privé.

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Bases de données** > **Compte Cosmos DB**, ou recherchez **Compte Cosmos DB** dans la zone de recherche.

2. Sous l’onglet **Informations de base** de la page **Créer un compte Cosmos DB**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionnez votre abonnement Azure. |
    | Groupe de ressources | Sélectionnez **myResourceGroup**. |
    | **Détails de l’instance** |  |
    | Nom du compte | Entez **mycosmosdb**. Si le nom n’est pas disponible, entrez un nom unique. |
    | API | Sélectionnez **Core (SQL)**. |
    | Emplacement | Sélectionnez **USA Est**. |
    | Mode de capacité | Conservez la valeur par défaut **Débit provisionné**. |
    | Appliquer la remise de niveau gratuit | Conservez la valeur par défaut **Ne pas appliquer**. |
    | Géoredondance | Conservez la valeur par défaut **Désactiver**. |
    | Écritures multirégions | Conservez la valeur par défaut **Désactiver**. |
   
3. Sélectionnez l’onglet **Mise en réseau** ou sélectionnez **Suivant : Réseau**.

4. Sous l’onglet **Mise en réseau**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Connectivité réseau** | |
    | Méthode de connexion | Sélectionnez **Point de terminaison privé**. |
    | **Configurer le pare-feu** | |
    | Autoriser l’accès à partir du portail Azure | Conservez la valeur par défaut **Autoriser**. |
    | Autoriser l’accès à partir de mon adresse IP | Conservez la valeur par défaut **Refuser**. |

5. Dans **Point de terminaison privé**, sélectionnez **+ Ajouter**.

6. Dans **Créer un point de terminaison privé**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
    | Emplacement | Sélectionnez **USA Est**. |
    | Nom | Entrez **myPrivateEndpoint** |
    | Sous-ressource cible | Conservez la valeur par défaut **Core (SQL)** |
    | **Mise en réseau** |  |
    | Réseau virtuel | Sélectionnez **myVNet** |
    | Subnet | Sélectionnez **mySubnet** |
    | **Intégration à un DNS privé** |
    | Intégrer à une zone DNS privée | Conservez la valeur par défaut **Oui**. |
    | Zone DNS privée | Conservez la valeur par défaut (New) privatelink.documents.azure.com |

7. Sélectionnez **OK**.

8. Sélectionnez **Revoir + créer**.

9. Sélectionnez **Create** (Créer).

### <a name="add-a-database-and-a-container"></a>Ajouter une base de données et un conteneur

1. Sélectionnez **Accéder à la ressource** ou, dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources** > **mycosmosdb**.

2. Dans le menu de gauche, sélectionnez **Explorateur de données** .

3. Dans la fenêtre **Explorateur de données**, sélectionnez **Nouveau conteneur**.

4. Dans **Ajouter un conteneur**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | ID de base de données | Conservez la valeur par défaut **Créer**. </br> Entrez **mydatabaseid** dans la zone de texte. |
    | Débit (400 - 100 000 RU/s) | Conservez la valeur par défaut **Manuel**. </br> Entrez **400** dans la zone de texte. |
    | ID de conteneur | Entrez **mycontainerid** |
    | Clé de partition | Entrez **/mykey** |

5. Sélectionnez **OK**.

10. Dans la section **Paramètres** du compte Cosmos DB, sélectionnez **Clés**.

11. Sélectionnez **myResourceGroup**.

12. Sélectionnez le compte de stockage que vous avez créé dans les étapes précédentes.

14. Sélectionnez Copier sur la **CHAÎNE DE CONNEXION PRINCIPALE**.

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter au compte Cosmos DB sur un point de terminaison privé.

1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **myVM**.

4. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

5. Sélectionnez le bouton bleu **Utiliser le bastion**.

6. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

7. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

8. Entrez `nslookup <storage-account-name>.documents.azure.com`. Remplacez **\<storage-account-name>** par le nom du compte de stockage que vous avez créé dans les étapes précédentes. 

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mycosmosdb8675.privatelink.documents.azure.com
    Address:  10.1.0.5
    Aliases:  mycosmosdb8675.documents.azure.com
    ```

    L’adresse IP privée **10.1.0.5** est retournée pour le nom du compte Cosmos DB.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.

9. Installez l’[Explorateur Stockage Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) sur la machine virtuelle.

10. Sélectionnez **Terminer** après l’installation de l’**Explorateur Stockage Microsoft Azure**.  Laissez la case cochée pour ouvrir l’application.

11. Dans l’écran **Se connecter au Stockage Azure**, sélectionnez **Annuler**.

12. Dans l’Explorateur Stockage, sélectionnez le bouton droit de la souris sur **Comptes Cosmos DB**, puis sélectionnez **Se connecter à Cosmos DB**.

13. Conservez la valeur par défaut **SQL** sous **Sélectionner une API**.

14. Dans la zone sous **Chaîne de connexion**, collez la chaîne de connexion du compte Cosmos DB que vous avez copiée lors des étapes précédentes.

15. Sélectionnez **Suivant**.

16. Vérifiez que les paramètres sont corrects dans **Résumé de la connexion**.  

17. Sélectionnez **Connecter**.

18. Fermez la connexion à **myVM**.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le réseau virtuel, la machine virtuelle et le compte Cosmos DB en effectuant les étapes suivantes :

1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **Supprimer le groupe de ressources**.

4. Entrez **myResourceGroup** dans **TAPEZ LE NOM DU GROUPE DE RESSOURCES**.

5. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé les éléments suivants :

* Un réseau virtuel et un hôte bastion.
* Une machine virtuelle.
* Un compte Cosmos DB.

Découvrez comment créer un service Private Link :
> [!div class="nextstepaction"]
> [Créer un service Private Link](create-private-link-service-portal.md)