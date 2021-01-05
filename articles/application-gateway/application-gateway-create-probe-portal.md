---
title: Créer une sonde personnalisée à l’aide du portail
titleSuffix: Azure Application Gateway
description: Découvrez comment créer une sonde personnalisée pour Application Gateway à l’aide du portail
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/09/2020
ms.author: victorh
ms.openlocfilehash: 5d2760415e4f4ef3b181f2fb69802659fec3ef66
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95975953"
---
# <a name="create-a-custom-probe-for-application-gateway-by-using-the-portal"></a>Créer une sonde personnalisée pour Application Gateway à l’aide du portail

> [!div class="op_single_selector"]
> * [Azure portal](application-gateway-create-probe-portal.md)
> * [Commandes PowerShell pour Azure Resource Manager](application-gateway-create-probe-ps.md)
> * [Azure Classic PowerShell](application-gateway-create-probe-classic-ps.md)

Dans cet article, une sonde d’intégrité personnalisée est ajoutée à une passerelle d’application existante via le portail Azure. À l’aide des sondes d’intégrité, Azure Application Gateway surveille l’intégrité des ressources dans le pool back-end.

## <a name="before-you-begin"></a>Avant de commencer

Si vous ne disposez pas déjà d’une passerelle d’application, consultez [Créer une passerelle d’application](./quick-create-portal.md) pour créer une passerelle d’application fonctionnelle.

## <a name="create-probe-for-application-gateway-v2-sku"></a>Créer une sonde pour la référence SKU Application Gateway v2

Les sondes sont configurées via un processus en deux étapes sur le portail. La première étape consiste à entrer les valeurs requises pour la configuration de la sonde. Dans la deuxième étape, vous testez l’intégrité du serveur back-end à l’aide de la configuration de cette sonde et enregistrez la sonde. 

### <a name="enter-probe-properties"></a><a name="createprobe"></a>Entrer les propriétés de la sonde

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous ne possédez pas encore de compte, vous pouvez [vous inscrire pour bénéficier d’un essai gratuit d’un mois](https://azure.microsoft.com/free).

2. Dans le volet Favoris du portail Azure, cliquez sur Toutes les ressources. Cliquez sur la passerelle d’application dans le panneau Toutes les ressources. Si l’abonnement que vous avez déjà sélectionné comporte déjà plusieurs ressources, vous pouvez entrer partners.contoso.net dans la zone Filtrer par nom… pour accéder facilement à la passerelle d’application.

3. Sélectionnez **Sondes d’intégrité**, puis sélectionnez **Ajouter** pour ajouter une nouvelle sonde d’intégrité.

   ![Ajouter une nouvelle sonde][4]

4. Sur la page **Ajouter une sonde d’intégrité**, fournissez les informations nécessaires à la sonde, puis sélectionnez **OK** une fois que vous avez terminé.

   |**Paramètre** | **Valeur** | **Détails**|
   |---|---|---|
   |**Nom**|customProbe|Cette valeur est le nom convivial donné à la sonde accessible par le biais du portail.|
   |**Protocole**|HTTP ou HTTPS | Protocole utilisé par la sonde d’intégrité. |
   |**Hôte**|Par exemple, contoso.com|Cette valeur est le nom de l’hôte virtuel (différent du nom d’hôte de machine virtuelle) en cours d’exécution sur le serveur d’application. La sonde est envoyée à \<protocol\>://\<host name\>:\<port\>/\<urlPath\>|
   |**Choisir un nom d'hôte à partir de paramètres HTTP back-end**|Oui ou Non|Définit l’en-tête *host* dans la sonde sur le nom d’hôte indiqué dans les paramètres HTTP auxquels cette sonde est associée. Spécialement requis en cas de serveurs back-end multilocataire, comme Azure App Service. [En savoir plus](./configuration-http-settings.md#pick-host-name-from-back-end-address)|
   |**Choisir un port à partir de paramètres HTTP back-end**| Oui ou Non|Définit le *port* de la sonde d’intégrité sur le port indiqué dans les paramètres HTTP auxquels cette sonde est associée. Si vous choisissez Non, vous pouvez entrer un port de destination personnalisé à utiliser |
   |**Port**| 1-65535 | Port personnalisé à utiliser pour les sondes d’intégrité | 
   |**Chemin d’accès**|/ ou n’importe quel chemin valide|Reste de l’URL complète de la sonde personnalisée. Un chemin valide commence par « / ». Pour le chemin par défaut de http:\//contoso.com, utilisez simplement « / » |
   |**Intervalle (secondes)**|30|Fréquence d’exécution de la sonde pour le contrôle d’intégrité. Il n’est pas recommandé de définir la valeur sur moins de 30 secondes.|
   |**Délai d’expiration (secondes)**|30|Intervalle de temps précédant l’expiration de la sonde. Si aucune réponse valide n’est reçue dans le délai imparti, la sonde est marquée comme étant en échec. L’intervalle de délai d’attente doit être suffisamment élevé pour qu’un appel HTTP puisse être envoyé afin de garantir que la page d’intégrité backend est disponible. Notez que la valeur du délai d’attente ne doit pas être supérieure à la valeur « Intervalle » utilisée dans ce paramètre de sonde ou la valeur « Délai d’attente de la demande » dans le paramètre HTTP qui sera associé à cette sonde.|
   |**Seuil de défaillance sur le plan de l’intégrité**|3|Nombre d’échecs consécutifs nécessaires pour marquer l’instance comme étant défaillante. Le seuil peut être défini sur 1 ou plus.|
   |**Utiliser des conditions de correspondance de sonde**|Oui ou Non|Par défaut, une réponse HTTP(S) avec un code d’état compris entre 200 et 399 est considérée comme intègre. Vous pouvez modifier la plage acceptable du code de réponse back-end ou du corps de la réponse back-end. [En savoir plus](./application-gateway-probe-overview.md#probe-matching)|
   |**Paramètres HTTP**|sélection dans une liste déroulante|La sonde est associée au(x) paramètre(s) HTTP sélectionné(s) ici et, par conséquent, surveille l’intégrité de ce pool back-end qui est associé au paramètre HTTP sélectionné. Le port utilisé pour la requête de la sonde est le même que celui utilisé dans le paramètre HTTP sélectionné. Vous ne pouvez choisir que le ou les paramètres HTTP qui ne sont associés à aucune autre sonde personnalisée. <br>Notez que seuls les paramètres HTTP qui ont le même protocole que celui choisi dans cette configuration de sonde et qui ont le même état pour le commutateur *Choisir un nom d’hôte à partir du paramètre HTTP back-end* sont disponibles pour l’association.|
   
   > [!IMPORTANT]
   > La sonde surveille l’intégrité du back-end uniquement lorsqu’elle est associée à un ou plusieurs paramètres HTTP. Elle surveille les ressources back-end de ces pools back-end qui sont associés aux paramètres HTTP auxquels cette sonde est liée. La requête de sonde sera envoyée sous la forme \<protocol\>://\<hostName\>:\<port\>/\<urlPath\>.

### <a name="test-backend-health-with-the-probe"></a>Tester l’intégrité du back-end avec la sonde

Après avoir entré les propriétés de la sonde, vous pouvez tester l’intégrité des ressources back-end pour vérifier que la configuration de la sonde est correcte et que les ressources back-end fonctionnent comme prévu.

1. Sélectionnez **Tester** et notez le résultat de la sonde. Application Gateway teste l’intégrité de toutes les ressources back-end dans les pools back-end associés aux paramètres HTTP utilisés pour cette sonde. 

   ![Tester l’intégrité du back-end][5]

2. S’il existe des ressources back-end non intègres, vérifiez la colonne **Détails** pour comprendre la raison de l’état non intègre de la ressource. Si la ressource a été marquée comme non intègre en raison d’une configuration de sonde incorrecte, sélectionnez le lien **Revenir à la sonde** et modifiez la configuration de la sonde. Sinon, si la ressource a été marquée comme non intègre en raison d’un problème avec le back-end, résolvez les problèmes avec la ressource back-end, puis testez de nouveau le back-end en sélectionnant le lien **Revenir à la sonde** et **Tester**.

   > [!NOTE]
   > Vous pouvez choisir d’enregistrer la sonde même avec des ressources back-end non intègres, mais cela n’est pas recommandé. Cela est dû au fait qu’Application Gateway ne transfère pas les requêtes aux serveurs back-end du pool de back-ends qui sont considérés comme non sains par la sonde. Dans le cas où il n’y a pas de ressources saines dans un pool de back-ends, vous ne pouvez pas accéder à votre application et vous obtenez une erreur HTTP 502.

   ![Afficher le résultat de la sonde][6]

3. Sélectionnez **Ajouter** pour enregistrer la sonde. 

## <a name="create-probe-for-application-gateway-v1-sku"></a>Créer une sonde pour la référence SKU Application Gateway v1

Les sondes sont configurées via un processus en deux étapes sur le portail. La première étape consiste à créer la sonde. Dans la deuxième étape, vous allez ajouter la sonde aux paramètres http backend de la passerelle d’application.

### <a name="create-the-probe"></a><a name="createprobe"></a>Créer la sonde

1. Connectez-vous au [portail Azure](https://portal.azure.com). Si vous ne possédez pas encore de compte, vous pouvez [vous inscrire pour bénéficier d’un essai gratuit d’un mois](https://azure.microsoft.com/free).

2. Dans le volet Favoris du portail Azure, sélectionnez **Toutes les ressources**. Sélectionnez la passerelle d’application dans la page **Toutes les ressources**. Si l’abonnement que vous avez déjà sélectionné comporte déjà plusieurs ressources, vous pouvez entrer partners.contoso.net dans la zone Filtrer par nom… pour accéder facilement à la passerelle d’application.

3. Sélectionnez **Sondes** puis sélectionnez **Ajouter** pour ajouter une sonde.

   ![Ajouter un panneau Sonde contenant toutes les informations][1]

4. Dans le volet **Ajouter une sonde d’intégrité**, fournissez les informations nécessaires à la sonde, puis sélectionnez **OK** une fois que vous avez terminé.

   |**Paramètre** | **Valeur** | **Détails**|
   |---|---|---|
   |**Nom**|customProbe|Cette valeur est le nom convivial donné à la sonde accessible par le biais du portail.|
   |**Protocole**|HTTP ou HTTPS | Protocole utilisé par la sonde d’intégrité. |
   |**Hôte**|Par exemple, contoso.com|Cette valeur est le nom de l’hôte virtuel (différent du nom d’hôte de machine virtuelle) en cours d’exécution sur le serveur d’application. La sonde est envoyée à (protocole)://(nom d’hôte):(port de httpsetting)/urlPath.  Cela s’applique lorsque plusieurs sites sont configurés sur Application Gateway. Si Application Gateway est configurée pour un seul site, entrez « 127.0.0.1 ».|
   |**Choisir un nom d'hôte à partir de paramètres HTTP back-end**|Oui ou Non|Définit l’en-tête *hôte* dans la sonde sur le nom d’hôte de la ressource back-end dans le pool back-end associé au paramètre HTTP auquel cette sonde est liée. Spécialement requis en cas de serveurs back-end multilocataire, comme Azure App Service. [En savoir plus](./configuration-http-settings.md#pick-host-name-from-back-end-address)|
   |**Chemin d’accès**|/ ou n’importe quel chemin valide|Reste de l’URL complète de la sonde personnalisée. Un chemin valide commence par « / ». Pour le chemin par défaut de http:\//contoso.com, utilisez simplement « / » |
   |**Intervalle (secondes)**|30|Fréquence d’exécution de la sonde pour le contrôle d’intégrité. Il n’est pas recommandé de définir la valeur sur moins de 30 secondes.|
   |**Délai d’expiration (secondes)**|30|Intervalle de temps précédant l’expiration de la sonde. Si aucune réponse valide n’est reçue dans le délai imparti, la sonde est marquée comme étant en échec. L’intervalle de délai d’attente doit être suffisamment élevé pour qu’un appel HTTP puisse être envoyé afin de garantir que la page d’intégrité backend est disponible. Notez que la valeur du délai d’attente ne doit pas être supérieure à la valeur « Intervalle » utilisée dans ce paramètre de sonde ou la valeur « Délai d’attente de la demande » dans le paramètre HTTP qui sera associé à cette sonde.|
   |**Seuil de défaillance sur le plan de l’intégrité**|3|Nombre d’échecs consécutifs nécessaires pour marquer l’instance comme étant défaillante. Le seuil peut être défini sur 1 ou plus.|
   |**Utiliser des conditions de correspondance de sonde**|Oui ou Non|Par défaut, une réponse HTTP(S) avec un code d’état compris entre 200 et 399 est considérée comme intègre. Vous pouvez modifier la plage acceptable du code de réponse back-end ou du corps de la réponse back-end. [En savoir plus](./application-gateway-probe-overview.md#probe-matching)|

   > [!IMPORTANT]
   > Le nom d’hôte n’est pas identique au nom du serveur. Cette valeur est le nom de l’hôte virtuel en cours d’exécution sur le serveur d’application. La sonde est envoyée à \<protocol\>://\<hostName\>:\<port from http settings\>/\<urlPath\>

### <a name="add-probe-to-the-gateway"></a>Ajouter une sonde à la passerelle

Maintenant que la sonde a été créée, il est temps de l’ajouter à la passerelle. Les paramètres de la sonde sont définis sur les paramètres http principaux de la passerelle d’application.

1. Cliquez sur **Paramètres HTTP** dans la passerelle d’application pour afficher le panneau de configuration, puis cliquez sur les paramètres http backend actuels répertoriés dans la fenêtre.

   ![fenêtre de paramètres https][2]

2. Dans la page des paramètres **appGatewayBackEndHttpSettings**, cochez la case **Utiliser la sonde personnalisée**, puis choisissez la sonde créée à la section [Créer la sonde](#createprobe) dans la liste déroulante **Sonde personnalisée**.
   Quand vous avez terminé, cliquez sur **Enregistrer** pour appliquer les paramètres.

## <a name="next-steps"></a>Étapes suivantes

Affichez l’intégrité des ressources back-end comme déterminé par la sonde à l’aide de la [vue d’intégrité du back-end](./application-gateway-diagnostics.md#back-end-health).

[1]: ./media/application-gateway-create-probe-portal/figure1.png
[2]: ./media/application-gateway-create-probe-portal/figure2.png
[4]: ./media/application-gateway-create-probe-portal/figure4.png
[5]: ./media/application-gateway-create-probe-portal/figure5.png
[6]: ./media/application-gateway-create-probe-portal/figure6.png
