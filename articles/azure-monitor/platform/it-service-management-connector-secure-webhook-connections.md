---
title: Connecteur de gestion des services informatiques – Exportation sécurisée dans Azure Monitor
description: Cet article vous montre comment connecter vos produits/services ITSM à l’aide de l’exportation sécurisée dans Azure Monitor, afin de superviser et de gérer de manière centralisée les éléments de travail ITSM.
ms.subservice: logs
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 09/08/2020
ms.openlocfilehash: 1cd8041f801a418f67d26461c5f4e9ebff7e5c30
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507300"
---
# <a name="connect-azure-to-itsm-tools-by-using-secure-export"></a>Connecter Azure aux outils ITSM en utilisant l’exportation sécurisée

Cet article explique comment configurer la connexion entre votre service ou produit Gestion des services informatiques (ITSM) à l’aide de l’exportation sécurisée.

L’exportation sécurisée est une version actualisée de [Connecteur de gestion des services informatiques (connecteur ITSMC)](./itsmc-overview.md). Les deux versions vous permettent de créer des éléments de travail dans un outil ITSM lorsque Azure Monitor envoie des alertes. La fonctionnalité comprend des alertes de métriques, des alertes de journaux et des alertes du journal d’activité.

ITSMC utilise les informations d’identification de nom d’utilisateur et de mot de passe. L’exportation sécurisée dispose d’une authentification renforcée, car elle utilise Azure Active Directory (Azure AD). Azure AD est le service Microsoft basé sur le cloud qui gère les identités et les accès. Il permet aux utilisateurs de se connecter et d’accéder aux ressources internes ou externes. L’utilisation d’Azure AD avec ITSM donne la possibilité d’identifier les alertes Azure (par l’intermédiaire de l’ID d’application Azure AD) qui ont été envoyées au système externe.

> [!NOTE]
> La capacité de connecter des outils Azure à ITSM en utilisant l’exportation sécurisée est en préversion.

## <a name="secure-export-architecture"></a>Architecture de l’exportation sécurisée

L’architecture de l’exportation sécurisée présente les nouvelles fonctionnalités suivantes :

* **Nouveau groupe d’actions** : les alertes sont envoyées à l’outil ITSM via le groupe d’actions Webhook sécurisé, à la place du groupe d’actions ITSM utilisé par le connecteur ITSMC.
* **Authentification Azure AD** : l’authentification s’effectue via Azure AD, à la place des informations d’identification de nom d’utilisateur/mot de passe.

## <a name="secure-export-data-flow"></a>Flux de données de l’exportation sécurisée

Les étapes du flux de données de l’exportation sécurisée sont les suivantes :

1. Azure Monitor envoie une alerte qui est configurée pour utiliser l’exportation sécurisée.
2. La charge utile de l’alerte est envoyée par une action Webhook sécurisé à l’outil ITSM.
3. L’application ITSM vérifie avec Azure AD si l’alerte est autorisée à entrer dans l’outil ITSM.
4. Si l’alerte est autorisée, l’application :
   
   1. Crée un élément de travail (par exemple, un incident) dans l’outil ITSM.
   2. Lie l’ID de l’élément de configuration (CI) à la base de données de gestion client (CMDB).

![Diagramme montrant comment l’outil ITSM communique avec Azure AD, les alertes Azure et un groupe d’actions.](media/it-service-management-connector-secure-webhook-connections/secure-export-diagram.png)

## <a name="benefits-of-secure-export"></a>Avantages de l’exportation sécurisée

Voici les avantages principaux de l’intégration :

* **Meilleure authentification** : Azure AD fournit une authentification plus sécurisée, sans les délais d’attente qui se produisent couramment dans le connecteur ITSMC.
* **Alertes résolues dans l’outil ITSM** : les alertes de métriques implémentent les états « déclenché » et « résolu ». Lorsque la condition est remplie, l’état de l’alerte est « déclenché ». Lorsque la condition n’est plus remplie, l’état de l’alerte est « résolu ». Dans le connecteur ITSMC, les alertes ne peuvent pas être résolues automatiquement. Avec l’exportation sécurisée, l’état résolu est transmis à l’outil ITSM, et donc mis à jour automatiquement.
* **[Schéma d’alerte courant](./alerts-common-schema.md)**  : dans le connecteur ITSMC, le schéma de la charge utile de l’alerte diffère selon le type d’alerte. Dans l’exportation sécurisée, il existe un schéma commun pour tous les types d’alerte. Ce schéma commun contient l’élément de configuration pour tous les types d’alerte. Tous les types d’alerte peuvent lier leur élément de configuration à la base de données de gestion client.

Commencez par utiliser l’outil Connecteur ITSM en suivant ces étapes :

1. Inscrire votre application auprès d’Azure AD.
2. Créer un groupe d’actions Webhook sécurisé
3. Configurer votre environnement partenaire 

Secure Export prend en charge les connexions avec les outils ITSM suivants :
* [ServiceNow](#connect-servicenow-to-azure-monitor)
* [BMC Helix](#connect-bmc-helix-to-azure-monitor)

## <a name="register-with-azure-active-directory"></a>S’inscrire auprès d’Azure Active Directory

Pour inscrire l’application auprès d’Azure AD, suivez ces étapes :

1. Suivez la procédure décrite dans [Inscrire une application auprès de la plateforme d’identités Microsoft](../../active-directory/develop/quickstart-register-app.md).
2. Dans Azure AD, sélectionnez **Exposer l’application**.
3. Sélectionnez **Définir** pour l’**URI d’ID d’application**.

   [![Capture d’écran de l’option permettant de définir l’URI de l’ID d’application.](media/it-service-management-connector-secure-webhook-connections/azure-ad.png)](media/it-service-management-connector-secure-webhook-connections/azure-ad-expand.png#lightbox)
4. Sélectionnez **Enregistrer**.

## <a name="create-a-secure-webhook-action-group"></a>Créer un groupe d’actions Webhook sécurisé

Une fois votre application inscrite auprès d’Azure AD, vous pouvez créer des éléments de travail dans votre outil ITSM, en fonction des alertes Azure et au moyen de l’action Webhook sécurisé dans des groupes d’actions.

Les groupes d’actions offrent une méthode modulaire et réutilisable de déclenchement d’actions pour les alertes Azure. Vous pouvez utiliser des groupes d’actions avec des alertes de métriques, des alertes de journal d’activité et des alertes Azure Log Analytics dans le portail Azure.
Pour en savoir plus sur les groupes d’actions, consultez [Créer et gérer des groupes d’actions dans le portail Azure](./action-groups.md).

Pour ajouter un Webhook à une action, suivez les instructions ci-dessous pour le Webhook sécurisé :

1. Dans le [portail Azure](https://portal.azure.com/), recherchez et sélectionnez **Monitor** Le volet **Moniteur** consolide tous vos paramètres et données de supervision dans une même vue.
2. Sélectionnez **Alertes** > **Gérer les actions**.
3. Sélectionnez [Ajouter un groupe d’actions](./action-groups.md#create-an-action-group-by-using-the-azure-portal) et renseignez les champs.
4. Entrez un nom dans la boîte **Nom du groupe d’actions** et entrez un nom dans la boîte **Nom court**. Le nom court est utilisé à la place du nom complet du groupe d’actions lorsque les notifications sont envoyées à l’aide de ce groupe.
5. Sélectionnez **Webhook sécurisé**.
6. Sélectionnez les informations suivantes :
   1. Sélectionnez l’ID d’objet de l’instance Azure Active Directory que vous avez inscrite.
   2. Pour l’URI, collez l’URL du webhook que vous avez copiée dans l’[environnement de l’outil ITSM](#configure-the-itsm-tool-environment).
   3. Définissez **Activer le schéma d’alerte commun** sur **Oui**. 

   L’illustration suivante montre la configuration d’un exemple d’action Webhook sécurisé :

   ![Capture d’écran illustrant une action Webhook sécurisé.](media/it-service-management-connector-secure-webhook-connections/secure-webhook.png)

## <a name="configure-the-itsm-tool-environment"></a>Configurer l’environnement de l’outil ITSM

La configuration comporte 2 étapes :
1. Obtenir l’URI pour la définition d’exportation sécurisée.
2. Définitions selon le flux de l’outil ITSM.


### <a name="connect-servicenow-to-azure-monitor"></a>Connecter ServiceNow à Azure Monitor

Les sections suivantes fournissent des détails sur la connexion de votre produit ServiceNow et l’exportation sécurisée dans Azure.

### <a name="prerequisites"></a>Prérequis

Assurez-vous d’avoir respecté les prérequis suivants :

* Azure AD est inscrit.
* Vous avez la version prise en charge de la Gestion des événements ServiceNow – ITOM (version Orlando ou ultérieure).

### <a name="configure-the-servicenow-connection"></a>Configurer la connexion ServiceNow

1. Utilisez le lien https://(nom de l’instance).service-now.com/api/sn_em_connector/em/inbound_event?source=azuremonitor afin d’obtenir l’URI pour la définition d’exportation sécurisée.

2. Suivez les instructions correspondant à la version :
   * [Paris](https://docs.servicenow.com/bundle/paris-it-operations-management/page/product/event-management/task/azure-events-authentication.html)
   * [Orlando](https://docs.servicenow.com/bundle/orlando-it-operations-management/page/product/event-management/task/azure-events-authentication.html)
   * [New York](https://docs.servicenow.com/bundle/newyork-it-operations-management/page/product/event-management/task/azure-events-authentication.html)

### <a name="connect-bmc-helix-to-azure-monitor"></a>Connecter BMC Helix à Azure Monitor

Les sections suivantes détaillent la connexion de votre produit BMC Helix à l’exportation sécurisée dans Azure.

### <a name="prerequisites"></a>Prérequis

Assurez-vous d’avoir respecté les prérequis suivants :

* Azure AD est inscrit.
* Vous disposez de la version prise en charge de BMC Helix Multi-Cloud Service Management (version 19.08 ou ultérieure).

### <a name="configure-the-bmc-helix-connection"></a>Configurer la connexion BMC Helix

1. Utilisez la procédure suivante dans l’environnement BMC Helix afin d’obtenir l’URI pour l’exportation sécurisée :

   1. Connectez-vous à Integration Studio.
   2. Recherchez le flux **Create Incident from Azure Alerts** (Créer un incident à partir des alertes Azure).
   3. Copiez l’URL du webhook.
   
   ![Capture d’écran de l’URL du Webhook dans Integration Studio.](media/it-service-management-connector-secure-webhook-connections/bmc-url.png)
   
2. Suivez les instructions correspondant à la version :
   * [Activation de l’intégration prédéfinie à Azure Monitor pour la version 20.02](https://docs.bmc.com/docs/multicloud/enabling-prebuilt-integration-with-azure-monitor-879728195.html)
   * [Activation de l’intégration prédéfinie à Azure Monitor pour la version 19.11](https://docs.bmc.com/docs/multicloudprevious/enabling-prebuilt-integration-with-azure-monitor-904157623.html)

3. Dans la partie de configuration de la connexion dans BMC Helix, accédez à votre instance BMC d’intégration et suivez ces instructions :

   1. Sélectionnez **Catalog**.
   2. Sélectionnez **Azure alerts**.
   3. Sélectionnez **connectors**.
   4. Sélectionnez **configuration**.
   5. Sélectionnez la configuration **add new connection**.
   6. Renseignez les informations suivantes dans la section de configuration :
      - **Name** : Indiquez ce qui vous convient.
      - **Authorization type**: **NONE**
      - **Description** : Indiquez ce qui vous convient.
      - **Site**: **Cloud**
      - **Number of instances**: **2**, la valeur par défaut.
      - **Check**: Sélectionné par défaut pour activer l’utilisation.
      - L’ID du locataire Azure et l’ID de l’application Azure sont extraits de l’application que vous avez définie précédemment.

![Capture d’écran illustrant la configuration BMC.](media/it-service-management-connector-secure-webhook-connections/bmc-configuration.png)




## <a name="next-steps"></a>Étapes suivantes

* [Créer des éléments de travail ITSM à partir des alertes Azure](./itsmc-overview.md)
