---
title: Azure Schema Registry dans Event Hubs (préversion)
description: Cet article fournit une vue d’ensemble de la prise en charge du registre de schémas par Azure Event Hubs (version préliminaire).
ms.topic: overview
ms.date: 09/22/2020
ms.custom: references_regions
ms.openlocfilehash: a876651b76aa259754623854b8fc4a7c6c8a939e
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330493"
---
# <a name="azure-schema-registry-in-event-hubs-preview"></a>Azure Schema Registry dans Event Hubs (préversion)
Dans de nombreux scénarios de diffusion en continu d’événements et de messagerie, l’événement ou la charge utile de message contient des données structurées qui sont en cours de sérialisation ou de désérialisation à l’aide d’un format piloté par schéma comme Apache Avro. Les expéditeurs et les destinataires peuvent être amenés à valider l’intégrité des données avec un document de schéma comme avec le schéma JSON. Pour les formats pilotés par schéma, le fait de rendre le schéma disponible pour le consommateur du message est une condition préalable pour que le consommateur puisse désérialiser les données. 

Le **Registre de schéma Azure** est une fonctionnalité de Event Hubs, qui fournit un référentiel central pour les documents de schéma pour les applications pilotées par les événements et les applications axées sur la messagerie. Il offre aux producteurs et aux consommateurs la flexibilité nécessaire pour échanger des données sans qu’ils doivent gérer et partager le schéma entre eux, et pour évoluer à des vitesses différentes. Le registre de schémas fournit également une infrastructure de gouvernance simple pour les schémas réutilisables et définit la relation entre les schémas par le biais d’une construction de regroupement (groupes de schémas).

> [!NOTE]
> - La fonctionnalité **Registre de schémas** est actuellement en **préversion** et n’est pas recommandée pour les charges de travail de production.
> - Cette fonctionnalité est disponible uniquement pour les niveaux **standard** et **dédié** et non pour le niveau **de base** .

Avec les infrastructures de sérialisation basées sur des schémas comme Apache Avro, l’externalisation des métadonnées de sérialisation dans des schémas partagés peut également réduire considérablement la surcharge par message des informations sur le type et des noms de champs inclus avec chaque jeu de données, comme c’est le cas avec des formats étiquetés tels que JSON. Le fait de disposer de schémas stockés avec les événements et à l’intérieur de l’infrastructure d’événements permet de s’assurer que les métadonnées requises pour la sérialisation/désérialisation sont toujours accessibles et que les schémas ne peuvent pas être mal placés. 

## <a name="event-hubs-namespace"></a>Espace de noms Event Hubs
Un espace de noms Event Hubs peut désormais héberger des groupes de schémas avec Event Hubs (ou des rubriques Kafka). Il héberge un registre de schémas et peut comporter plusieurs groupes de schémas. En dépit de l’hébergement dans Azure Event Hubs, le registre de schémas peut être utilisé de manière universelle avec tous les services de messagerie Azure et tout autre message ou répartiteur d’événements. Chacun de ces groupes de schémas est un référentiel sécurisable distinct pour un ensemble de schémas. Les groupes peuvent être alignés sur une application ou une unité d’organisation particulière. 

## <a name="schema-groups"></a>Groupes de schémas
Le groupe de schémas est un groupe logique de schémas similaires basés sur les critères de votre entreprise. Un groupe de schémas peut contenir plusieurs versions d’un schéma. Le paramètre de mise en œuvre de la compatibilité sur un groupe de schémas peut assurer la compatibilité descendante des versions de schéma plus récentes.

La limite de sécurité imposée par le mécanisme de regroupement permet de s’assurer que les secrets commerciaux ne fuient pas par inadvertance à travers les métadonnées dans des situations où l’espace de noms est partagé entre plusieurs partenaires. Cela permet également aux propriétaires d’applications de gérer les schémas indépendamment d’autres applications qui partagent le même espace de noms.


## <a name="schemas"></a>Schémas
Les schémas définissent le contrat entre les producteurs et les consommateurs. Un schéma défini dans un registre de schémas Event Hubs permet de gérer le contrat en dehors des données d’événement, ce qui supprime la surcharge de charge utile. Un schéma a un nom, un type (exemple : enregistrement, tableau, etc.), le mode de compatibilité (aucune, compatibilité ascendante, compatibilité descendante, totale) et le type de sérialisation (uniquement Avro pour le moment). Vous pouvez créer plusieurs versions d’un schéma et récupérer et utiliser une version spécifique d’un schéma. 

## <a name="client-sdks"></a>Kits de développement logiciel (SDK) client
Vous pouvez utiliser l’une des bibliothèques suivantes qui incluent un sérialiseur Avro, qui vous permettent de sérialiser et désérialiser des charges utiles contenant des identificateurs de schéma de registre de schémas et des données encodées Avro.

- [.NET - Microsoft.Azure.Data.SchemaRegistry.ApacheAvro](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/schemaregistry/Microsoft.Azure.Data.SchemaRegistry.ApacheAvro)
- [Java - azure-data-schemaregistry-avro](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/schemaregistry/azure-data-schemaregistry-avro/)
- [Python - azure-schemaregistry-avroserializer](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/schemaregistry/azure-schemaregistry-avroserializer)
- [JavaScript - @azure/schema-registry-avro](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/schemaregistry/schema-registry-avro)
- [Apache Kafka](https://github.com/Azure/azure-schema-registry-for-kafka/) - Exécuter des sérialiseurs et des désérialiseurs Apache Avro intégrés sauvegardés par des registres de schémas Azure. Le sérialiseur client Apache Kafka du client Java pour le registre de schémas Azure peut être utilisé dans n’importe quel scénario Apache Kafka et avec un déploiement Apache Kafka® ou un service cloud. 

L’illustration suivante montre le flux d’informations du registre de schémas avec Event Hubs : 

:::image type="content" source="./media/schema-registry-overview/flow-diagram.png" alt-text="Diagramme de flux":::

## <a name="standard-vs-dedicated-limits"></a>Limites standard et limites dédiées
Pour connaître les limites (par exemple, le nombre de groupes de schémas dans un espace de noms) qui sont identiques et différents pour les niveaux standard et dédiés d’Event Hubs, consultez [Limites du registre de schémas](../azure-resource-manager/management/azure-subscription-service-limits.md#schema-registry-limitations)

## <a name="azure-role-based-access-control"></a>Contrôle d'accès en fonction du rôle Azure
Quand vous accédez au registre de schémas programmatiquement, vous devez inscrire une application dans Azure Active Directory (Azure AD) et ajouter le principal de sécurité de l’application à l’un des rôles de contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

| Role | Description | 
| ---- | ----------- | 
| Propriétaire | Lire, écrire et supprimer des groupes de registres de schémas et des schémas. |
| Contributeur | Lire, écrire et supprimer des groupes de registres de schémas et des schémas. |
| [Lecteur du registre de schémas (préversion)](../role-based-access-control/built-in-roles.md#schema-registry-reader-preview) | Lire et répertorier les groupes de registres de schémas et les schémas. |
| [Contributeur du registre de schémas (préversion)](../role-based-access-control/built-in-roles.md#schema-registry-reader-preview) | Lire, écrire et supprimer des groupes de registres de schémas et des schémas. |

Pour obtenir des instructions sur la création et l’inscription d’une application à l’aide du portail Azure, consultez [Inscrire une application avec Azure AD](../active-directory/develop/quickstart-register-app.md). Notez l’ID client (ID d’application), l’ID de locataire et le secret à utiliser dans le code. 

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer un registre de schémas à l’aide du portail Azure, consultez [Créer un registre de schémas Event Hubs à l’aide du portail Azure](create-schema-registry.md).
- Reportez-vous aux exemples suivants de la **Bibliothèque client Avro de registre de schémas** .
    - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/schemaregistry/Microsoft.Azure.Data.SchemaRegistry.ApacheAvro/tests/Samples)
    - [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/schemaregistry/azure-data-schemaregistry-avro/src/samples)
    - [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/schemaregistry/schema-registry-avro/samples )
    - [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/schemaregistry/azure-schemaregistry-avroserializer/samples )
    - [Intégration de Kafka Avro pour le registre de schémas Azure](https://github.com/Azure/azure-schema-registry-for-kafka/tree/master/csharp/avro/samples)
