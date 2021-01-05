---
title: Applications et principaux de service dans Azure AD | Azure
titleSuffix: Microsoft identity platform
description: Découvrez la relation entre les objets application et principal du service dans Azure Active Directory.
author: rwike77
manager: CelesteDG
services: active-directory
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/22/2020
ms.author: ryanwi
ms.custom: aaddev, identityplatformtop40
ms.reviewer: sureshja
ms.openlocfilehash: a18e3cb65b2ef70a04ca1d7e74dd9d5f42e3a933
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97355764"
---
# <a name="application-and-service-principal-objects-in-azure-active-directory"></a>Objets application et principal du service dans Azure Active Directory

Cet article décrit l’inscription d’application, les objets application et les principaux du service dans Azure Active Directory : ce qu’ils sont, comment ils sont utilisés et comment ils sont liés entre eux. Un exemple de scénario mutualisé est également présenté pour illustrer la relation entre l’objet application d’une application et les objets principal de service correspondants.

## <a name="application-registration"></a>Inscription de l’application
Afin de pouvoir déléguer les fonctions de gestion des identités et des accès, une application doit être inscrite auprès d’un [client](developer-glossary.md#tenant) Azure AD. Lorsque vous inscrivez votre application auprès d’Azure AD, vous créez une configuration d’identité pour votre application, ce qui lui permet de s’intégrer avec Azure AD. Quand vous inscrivez une application dans le [portail Azure][AZURE-Portal], vous choisissez s’il s’agit d’une application monolocataire (accessible uniquement dans votre locataire) ou multilocataire (accessible dans d’autres locataires). De plus, vous pouvez éventuellement définir un URI de redirection (auquel le jeton d’accès est envoyé).

:::image type="content" source="media/app-objects-and-service-principals/app-registration.png" alt-text="Capture d’écran du volet Enregistrer une application du Portail Azure":::

Une fois l’inscription de l’application terminée, vous disposez d’une instance globale unique de l’application (objet application) qui réside dans votre locataire ou annuaire de base.  Vous disposez également d’un ID global unique pour votre application (l’ID de l’application ou du client).  Dans le portail, vous pouvez ensuite ajouter des secrets ou des certificats, ainsi que des étendues pour que votre application fonctionne, personnaliser votre application dans la boîte de dialogue de connexion, et bien plus.

Si vous inscrivez une application dans le portail, un objet application et un objet principal de service sont automatiquement créés dans votre locataire de base.  Si vous inscrivez/créez une application à l’aide des API Microsoft Graph, la création de l’objet principal de service est une étape distincte.

## <a name="application-object"></a>Objet application
Une application Azure AD est définie par son seul et unique objet application, qui réside dans le locataire Azure AD dans lequel l’application a été inscrite (locataire « de base » de l’application).  Un objet application est utilisé en tant que modèle ou blueprint pour créer un ou plusieurs objets principal de service.  Un principal de service est créé dans chaque locataire dans lequel l’application est utilisée. À l’image d’une classe en programmation orientée objet, l’objet d’application a des propriétés statiques qui sont appliquées à tous les principaux de service créés (ou instances d’application).

L’objet application décrit trois aspects d’une application : la manière dont le service peut émettre des jetons pour accéder à l’application, les ressources auxquelles l’application peut avoir besoin d’accéder, et les actions que l’application peut effectuer.

Le panneau **Inscriptions d’applications** dans le [portail Azure][AZURE-Portal] permet de répertorier et gérer les objets application dans votre locataire de base.

![Panneau Inscriptions d’applications](./media/app-objects-and-service-principals/app-registrations-blade.png)

[L’entité Application][MS-Graph-App-Entity] de Microsoft Graph définit le schéma pour les propriétés d’un objet d’application.

## <a name="service-principal-object"></a>Objet principal du service
Pour accéder aux ressources qui sont sécurisées par un locataire Azure AD, l’entité qui nécessite l’accès doit être représentée par un principal de sécurité. Cette exigence est valable aussi bien pour les utilisateurs (principal d’utilisateur) que pour les applications (principal de service). Le principal de sécurité définit la stratégie d’accès et les autorisations pour l’utilisateur ou l’application du locataire Azure AD. Cela rend possibles les fonctionnalités de base, telles que l’authentification de l’application ou de l’utilisateur lors de la connexion, et l’autorisation lors de l’accès aux ressources.

Un principal de service est la représentation locale, ou instance d’application, d’un objet application global dans un locataire ou répertoire unique. Un principal de service est une instance concrète créée à partir de l’objet application, qui hérite de certaines propriétés de celui-ci. Un principal de service est créé dans chaque locataire dans lequel l’application est utilisée, et fait référence à l’objet application global unique.  L’objet principal de service définit ce que l’application peut réellement faire dans le locataire spécifique, qui peut accéder à l’application, ainsi que les ressources auxquelles l’application peut accéder.

Lorsqu’une application reçoit l’autorisation d’accéder aux ressources d’un locataire (après inscription ou [consentement](developer-glossary.md#consent)), un objet de principal de service est créé. Vous pouvez également créer des objets principal de service dans un locataire à l’aide d’[Azure PowerShell](howto-authenticate-service-principal-powershell.md), de l’interface [Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli), de [Microsoft Graph](/graph/api/serviceprincipal-post-serviceprincipals?tabs=http), du [portail Azure][AZURE-Portal] et d’autres outils. Lorsque vous utilisez le portail, un principal de service est créé automatiquement lorsque vous inscrivez une application.

Le panneau **Applications d’entreprise** dans le portail permet de répertorier et gérer les principaux de service dans un locataire. Vous pouvez voir les autorisations du principal de service, les autorisations accordées par l’utilisateur, les utilisateurs ayant accordé ces autorisations, les informations de connexion, et bien plus.

![Panneau Applications d’entreprise](./media/app-objects-and-service-principals/enterprise-apps-blade.png)

L’entité [ServicePrincipal][MS-Graph-Sp-Entity] de Microsoft Graph définit le schéma pour les propriétés d’un objet principal de service.

## <a name="relationship-between-application-objects-and-service-principals"></a>Relation entre les objets application et les principaux de service

L’objet application est la représentation *globale* de votre application à l’usage de tous les locataires, et le principal de service est la représentation *locale* de l’application à l’usage d’un locataire spécifique.

L’objet application fait office de modèle à partir duquel les propriétés communes et par défaut sont *dérivées* pour une utilisation visant à créer des objets de principal de service correspondants. Un objet application présente donc une relation 1 à 1 avec l’application logicielle et une relation 1 à plusieurs avec le ou les objets de principal de service correspondants.

Un principal de service doit être créé dans chaque locataire sur lequel l’application est utilisée, ce qui lui permet d’établir une identité pour la connexion et/ou l’accès aux ressources sécurisées par le locataire. Une application à locataire unique n’a qu’un seul principal de service (dans son locataire de base), créé et pouvant être utilisé pendant l’inscription de l’application. Une application web/API multi-locataire a également un principal de service créé dans chaque locataire où un utilisateur de ce locataire a consenti à son utilisation.

> [!NOTE]
> Toute modification apportée à l’objet application de votre application est également répercutée dans son objet principal du service, uniquement dans le client de base de l’application (le client où elle a été inscrite). Pour l’accès mutualisé, les modifications apportées à l’objet application ne sont pas répercutées dans les objets principal de service des clients consommateurs jusqu’à ce que l’accès soit supprimé via le [volet d’accès à l’application](https://myapps.microsoft.com) et à nouveau octroyé.
>
> Notez également que les applications natives sont enregistrées en tant que mutualisées par défaut.

## <a name="example"></a>Exemple

Le schéma suivant illustre la relation entre un objet application d’une application et les objets principal du service correspondants dans le contexte d’un exemple d’application mutualisée appelée **RH**. Il existe trois clients Azure AD dans cet exemple de scénario :

- **Adatum** : le client utilisé par la société qui a développé **l’application RH** ;
- **Contoso** : le client utilisé par l’entreprise Contoso, qui est un consommateur de **l’application RH** ;
- **Fabrikam** : le client utilisé par l’entreprise Fabrikam, qui est également un consommateur de **l’application RH**.

![Relation entre un objet d’application et un objet de principal de service](./media/app-objects-and-service-principals/application-objects-relationship.svg)

Dans cet exemple de scénario :

| Étape | Description |
|------|-------------|
| 1    | Correspond au processus de création des objets application et principal du service dans le client de base de l’application. |
| 2    | Lorsque les administrateurs de Contoso et Fabrikam accordent leur consentement, un objet principal du service est créé dans le client Azure AD de leur entreprise et se voit attribuer les autorisations accordées par l’administrateur. Notez également que l’application RH peut être configurée/conçue de manière à autoriser le consentement par les utilisateurs à des fins d’utilisation individuelle. |
| 3    | Les clients consommateurs de l’application RH (Contoso et Fabrikam) ont chacun leur propre objet principal de service. Chacun représente leur utilisation d’une instance de l’application lors de l’exécution, régie par les autorisations consenties par l’administrateur respectif. |

## <a name="next-steps"></a>Étapes suivantes

- Vous pouvez utiliser [l’Afficheur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer) pour interroger à la fois les objets d’application et de principal de service.
- Vous pouvez accéder à l’objet d’application d’une application via l’API Microsoft Graph, l’éditeur de manifeste d’application du [Portail Azure][AZURE-Portal] ou les [cmdlets Azure AD PowerShell](/powershell/azure/), tel que représenté par son [entité Application][MS-Graph-App-Entity] OData.
- Vous pouvez accéder à l’objet de principal de service d’une application via l’API Microsoft Graph ou les [cmdlets Azure AD PowerShell](/powershell/azure/), tel que représenté par son [entité ServicePrincipal][MS-Graph-Sp-Entity] OData.

<!--Image references-->

<!--Reference style links -->
[MS-Graph-App-Entity]: /graph/api/resources/application
[MS-Graph-Sp-Entity]: /graph/api/resources/serviceprincipal
[AZURE-Portal]: https://portal.azure.com
