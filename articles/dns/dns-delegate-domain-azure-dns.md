---
title: 'Tutoriel : Héberger votre domaine et votre sous-domaine - Azure DNS'
description: Dans ce tutoriel, vous découvrez comment configurer Azure DNS pour héberger vos zones DNS.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 3/11/2019
ms.author: rohink
ms.openlocfilehash: a8f64ab3141459142def12a1758b0fe0a94ca432
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92282174"
---
# <a name="tutorial-host-your-domain-in-azure-dns"></a>Tutoriel : Hébergez votre domaine dans Azure DNS

Vous pouvez utiliser Azure DNS pour héberger vos domaines DNS et gérer vos enregistrements DNS. En hébergeant vos domaines dans Azure, vous pouvez gérer vos enregistrements DNS à l’aide des mêmes informations d’identification, les mêmes API, les mêmes outils et la même facturation que vos autres services Azure.

Par exemple, supposons que vous achetez le domaine contoso.net à partir d’un registre de noms de domaine et que vous créez une zone avec le nom contoso.net dans Azure DNS. En tant que propriétaire du domaine, votre registre vous permet de configurer les enregistrements de serveur de noms (NS) pour votre domaine. Le registre stocke ces enregistrements NS dans la zone parent .NET. Les utilisateurs internet du monde entier sont ensuite redirigés vers votre domaine dans la zone Azure DNS lorsqu’ils tentent de résoudre des enregistrements DNS dans contoso.net.


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une zone DNS.
> * Récupérer une liste des serveurs de noms.
> * Déléguer le domaine.
> * Vérifier que la délégation fonctionne.


Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour le test, vous devez disposer d’un nom de domaine disponible que vous pouvez héberger dans Azure DNS. Vous devez disposer d’un contrôle total de ce domaine. Le contrôle total comprend notamment la possibilité de définir les enregistrements de serveur de noms pour le domaine.

Dans cet exemple, nous allons faire référence au domaine parent en tant que **contoso.net**

## <a name="create-a-dns-zone"></a>Création d’une zone DNS

1. Accédez au [Portail Azure](https://portal.azure.com/) pour créer une zone DNS. Recherchez et sélectionnez **zones DNS** .

   ![Zone DNS](./media/dns-delegate-domain-azure-dns/openzone650.png)

1. Sélectionnez **Créer une zone DNS** .
1. Sur la page **Créer une zone DNS** , entrez les valeurs suivantes, puis cliquez sur **Créer** : par exemple, **contoso.net**
      > [!NOTE] 
      > Si la nouvelle zone que vous créez est une zone enfant (par exemple, zone parent = contoso.net, zone enfant = child.contoso.net), consultez notre [tutoriel Création d’une nouvelle zone DNS enfant](./tutorial-public-dns-zones-child.md)

    | **Paramètre** | **Valeur** | **Détails** |
    |--|--|--|
    | **Détails du projet :**  |  |  |
    | **Groupe de ressources**    | ContosoRG | Créez un groupe de ressources. Le nom du groupe de ressources doit être unique au sein de l’abonnement sélectionné. L’emplacement du groupe de ressources n’a aucun impact sur la zone DNS. L’emplacement de la zone DNS est toujours « global » et n’est pas affiché. |
    | **Détails de l’instance :** |  |  |
    | **Enfant de zone**        | laissez la case non activée | Dans la mesure où cette zone n’est **pas** une [zone enfant](./tutorial-public-dns-zones-child.md) vous devez la laisser désactiver |
    | **Nom**              | contoso.net | Champ pour le nom de votre zone parent      |
    | **Lieu**          | USA Est | Ce champ est basé sur l’emplacement sélectionné dans le cadre de la création du groupe de ressources.  |
    

## <a name="retrieve-name-servers"></a>Récupérer les serveurs de noms

Avant de pouvoir déléguer votre zone DNS à Azure DNS, vous devez d’abord connaître les serveurs de noms correspondant à votre zone. Azure DNS alloue des serveurs de noms à partir d’un pool chaque fois qu’une zone est créée.

1. Une fois la zone DNS créée, accédez au panneau **Favoris** du portail Azure et sélectionnez **Toutes les ressources** . Sur la page **Toutes les ressources** , sélectionnez votre zone DNS. Si l’abonnement que vous avez sélectionné comporte déjà plusieurs ressources, vous pouvez saisir votre nom de domaine dans la case **Filtrer par nom** pour accéder facilement à la passerelle d’application. 

1. Récupérez les serveurs de noms à partir de la page de la zone DNS. Dans cet exemple, la zone contoso.net a été attribuée aux serveurs de noms *ns1-01.azure-dns.com* , *ns2-01.azure-dns.net* , *ns3-01.azure-dns.org* , et *ns4-01.azure-dns.info*  :

   ![Liste des serveurs de noms](./media/dns-delegate-domain-azure-dns/viewzonens500.png)

Azure DNS crée automatiquement des enregistrements NS faisant autorité dans votre zone pour les serveurs de noms attribués.

## <a name="delegate-the-domain"></a>Déléguer le domaine

Maintenant que la zone DNS est créée et que vous disposez des serveurs de noms, vous devez mettre à jour le domaine parent avec les serveurs de noms Azure DNS. Chaque bureau d’enregistrement a ses propres outils de gestion DNS pour modifier les enregistrements de serveur de noms pour un domaine. 

1. Dans la page de gestion des DNS du registre, modifiez les enregistrements NS et remplacez-les par ceux créés par les serveurs de noms Azure DNS.

1. Lors de la délégation d’un domaine à Azure DNS, vous devez utiliser les serveurs de noms fournis par Azure DNS. Utilisez les quatre serveurs de noms, quel que soit le nom de votre domaine. La délégation de domaine ne requiert pas que le serveur de noms utilise le même domaine de premier niveau que votre domaine.

> [!NOTE]
> Lorsque vous copiez chaque adresse de serveur de nom, assurez-vous de copier le point final à la fin de l’adresse. Le point final indique la fin d’un nom de domaine complet. Certains bureaux d’enregistrement ajoutent le point final s’il n’est pas présent dans le nom NS. Pour être conforme à la norme RFC DNS, incluez le point final.

Les délégations utilisant les serveurs de noms dans votre propre zone (parfois appelés *serveurs de noms de redirection vers un microsite* ) ne sont actuellement pas prises en charge dans Azure DNS.

## <a name="verify-the-delegation"></a>Vérifier la délégation

Une fois la délégation effectuée, vous pouvez vérifier qu’elle fonctionne à l’aide d’un outil tel que *nslookup* pour interroger l’enregistrement Start of Authority (SOA) de votre zone. L’enregistrement SOA est créé automatiquement en même temps que la zone. Vous devrez peut-être patienter 10 minutes ou plus une fois la délégation effectuée avant de pouvoir vérifier qu’elle fonctionne. La propagation des modifications dans le système DNS peut prendre du temps.

Il est inutile de spécifier les serveurs de noms Azure DNS. Si la délégation est correctement configurée, le processus de résolution DNS normal détecte automatiquement les serveurs de noms.

1. À partir d’une invite de commandes, saisissez une commande nslookup semblable à l’exemple suivant :

   ```
   nslookup -type=SOA contoso.net
   ```

1. Vérifiez que la réponse ressemble à la sortie nslookup suivante :

   ```
   Server: ns1-04.azure-dns.com
   Address: 208.76.47.4

   contoso.net
   primary name server = ns1-04.azure-dns.com
   responsible mail addr = msnhst.microsoft.com
   serial = 1
   refresh = 900 (15 mins)
   retry = 300 (5 mins)
   expire = 604800 (7 days)
   default TTL = 300 (5 mins)
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez conserver le groupe de ressources **contosoRG** si vous avez l’intention d’effectuer le didacticiel suivant. Sinon, supprimez le groupe de ressources **contosoRG** pour supprimer les ressources créées dans ce didacticiel.

- Sélectionnez le groupe de ressources **contosoRG** , puis cliquez sur **Supprimer le groupe de ressources** . 

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une zone DNS pour votre domaine et l’avez délégué à Azure DNS. Pour en savoir plus sur Azure DNS et les applications web, passez au tutoriel sur les applications web.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)
