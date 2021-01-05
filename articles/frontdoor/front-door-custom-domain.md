---
title: Tutoriel - Ajouter un domaine personnalisé à votre configuration Azure Front Door
description: Dans ce tutoriel, vous allez apprendre à intégrer un domaine personnalisé à Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/24/2020
ms.author: duau
ms.openlocfilehash: e153edd807dcb119c34f60dc34e33fed510916bb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011521"
---
# <a name="tutorial-add-a-custom-domain-to-your-front-door"></a>Tutoriel : Ajouter un domaine personnalisé à votre Front Door
Ce tutoriel montre comment ajouter un domaine personnalisé à votre Front Door. Quand vous utilisez Azure Front Door pour la livraison d’applications, vous avez besoin d’un domaine personnalisé si vous souhaitez que votre propre nom de domaine soit visible dans votre demande d’utilisateur final. Un nom de domaine visible peut être pratique pour vos clients et utile à des fins de personnalisation.

Une fois que vous avez créé une instance Front Door, l’hôte front-end par défaut, qui est un sous-domaine de `azurefd.net`, est inclus dans l’URL de livraison de contenu Front Door à partir de votre back-end par défaut (par exemple https:\//contoso-frontend.azurefd.net/activeusers.htm). Par commodité, Azure Front Door permet d’associer un domaine personnalisé à l’hôte par défaut. Avec cette option, vous livrez votre contenu avec un domaine personnalisé dans votre URL au lieu d’un nom de domaine détenu par Front Door (par exemple https:\//www.contoso.com/photo.png). 

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Créez un enregistrement CNAME DNS.
> - Associez le domaine personnalisé à votre Front Door.
> - Vérifiez le domaine personnalisé.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!NOTE]
> Front Door ne prend **pas** en charge les domaines personnalisés avec des caractères [Punycode](https://en.wikipedia.org/wiki/Punycode). 

## <a name="prerequisites"></a>Prérequis

* Avant de pouvoir effectuer les étapes de ce tutoriel, vous devez d’abord créer un Front Door. Pour plus d’informations, consultez [Démarrage rapide : Créer une porte d’entrée](quickstart-create-front-door.md)

* Si vous ne disposez pas déjà d’un domaine personnalisé, vous devez tout d’abord en acheter un auprès d’un fournisseur de domaine. Par exemple, consultez [Acheter un nom de domaine personnalisé](../app-service/manage-custom-dns-buy-domain.md).

* Si vous utilisez Azure pour héberger vos [domaines DNS](../dns/dns-overview.md), vous devez déléguer le DNS (Domain Name System) du fournisseur de domaine à un DNS Azure. Pour plus d’informations, voir [Délégation de domaine à Azure DNS](../dns/dns-delegate-domain-azure-dns.md). Sinon, si vous utilisez un fournisseur de domaine pour gérer votre domaine DNS, passez à [Créer un enregistrement CNAME DNS](#create-a-cname-dns-record).


## <a name="create-a-cname-dns-record"></a>Créer un enregistrement CNAME DNS

Avant de pouvoir utiliser un domaine personnalisé avec votre Front Door, vous devez d’abord créer un enregistrement de nom canonique (CNAME) auprès de votre fournisseur de domaine pour pointer vers l’hôte frontend par défaut du Front Door (par exemple, contoso.azurefd.net). Un enregistrement CNAME est un type de fonctionnalité DNS (Domain Name System) qui mappe un nom de domaine source à un nom de domaine de destination. Pour Azure Front Door, le nom de domaine source correspond à votre nom de domaine personnalisé, et le nom de domaine de destination correspond à votre nom d’hôte par défaut Front Door. Une fois que Front Door a vérifié l’enregistrement CNAME que vous avez créé, le trafic adressé au domaine personnalisé source (par exemple www\.contoso.com) est acheminé vers l’hôte front-end par défaut du Front Door de destination spécifié (par exemple contoso-frontend.azurefd.net). 

Un domaine personnalisé et son sous-domaine ne peuvent être associés qu’à une seule instance Front Door à la fois. Toutefois, vous pouvez utiliser différents sous-domaines du même domaine personnalisé pour différentes instances Front Door à l’aide de plusieurs enregistrements CNAME. Vous pouvez également mapper un domaine personnalisé ayant différents sous-domaines à la même instance Front Door.


## <a name="map-the-temporary-afdverify-subdomain"></a>Mapper le sous-domaine temporaire afdverify

Lorsque vous mappez un domaine existant qui est en production, plusieurs points demandent une attention particulière. Lorsque vous inscrivez votre domaine personnalisé dans le Portail Azure, il se peut que le domaine connaisse un court temps d’arrêt. Pour éviter toute interruption du trafic web, commencez par mapper votre domaine personnalisé à l’hôte front-end par défaut Front Door ayant le sous-domaine Azure afdverify pour créer un mappage CNAME temporaire. Avec cette méthode, les utilisateurs peuvent accéder à votre domaine sans interruption pendant que le mappage DNS est en cours.

Sinon, si vous utilisez votre domaine personnalisé pour la première fois et si aucun trafic de production n’est en cours d’exécution sur ce dernier, vous pouvez mapper directement votre domaine personnalisé à votre Front Door. Passez à [Mapper le domaine personnalisé permanent](#map-the-permanent-custom-domain).

Pour créer un enregistrement CNAME avec le sous-domaine afdverify :

1. Connectez-vous au site web du fournisseur de domaine pour votre domaine personnalisé.

2. Recherchez la page permettant de gérer les enregistrements DNS en consultant la documentation du fournisseur ou en recherchant les sections du site web intitulées **Nom de domaine**, **DNS** ou **Gestion des noms de serveur**. 

3. Créez une entrée d’enregistrement CNAME pour votre domaine personnalisé et renseignez les champs comme indiqué dans le tableau suivant (les noms de champs peuvent varier) :

    | Source                    | Type  | Destination                     |
    |---------------------------|-------|---------------------------------|
    | afdverify. www.contoso.com | CNAME | afdverify.contoso-frontend.azurefd.net |

    - Source : entrez votre nom de domaine personnalisé, y compris le sous-domaine afdverify, au format suivant : afdverify. _&lt;nom_de_domaine_personnalisé&gt;_ . Exemple : afdverify. www.contoso.com.

    - Tapez : entrez *CNAME*.

    - Destination : entrez votre hôte front-end Front Door par défaut, y compris le sous-domaine afdverify, au format suivant : afdverify. _&lt;nom_du_point_de_terminaison&gt;_ .azurefd.net. Par exemple : afdverify.contoso-frontend.azurefd.net.

4. Enregistrez vos modifications.

Par exemple, la procédure pour le bureau d’enregistrement de domaines GoDaddy est la suivante :

1. Connectez-vous et sélectionnez le domaine personnalisé que vous souhaitez utiliser.

2. Dans la section Domaines, sélectionnez **Gérer tous**, puis sélectionnez **DNS** | **Gérer les zones**.

3. Pour le **Nom de domaine**, entrez votre domaine personnalisé, puis sélectionnez **Rechercher**.

4. À partir de la page **Gestion DNS**, sélectionnez **Ajouter**, puis sélectionnez **CNAME** dans la liste **Type**.

5. Renseignez les champs suivants de l’entrée CNAME :

    - Tapez : laissez *CNAME* sélectionné.

    - Hôte : entrez le sous-domaine du domaine personnalisé à utiliser, y compris le nom de sous-domaine afdverify. Exemple : afdverify. www.

    - Pointe vers : entrez le nom d’hôte de votre hôte front-end Front Door par défaut, y compris le nom de sous-domaine afdverify. Par exemple : afdverify.contoso-frontend.azurefd.net. 

    - TTL : laissez la valeur *Une heure* sélectionnée.

6. Sélectionnez **Enregistrer**.
 
    L’entrée de l’enregistrement CNAME est ajoutée à la table des enregistrements DNS.


## <a name="associate-the-custom-domain-with-your-front-door"></a>Associer le domaine personnalisé à votre Front Door

Après avoir inscrit votre domaine personnalisé, vous pouvez l’ajouter à votre Front Door.

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) et accédez au Front Door contenant l’hôte frontend à mapper à un domaine personnalisé.
    
2. Dans la page **Concepteur Front Door**, cliquez sur + pour ajouter un domaine personnalisé.
    
3. Spécifiez le **domaine personnalisé**. 

4. Pour **Hôte frontend**, l’hôte frontend à utiliser en tant que domaine de destination de votre enregistrement CNAME est prérempli et est dérivé de votre Front Door : *&lt;nom d’hôte par défaut&gt;* .azurefd.net. Cette valeur n’est pas modifiable.

5. Pour **Nom d’hôte personnalisé**, entrez votre domaine personnalisé, y compris le sous-domaine, à utiliser comme domaine source de votre enregistrement CNAME. Par exemple, www\.contoso.com ou cdn.contoso.com. N’utilisez pas le nom de sous-domaine afdverify.

6. Sélectionnez **Ajouter**.

   Azure vérifie que l’enregistrement CNAME existe pour le nom de domaine personnalisé que vous avez entré. Si l'enregistrement CNAME est correct, votre domaine personnalisé sera validé.

>[!WARNING]
> Vous **devez** vérifier que chacun des hôtes frontend (notamment les domaines personnalisés) de votre Front Door dispose d’une règle d’acheminement à laquelle est associé un chemin par défaut (« /\* »). En d’autres termes, parmi toutes vos règles d’acheminement, il doit exister au moins une règle d’acheminement pour chacun des hôtes frontend définis dans le chemin par défaut (« /\* »). Sinon, le trafic de l’utilisateur final risque de ne pas être routé correctement.

## <a name="verify-the-custom-domain"></a>Vérifier le domaine personnalisé

Une fois l’inscription de votre domaine personnalisé effectuée, vérifiez que celui-ci référence l’hôte frontend par défaut du Front Door.
 
Dans votre navigateur, accédez à l’adresse du fichier à l’aide du domaine personnalisé. Par exemple, si votre domaine personnalisé est robotics.contoso.com, l’URL du fichier mis en cache doit être similaire à l’URL suivante : http:\//robotics.contoso.com/mon-conteneur-public/mon-fichier.jpg. Vérifiez que le résultat est identique à celui obtenu quand vous accédez directement au Front Door sur *&lt;Hôte Front Door&gt;* .azurefd.net.


## <a name="map-the-permanent-custom-domain"></a>Mapper le domaine personnalisé permanent

Si vous avez vérifié que le sous-domaine afdverify a été mappé correctement à votre Front Door (ou si vous utilisez un nouveau domaine personnalisé qui n’est pas en production), vous pouvez ensuite mapper le domaine personnalisé directement à votre hôte frontend par défaut.

Pour créer un enregistrement CNAME pour votre domaine personnalisé :

1. Connectez-vous au site web du fournisseur de domaine pour votre domaine personnalisé.

2. Recherchez la page permettant de gérer les enregistrements DNS en consultant la documentation du fournisseur ou en recherchant les sections du site web intitulées **Nom de domaine**, **DNS** ou **Gestion des noms de serveur**. 

3. Créez une entrée d’enregistrement CNAME pour votre domaine personnalisé et renseignez les champs comme indiqué dans le tableau suivant (les noms de champs peuvent varier) :

    | Source          | Type  | Destination           |
    |-----------------|-------|-----------------------|
    | <www.contoso.com> | CNAME | contoso-frontend.azurefd.net |

   - Source : Entrez votre nom de domaine personnalisé (par exemple www\.contoso.com).

   - Tapez : entrez *CNAME*.

   - Destination : entrez votre hôte front-end Front Door par défaut. Il doit être au format suivant : _&lt;nom_hôte&gt;_ .azurefd.net. Par exemple : contoso-frontend.azurefd.net.

4. Enregistrez vos modifications.

5. Si vous avez créé un enregistrement CNAME temporaire de sous-domaine afdverify, supprimez-le. 

6. Si vous utilisez ce domaine personnalisé en production pour la première fois, suivez les étapes pour [Associer le domaine personnalisé à votre Front Door](#associate-the-custom-domain-with-your-front-door) et [Vérifier le domaine personnalisé](#verify-the-custom-domain).

Par exemple, la procédure pour le bureau d’enregistrement de domaines GoDaddy est la suivante :

1. Connectez-vous et sélectionnez le domaine personnalisé que vous souhaitez utiliser.

2. Dans la section Domaines, sélectionnez **Gérer tous**, puis sélectionnez **DNS** | **Gérer les zones**.

3. Pour le **Nom de domaine**, entrez votre domaine personnalisé, puis sélectionnez **Rechercher**.

4. À partir de la page **Gestion DNS**, sélectionnez **Ajouter**, puis sélectionnez **CNAME** dans la liste **Type**.

5. Renseignez les champs de l’entrée CNAME :

    - Tapez : laissez *CNAME* sélectionné.

    - Hôte : entrez le sous-domaine de votre domaine personnalisé à utiliser. Par exemple, www ou profil.

    - Pointe vers : entrez le nom d’hôte par défaut de votre porte d’entrée. Exemple : contoso.azurefd.net. 

    - TTL : laissez la valeur *Une heure* sélectionnée.

6. Sélectionnez **Enregistrer**.
 
    L’entrée de l’enregistrement CNAME est ajoutée à la table des enregistrements DNS.

7. Si vous avez un enregistrement CNAME afdverify, sélectionnez l’icône en forme de crayon à côté de celui-ci, puis l’icône en forme de corbeille.

8. Sélectionnez **Supprimer** pour supprimer l’enregistrement CNAME.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Dans les étapes précédentes, vous avez ajouté un domaine personnalisé à un Front Door. Si vous ne souhaitez plus associer votre Front Door à un domaine personnalisé, vous pouvez supprimer le domaine personnalisé en effectuant les étapes suivantes :
 
1. Dans votre Concepteur Front Door, sélectionnez le domaine personnalisé à supprimer.

2. Cliquez dans le menu contextuel du domaine personnalisé sur Supprimer.  

   Le domaine personnalisé est dissocié de votre point de terminaison.


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

* Créez un enregistrement CNAME DNS.
* Associez le domaine personnalisé à votre Front Door.
* Vérifiez le domaine personnalisé.

Pour savoir comment activer le protocole HTTPS de votre domaine personnalisé, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Activer le protocole HTTPS pour un domaine personnalisé](front-door-custom-domain-https.md)