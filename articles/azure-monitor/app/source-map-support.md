---
title: Prise en charge du mappage de source pour les applications JavaScript – Azure Monitor Application Insights
description: Découvrez comment charger des mappages de source dans le conteneur Blob de votre propre compte de stockage avec Application Insights.
ms.topic: conceptual
author: DavidCBerry13
ms.author: daberry
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 9326531d9071305ad804e9c60f8ac90bcb5eaee6
ms.sourcegitcommit: 31d242b611a2887e0af1fc501a7d808c933a6bf6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97809630"
---
# <a name="source-map-support-for-javascript-applications"></a>Prise en charge du mappage de source pour les applications JavaScript

Application Insights prend en charge le chargement des mappages de source dans le conteneur Blob de votre propre compte de stockage.
Les mappages de source peuvent être utilisés pour déminifier les piles d’appels rencontrées dans la page de détails de transaction de bout en bout. Toute exception envoyée par le kit [SDK JavaScript][ApplicationInsights-JS] ou [SDK Node.js][ApplicationInsights-Node.js] peut être déminifiée avec des mappages de source.

![Déminifier une pile d’appels en la liant à un compte de stockage](./media/source-map-support/details-unminify.gif)

## <a name="create-a-new-storage-account-and-blob-container"></a>Créer un compte de stockage et un conteneur Blob

Si vous disposez déjà d’un compte de stockage ou d’un conteneur d’objets blob, vous pouvez ignorer cette étape.

1. [Créer un compte de stockage][create storage account]
2. [Créez un conteneur d’objets blob][create blob container] à l’intérieur de votre compte de stockage. Veillez à définir le « niveau d’accès public » sur `Private` de sorte que vos mappages de source ne soient pas accessibles publiquement.

> [!div class="mx-imgBorder"]
>![Le niveau d’accès de votre conteneur doit être défini sur Privé](./media/source-map-support/container-access-level.png)

## <a name="push-your-source-maps-to-your-blob-container"></a>Envoyer (push) vos mappages de source à votre conteneur Blob

Vous devez intégrer votre pipeline de déploiement continu à votre compte de stockage en le configurant de telle sorte qu’il charge automatiquement vos mappages de source dans le conteneur Blob configuré.

Les mappages de sources peuvent être téléchargés vers votre conteneur de stockage d’objets BLOB avec la même structure de dossiers que celle dans laquelle ils ont été compilés et déployés. Un cas d’usage courant consiste à préfixer un dossier de déploiement avec sa version, par exemple `1.2.3/static/js/main.js`. Lors de la déminimisation via un conteneur d’objets BLOB Azure appelé `sourcemaps`, il tente de récupérer un mappage source situé dans `sourcemaps/1.2.3/static/js/main.js.map`.

### <a name="upload-source-maps-via-azure-pipelines-recommended"></a>Charger les mappages de source via Azure Pipelines (recommandé)

Si vous utilisez Azure Pipelines pour générer et déployer votre application en continu, ajoutez une tâche [Copie de fichiers Azure][azure file copy] à votre pipeline pour charger automatiquement vos mappages de source.

> [!div class="mx-imgBorder"]
> ![Ajoutez une tâche Copie de fichiers Azure à votre pipeline pour charger vos mappages de source dans Stockage Blob Azure](./media/source-map-support/azure-file-copy.png)

## <a name="configure-your-application-insights-resource-with-a-source-map-storage-account"></a>Configurer votre ressource Application Insights avec un compte de stockage Mappage de source

### <a name="from-the-end-to-end-transaction-details-page"></a>À partir de la page de détails de transaction de bout en bout

À partir de la page de détails de transaction de bout en bout, vous pouvez cliquer sur *Unminify* (Déminifier). Vous êtes alors invité à configurer votre ressource si ce n’est pas déjà fait.

1. Sur le portail, affichez les détails d’une exception minifiée.
2. Cliquer sur *Unminify* (Déminifier)
3. Si votre ressource n’a pas été configurée, un message s’affiche pour vous inviter à le faire.

### <a name="from-the-properties-page"></a>À partir de la page de propriétés

Si vous souhaitez configurer ou changer le compte de stockage ou le conteneur Blob qui est lié à votre ressource Application Insights, vous pouvez le faire sous l’onglet *Properties* (Propriétés) de la ressource Application Insights.

1. Accédez à l’onglet *Properties* (Propriétés) de votre ressource Application Insights.
2. Cliquez sur *Change source map blob container* (Changer de conteneur d’objets blob de mappage de source).
3. Sélectionnez un autre conteneur Blob en guise de conteneur de mappages de source.
4. Cliquez sur `Apply`.

> [!div class="mx-imgBorder"]
> ![Reconfigurez votre conteneur Blob Azure en accédant au panneau des propriétés](./media/source-map-support/reconfigure.png)

## <a name="troubleshooting"></a>Dépannage

### <a name="required-azure-role-based-access-control-azure-rbac-settings-on-your-blob-container"></a>Paramètres de contrôle d’accès en fonction du rôle Azure (Azure RBAC) nécessaires sur votre conteneur Blob

Tout utilisateur sur le portail utilisant cette fonctionnalité doit au moins disposer d’une autorisation [Lecteur de données Blob de stockage][storage blob data reader] sur votre conteneur Blob. Vous devez attribuer ce rôle à toute autre personne appelée à utiliser les mappages de source via cette fonctionnalité.

> [!NOTE]
> Selon la façon dont le conteneur a été créé, il est possible que ce rôle ne vous ait pas été attribué automatiquement, à vous ou à votre équipe.

### <a name="source-map-not-found"></a>Mappage de source introuvable

1. Vérifiez que le mappage de source correspondant est chargé dans le bon conteneur d’objets blob.
2. Vérifiez que le nom du fichier de mappage de source reprend le nom du fichier JavaScript auquel il est mappé, avec le suffixe `.map`.
    - Par exemple, `/static/js/main.4e2ca5fa.chunk.js` recherche l’objet blob nommé `main.4e2ca5fa.chunk.js.map`.
3. Vérifiez si des erreurs sont journalisées dans la console de votre navigateur. Incluez-les dans votre ticket de support, le cas échéant.

## <a name="next-steps"></a>Étapes suivantes

* [Tâche Copie de fichiers Azure](/azure/devops/pipelines/tasks/deploy/azure-file-copy?view=azure-devops)


<!-- Remote URLs -->
[create storage account]: ../../storage/common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fblobs%2Ftoc.json&tabs=azure-portal
[create blob container]: ../../storage/blobs/storage-quickstart-blobs-portal.md
[storage blob data reader]: ../../role-based-access-control/built-in-roles.md#storage-blob-data-reader
[ApplicationInsights-JS]: https://github.com/microsoft/applicationinsights-js
[ApplicationInsights-Node.js]: https://github.com/microsoft/applicationinsights-node.js
[azure file copy]: https://aka.ms/azurefilecopyreadme
