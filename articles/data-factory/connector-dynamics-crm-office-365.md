---
title: Copier des données dans Dynamics (Common Data Service)
description: Découvrez comment copier des données à partir de Microsoft Dynamics CRM ou Microsoft Dynamics 365 (Common Data Service) vers des banques de données réceptrices prises en charge ou à partir de banques de données sources prises en charge vers Dynamics CRM ou Dynamics 365 à l’aide de l’activité de copie disponible dans un pipeline de fabrique de données.
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.custom: seo-lt-2019
ms.date: 09/23/2020
ms.openlocfilehash: 204399186ae229324f9dc478e0ef58a173060013
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92638174"
---
# <a name="copy-data-from-and-to-dynamics-365-common-data-service-or-dynamics-crm-by-using-azure-data-factory"></a>Copier des données à partir et vers Dynamics 365 (Common Data Service) ou Dynamics CRM à l’aide d’Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données depuis et vers Microsoft Dynamics 365 et Microsoft Dynamics CRM. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md) qui offre une présentation générale de l’activité d’une copie.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur est pris en charge pour les activités suivantes :

- [Activité de copie](copy-activity-overview.md) avec [prise en charge de la matrice de source/récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données à partir de Dynamics 365 (Common Data Service) ou Dynamics CRM vers toute banque de données réceptrice prise en charge. Vous pouvez également copier des données à partir de n’importe quelle banque de données source prise en charge vers Dynamics 365 (Common Data Service) ou Dynamics CRM. Pour obtenir la liste des magasins de données prises en charge par l'activité de copie en tant que sources et récepteurs, consultez la table [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Ce connecteur Dynamics prend en charge Dynamics versions 7 à 9, aussi bien en ligne qu’en local. Plus précisément :

- La version 7 mappe vers Dynamics CRM 2015.
- La version 8 mappe vers Dynamics CRM 2016 et la version anticipée de Dynamics 365.
- La version 9 mappe vers la dernière version de Dynamics 365.

Consultez le tableau suivant sur les configurations et les types d’authentification pris en charge pour en savoir plus sur les versions et produits Dynamics.

| Versions de Dynamics | Types d’authentification | Exemples de services liés |
|:--- |:--- |:--- |
| Common Data Service <br/><br/> Dynamics 365 (en ligne) <br/><br/> Dynamics CRM (en ligne) | Principal de service Azure Active Directory (Azure AD) <br/><br/> Office 365 | [Service Dynamics Online et Azure AD : principal ou authentification Office 365](#dynamics-365-and-dynamics-crm-online) |
| Dynamics 365 en local avec un déploiement accessible sur Internet (IFD) <br/><br/> Dynamics CRM 2016 local avec IFD <br/><br/> Dynamics CRM 2015 local avec IFD | IFD | [Dynamics local avec IFD et authentification IFD](#dynamics-365-and-dynamics-crm-on-premises-with-ifd) |

Plus spécifiquement pour Dynamics 365, les types d’applications suivants sont pris en charge :

- Dynamics 365 pour les ventes
- Dynamics 365 pour le service client
- Dynamics 365 pour le service après-vente
- Dynamics 365 pour l’automatisation de service de projet
- Dynamics 365 pour le marketing

Ce connecteur ne prend pas en charge d’autres types d’applications comme Finance, Operations et Talent.

Ce connecteur Dynamics se base sur les [outils Dynamics XRM](/dynamics365/customer-engagement/developer/build-windows-client-applications-xrm-tools).

>[!TIP]
>Pour copier des données issues de Dynamics 365 for Finance and Operations, vous pouvez utiliser le [connecteur Dynamics AX](connector-dynamics-ax.md).

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce connecteur avec l’authentification du principal du service Azure AD, vous devez configurer l’authentification S2S (Server-to-Server) dans Common Data Service ou Dynamics. Pour obtenir des instructions détaillées, consultez [cet article](/powerapps/developer/common-data-service/build-web-applications-server-server-s2s-authentication).

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques de Dynamics.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Dynamics sont les suivantes :

### <a name="dynamics-365-and-dynamics-crm-online"></a>Dynamics 365 et Dynamics CRM en ligne

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur « Dynamics », « DynamicsCrm » ou « CommonDataServiceForApps ». | Oui |
| deploymentType | Type de déploiement de l’instance Dynamics. La valeur doit être « En ligne » pour Dynamics en ligne. | Oui |
| serviceUri | URL du service de votre instance Dynamics, la même que celle à laquelle vous accédez à partir du navigateur. Par exemple, « https://\<organization-name>.crm[x].dynamics.com ». | Oui |
| authenticationType | Type d’authentification pour se connecter à un serveur Dynamics. Les valeurs valides sont « AADServicePrincipal » et « Office 365 ». | Oui |
| servicePrincipalId | ID client de l’application Azure AD. | Oui lorsque l’authentification est « AADServicePrincipal » |
| servicePrincipalCredentialType | Type d’informations d’identification à utiliser pour l’authentification de principal du service. Les valeurs valides sont « ServicePrincipalKey » et « ServicePrincipalCert ». | Oui lorsque l’authentification est « AADServicePrincipal » |
| servicePrincipalCredential | Informations d’identification du principal du service. <br/><br/>Quand vous utilisez « ServicePrincipalKey » comme type d’informations d’identification, `servicePrincipalCredential` peut être une chaîne chiffrée par Azure Data Factory lors du déploiement du service lié. Il peut aussi s’agir d’une référence à un secret dans Azure Key Vault. <br/><br/>Quand vous utilisez « ServicePrincipalCert » comme informations d’identification, `servicePrincipalCredential` doit être une référence à un certificat dans Azure Key Vault. | Oui lorsque l’authentification est « AADServicePrincipal » |
| username | Nom d'utilisateur pour la connexion à Dynamics. | Oui, lorsque l’authentification est « Office365 ». |
| mot de passe | Mot de passe du compte d’utilisateur défini pour le nom d'utilisateur. Marquez ce champ avec « SecureString » afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui, lorsque l’authentification est « Office365 ». |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Si aucune valeur n’est spécifiée, la propriété utilise le runtime d'intégration Azure par défaut. | Non pour la source et oui pour le récepteur si le service lié à la source n’a pas de runtime d’intégration |

>[!NOTE]
>Le connecteur Dynamics a précédemment utilisé la propriété facultative **organizationName** pour identifier votre instance Dynamics CRM ou Dynamics 365 en ligne. Pendant qu’il continue de fonctionner, vous êtes invité à spécifier à la place la nouvelle propriété **serviceUri** pour obtenir de meilleures performances pour l’instance de détection.

#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-key-authentication"></a>Exemple : Dynamics en ligne à l’aide du principal de service Azure AD et de l’authentification de la clé

```json
{  
    "name": "DynamicsLinkedService",  
    "properties": {  
        "type": "Dynamics",  
        "typeProperties": {  
            "deploymentType": "Online",  
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",  
            "authenticationType": "AADServicePrincipal",  
            "servicePrincipalId": "<service principal id>",  
            "servicePrincipalCredentialType": "ServicePrincipalKey",  
            "servicePrincipalCredential": "<service principal key>"
        },  
        "connectVia": {  
            "referenceName": "<name of Integration Runtime>",  
            "type": "IntegrationRuntimeReference"  
        }  
    }  
}  
```
#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-certificate-authentication"></a>Exemple : Dynamics en ligne à l’aide du principal de service Azure AD et de l’authentification du certificat

```json
{ 
    "name": "DynamicsLinkedService", 
    "properties": { 
        "type": "Dynamics", 
        "typeProperties": { 
            "deploymentType": "Online", 
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com", 
            "authenticationType": "AADServicePrincipal", 
            "servicePrincipalId": "<service principal id>", 
            "servicePrincipalCredentialType": "ServicePrincipalCert", 
            "servicePrincipalCredential": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<AKV reference>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<certificate name in AKV>" 
            } 
        }, 
        "connectVia": { 
            "referenceName": "<name of Integration Runtime>", 
            "type": "IntegrationRuntimeReference" 
        } 
    } 
} 
```

#### <a name="example-dynamics-online-using-office-365-authentication"></a>Exemple : Dynamics en ligne utilisant l’authentification Office 365

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "typeProperties": {
            "deploymentType": "Online",
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",
            "authenticationType": "Office365",
            "username": "test@contoso.onmicrosoft.com",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="dynamics-365-and-dynamics-crm-on-premises-with-ifd"></a>Dynamics 365 et Dynamics CRM locaux avec IFD

Les propriétés supplémentaires comparables à celles de Dynamics en ligne sont **hostName** et **port** .

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur « Dynamics », « DynamicsCrm » ou « CommonDataServiceForApps ». | Oui. |
| deploymentType | Type de déploiement de l’instance Dynamics. La valeur doit être « OnPremisesWithIfd » pour Dynamics local avec IFD.| Oui. |
| hostName | Nom d’hôte du serveur Dynamics local. | Oui. |
| port | Port du serveur Dynamics local. | Non. La valeur par défaut est 443. |
| organizationName | Nom d’organisation de l’instance Dynamics. | Oui. |
| authenticationType | Type d’authentification pour se connecter au serveur Dynamics. Spécifiez « Ifd » pour Dynamics local avec IFD. | Oui. |
| username | Nom d'utilisateur pour la connexion à Dynamics. | Oui. |
| mot de passe | Le mot de passe du compte d’utilisateur que vous avez défini pour le nom d’utilisateur. Vous pouvez marquer ce champ avec « SecureString » pour le stocker en toute sécurité dans Data Factory. Vous pouvez également stocker le mot de passe dans Key Vault et laisser l’activité de copie l’y récupérer quand vous effectuez une copie de données. Pour plus d’informations, consultez [Stocker des informations d’identification dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui. |
| connectVia | Le [runtime d’intégration](concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Si aucune valeur n’est spécifiée, la propriété utilise le runtime d'intégration Azure par défaut. | Non pour Source et oui pour Récepteur. |

#### <a name="example-dynamics-on-premises-with-ifd-using-ifd-authentication"></a>Exemple : Dynamics local avec IFD utilisant l’authentification IFD

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics on-premises with IFD linked service using IFD authentication",
        "typeProperties": {
            "deploymentType": "OnPremisesWithIFD",
            "hostName": "contosodynamicsserver.contoso.com",
            "port": 443,
            "organizationName": "admsDynamicsTest",
            "authenticationType": "Ifd",
            "username": "test@contoso.onmicrosoft.com",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Dynamics.

Pour copier des données depuis et vers Dynamics, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur « DynamicsEntity », « DynamicsCrmEntity » ou « CommonDataServiceForAppsEntity ». |Oui |
| entityName | Nom logique de l’entité à récupérer. | Non pour la source si « query » est spécifié dans la source de l’activité et oui pour le récepteur |

#### <a name="example"></a>Exemple

```json
{
    "name": "DynamicsDataset",
    "properties": {
        "type": "DynamicsEntity",
        "schema": [],
        "typeProperties": {
            "entityName": "account"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics linked service name>",
            "type": "linkedservicereference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par les types de source et de récepteur Dynamics.

### <a name="dynamics-as-a-source-type"></a>Dynamics en tant que type de source

Pour copier des données depuis Dynamics, les propriétés suivantes sont prises en charge dans la section **source** de l’activité de copie :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source de l’activité de copie doit être définie sur « DynamicsSource », « DynamicsCrmSource » ou « CommonDataServiceForAppsSource ». | Oui |
| query | FetchXML est un langage de requête propriétaire qui est utilisé dans Dynamics en ligne et local. Consultez l’exemple qui suit. Pour en savoir plus, consultez [Générer des requêtes avec FetchXML](/previous-versions/dynamicscrm-2016/developers-guide/gg328332(v=crm.8)). | Non si `entityName` est spécifié dans le jeu de données |

>[!NOTE]
>La colonne PK sera toujours copiée, même si la projection de colonne que vous avez configurée dans la requête FetchXML ne la contient pas.

> [!IMPORTANT]
>- Lorsque vous copiez des données à partir de Dynamics, le mappage de colonnes explicite de Dynamics au récepteur est facultatif. Toutefois, nous recommandons vivement le mappage pour garantir un résultat de copie déterministe.
>- Lorsque Data Factory importe un schéma dans l’interface utilisateur de création, il déduit le schéma. Pour ce faire, il échantillonne les premières lignes du résultat de la requête Dynamics pour initialiser la liste des colonnes sources. Dans ce cas, les colonnes sans valeurs dans les lignes du haut sont omises. Le même comportement s’applique aux exécutions de copies en l’absence d’un mappage explicite. Vous pouvez examiner le mappage et y ajouter des colonnes, ce qui sera respecté pendant l’exécution de la copie.

#### <a name="example"></a>Exemple

```json
"activities":[
    {
        "name": "CopyFromDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DynamicsSource",
                "query": "<FetchXML Query>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sample-fetchxml-query"></a>Exemple de requête FetchXML

```xml
<fetch>
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="marketingonly" />
    <attribute name="modifiedon" />
    <order attribute="modifiedon" descending="false" />
    <filter type="and">
      <condition attribute ="modifiedon" operator="between">
        <value>2017-03-10 18:40:00z</value>
        <value>2017-03-12 20:40:00z</value>
      </condition>
    </filter>
  </entity>
</fetch>
```

### <a name="dynamics-as-a-sink-type"></a>Dynamics comme type de récepteur

Pour copier des données dans Dynamics, les propriétés suivantes sont prises en charge dans la section **récepteur** de l’activité de copie :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du récepteur d’activité de copie doit être définie sur « DynamicsSink », « DynamicsCrmSink » ou « CommonDataServiceForAppsSink ». | Oui. |
| writeBehavior | Comportement d’écriture de l’opération. La valeur doit être « Upsert ». | Oui |
| alternateKeyName | Nom de clé de remplacement défini sur votre entité pour exécuter une opération upsert. | Non. |
| writeBatchSize | Nombre de lignes de données écrites dans Dynamics pour chaque lot. | Non. La valeur par défaut est 10. |
| ignoreNullValues | Indique s’il faut ignorer les valeurs null des données d’entrée autres que les champs clés lors d’une opération d’écriture.<br/><br/>Les valeurs valides sont **TRUE** et **FALSE**  :<ul><li>**TRUE**  : Conserver les données dans l’objet de destination quand vous effectuez une opération upsert ou de mise à jour. Insérer une valeur définie par défaut lorsque vous effectuez une opération insert.</li><li>**FALSE**  : Mettre à jour les données dans l’objet de destination avec la valeur null quand vous effectuez une opération upsert ou de mise à jour. Insérer une valeur null lorsque vous effectuez une opération insert.</li></ul> | Non. La valeur par défaut est **FALSE** . |

>[!NOTE]
>La valeur par défaut du récepteur **writeBatchSize** et de l’activité de copie **[parallelCopies](copy-activity-performance-features.md#parallel-copy)** pour le récepteur Dynamics est de 10. Par conséquent, 100 enregistrements sont soumis simultanément à Dynamics par défaut.

Pour Dynamics 365 (en ligne), il existe une limite de [deux appels simultanés de lot par organisation](/previous-versions/dynamicscrm-2016/developers-guide/jj863631(v=crm.8)#Run-time%20limitations). Si cette limite est dépassée, une exception « Serveur occupé » est levée avant l’exécution de la première requête. Conservez une valeur **writeBatchSize** inférieure ou égale à 10 pour éviter la limitation des appels simultanés.

La combinaison optimale de **writeBatchSize** et **parallelCopies** dépend du schéma de votre entité. Les éléments de schéma incluent le nombre de colonnes, la taille des lignes et le nombre de plug-ins, de flux de travail ou d’activités de flux de travail raccordés à ces appels. Le paramètre par défaut de **writeBatchSize** (10) &times; et **parallelCopies** (10) est la suggestion en fonction du service Dynamics. Cette valeur fonctionne pour la plupart des entités Dynamics, même si elle peut ne pas offrir les meilleures performances. Vous pouvez améliorer les performances en ajustant cette combinaison dans les paramètres de votre activité de copie.

#### <a name="example"></a>Exemple

```json
"activities":[
    {
        "name": "CopyToDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Dynamics output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DynamicsSink",
                "writeBehavior": "Upsert",
                "writeBatchSize": 10,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="data-type-mapping-for-dynamics"></a>Mappage de type de données pour Dynamics

Lorsque vous copiez de données de Dynamics, les tableaux suivants montrent le mappage utilisé entre les types de données Dynamics et les types de données intermédiaires Data Factory. Pour découvrir comment l’activité de copie mappe le schéma et le type de données sources au récepteur, consultez [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

Configurez le type de données Data Factory correspondant dans la structure du jeu de données en fonction du type de données Dynamics de votre source à l’aide des tables de mappage suivantes :

| Type de données Dynamics | Type de données intermédiaires d’Azure Data Factory | Prise en charge en tant que source | Prise en charge en tant que récepteur |
|:--- |:--- |:--- |:--- |
| AttributeTypeCode.BigInt | Long | ✓ | ✓ |
| AttributeTypeCode.Boolean | Boolean | ✓ | ✓ |
| AttributeType.Customer | GUID | ✓ | ✓ (Consultez les [conseils](#writing-data-to-a-lookup-field)) |
| AttributeType.DateTime | Datetime | ✓ | ✓ |
| AttributeType.Decimal | Decimal | ✓ | ✓ |
| AttributeType.Double | Double | ✓ | ✓ |
| AttributeType.EntityName | String | ✓ | ✓ |
| AttributeType.Integer | Int32 | ✓ | ✓ |
| AttributeType.Lookup | GUID | ✓ | ✓ (Consultez les [conseils](#writing-data-to-a-lookup-field)) |
| AttributeType.ManagedProperty | Boolean | ✓ | |
| AttributeType.Memo | String | ✓ | ✓ |
| AttributeType.Money | Decimal | ✓ | ✓ |
| AttributeType.Owner | GUID | ✓ | ✓ (Consultez les [conseils](#writing-data-to-a-lookup-field)) |
| AttributeType.Picklist | Int32 | ✓ | ✓ |
| AttributeType.Uniqueidentifier | GUID | ✓ | ✓ |
| AttributeType.String | String | ✓ | ✓ |
| AttributeType.State | Int32 | ✓ | ✓ |
| AttributeType.Status | Int32 | ✓ | ✓ |

> [!NOTE]
> Les types de données **AttributeType.CalendarRules** , **AttributeType.MultiSelectPicklist** et **AttributeType.PartyList de Dynamics** ne sont pas pris en charge.

## <a name="writing-data-to-a-lookup-field"></a>Écriture de données dans un champ de recherche

Pour écrire des données dans un champ de recherche avec plusieurs cibles comme le client et le propriétaire, suivez les instructions et les exemples suivants :

1. Faites en sorte que votre source contienne à la fois la valeur du champ et le nom de l’entité cible correspondante.
   - Si tous les enregistrements sont mappés à la même entité cible, assurez-vous que l’une des conditions suivantes est présente :
      - Vos données sources comportent une colonne qui stocke le nom de l’entité cible.
      - Vous avez ajouté une colonne supplémentaire dans la source de l’activité de copie pour définir l’entité cible.
   - Si différents enregistrements sont mappés à des entités cibles différentes, assurez-vous que vos données sources contiennent une colonne qui stocke le nom de l’entité cible correspondante.

1. Mappez la valeur et les colonnes de référence d’entité de la source au récepteur. La colonne de référence d’entité doit être mappée à une colonne virtuelle avec le modèle d’affectation de noms spécial `{lookup_field_name}@EntityReference`. La colonne n’existe pas réellement dans Dynamics. Elle est utilisée pour indiquer que cette colonne est la colonne de métadonnées du champ de recherche multicible donné.

Supposons, par exemple, que la source comporte les deux colonnes suivantes :

- La colonne **CustomerField** de type **GUID** , qui est la valeur de clé primaire de l’entité cible dans Dynamics.
- La colonne **Target** de type **String** , qui est le nom logique de l’entité cible.

Supposons également que vous souhaitez copier ces données dans le champ de l’entité Dynamics du récepteur **CustomerField** de type **Customer** .

Dans le mappage de colonne de copie-activité, mappez les deux colonnes comme suit :

- **CustomerField** pour **CustomerField** . Ce mappage est le mappage de champs normal.
- **Target** pour **CustomerField\@EntityReference** . La colonne du récepteur est une colonne virtuelle représentant la référence d’entité. Saisissez les noms de champs dans un mappage, car ils ne s’affichent pas en important des schémas.

![Mappage de colonnes de recherche Dynamics](./media/connector-dynamics-crm-office-365/connector-dynamics-lookup-field-column-mapping.png)

Si tous vos enregistrements sources sont mappés à la même entité cible et que vos données sources ne contiennent pas le nom de l’entité cible, voici un raccourci : dans la source de l’activité de copie, ajoutez une colonne supplémentaire. Nommez la nouvelle colonne à l’aide du modèle `{lookup_field_name}@EntityReference`, définissez la valeur sur le nom de l’entité cible, puis poursuivez le mappage de colonne comme d’habitude. Si vos noms de colonne source et récepteur sont identiques, vous pouvez également ignorer le mappage de colonnes explicite, car l’activité de copie par défaut mappe les colonnes par nom.

![Champ de recherche Dynamics pour ajouter une colonne de référence d’entité](./media/connector-dynamics-crm-office-365/connector-dynamics-add-entity-reference-column.png)

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des magasins de données pris en charge par l’activité Copy dans Data Factory en tant que sources et récepteurs, consultez [Magasins de données pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).