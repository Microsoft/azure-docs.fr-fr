---
title: Peering de connexions pour les partenaires Exchange avec un serveur d’itinéraire à l’aide du portail
titleSuffix: Azure
description: Créer ou modifier un Peering Exchange avec un serveur d’itinéraire à l’aide du Portail Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/19/2020
ms.author: derekol
ms.openlocfilehash: 7e10cd91eadd338217845b1504e8e9160bccfc98
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537189"
---
# <a name="create-or-modify-an-exchange-peering-with-route-server-in-azure-portal"></a>Créer ou modifier un Peering Exchange avec un serveur d’itinéraire à l’aide du Portail Azure

Cet article explique comment créer un Peering Microsoft Exchange avec un serveur d’itinéraire à l’aide du Portail Azure. Il montre également comment vérifier l’état de la ressource, la mettre à jour, la supprimer et la déprovisionner.


## <a name="before-you-begin"></a>Avant de commencer
* Passez en revue les [prérequis](prerequisites.md) et la [procédure pas à pas du Peering Exchange](walkthrough-exchange-all.md) avant de commencer la configuration.
* Si vous avez déjà des Peerings Exchange avec Microsoft qui ne sont pas convertis en ressources Azure, consultez [Convertir un Peering Exchange hérité en ressource Azure à l’aide du portail](howto-legacy-exchange-portal.md).

## <a name="create-and-provision-an-exchange-peering"></a>Créer et provisionner un Peering Exchange

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Se connecter au portail et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="create-an-exchange-peering-with-route-server"></a><a name=create></a>Créer un Peering Exchange avec un serveur d’itinéraire


En tant que fournisseur Internet Exchange, vous pouvez créer une requête de Peering Exchange en [créant un Peering]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. Dans la page **Créer un Peering**, sur l’onglet **De base**, remplissez les cases comme illustré ici :

    > [!div class="mx-imgBorder"] 
    > ![Inscrire Peering Service](./media/setup-basics-tab.png)

* Sélectionnez votre abonnement Azure.

* Sélectionnez un groupe de ressources existant dans la liste déroulante, ou créez un groupe en sélectionnant Créer nouveau. Dans cet exemple, ,nous allons créer un nouveau groupe de ressources.

* Le nom correspond au nom de la ressource et vous pouvez le choisir librement.

* La région est sélectionnée automatiquement si vous avez choisi un groupe de ressources existant. Si vous avez choisi de créer un nouveau groupe de ressources, vous devez également choisir la région Azure dans laquelle vous souhaitez que la ressource réside.

    >[!NOTE]
    >La région dans laquelle un groupe de ressources réside est indépendante de l’emplacement où vous souhaitez créer le Peering avec Microsoft. Toutefois, il est recommandé d’organiser vos ressources de Peering dans des groupes de ressources qui résident dans les régions Azure les plus proches. Par exemple : pour les Peerings à Ashburn, vous pouvez créer un groupe de ressources dans les régions USA Est ou USA Est2.

* Sélectionnez votre ASN dans la zone **PeerASN**.

    >[!IMPORTANT] 
    >Vous pouvez uniquement choisir un ASN dont le paramètre ValidationState est « approuvé » avant de soumettre une demande de Peering. Si vous venez d’envoyer votre demande PeerAsn, patientez environ 12 heures pour que l’association ASN soit « approuvée ». Si l’ASN que vous sélectionnez est en attente de validation, un message d’erreur s’affiche. Si vous ne voyez pas l’ASN que vous devez choisir, vérifiez que vous avez sélectionné le bon abonnement. Si c’est le cas, vérifiez que vous avez déjà créé PeerAsn à l’aide de **[l’association entre l’ASN pair et un abonnement Azure](https://go.microsoft.com/fwlink/?linkid=2129592)** .

* Sélectionnez **Suivant : Configuration** pour continuer.

#### <a name="configure-connections-and-submit"></a>Configurer les connexions et les soumettre

1. Sur l’onglet Configuration de la page Créer un Peering, remplissez les cases comme illustré ici :

    > [!div class="mx-imgBorder"]
    > ![Configurer un serveur d’itinéraire](./media/setup-exchange-conf-tab-routeserver.png)
 
    * Sélectionnez le type de Peering **Direct**.
    * Pour un réseau Microsoft, sélectionnez **AS8075 with Exchange Route Server** (AS8075 avec serveur d’itinéraire Exchange). 
    * Sélectionnez **Gratuit de base** pour la SKU. Ne sélectionnez pas Premium gratuit, car cette option est réservée aux applications spéciales.
    * Sélectionnez **Zone urbaine** comme l’emplacement où vous souhaitez configurer le Peering.

1. Sous **Connexions de Peering**, sélectionnez **Créer**.

1.  Sous **Direct Peering Connection** (Connexion de Peering directe), renseignez les détails de la session BGP :

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran représentant le volet Connexion de Peering directe dans lequel des détails ont été ajoutés.](./media/setup-exchange-conf-tab-direct-route.png)


     * Installation de Peering : sélectionnez l’emplacement physique approprié pour le Peering.
     * Pour ce qui est du fournisseur d’adresse de session, sélectionnez Pair.
     * Le préfixe IPv4 de la session sera fourni par le pair du fournisseur Exchange.
     * L’adresse IPv4 de session du pair est sélectionnée par le pair Exchange pour le serveur d’itinéraire, à partir de sa plage de préfixes IP.
     * L’adresse IPv4 de la session Microsoft correspond à l’adresse IP du routeur allouée à partir de la plage de préfixes IP.
     * L’adresse IPv6 de la session est facultative pour l’instant.
     * Le préfixe IPv4 publié peut aller jusqu’à 20000 maximum. 
     * L’utilisation de Peering Service est désactivée par défaut. Vous pouvez l’activer une fois que le fournisseur Exchange a signé un contrat Peering Service avec Microsoft.

1. Lorsque l’opération est terminée, cliquez sur **Enregistrer**. 

1. Sous Créer un Peering, vous pouvez constater que la validation a abouti. Une fois la validation effectuée, cliquez sur **Créer**.

    > [!div class="mx-imgBorder"]
    > ![Validation des paramètres](./media/setup-exchange-conf-tab-validation.png)

    >[!NOTE]
    >Pour les fournisseurs de services Internet (ISP) normaux reconnus comme des partenaires Microsoft Peering Service, l’inscription des préfixes IP du client est nécessaire. Toutefois, dans le cas des partenaires Exchange avec un serveur d’itinéraire, il est nécessaire d’inscrire les numéros ASN des clients, et non les préfixes. La même clé ASN est valide pour l’inscription du préfixe du client.

1. Sélectionnez **Registered ASNs** (Numéros ASN inscrits) dans la section des paramètres.

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran représentant le volet Peering dans lequel l'élément de menu Numéros ASN inscrits est sélectionné.](./media/setup-exchange-registered-asn.png)

1. Sélectionnez **Add Registered ASN** (Ajouter le numéro ASN inscrit) pour créer un numéro ASN client dans le cadre de votre abonnement Exchange.

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran représentant le volet Inscrire un numéro ASN, et les zones de texte Nom et Numéro ASN.](./media/setup-exchange-register-new-asn.png)

1. Sous Register an ASN (Inscrire un numéro ASN), sélectionnez un nom, indiquez le numéro ASN du client, puis cliquez sur Enregistrer.

1. Une clé de préfixe associée à chaque numéro ASN s’affiche sous les numéros ASN inscrits. En tant que fournisseur Exchange, vous devrez fournir cette clé de préfixe à votre client pour qu’il puisse inscrire Peering Service dans le cadre de son abonnement.

    > [!div class="mx-imgBorder"]
    > ![Capture d'écran représentant le volet Numéros ASN inscrits avec des clés de préfixe.](./media/setup-exchange-register-asn-prefixkey.png)




### <a name="verify-an-exchange-peering"></a><a name=get></a>Vérifier un Peering Exchange
[!INCLUDE [peering-exchange-get-portal](./includes/exchange-portal-get.md)]

## <a name="modify-an-exchange-peering"></a><a name="modify"></a>Modifier un Peering Exchange
[!INCLUDE [peering-exchange-modify-portal](./includes/exchange-portal-modify.md)]

## <a name="deprovision-an-exchange-peering"></a><a name="delete"></a>Déprovisionner un Peering Exchange
[!INCLUDE [peering-exchange-delete-portal](./includes/delete.md)]

## <a name="next-steps"></a>Étapes suivantes

* [Créer ou modifier un Peering direct à l’aide du portail](howto-direct-portal.md)
* [Convertir un Peering direct hérité en ressource Azure à l’aide du portail](howto-legacy-direct-portal.md)

## <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les [FAQ sur le Peering Internet](faqs.md).
