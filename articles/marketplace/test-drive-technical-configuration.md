---
title: Version d'évaluation de la configuration technique, place de marché commerciale Microsoft
description: Découvrez-en plus sur les versions d’évaluation. Les versions d’évaluation permettent aux nouveaux clients de tester votre offre avant de valider l’achat.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
ms.date: 08/13/2019
author: trkeya
ms.author: trkeya
ms.openlocfilehash: 553eba3898a5ea42e5d478603e35e82c68abcab1
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462893"
---
# <a name="test-drive-technical-configuration"></a>Configuration technique de la version d’évaluation

L’option Version d’évaluation de la place de marché commerciale Microsoft vous permet de configurer une visite guidée et interactive des principales fonctionnalités de votre produit. Avec une version d’évaluation, les nouveaux clients peuvent tester votre offre avant de valider l’achat. Pour plus d’informations, consultez la rubrique [Qu’est-ce qu’une version d’évaluation ?](what-is-test-drive.md)

Si vous ne souhaitez plus fournir une version d’évaluation pour votre offre, retournez à la page **Configuration de l’offre** et décochez **Activer la version d’évaluation**. Tous les types d’offre ne disposent pas d’une version d'évaluation.

## <a name="azure-resource-manager-test-drive"></a>Version d’évaluation d’Azure Resource Manager

Il s’agit de la seule option de version d’évaluation pour les machines virtuelles ou Azure App et elle nécessite une configuration assez détaillée. Lisez les sections ci-dessous pour consulter plus de [détails sur l’abonnement au déploiement](#deployment-subscription-details) et [les listes des versions d'évaluation](#test-drive-listings), puis passez à la rubrique consacrée à la [configuration de la version d'évaluation d’Azure Resource Manager](azure-resource-manager-test-drive.md).

## <a name="hosted-test-drive"></a>Version d’évaluation hébergée

Microsoft peut supprimer la complexité de la configuration d’une version d’évaluation en hébergeant et en assurant l’approvisionnement et le déprovisionnement du service. La configuration de ce type de version d’évaluation est la même, que la version d’évaluation cible un public Dynamics 365 Customer Engagement ou Dynamics 365 Operations.

- **Nombre maximal de versions d’évaluation simultanées** (obligatoire) : définissez le nombre maximum de clients pouvant utiliser votre version d’évaluation simultanément. Chaque utilisateur simultané consommera une licence Dynamics 365 tant que la version d’évaluation est active. Vérifiez donc que vous avez assez de licences pour prendre en charge la limite maximale définie. La valeur recommandée se situe entre 3 et 5.

- **Durée de la version d’évaluation** (obligatoire) : entrez le nombre d’heures durant lesquelles la version d’évaluation restera active pour chaque client. Passée cette période, la session prendra fin et ne consommera plus aucune de vos licences. La valeur recommandée est de 2 à 24 heures, selon la complexité de votre offre. Cette durée ne peut être définie qu’en heures pleines (par exemple, « 2 heures » est valide, mais pas « 1,5 heure »). L’utilisateur peut demander une nouvelle session s’il n’a pas eu assez de temps et s’il souhaite de nouveau accéder à la version d’évaluation.

- **URL de l’instance** (obligatoire) : URL où le client commencera sa version d’évaluation. Il s’agit généralement de l’URL de votre instance Dynamics 365 exécutant votre application et où les exemples de données sont installés (par exemple `https://testdrive.crm.dynamics.com`).

- **URL de l’API web de l’instance**(obligatoire) – Récupérez l’URL de l’API web pour votre instance Dynamics 365 en vous connectant à votre compte Microsoft 365 et en accédant à **Paramètres** > **Personnalisation** > **Ressources développeur** > **API web de l’instance (URL racine du service)**, puis copiez l’URL trouvée ici (par exemple `https://testdrive.crm.dynamics.com/api/data/v9.0`).

- **Nom de rôle** (obligatoire) : renseignez le nom du rôle de sécurité que vous avez défini dans votre version d’évaluation personnalisée de Dynamics 365, qui sera attribué à l’utilisateur lors de la version d’évaluation (par exemple, test-drive-role).

Pour obtenir de l’aide pour la configuration de votre environnement Dynamics 365 pour une version d'évaluation et autoriser AppSource à approvisionner et à déprovisionner les utilisateurs de la version d'évaluation dans votre locataire, suivez [ces instructions](./test-drive-azure-subscription-setup.md).

Pour obtenir des instructions pas à pas sur l’affichage et la configuration de votre version d’évaluation hébergée, consultez la page [Configuration détaillée pour une version d'évaluation hébergée](./test-drive-hosted-detailed-config.md).

## <a name="logic-app-test-drive"></a>Version d’évaluation d’une application logique

Ce type de version d'évaluation n’est pas hébergé par Microsoft. Utilisez-le pour vous connecter à une offre Dynamics 365 ou à une autre ressource personnalisée, qui englobe une variété d’architectures de solution complexes. Pour plus d’informations sur la configuration de versions d’évaluation d’applications logiques, consultez [Operations](https://github.com/Microsoft/AppSource/blob/master/Setup-your-Azure-subscription-for-Dynamics365-Operations-Test-Drives.md) et [Customer Engagement](https://github.com/Microsoft/AppSource/wiki/Setting-up-Test-Drives-for-Dynamics-365-app) sur GitHub.

- **Région** (obligatoire, liste déroulante à sélection unique) : à l’heure actuelle, vous pouvez mettre votre version d’évaluation à disposition dans 26 régions Azure. Les ressources pour votre application logique seront déployées dans la région que vous sélectionnez. Si votre application logique compte des ressources personnalisées stockées dans une région spécifique, assurez-vous que celle-ci est sélectionnée ici. La meilleure façon de procéder consiste à déployer entièrement votre application logique localement vers votre abonnement Azure dans le portail et à vérifier qu’elle fonctionne correctement avant de la sélectionner.

- **Nombre maximal de versions d’évaluation simultanées** (obligatoire) : définissez le nombre maximum de clients pouvant utiliser votre version d’évaluation simultanément. Ces versions d’évaluation sont déjà déployées, permettant ainsi aux clients d’y accéder instantanément sans attendre un déploiement.

- **Durée de la version d’évaluation** (obligatoire) : entrez la durée d’activité de la version d’évaluation, en nombre d’heures. La version d’évaluation se termine automatiquement à la fin de cette période.

- **Nom du groupe de ressources Azure** (obligatoire) : entrez le nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md#resource-groups) dans lequel votre version d’évaluation d’application logique est enregistrée.

- **Nom de l’application logique Azure** (obligatoire) : entrez le nom de l’application logique qui attribue la version d’évaluation à l’utilisateur. Cette application logique doit être enregistrée dans le groupe de ressources Azure ci-dessus.

- **Déprovisionner le nom d’application logique** (obligatoire) : entrez le nom de l’application logique qui déprovisionne la version d’évaluation une fois que le client a terminé. Cette application logique doit être enregistrée dans le groupe de ressources Azure ci-dessus.

## <a name="power-bi-test-drive"></a>Version d’évaluation de Power BI

Les produits souhaitant afficher un visuel interactif Power BI peuvent utiliser un lien hypertexte pour partager un tableau de bord personnalisé et intégré qui sert de version d’évaluation, sans configuration technique supplémentaire. Il vous suffit de charger votre URL Power BI incorporée.

Pour plus d’informations sur la configuration des applications Power BI, consultez [Que sont les applications Power BI ?](/power-bi/service-template-apps-overview)

## <a name="deployment-subscription-details"></a>Détails de l’abonnement de déploiement

Pour permettre à Microsoft de déployer la version d’évaluation en votre nom, créez et fournissez un abonnement Azure distinct et unique (non obligatoire pour les versions d’évaluation Power BI).

- **ID d’abonnement Azure** (obligatoire pour Azure Resource Manager et les applications logiques) – Entrez l’ID d’abonnement pour accorder l’accès aux services de votre compte Azure pour les rapports d’utilisation des ressources et la facturation. Nous vous recommandons d’envisager la [création d’un abonnement Azure distinct](../cost-management-billing/manage/create-subscription.md) à utiliser pour les versions d’évaluation si vous n’en avez pas déjà. Pour trouver vos ID d’abonnement Azure en vous connectant au [portail Azure](https://portal.azure.com/) et en accédant à l’onglet **Abonnements** dans le menu de gauche. Sélectionnez l’onglet pour afficher votre ID d’abonnement (par exemple, « a83645ac-1234-5ab6-6789-1h234g764ghty »).

- **ID de locataire Azure AD** (obligatoire) – Entrez votre [ID de locataire](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) Azure Active Directory (AD). Pour trouver cet ID, connectez-vous au [Portail Azure](https://portal.azure.com/), sélectionnez l’onglet Active Directory dans le menu de gauche, sélectionnez **Propriétés**, puis cherchez l’**ID du répertoire** (par exemple, 50c464d3-4930-494c-963c-1e951d15360e). Vous pouvez aussi chercher l’ID de locataire de votre organisation à partir de l’adresse de votre nom de domaine à l’adresse [https://www.whatismytenantid.com](https://www.whatismytenantid.com).

- **Nom de locataire Azure AD** (obligatoire pour Dynamic 365) : entrez votre nom Azure Active Directory (AD). Pour rechercher ce nom, connectez-vous au [portail Azure](https://portal.azure.com/), et dans le coin supérieur droit, le nom de votre locataire s’affichera sous le nom de votre compte.

- **ID d’application Azure AD** (obligatoire) : entrez votre [ID d’application](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in) Azure Active Directory (AD). Pour trouver cet ID, connectez-vous au [Portail Azure](https://portal.azure.com/), sélectionnez l’onglet Active Directory dans le menu de gauche, sélectionnez **Inscriptions d’applications**, puis cherchez l’**ID d’application** (comme `50c464d3-4930-494c-963c-1e951d15360e`).

- **Clé secrète client Azure AD** (obligatoire) : entrez la [clé secrète client](../active-directory/develop/howto-create-service-principal-portal.md#option-2-create-a-new-application-secret) de votre application Azure AD. Pour trouver cette valeur, connectez-vous au [portail Azure](https://portal.azure.com/). Dans le menu de gauche, sélectionnez l’onglet **Azure Active Directory**, puis **Inscriptions d’applications**, et enfin la version d’évaluation de votre application. Ensuite, sélectionnez **Certificats et secrets** et **Nouvelle clé secrète client**. Entrez une description, sélectionnez **Jamais** sous **Expiration** et choisissez **Ajouter**. Veillez à copier la valeur. Ne quittez pas la page avant de copier la valeur.

## <a name="test-drive-listings"></a>Listes des versions d’évaluation

L’option **Liste des versions d’évaluation** disponible sous l’onglet **Version d’évaluation** dans l’Espace partenaires affiche les langues (et les marchés) disponibles pour votre version d’évaluation. English (États-Unis) est actuellement le seul emplacement disponible. De plus, cette page affiche le statut de l’annonce spécifique à une langue et l’heure et la date à laquelle elle a été ajoutée. Vous devez définir les détails de la version d’évaluation (description, manuel de l’utilisateur, vidéos, etc.) pour chaque langue et marché.

- **Description** (obligatoire) : Décrivez votre version d’évaluation, ce qui sera présenté, les objectifs que l’utilisateur pourra essayer, les fonctionnalités à explorer, et toute autre information qui aidera l’utilisateur à déterminer ou non s’il souhaite faire l’acquisition de votre offre. Jusqu’à 3 000 caractères peuvent être entrés dans ce champ.

- **Accéder aux informations** (obligatoire pour les versions d’évaluation Azure Resource Manager et d’applications logiques) : Expliquez ce qu’un client doit savoir pour accéder à cette version d’évaluation et l’utiliser. Détaillez un scénario d’utilisation de votre offre ainsi que tout ce que le client doit savoir pour accéder aux fonctionnalités disponibles dans la version d’évaluation. Jusqu’à 10 000 caractères peuvent être entrés dans ce champ.

- **Manuel de l’utilisateur** (obligatoire) : Procédure approfondie de l’expérience de votre version d’évaluation. Le manuel de l’utilisateur doit couvrir exactement ce que vous souhaitez que le client accomplisse dans la version d’évaluation et sert de référence pour toutes les questions qu’ils peuvent avoir. Le fichier doit être au format PDF et être nommé (255 caractères maximum) après le chargement.

- **Vidéos : Ajouter des vidéos** (facultatif) : Les vidéos hébergées ailleurs peuvent être référencées ici via un lien et une image miniature (533 x 324 pixels), pour proposer au client un guide à suivre et l’aider à mieux comprendre la version d’évaluation, dont l’utilisation des fonctionnalités de votre offre et la compréhension des scénarios qui présentent les avantages.
  - **Nom** (obligatoire)
  - **URL (YouTube ou Vimeo uniquement)** (obligatoire)
  - **Miniature** (533 x 324 pixels) : cette image doit être au format PNG.

Si vous êtes en train de créer votre version d'évaluation dans l’Espace partenaires, sélectionnez **Enregistrer le brouillon** avant de continuer.

Pour obtenir des instructions pas à pas sur l’affichage et la configuration de votre version d’évaluation hébergée, consultez la page [Configuration détaillée pour une version d'évaluation hébergée](./test-drive-hosted-detailed-config.md).

## <a name="additional-resources"></a>Ressources supplémentaires

## <a name="next-steps"></a>Étapes suivantes

- [Test drive des meilleures pratiques](https://github.com/Azure/AzureTestDrive/wiki/Test-Drive-Best-Practices)
- [Vue d’ensemble](https://assetsprod.microsoft.com/mpn/azure-marketplace-appsource-test-drives.pdf) (PDF ; veillez à désactiver votre bloqueur de fenêtres publicitaires)
- [Mettre à jour une offre existante dans la Place de marché commerciale](partner-center-portal/update-existing-offer.md)