---
title: 'Tutoriel : Création d’une zone DNS enfant Azure'
titleSuffix: Azure DNS
description: Didacticiel sur la création de zones DNS enfant dans le portail Azure.
author: jonbeck
ms.assetid: be4580d7-aa1b-4b6b-89a3-0991c0cda897
ms.service: dns
ms.topic: tutorial
ms.custom: ''
ms.workload: infrastructure-services
ms.date: 7/16/2020
ms.author: jonbeck
ms.openlocfilehash: d805fb926131e047572887758b10506e4edd5944
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183377"
---
# <a name="tutorial-creating-a-new-child-dns-zone"></a>Tutoriel : Création d’une zone DNS enfant

Dans ce tutoriel, vous allez apprendre à : 

> [!div class="checklist"]
> * Connectez-vous au portail Azure.
> * Création d’une zone DNS enfant via une nouvelle zone DNS.
> * Création d’une zone DNS enfant via une zone DNS parent.
> * Vérification de la délégation NS pour la nouvelle zone DNS enfant.



## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif.  Si vous n’avez pas de compte, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Zone Azure DNS parent existante.  

Dans le cadre de ce didacticiel, nous allons utiliser contoso.com comme zone parent et subdomain.contoso.com comme nom de domaine enfant.  Remplacez *contoso.com* par votre nom de domaine parent et *subdomain* par votre domaine enfant.  Si vous n’avez pas créé votre zone DNS parent, consultez les étapes pour [Créer une zone DNS à l’aide du portail Azure](./dns-getstarted-portal.md#create-a-dns-zone). 


## <a name="sign-in-to-azure-portal"></a>Se connecter au portail Azure

Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte Azure.
Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer.

Il existe deux façons de créer votre zone DNS enfant.
1.  Via la page du portail « Créer une zone DNS ».
1.  Via la page de configuration de la zone DNS parent.


## <a name="create-child-dns-zone-via-create-dns-zone"></a>Créer une zone DNS enfant via Créer une zone DNS

Dans cette étape, nous allons créer une zone DNS enfant avec le nom **subdomain.contoso.com** et la déléguer à la zone DNS parent **contoso.com**. Vous allez créer la zone DNS à l’aide des onglets de la page **Créer une zone DNS**.
1.  Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**. La fenêtre **Nouvelle** apparaît.
1.  Sélectionnez **Réseau**, puis **Zone DNS** en enfin **Ajouter**.

1.  Sous l’onglet **Général**, tapez ou sélectionnez les valeurs suivantes :
    * **Abonnement**: Sélectionnez l’abonnement dans lequel créer la zone.
    * **Groupe de ressources** : Entrez votre groupe de ressources existant, ou créez-en un nouveau en sélectionnant **Créer**, entrez *MyResourceGroup*, puis sélectionnez **OK**. Le nom du groupe de ressources doit être unique au sein de l’abonnement Azure.
    * Activez cette case à cocher : **Cette zone est l’enfant d’une zone existante déjà hébergée dans Azure DNS**
    * **Abonnement de zone parent** : Dans cette liste déroulante, recherchez et/ou sélectionnez le nom de l’abonnement sous lequel la zone DNS parent *contoso.com* a été créée.
    * **Zone parent** : Dans la barre de recherche, entrez *contoso.com* pour la charger dans la liste déroulante. Une fois chargée, sélectionnez *contoso.com* dans la liste déroulante.
    * **Nom :** Entrez *subdomain* pour l’exemple de ce didacticiel. Notez que le nom de la zone DNS parent *contoso.com* est automatiquement ajouté comme suffixe au nom quand vous sélectionnez la zone parent dans l’étape ci-dessus.

1. Sélectionnez **Suivant : Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer**, passez en revue le résumé, corrigez les éventuelles erreurs de validation et sélectionnez **Créer**.
La création de la zone peut prendre plusieurs minutes.

 
    :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-dns-zone-inline.png" alt-text="Capture d’écran de la page Créer une zone DNS." lightbox="./media/dns-delegate-domain-azure-dns/create-dns-zone-expanded.png":::

## <a name="create-child-dns-zone-via-parent-dns-zone-overview-page"></a>Page Créer une zone DNS enfant via la zone DNS parent
Vous pouvez également créer une zone DNS enfant et la déléguer dans la zone DNS parent à l’aide du bouton **Zone enfant** à partir de la page Vue d’ensemble de la zone parent. L’utilisation de ce bouton prérenseigne automatiquement les paramètres du parent pour la zone enfant. 

1.  Sur le portail Azure, sous **Toutes les ressources**, ouvrez la zone DNS *contoso.com* dans le groupe de ressources **MyResourceGroup**. Vous pouvez entrer *contoso.com* dans la zone **Filtrer par nom** pour la trouver plus facilement.
1.  Sur la page Vue d’ensemble de la zone DNS, sélectionnez le bouton **+ Zone enfant**.

      :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-inline.png" alt-text="Capture du bouton Zone enfant." border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-expanded.png":::

1.  La page Créer une zone DNS s’ouvre. L’option Zone enfant est déjà activée, et l’abonnement à la zone parent et la zone parent sont déjà renseignés pour vous sur cette page.
1.  Entrez le nom *child* pour l’exemple de ce didacticiel. Notez que le nom de la zone DNS parent contoso.com est automatiquement ajouté comme préfixe au nom.
1.  Sélectionnez **Suivant : Balises**, puis sur **Suivant : Vérifier + créer**.
1.  Sous l’onglet **Vérifier + créer**, passez en revue le résumé, corrigez les éventuelles erreurs de validation et sélectionnez **Créer**.

    :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-dns-zone-child-inline.png" alt-text="Capture d’écran de la zone enfant sélectionnée" border="true"  lightbox="./media/dns-delegate-domain-azure-dns/create-dns-zone-child-expanded.png":::
## <a name="verify-child-dns-zone"></a>Vérifier la zone DNS enfant
Maintenant, vous avez une nouvelle zone DNS enfant *subdomain.contoso.com* créée. Pour vérifier que la délégation s’est correctement déroulée, vous pouvez vérifier que les enregistrements de serveur de noms (NS) de votre zone enfant se trouvent dans la zone parent, comme décrit ci-dessous.  

**Récupérer les serveurs de noms de la zone DNS enfant :**

1.  Sur le portail Azure, sous **Toutes les ressources**, ouvrez la zone DNS *subdomain.contoso.xyz* dans le groupe de ressources **MyResourceGroup**. Vous pouvez entrer *subdomain.contoso.com* dans la zone **Filtrer par nom** pour la trouver plus facilement.
1.  Récupérez les serveurs de noms à partir de la page Vue d’ensemble de la zone DNS. Dans cet exemple, la zone contoso.net a été attribuée aux serveurs de noms *ns1-08.azure-dns.com, ns2-08.azure-dns.net, ns3-08.azure-dns.org* et *ns4-08.azure-dns.info* :

      :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-inline.png" alt-text="Capture d’écran des serveurs de noms de la zone enfant" border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-expanded.png":::
**Vérifiez l’enregistrement NS dans la zone DNS parent :**

Au cours de cette étape, nous allons accéder à la zone DNS parent *contoso.com* et vérifier que son entrée de jeu d’enregistrements NS pour les serveurs de noms des zones enfant a été créée.

1. Sur le portail Azure, sous **Toutes les ressources**, ouvrez la zone DNS contoso.com dans le groupe de ressources **MyResourceGroup**. Vous pouvez entrer contoso.com dans la zone **Filtrer par nom** pour la trouver plus facilement.
1.  Sur la page Vue d’ensemble des zones DNS *contoso.com*, vérifiez les jeux d’enregistrements.
1.  Le jeu d’enregistrements de type NS et de nom de sous-domaine est déjà créé dans la zone DNS parent. Vérifiez que les valeurs de ce jeu d’enregistrements correspondent à la liste de serveurs de noms que nous avons récupérée à partir de la zone DNS enfant dans l’étape ci-dessus.

     :::image type="content" source="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-validate-inline.png" alt-text="Capture d’écran de la validation des serveurs de noms de la zone enfant" border="true" lightbox="./media/dns-delegate-domain-azure-dns/create-child-zone-ns-validate-expanded.png":::
## <a name="clean-up-resources"></a>Nettoyer les ressources
Dès lors que les ressources que vous avez créées dans ce tutoriel ne vous sont plus utiles, supprimez-les en supprimant le groupe de ressources **MyResourceGroup**. Ouvrez le groupe de ressources **MyResourceGroup**, puis sélectionnez **Supprimer le groupe de ressources**.



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Scénarios Azure DNS Private Zones](private-dns-scenarios.md)