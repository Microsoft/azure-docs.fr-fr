---
title: Restrictions d’accès dans Azure App Service
description: Apprenez à sécuriser votre application dans Azure App Service en configurant des restrictions d’accès.
author: ccompy
ms.assetid: 3be1f4bd-8a81-4565-8a56-528c037b24bd
ms.topic: article
ms.date: 06/06/2019
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: e1549dda367105db34272eab8a90c1760dd5bb5c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010178"
---
# <a name="set-up-azure-app-service-access-restrictions"></a>Configurer des restrictions d’accès dans Azure App Service

En configurant des restrictions d’accès, vous pouvez définir une liste verte/d’exclusion classée par ordre de priorité qui contrôle l’accès réseau à votre application. La liste peut inclure des adresses IP ou des sous-réseaux de Réseau virtuel Microsoft Azure. Lorsqu’il y a une ou plusieurs entrées, une règle implicite *Tout refuser* se trouve à la fin de la liste.

Les restrictions d’accès fonctionnent avec toutes les charges de travail hébergées dans Azure App Service. Les charges de travail peuvent inclure des applications web, des applications API, des applications Linux, des applications de conteneur Linux et des fonctions.

Lorsqu’une requête est envoyée à votre application, l’adresse dont provient la demande est vérifiée par rapport aux règles d’adresses IP de votre liste de restriction des accès. Si l’adresse dont provient la demande se trouve dans un sous-réseau configuré avec des points de terminaison de service sur Microsoft.Web, le sous-réseau source est comparé aux règles de réseau virtuel de votre liste des restrictions d’accès. Si l’adresse n’est pas autorisée à y accéder selon les règles définies dans la liste, le service répond avec le code d’état [HTTP 403](https://en.wikipedia.org/wiki/HTTP_403).

La fonctionnalité de restriction des accès est implémentée dans les rôles frontaux App Service, qui sont en amont des hôtes de travail où votre code s’exécute. C’est pourquoi les restrictions d’accès sont comparables à des listes de contrôle d’accès (ACL) réseau.

La possibilité de restreindre l’accès à votre application web à partir d’un réseau virtuel Azure est activée par des [points de terminaison de service][serviceendpoints]. Avec les points de terminaison de service, vous pouvez restreindre l’accès à un service multilocataire à partir de sous-réseaux sélectionnés. Il est impossible de limiter le trafic vers les applications hébergées dans un environnement App Service Environment. Si vous êtes dans un environnement App Service Environment, vous pouvez contrôler l’accès à votre application en appliquant des règles d’adresses IP.

> [!NOTE]
> Les points de terminaison de service doivent être activés côté mise en réseau et pour le service Azure avec lequel il sont activés. Pour obtenir la liste des services Azure qui prennent en charge les points de terminaison de service, consultez [Points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md).
>

![Diagramme du flux des restrictions d’accès.](media/app-service-ip-restrictions/access-restrictions-flow.png)

## <a name="add-or-edit-access-restriction-rules-in-the-portal"></a>Ajouter ou modifier des règles de restriction d’accès dans le portail

Pour ajouter une règle de restriction d’accès à votre application, procédez comme suit :

1. Connectez-vous au portail Azure.

1. Dans le volet gauche, sélectionnez **Mise en réseau**.

1. Dans le volet **Mise en réseau**, sous **Restrictions d’accès**, sélectionnez **Configurer des restrictions d’accès**.

   ![Capture d’écran du volet Options réseau d’App Service dans le portail Azure.](media/app-service-ip-restrictions/access-restrictions.png)  

1. Sur la page **Restrictions d'accès**, vous pouvez consulter la liste des règles de restriction d’accès définies pour votre application.

   ![Capture d’écran de la page Restrictions d’accès dans le portail Azure montrant la liste des règles de restriction d’accès définies pour l’application sélectionnée.](media/app-service-ip-restrictions/access-restrictions-browse.png)

   La liste affiche toutes les restrictions actuelles qui sont appliquées à l’application. Si une restriction de réseau virtuel s’applique à votre application, le tableau indique si des points de terminaison de service sont activés pour Microsoft.Web. Si aucune restriction n’est définie sur votre application, l’application est accessible de partout.  

### <a name="add-an-access-restriction-rule"></a>Ajouter une règle de restriction d’accès

Pour ajouter une règle de restriction d’accès à votre application, dans le volet **Restrictions d’accès**, sélectionnez **Ajouter une règle**. Une fois qu’une règle a été ajoutée, elle est appliquée immédiatement. 

Les règles sont appliquées par ordre de priorité, en commençant par le chiffre le moins élevé dans la colonne **Priorité**. Une règle implicite *Tout refuser* s’applique dès que vous ajoutez une règle.

Dans le volet **Ajouter une restriction d’adresse IP**, lorsque vous créez une règle, procédez comme suit :

1. Sous **Action**, sélectionnez **Autoriser** ou **Refuser**.  

   ![Capture d’écran du volet « Ajouter une restriction d’adresse IP ».](media/app-service-ip-restrictions/access-restrictions-ip-add.png)
   
1. En option, vous pouvez entrez un nom et une description pour la règle.  
1. Dans la liste déroulante **Type**, sélectionnez le type de règle.  
1. Dans la zone **Priorité**, entrez une valeur de priorité.  
1. Dans les listes déroulantes **Abonnement**, **Réseau virtuel** et **Sous-réseau**, sélectionnez les éléments auxquels vous souhaitez restreindre l’accès.  

### <a name="set-an-ip-address-based-rule"></a>Définir une règle basée sur une adresse IP

Suivez la procédure décrite dans la section précédente, mais avec la variation suivante :
* Pour l’étape 3, dans la liste déroulante **Type**, sélectionnez **IPv4** ou **IPv6**. 

Spécifiez l’adresse IP dans la notation de routage CIDR (Classless InterDomain Routing) pour les adresses IPv4 et IPv6. Pour spécifier une adresse, vous pouvez utiliser le format *1.2.3.4/32*, où les quatre premiers octets représentent votre adresse IP et */32* correspond au masque. La notation CIDR IPv4 est 0.0.0.0/0 pour toutes les adresses. Pour plus d’informations sur la notation de routage CIDR, consultez [Routage CIDR (Classless InterDomain Routing)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). 

## <a name="use-service-endpoints"></a>Utiliser des points de terminaison de service

Grâce aux points de terminaison de service, vous pouvez restreindre l’accès à une sélection de sous-réseaux de réseau virtuel Azure. Pour restreindre l’accès à un sous-réseau spécifique, créez une règle de restriction avec le type **Réseau virtuel**. Vous pouvez ensuite sélectionner l’abonnement, le réseau virtuel et le sous-réseau auxquels vous souhaitez accorder ou refuser l’accès. 

Si les points de terminaison de service ne sont pas encore activés avec Microsoft.Web pour le sous-réseau que vous avez sélectionné, ils seront automatiquement activés, sauf si vous cochez la case **Ignorer les points de terminaison de service Microsoft.Web manquants**. Le scénario dans lequel vous voudriez activer des points de terminaison de service sur l’application, mais pas sur le sous-réseau, dépend principalement du fait que vous disposiez ou non des autorisations pour les activer sur le sous-réseau. 

Si vous avez besoin que quelqu’un d’autre active les points de terminaison de service sur le sous-réseau, cochez la case **Ignorer les points de terminaison de service Microsoft.Web manquants**. Votre application sera configurée pour les points de terminaison de service en prévision de leur activation ultérieure sur le sous-réseau. 

![Capture d’écran du volet « Ajouter une restriction d’adresse IP » avec le type Réseau virtuel sélectionné.](media/app-service-ip-restrictions/access-restrictions-vnet-add.png)

Vous ne pouvez pas utiliser des points de terminaison de service pour restreindre l’accès aux applications qui s’exécutent dans un environnement App Service Environment. Si votre application se trouve dans un environnement App Service Environment, vous pouvez contrôler l’accès à celle-ci avec des règles d’accès IP. 

Avec les points de terminaison de service, vous pouvez configurer votre application avec les passerelles d’application ou d’autres appareils de pare-feu d'applications web (WAF). Vous pouvez également configurer des applications à plusieurs niveaux avec des serveurs principaux sécurisés. Pour plus d’informations, consultez [Fonctionnalités de mise en réseau et App Service](networking-features.md) et [Intégration d’Application Gateway avec les points de terminaison de service](networking/app-gateway-with-service-endpoints.md).

> [!NOTE]
> - Les points de terminaison de service ne sont actuellement pas pris en charge pour les applications web qui utilisent une adresse IP virtuelle IP SSL.
> - Il existe une limite de 512 lignes de restrictions d’adresse IP ou de point de terminaison de service. Si vous avez besoin de plus de 512 lignes de restrictions, nous vous suggérons d’envisager d’installer un produit de sécurité autonome, tel qu’Azure Front Door, Azure App Gateway ou un pare-feu d'applications web (WAF).
>

## <a name="manage-access-restriction-rules"></a>Gérer des règles de restriction d’accès

Vous pouvez modifier ou supprimer une règle de restriction d’accès existante.

### <a name="edit-a-rule"></a>Modifier une règle

1. Pour commencer à modifier une règle de restriction d’accès existante, sur la page **Restrictions d’accès**, double-cliquez sur la règle que vous souhaitez modifier.

1. Dans le volet **Modifier la restriction d’adresses IP**, effectuez vos modifications, puis sélectionnez **Mettre à jour la règle**. Les modifications sont appliquées immédiatement, y compris les changements de priorité.

   ![Capture d’écran du volet « Modifier la restriction d’adresses IP » dans le portail Azure montrant les champs d’une règle de restriction d’accès existante.](media/app-service-ip-restrictions/access-restrictions-ip-edit.png)

   > [!NOTE]
   > Lorsque vous modifiez une règle, vous ne pouvez pas basculer entre une règle d’adresse IP et une règle de réseau virtuel. 

   ![Capture d’écran du volet « Modifier la restriction d’adresses IP » dans le portail Azure montrant les paramètres d’une règle de réseau virtuel.](media/app-service-ip-restrictions/access-restrictions-vnet-edit.png)

### <a name="delete-a-rule"></a>Supprimer une règle

Pour supprimer une règle, sur la page **Restrictions d’accès**, sélectionnez les points de suspension ( **...** ) en regard de la règle à supprimer, puis sélectionnez **Supprimer**.

![Capture d’écran de la page « Restrictions d’accès » montrant les points de suspension « Supprimer » en regard de la règle de restriction d’accès à supprimer.](media/app-service-ip-restrictions/access-restrictions-delete.png)

## <a name="block-a-single-ip-address"></a>Bloquer une adresse IP unique

Lorsque vous ajoutez votre première règle de restriction IP, le service ajoute une règle *Tout refuser* ayant la priorité 2147483647. Dans la pratique, la règle explicite *Tout refuser* est la dernière règle exécutée et bloque l’accès à toute adresse IP n’étant pas autorisée de manière explicite par une règle *Autoriser*.

Si vous souhaitez bloquer explicitement une seule adresse IP ou un bloc d’adresses IP, mais autoriser l’accès à tout autre élément, ajoutez une règle explicite *Tout autoriser*.

![Capture d’écran de la page « Restrictions d’accès » dans le portail Azure montrant une seule adresse IP bloquée.](media/app-service-ip-restrictions/block-single-address.png)

## <a name="restrict-access-to-an-scm-site"></a>Limiter l'accès à un site du Gestionnaire de contrôle des services (SCM) 

Outre la possibilité de contrôler l’accès à votre application, vous pouvez également restreindre l’accès au site SCM utilisé par votre application. Le site SCM est le point de terminaison de déploiement web, ainsi que la console Kudu. Vous pouvez affecter des restrictions d’accès au site SCM à partir de l’application séparément ou utiliser le même ensemble de restrictions pour l’application et le site SCM. Lorsque vous activez la case à cocher **Restrictions identiques à \<app name>** , tout reste vide. Si vous décochez la case, les paramètres de votre site SCM sont réappliqués. 

![Capture d’écran de la page « Restrictions d’accès » dans le portail Azure indiquant qu’aucune restriction d’accès n’est définie pour le site SCM ou l’application.](media/app-service-ip-restrictions/access-restrictions-scm-browse.png)

## <a name="manage-access-restriction-rules-programatically"></a>Gérer des règles de restriction d’accès par programmation

Vous pouvez ajouter des restrictions d’accès par programmation en procédant de l’une des façons suivantes : 

* Utilisez [Azure CLI](/cli/azure/webapp/config/access-restriction?view=azure-cli-latest&preserve-view=true). Par exemple :
   
  ```azurecli-interactive
  az webapp config access-restriction add --resource-group ResourceGroup --name AppName \
  --rule-name 'IP example rule' --action Allow --ip-address 122.133.144.0/24 --priority 100
  ```

* Utilisez [Azure PowerShell](/powershell/module/Az.Websites/Add-AzWebAppAccessRestrictionRule?view=azps-3.1.0&preserve-view=true). Par exemple :


  ```azurepowershell-interactive
  Add-AzWebAppAccessRestrictionRule -ResourceGroupName "ResourceGroup" -WebAppName "AppName"
      -Name "Ip example rule" -Priority 100 -Action Allow -IpAddress 122.133.144.0/24
  ```

Vous pouvez également définir les valeurs manuellement en procédant de l’une des façons suivantes :

* Utilisez une opération PUT de l’[API REST Azure](/rest/api/azure/) sur la configuration de l’application dans Azure Resource Manager. Dans Azure Resource Manager, ces informations se trouvent à l’emplacement suivant :

  management.azure.com/subscriptions/**subscription ID**/resourceGroups/**resource groups**/providers/Microsoft.Web/sites/**web app name**/config/web?api-version=2018-02-01

* Utilisez un modèle ARM. Par exemple, vous pouvez utiliser resources.azure.com et modifier le bloc ipSecurityRestrictions pour ajouter le code JSON nécessaire.

  La syntaxe JSON de l’exemple précédent est la suivante :

  ```json
  {
    "properties": {
      "ipSecurityRestrictions": [
        {
          "ipAddress": "122.133.144.0/24",
          "action": "Allow",
          "priority": 100,
          "name": "IP example rule"
        }
      ]
    }
  }
  ```

## <a name="set-up-azure-functions-access-restrictions"></a>Configurer des restrictions d’accès Azure Functions

Les restrictions d’accès sont également disponibles pour les applications de fonctions, avec les mêmes fonctionnalités que les plans App Service. Lorsque vous activez des restrictions d’accès, vous désactivez également l’éditeur de code du portail Azure pour les adresses IP non autorisées.

## <a name="next-steps"></a>Étapes suivantes
[Restrictions d’accès pour Azure Functions](../azure-functions/functions-networking-options.md#inbound-access-restrictions)  
[Intégration d’Application Gateway par des points de terminaison de service](networking/app-gateway-with-service-endpoints.md)

<!--Links-->
[serviceendpoints]: ../virtual-network/virtual-network-service-endpoints-overview.md
