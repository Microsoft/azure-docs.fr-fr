---
title: Pratiques de sécurité recommandées
description: Lorsque vous utilisez Azure Lighthouse, il est important de prendre en compte la sécurité et le contrôle d’accès.
ms.date: 08/12/2020
ms.topic: conceptual
ms.openlocfilehash: ef2c41cf052e5f79ecf4abf01c8f3fab3dd1de14
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843721"
---
# <a name="recommended-security-practices"></a>Pratiques de sécurité recommandées

Lorsque vous utilisez [Azure Lighthouse](../overview.md), il est important de prendre en compte la sécurité et le contrôle d’accès. Les utilisateurs de votre locataire ayant un accès direct aux abonnements et aux groupes de ressources du client, il est souhaitable de prendre des mesures pour maintenir la sécurité de votre locataire. Vous devez également vous assurer que vous autorisez uniquement l’accès nécessaire pour gérer efficacement les ressources de vos clients. Cette rubrique fournit des recommandations pour vous y aider.

> [!TIP]
> Ces recommandations s’appliquent également aux [entreprises qui gèrent plusieurs locataires](enterprise.md) avec Azure Lighthouse.

## <a name="require-azure-ad-multi-factor-authentication"></a>Demander l’authentification multifacteur Azure AD

La fonctionnalité [Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md) (également appelée vérification en deux étapes) permet d'empêcher des attaquants d'accéder à un compte en exigeant plusieurs étapes d'authentification. Vous devez exiger l’application de la solution Microsoft Azure Multi-Factor Authentication à tous les utilisateurs de votre locataire gérant, notamment aux utilisateurs qui auront accès aux ressources déléguées du client.

Nous vous suggérons de demander à vos clients d'implémenter également Azure AD Multi-Factor Authentication dans leurs locataires.

## <a name="assign-permissions-to-groups-using-the-principle-of-least-privilege"></a>Affecter des autorisations à des groupes selon le principe de privilège minimum

Pour faciliter la gestion, utilisez des groupes Azure Active Directory (Azure AD) pour chaque rôle requis pour gérer les ressources de vos clients. Cela vous permet d’ajouter ou de supprimer des utilisateurs dans un groupe en fonction des besoins, plutôt que d’attribuer directement des autorisations à chaque utilisateur.

> [!IMPORTANT]
> Pour que vous puissiez ajouter des autorisations pour un groupe Azure AD, le **Type de groupe** doit être défini sur **Sécurité**. Cette option est sélectionnée lors de la création du groupe. Pour plus d’informations, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Lors de la création de votre structure d’autorisation, veillez à suivre le principe du privilège minimum afin que les utilisateurs disposent uniquement des autorisations nécessaires pour accomplir leur travail, ce qui contribue à réduire le risque d’erreurs accidentelles.

Par exemple, vous pouvez utiliser une structure telle que celle-ci :

|Nom du groupe  |Type  |principalId  |Définition de rôle  |ID de définition de rôle  |
|---------|---------|---------|---------|---------|
|Architectes     |Groupe d'utilisateurs         |\<principalId\>         |Contributeur         |b24988ac-6180-42a0-ab88-20f7382dd24c  |
|Évaluation     |Groupe d'utilisateurs         |\<principalId\>         |Lecteur         |acdd72a7-3385-48ef-bd42-f606fba81ae7  |
|Spécialistes des machines virtuelles     |Groupe d'utilisateurs         |\<principalId\>         |Contributeur de machine virtuelle         |9980e02c-c2be-4d73-94e8-173b1dc7cf3c  |
|Automatisation     |Nom du principal du service (SPN)         |\<principalId\>         |Contributeur         |b24988ac-6180-42a0-ab88-20f7382dd24c  |

Une fois que vous avez créé ces groupes, vous pouvez affecter des utilisateurs en fonction des besoins. Ajoutez uniquement les utilisateurs qui ont vraiment besoin d’un accès. Veillez à vérifier régulièrement l’appartenance aux groupes et à supprimer tous les utilisateurs dont l’inclusion n’est plus appropriée ou nécessaire.

N’oubliez pas que, lorsque vous [intégrez des clients via une offre de service managé public](../how-to/publish-managed-services-offers.md), tout groupe (ou utilisateur ou principal de service) que vous incluez aura les mêmes autorisations pour chaque client achetant le plan. Pour affecter différents groupes afin de collaborer avec chaque client, vous devez publier un plan privé distinct exclusif pour chaque client, ou intégrer des clients individuellement à l’aide de modèles Azure Resource Manager. Par exemple, vous pouvez publier un plan public disposant d’un accès très limité, puis travailler directement avec le client pour intégrer ses ressources afin d’obtenir un accès supplémentaire à l’aide d’un modèle de ressource Azure personnalisé qui accorde un accès supplémentaire en fonction des besoins.

## <a name="next-steps"></a>Étapes suivantes

- [Déployer Azure AD Multi-Factor Authentication](../../active-directory/authentication/howto-mfa-getstarted.md).
- Découvrez les [Expériences de gestion inter-locataire](cross-tenant-management-experience.md).
