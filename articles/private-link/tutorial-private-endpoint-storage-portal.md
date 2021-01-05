---
title: 'Tutoriel : Se connecter à un compte de stockage en utilisant un point de terminaison privé Azure'
titleSuffix: Azure Private Link
description: Suivez ce tutoriel pour établir une connexion privée avec un compte de stockage à l’aide d’Azure Private Endpoint.
author: asudbring
ms.author: allensu
ms.service: private-link
ms.topic: tutorial
ms.date: 9/25/2020
ms.openlocfilehash: eefef0eb61df9aa0597206b537f3d58e8b8b3213
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484819"
---
# <a name="tutorial-connect-to-a-storage-account-using-an-azure-private-endpoint"></a>Tutoriel : Se connecter à un compte de stockage en utilisant un point de terminaison privé Azure

Azure Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de façon privée avec des ressources Private Link.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un hôte Azure bastion.
> * Création d’une machine virtuelle
> * Créer un compte de stockage avec un point de terminaison privé.
> * Tester la connectivité au point de terminaison privé du compte de stockage.

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

## <a name="create-storage-account-with-a-private-endpoint"></a>Créer un compte de stockage avec un point de terminaison privé

Dans cette section, vous allez créer un compte de stockage et configurer le point de terminaison privé.

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Stockage** > **Compte de stockage**, ou recherchez **Compte de stockage** dans la zone de recherche.

2. Dans l’onglet **Général** de la page **Créer un compte de stockage**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
    | **Détails de l’instance** |  |
    | Nom du compte de stockage | Entrez **mystorageaccount**. Si le nom n’est pas disponible, entrez un nom unique. |
    | Emplacement | Sélectionnez **USA Est**. |
    | Performances | Conservez la valeur par défaut **Standard**. |
    | Type de compte | Conservez la valeur par défaut **Stockage (usage général v2)** . |
    | Réplication| Conservez la valeur par défaut **Stockage géo-redondant avec accès en lecture (RA-GRS)** . |
   
3. Sélectionnez l’onglet **Mise en réseau** ou sélectionnez **Suivant : Réseau**.

4. Sous l’onglet **Mise en réseau**, sous **Méthode de connectivité**, sélectionnez **Point de terminaison privé**.

5. Dans **Point de terminaison privé**, sélectionnez **+ Ajouter**.

6. Dans **Créer un point de terminaison privé**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionner **myResourceGroup** |
    | Emplacement | Sélectionnez **USA Est**. |
    | Nom | Entrez **myPrivateEndpoint** |
    | Sous-ressource de stockage | Conservez la valeur par défaut **Objet blob**. |
    | **Mise en réseau** |  |
    | Réseau virtuel | Sélectionnez **myVNet** |
    | Subnet | Sélectionnez **mySubnet** |
    | **Intégration à un DNS privé** |
    | Intégrer à une zone DNS privée | Conservez la valeur par défaut **Oui**. |
    | Zone DNS privée | Conservez la valeur par défaut (Nouveau) privatelink.blob.core.windows.net. |

7. Sélectionnez **OK**.

8. Sélectionnez **Revoir + créer**.

9. Sélectionnez **Create** (Créer).

10. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

11. Sélectionnez **myResourceGroup**.

12. Sélectionnez le compte de stockage que vous avez créé dans les étapes précédentes.

13. Dans la section **Paramètres** du compte de stockage, sélectionnez **Clés d’accès**.

14. Sélectionnez Copier sur la **Chaîne de connexion** pour **key1**.

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter au compte de stockage via point de terminaison privé.

1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **myVM**.

4. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

5. Sélectionnez le bouton bleu **Utiliser le bastion**.

6. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

7. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

8. Entrez `nslookup <storage-account-name>.blob.core.windows.net`. Remplacez **\<storage-account-name>** par le nom du compte de stockage que vous avez créé dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mystorageaccount8675.privatelink.blob.core.windows.net
    Address:  10.1.0.5
    Aliases:  mystorageaccount8675.blob.core.windows.net
    ```

    Une adresse IP privée **10.1.0.5** est retournée pour le nom du compte de stockage.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.

9. Installez l’[Explorateur Stockage Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows&toc=%2fazure%2fstorage%2fblobs%2ftoc.json) sur la machine virtuelle.

10. Sélectionnez **Terminer** après l’installation de l’**Explorateur Stockage Microsoft Azure**.  Laissez la case cochée pour ouvrir l’application.

11. Dans l’écran **Se connecter à Stockage Azure**, sélectionnez **Utiliser une chaîne de connexion**.

12. Sélectionnez **Suivant**.

13. Entrez le nom de votre compte de stockage utilisé dans les étapes précédentes dans **Nom d’affichage**.

14. Dans la zone sous **Chaîne de connexion**, collez la chaîne de connexion du compte de stockage que vous avez copiée dans les étapes précédentes.

15. Sélectionnez **Suivant**.

16. Vérifiez que les paramètres sont corrects dans **Résumé de la connexion**.  

17. Sélectionnez **Connecter**.

18. Fermez la connexion à **myVM**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le réseau virtuel, la machine virtuelle et le compte de stockage en effectuant les étapes suivantes :

1. Dans le menu de gauche, sélectionnez **Groupes de ressources**.

2. Sélectionnez **myResourceGroup**.

3. Sélectionnez **Supprimer le groupe de ressources**.

4. Entrez **myResourceGroup** dans **TAPEZ LE NOM DU GROUPE DE RESSOURCES**.

5. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment créer un service Private Link :
> [!div class="nextstepaction"]
> [Créer un service Private Link](create-private-link-service-portal.md)