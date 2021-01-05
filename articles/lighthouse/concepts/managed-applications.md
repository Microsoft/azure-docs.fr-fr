---
title: Azure Lighthouse et applications managées Azure
description: Découvrez comment les applications gérées par Azure Lighthouse et Azure peuvent vous aider à activer différents scénarios et comment elles peuvent être utilisées ensemble.
ms.date: 08/12/2020
ms.topic: conceptual
ms.openlocfilehash: b3d6c83a50f1da006f2618f7f4ff59759dac2555
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436519"
---
# <a name="azure-lighthouse-and-azure-managed-applications"></a>Azure Lighthouse et applications managées Azure

Azure Lighthouse et les applications managées Azure sont deux solutions qui permettent à un fournisseur de services d’accéder aux ressources qui résident dans le locataire du client. Il est utile de comprendre en quoi le fonctionnement de ces solutions diffère, les scénarios pour lesquels elles sont pertinentes ainsi que la façon dont elles peuvent être utilisées ensemble.

> [!TIP]
> Même si nous faisons référence aux fournisseurs de services et aux clients dans cette rubrique, les [entreprises gérant plusieurs locataires](enterprise.md) peuvent utiliser les mêmes processus et outils.

## <a name="comparing-azure-lighthouse-and-azure-managed-applications"></a>Comparaison d’Azure Lighthouse et des applications managées Azure

### <a name="azure-lighthouse"></a>Azure Lighthouse

Avec [Azure Lighthouse](../overview.md), un fournisseur de services peut effectuer une grande diversité de tâches de gestion directement sur l’abonnement d’un client (ou groupe de ressources). Cet accès s’effectue au moyen d’une projection logique, qui permet au fournisseur de services de se connecter à son propre locataire et d’accéder aux ressources qui appartiennent au locataire du client. Le client détermine quels abonnements ou groupes de ressources il souhaite déléguer au fournisseur de services, et il assure l’accès total à ces ressources. Il peut également supprimer l’accès du fournisseur de services à tout moment.

Pour pouvoir utiliser Azure Lighthouse, les clients sont intégrés à la [gestion des ressources déléguées Azure](azure-delegated-resource-management.md) soit en [déployant des modèles ARM](../how-to/onboard-customer.md), soit en publiant une [offre de service managé sur la Place de marché Azure](managed-services-offers.md). Vous pouvez suivre votre impact sur les engagements des clients en [liant votre ID partenaire](../how-to/partner-earned-credit.md).

Azure Lighthouse est généralement utilisé dans les scénarios où un fournisseur de services effectue des tâches de gestion pour un client de manière continue.

### <a name="azure-managed-applications"></a>Applications managées Azure

Les [applications managées Azure](../../azure-resource-manager/managed-applications/overview.md) permettent à un fournisseur de services ou à un ISV d’offrir des solutions cloud faciles à déployer et à utiliser par les clients dans leurs propres abonnements.

Dans une application managée, les ressources utilisées par l’application sont regroupées et déployées sur un groupe de ressources qui est géré par l’éditeur. Ce groupe de ressources est présent dans l’abonnement du client, mais une identité dans le locataire de l’éditeur y a accès. L’ISV garde le contrôle de la gestion et de la mise à jour de l’application managée ; le client n’a pas un accès direct pour l’utiliser dans son groupe de ressources, et n’a aucun accès à ses ressources.

Les applications managées prennent en charge les [expériences personnalisées dans le Portail Azure](../../azure-resource-manager/managed-applications/concepts-view-definition.md) et l’[intégration avec des fournisseurs personnalisés](../../azure-resource-manager/managed-applications/tutorial-create-managed-app-with-custom-provider.md). Ces options permettent d’offrir une expérience encore plus personnalisée et intégrée, et rendre ainsi certaines tâches de gestion plus faciles à effectuer par les clients eux-mêmes.

Les applications managées peuvent être [publiées sur la Place de marché Azure](../../marketplace/create-new-azure-apps-offer.md), soit en tant qu’offre privée pour une utilisation particulière d’un client, soit en tant qu’offre publique que plusieurs clients peuvent acheter. Elles peuvent également être fournies aux utilisateurs dans votre organisation par la [publication d’applications managées dans votre catalogue de services](../../azure-resource-manager/managed-applications/publish-service-catalog-app.md). Vous pouvez déployer des instances du catalogue de services et de la Place de marché à l’aide de modèles ARM, qui contiennent éventuellement l’identificateur unique du partenaire de la Place de marché commerciale pour suivre l’[attribution de l’utilisation de client](../../marketplace/azure-partner-customer-usage-attribution.md).

Les applications managées Azure sont généralement utilisées quand il est possible de répondre à un besoin spécifique du client au moyen d’une solution clés en main entièrement managée par le fournisseur de services.

## <a name="using-azure-lighthouse-and-azure-managed-applications-together"></a>Utilisation conjointe d’Azure Lighthouse et des applications managées Azure

Bien qu’Azure Lighthouse et les applications managées Azure utilisent des mécanismes d’accès différents pour remplir des objectifs différents, il y a certains scénarios où un fournisseur de services peut avoir intérêt à utiliser les deux solutions ensemble avec le même client.

Par exemple, un client peut souhaiter que les services managés soient fournis par un fournisseur de services par le biais d’Azure Lighthouse, afin d’avoir une visibilité sur les actions du partenaire tout en conservant le contrôle continu de leur abonnement délégué. Toutefois, il se peut que le fournisseur de services ne souhaite pas que le client ait accès à certaines ressources qui seront stockées dans le locataire du client, ou autoriser les actions personnalisées sur ces ressources. Pour remplir ces objectifs, le fournisseur de services peut publier une offre privée en tant qu’application managée. L’application managée peut inclure un groupe de ressources qui est déployé dans le locataire du client, mais qui n’est pas accessible directement par le client.

Les clients peuvent également être intéressés par les applications managées de plusieurs fournisseurs de services, qu’ils utilisent ou non par ailleurs des services managés par Azure Lighthouse de l’un de ces fournisseurs de services. En outre, les partenaires du programme CSP (fournisseur de solutions Cloud) peuvent revendre certaines applications managées qui sont publiées par d’autres ISV à des clients qu’ils gèrent via Azure Lighthouse. Le large éventail d’options possibles permet aux fournisseurs de services de trouver le bon équilibre entre répondre aux besoins de leurs clients et limiter l’accès aux ressources comme ils le souhaitent.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [applications managées Azure](../../azure-resource-manager/managed-applications/overview.md).
- Découvrez comment [intégrer un abonnement à Azure Lighthouse](../how-to/onboard-customer.md).