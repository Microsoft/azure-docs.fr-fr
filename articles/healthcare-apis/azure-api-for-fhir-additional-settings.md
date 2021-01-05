---
title: Paramètres supplémentaires de l’API Azure pour FHIR
description: Vue d’ensemble des paramètres supplémentaires que vous pouvez définir pour l’API Azure pour FHIR
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.reviewer: matjazl
ms.author: cavoeg
author: CaitlinV39
ms.date: 11/22/2019
ms.openlocfilehash: b9949ab4cfa42553e5a8d28244b8f621f09334d1
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398162"
---
# <a name="additional-settings-for-azure-api-for-fhir"></a>Paramètres supplémentaires de l’API Azure pour FHIR

Ce guide pratique décrit les paramètres supplémentaires que vous pouvez définir dans votre API Azure pour FHIR. Il existe des pages supplémentaires qui explorent d’autres détails.

## <a name="configure-database-settings"></a>Configurer les paramètres de base de données

L’API Azure pour FHIR utilise une base de données pour stocker ses données. Les performances de la base de données sous-jacente dépendent du nombre d’unités de requête (RU) sélectionné au cours de l’approvisionnent du service ou des paramètres de base de données définis une fois le service approvisionné.

Un débit doit être approvisionné pour qu’il y ait suffisamment de ressources système disponibles en permanence pour votre base de données. Le nombre de RU dont vous avez besoin pour votre application dépend des opérations que vous effectuez. Les opérations peuvent aller de simples lectures et écritures à des requêtes plus complexes.

Pour plus d’informations sur la façon de modifier les paramètres par défaut, consultez [Configurer les paramètres de base de données](configure-database.md).

## <a name="access-control"></a>Contrôle d’accès

L’API Azure pour FHIR autorise uniquement les utilisateurs autorisés à accéder à l’API FHIR. Vous pouvez configurer des utilisateurs autorisés de deux façons. La méthode principale recommandée pour configurer le contrôle d’accès consiste à utiliser le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/index.yml), qui est accessible via le panneau **Contrôle d’accès (IAM)** . Un RBAC Azure fonctionne uniquement si vous souhaitez sécuriser l’accès au plan de données à l’aide du locataire Azure Active Directory associé à votre abonnement. Si vous souhaitez utiliser un autre locataire, l’API Azure pour FHIR offre un mécanisme de contrôle d’accès au plan de données FHIR local. Les options de configuration ne sont pas aussi riches lors de l’utilisation du mécanisme de RBAC local. Pour plus de détails, choisissez l’une des options suivantes :

* [RBAC Azure pour plan de données FHIR](configure-azure-rbac.md). Il s’agit de l’option recommandée lorsque vous utilisez le locataire Azure Active Directory associé à votre abonnement.
* [Contrôle d’accès au plan de données FHIR local](configure-local-rbac.md). Utilisez cette option uniquement si vous devez utiliser un locataire Azure Active Directory externe pour le contrôle d’accès au plan de données. 

## <a name="enable-diagnostic-logging"></a>Activer la journalisation des diagnostics
Vous pouvez activer la journalisation des diagnostics dans le cadre de votre configuration pour être en mesure de surveiller votre service et de recevoir des rapports précis à des fins de conformité. Pour plus d’informations sur la configuration de la journalisation des diagnostics, consultez notre [Guide pratique](enable-diagnostic-logging.md) sur la configuration de la journalisation des diagnostics, ainsi que des exemples de requêtes. 

## <a name="use-custom-headers-to-add-data-to-audit-logs"></a>Utiliser des en-têtes personnalisés pour ajouter des données aux journaux d’audit
Dans l’API Azure pour FHIR, vous pouvez inclure dans les journaux des informations supplémentaires qui proviennent du système appelant. Pour inclure ces informations, vous pouvez utiliser des en-têtes personnalisés.

Vous pouvez utiliser des en-têtes personnalisés pour capturer plusieurs types d’informations. Par exemple :

* Informations d’identité ou d’autorisation
* Origine de l’appelant
* Organisation d’origine
* Détails sur le système client (dossier médical électronique, portail des patients)

Pour ajouter ces données à vos journaux d’audit, consultez le guide pratique [Utiliser des en-têtes HTTP personnalisés pour ajouter des données aux journaux d’audit](use-custom-headers.md).

## <a name="next-steps"></a>Étapes suivantes

Ce guide pratique vous a expliqué comment configurer des paramètres supplémentaires pour l’API Azure pour FHIR.

Consultez ensuite la série de didacticiels pour créer une application Web qui lit les données FHIR.

>[!div class="nextstepaction"]
>[Déployer une application JavaScript](tutorial-web-app-fhir-server.md)