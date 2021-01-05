---
title: 'ML Studio (classique) : Gestion des services web – Azure'
description: Gérez vos services web Machine Learning nouveaux et classiques à l’aide du portail des services web Microsoft Azure Machine Learning. Étant donné que les services web classiques et nouveaux sont basés sur des technologies différentes , les fonctionnalités de gestion diffèrent légèrement pour chacun d’eux.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: previous-ms.author=yahajiza, previous-author=YasinMSFT
ms.date: 02/28/2017
ms.openlocfilehash: 37f10cc8eb3d606d99eb5b3f821a486e9e4ea2fe
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322342"
---
# <a name="manage-a-web-service-using-the-azure-machine-learning-studio-classic-web-services-portal"></a>Gérer un service web à l’aide du portail des services web Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Vous pouvez gérer vos services web Machine Learning nouveaux et classiques à l’aide du portail des services web Microsoft Azure Machine Learning. Étant donné que les services web classiques et nouveaux sont basés sur des technologies différentes , les fonctionnalités de gestion diffèrent légèrement pour chacun d’eux.

Le portail des services web Azure Machine Learning vous permet d’effectuer les opérations suivantes :

* surveiller l’utilisation du service web ;
* configurer la description, mettre à jour les clés du service web (nouveau uniquement), mettre à jour votre clé de compte de stockage (nouveau uniquement), activer l’enregistrement (nouveau uniquement) et activer ou désactiver les exemples de données ;
* supprimer le service web ;
* créer, supprimer ou mettre à jour des plans de facturation (nouveau uniquement) ;
* ajouter et supprimer des points de terminaison (classique uniquement).

>[!NOTE]
>Vous pouvez également gérer les services web classiques dans [Machine Learning Studio (classique)](https://studio.azureml.net) dans l’onglet **Services web**.

## <a name="permissions-to-manage-new-resources-manager-based-web-services"></a>Autorisations pour gérer des services web basés sur le gestionnaire de nouvelles ressources

De nouveaux services web sont déployés en tant que ressources Azure. Par conséquent, vous devez disposer des autorisations appropriées pour déployer et gérer de nouveaux services web.  Pour déployer ou gérer de nouveaux services web, vous devez avoir un rôle de contributeur ou d’administrateur dans l’abonnement dans lequel le service web est déployé. Si vous invitez un autre utilisateur dans un espace de travail Machine Learning, vous devez l’affecter à un rôle de contributeur ou d’administrateur dans l’abonnement avant qu’il ne puisse déployer ou gérer des services web. 

Si l’utilisateur n’a pas les autorisations appropriées pour accéder aux ressources dans le portail Service web Machine Learning, il recevra le message d’erreur suivant s’il tente de déployer un service web :

*Échec du déploiement d’un service web. Ce compte n’a pas de droits d’accès suffisants à l’abonnement Azure qui contient l’espace de travail. Pour déployer un service web sur Azure, le même compte doit être invité à l’espace de travail et disposer de l’accès à l’abonnement Azure qui contient l’espace de travail.*

Pour plus d’informations sur la création d’un espace de travail, consultez [Créer et partager un espace de travail Azure Machine Learning Studio (classique)](create-workspace.md).

Pour plus d’informations sur le paramétrage des autorisations d’accès, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).


## <a name="manage-new-web-services"></a>Gérer de nouveaux services web
Pour gérer vos nouveaux service web :

1. Connectez-vous au portail [Services web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart) à l’aide de votre compte Microsoft Azure (utilisez le compte associé à l’abonnement Azure).
2. Dans le menu, cliquez sur **Services Web**.

La liste des services web déployés pour votre abonnement s’affiche. 

Pour gérer un service web, cliquez sur Services web. À partir de la page Services Web, vous pouvez :

* Cliquer sur le service web pour le gérer.
* Cliquer sur le plan de facturation associé au service web pour le mettre à jour.
* Supprimer un service web.
* Copier un service web et le déployer dans une autre région.

Lorsque vous cliquez sur un service web, la page de démarrage rapide du service web s’ouvre. La page de démarrage rapide du service web comprend deux options de menu pour gérer votre service web :

* **TABLEAU DE BORD** : pour afficher l’utilisation du service web.
* **CONFIGURER** : pour ajouter un texte descriptif, mettre à jour la clé du compte de stockage associé au service web, et activer ou désactiver des exemples de données.

### <a name="monitoring-how-the-web-service-is-being-used"></a>Surveiller l’utilisation du service web
Cliquez sur l’onglet **TABLEAU DE BORD** .

Le tableau de bord vous permet de voir l’utilisation globale de votre service web sur une période donnée. Vous pouvez sélectionner la période à afficher dans le menu déroulant Période situé dans le coin supérieur droit des graphiques d’utilisation. Le tableau de bord affiche les informations suivantes :

* **Requests Over Time** (Requêtes sur une période donnée) affiche un graphique détaillé du nombre de requêtes au cours de la période sélectionnée. Ce graphique vous permet d’identifier les pics d’utilisation.
* **Request-Response Requests** (Demandes de requête-réponse) affiche le nombre total d’appels de requête-réponse que le service a reçu au cours de la période sélectionnée et combien d’entre eux ont échoué.
* **Average Request-Response Compute Time** (Temps de calcul moyen d’une requête-réponse) affiche le temps moyen nécessaire pour exécuter les demandes reçues.
* **Batch Requests** (Requêtes de lots) affiche le nombre total de requêtes de traitement par lots que le service a reçu au cours de la période sélectionnée et combien d’entre elles ont échoué.
* **Average Job Latency** (Latence de travail moyenne) affiche le temps moyen nécessaire pour exécuter les requêtes reçues.
* **Errors** (Erreurs) affiche le nombre agrégé d’erreurs qui se sont produites lors des appels au service web.
* **Services Costs** (Coûts de service) affiche les frais du plan de facturation associé au service.

### <a name="configuring-the-web-service"></a>Configuration du service web
Cliquez sur l’option de menu **CONFIGURER** .

Vous pouvez mettre à jour les propriétés suivantes :

* **Description** pour entrer une description du service web.
* **Titre** pour entrer le titre du service web.
* **Clés** , configurez la rotation de vos clés API principales et secondaires.
* **Clé du compte de stockage** pour mettre à jour la clé du compte de stockage associé aux modifications du service web. 
* **Activer l’exemple de données** pour fournir des exemples de données que vous pouvez utiliser pour tester votre service de demande-réponse. Si vous avez créé le service web dans Machine Learning Studio (classique), les exemples de données proviennent des données utilisées pour effectuer l’apprentissage de votre modèle. Si vous avez créé le service par programme, les données proviennent des exemples de données que vous avez fournis dans le cadre du package JSON.

### <a name="managing-billing-plans"></a>Gestion des plans de facturation
Cliquez sur l’option de menu **Plans** à partir de la page Démarrage rapide des services web. Vous pouvez également cliquer sur le plan associé au service web spécifique pour gérer ce plan.

* **Nouveau** vous permet de créer un nouveau plan.
* **Add/Remove Plan instance** (Ajouter/supprimer une instance de plan) vous permet d’effectuer un scale-out du plan existant pour ajouter de la capacité.
* **Upgrade/DownGrade** (Mettre à niveau/rétrograder) vous permet de mettre à l’échelle un plan existant pour ajouter de la capacité.
* **Supprimer** vous permet de supprimer un plan.

Cliquez sur un plan pour afficher son tableau de bord. Le tableau de bord affiche un instantané de l’utilisation du plan sur une période donnée. Pour sélectionner la période à afficher, cliquez sur le menu déroulant **Période** en haut à droite du tableau de bord. 

Le tableau de bord du plan affiche les informations suivantes :

* **Description du plan** : informations sur les coûts et la capacité associés au plan.
* **Plan Usage** (Utilisation du plan) : nombre de transactions et d’heures de calcul facturées dans le cadre du plan.
* **Services web** : nombre de services web qui utilisent ce plan.
* **Top web Service By Calls** (Principaux services web par appels) : quatre principaux services web qui effectuent des appels facturés dans le cadre du plan.
* **Top web Services by Compute Hrs** (Principaux services web par heures de calcul) : les quatre principaux services web qui utilisent des ressources de calcul facturées dans le cadre du plan.

## <a name="manage-classic-web-services"></a>Gérer des services web classiques
> [!NOTE]
> Les procédures décrites dans cette section ont trait à la gestion de services web classiques via le portail des services web Azure Machine Learning. Pour plus d’informations sur la gestion des services web classiques par le biais de Machine Learning Studio (classique) et le Portail Microsoft Azure, consultez [Gérer un espace de travail Azure Machine Learning Studio (classique)](manage-workspace.md).
> 
> 

Pour gérer vos services web classiques :

1. Connectez-vous au portail [Services web Microsoft Azure Machine Learning](https://services.azureml.net/quickstart) à l’aide de votre compte Microsoft Azure (utilisez le compte associé à l’abonnement Azure).
2. Dans le menu, cliquez sur **Services web classiques**.

Pour gérer un Service web classique, cliquez sur **Services web classiques**. Dans la page Services web classiques, vous pouvez :

* cliquer sur un service web pour afficher les points de terminaison associés ;
* Supprimer un service web.

Lorsque vous gérez un service web classique, vous gérer chacun des points de terminaison séparément. Lorsque vous cliquez sur un service web dans la page Services web, la liste des points de terminaison associés au service s’ouvre. 

Dans la page des points de terminaison du web classique, vous pouvez ajouter et supprimer des points de terminaison. Pour plus d’informations sur l’ajout de points de terminaison, consultez [Création de points de terminaison](create-endpoint.md).

Cliquez sur un des points de terminaison pour ouvrir la page de démarrage rapide du service web. Dans la page de démarrage rapide, deux options de menu permettent de gérer le service web :

* **TABLEAU DE BORD** : pour afficher l’utilisation du service web.
* **CONFIGURER** : pour ajouter un texte descriptif, activer et désactiver la journalisation des erreurs, mettre à jour la clé du compte de stockage associé au service web, et activer et désactiver des exemples de données.

### <a name="monitoring-how-the-web-service-is-being-used"></a>Surveiller l’utilisation du service web
Cliquez sur l’onglet **TABLEAU DE BORD** .

Le tableau de bord vous permet de voir l’utilisation globale de votre service web sur une période donnée. Vous pouvez sélectionner la période à afficher dans le menu déroulant Période situé dans le coin supérieur droit des graphiques d’utilisation. Le tableau de bord affiche les informations suivantes :

* **Requests Over Time** (Requêtes sur une période donnée) affiche un graphique détaillé du nombre de requêtes au cours de la période sélectionnée. Ce graphique vous permet d’identifier les pics d’utilisation.
* **Request-Response Requests** (Demandes de requête-réponse) affiche le nombre total d’appels de requête-réponse que le service a reçu au cours de la période sélectionnée et combien d’entre eux ont échoué.
* **Average Request-Response Compute Time** (Temps de calcul moyen d’une requête-réponse) affiche le temps moyen nécessaire pour exécuter les demandes reçues.
* **Batch Requests** (Requêtes de lots) affiche le nombre total de requêtes de traitement par lots que le service a reçu au cours de la période sélectionnée et combien d’entre elles ont échoué.
* **Average Job Latency** (Latence de travail moyenne) affiche le temps moyen nécessaire pour exécuter les requêtes reçues.
* **Errors** (Erreurs) affiche le nombre agrégé d’erreurs qui se sont produites lors des appels au service web.
* **Services Costs** (Coûts de service) affiche les frais du plan de facturation associé au service.

### <a name="configuring-the-web-service"></a>Configuration du service web
Cliquez sur l’option de menu **CONFIGURER** .

Vous pouvez mettre à jour les propriétés suivantes :

* **Description** pour entrer une description du service web. Le champ Description est obligatoire.
* **Journalisation** pour activer ou désactiver la journalisation des erreurs sur le point de terminaison. Pour plus d’informations sur la journalisation, voir [Activation de la journalisation pour les services web Machine Learning](web-services-logging.md).
* **Activer l’exemple de données** pour fournir des exemples de données que vous pouvez utiliser pour tester votre service de demande-réponse. Si vous avez créé le service web dans Machine Learning Studio (classique), les exemples de données proviennent des données utilisées pour effectuer l’apprentissage de votre modèle. Si vous avez créé le service par programme, les données proviennent des exemples de données que vous avez fournis dans le cadre du package JSON.