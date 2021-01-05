---
title: Tutoriel d’application web – Configurer l’API Azure pour FHIR
description: Ce tutoriel décrit en détail un exemple de déploiement d’application web simple. Ce premier tutoriel décrit les prérequis et le déploiement de l’API Azure pour FHIR
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.custom: devx-track-js
ms.openlocfilehash: cb183b5c8aff018d4dc73819b938b24ad0daa934
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975788"
---
# <a name="deploy-javascript-app-to-read-data-from-fhir-service"></a>Déployer une application JavaScript pour lire des données à partir d’un service FHIR
Dans ce tutoriel, vous allez déployer une petite application JavaScript, qui lit des données à partir d’un service FHIR. Les étapes de ce tutoriel sont les suivantes :
1. Déployer un serveur FHIR
1. Inscrire une application cliente publique
1. Tester l’accès à l’application
1. Créer une application web qui lit ces données FHIR

## <a name="prerequisites"></a>Prérequis
Avant de commencer cet ensemble de tutoriels, vous aurez besoin des éléments suivants :
1. Un abonnement Azure
1. Un client Azure Active Directory
1. [Postman](https://www.getpostman.com/) installé

> [!NOTE]
> Pour ce tutoriel, le service FHIR, l’application Azure AD et les utilisateurs Azure AD se trouvent tous dans le même locataire Azure AD. Si ce n’est pas le cas, vous pouvez quand même suivre ce tutoriel, mais vous serez peut-être amené à vous plonger dans quelques-uns des documents référencés et effectuer des étapes supplémentaires.

## <a name="deploy-azure-api-for-fhir"></a>Déployer l’API Azure pour FHIR
La première étape du tutoriel consiste à configurer correctement l’API Azure pour FHIR.

1. Si vous ne l’avez pas déjà fait, déployez l’[API Azure pour FHIR](fhir-paas-portal-quickstart.md).
1. Une fois que vous avez déployé l’API Azure pour FHIR, configurez les paramètres [CORS](configure-cross-origin-resource-sharing.md) en accédant à votre API Azure pour FHIR et en sélectionnant CORS. 
    1. Définissez **Origines** sur *.
    1. Définissez **En-têtes** sur *.
    1. Sous **Méthodes**, choisissez **Sélectionner tout**.
    1. Définissez **Âge maximal** sur **600**.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que votre API Azure pour FHIR est déployée, vous êtes prêt à inscrire une application cliente publique.

>[!div class="nextstepaction"]
>[Inscrire une application cliente publique](tutorial-web-app-public-app-reg.md)
