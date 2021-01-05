---
title: Copier des données depuis Office 365 avec Azure Data Factory
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données d’Office 365 vers des banques de données réceptrices prises en charge.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/20/2019
ms.author: jingwang
ms.openlocfilehash: 365896fec555340c3932192aa82086d140d4db0c
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92632989"
---
# <a name="copy-data-from-office-365-into-azure-using-azure-data-factory"></a>Copier des données dans Azure depuis Office 365 avec Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Azure Data Factory s’intègre avec [Microsoft Graph data connect](/graph/data-connect-concept-overview) et vous permet de transférer de façon progressive vers Azure les données de votre organisation présentes dans votre locataire Office 365, de créer des applications d’analytique et d’extraire des insights basés sur ces riches ressources de données. L’intégration avec Privileged Access Management fournit un contrôle d’accès sécurisé pour les données organisées dans Office 365.  Reportez-vous à [ce lien](/graph/data-connect-concept-overview) pour une vue d’ensemble sur Microsoft Graph data connect et à [ce lien](/graph/data-connect-policies#licensing) pour les informations de licence.

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour copier des données d’Office 365. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge
Le connecteur ADF Office 365 et Microsoft Graph data connect favorisent l’ingestion à grande échelle de différents types de jeux de données à partir de boîtes aux lettres prenant en charge l’e-mail Exchange, notamment le carnet d’adresses, les événements de calendrier, les messages, les informations utilisateur, les paramètres de boîte aux lettres, etc.  Reportez-vous [ici](/graph/data-connect-datasets) pour voir la liste complète des jeux de données disponibles.

À ce stade, au sein d’une seule activité de copie, vous pouvez seulement **copier des données depuis Office 365 dans [Stockage Blob Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) au format JSON** (type setOfObjects). Si vous voulez charger des données Office 365 dans d’autres types de banques de données ou dans d’autres formats, vous pouvez faire suivre la première activité de copie d’une autre activité de copie pour charger les données dans une des [banques de données de destination ADF prises en charge](copy-activity-overview.md#supported-data-stores-and-formats) (reportez-vous à la colonne « prise en charge en tant que récepteur »du tableau « Banques de données et formats pris en charge »).

>[!IMPORTANT]
>- L’abonnement Azure contenant la fabrique de données et la banque de données réceptrice doit être sous le même locataire Azure Active Directory (Azure AD) que le locataire Office 365.
>- Vérifiez que la région Azure Integration Runtime utilisée pour l’activité de copie et la destination se trouvent dans la même région que celle où se trouve la boîte aux lettres des utilisateurs du locataire Office 365. Pour comprendre comment l’emplacement d’Azure Integration Runtime est déterminé, suivez [ce lien](concepts-integration-runtime.md#integration-runtime-location). Consultez [ce tableau](/graph/data-connect-datasets#regions) pour obtenir la liste des régions Office et des régions Azure correspondantes prises en charge.
>- L’authentification d’un principal de service est le seul mécanisme d’authentification pris en charge pour Stockage Blob Azure, Azure Data Lake Storage Gen1 et Azure Data Lake Storage Gen2 comme magasins de destination.

## <a name="prerequisites"></a>Prérequis

Pour copier des données depuis Office 365 dans Azure, vous devez effectuer les étapes prérequises suivantes :

- L’administrateur de votre locataire Office 365 doit effectuer des actions d’intégration comme décrit [ici](/graph/data-connect-get-started).
- Créez et configurez une application web Azure AD dans Azure Active Directory.  Pour obtenir des instructions, consultez la page [Créer une application Azure AD](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal).
- Prenez note des valeurs suivantes, que vous utiliserez afin de définir le service lié pour Office 365 :
    - ID de locataire. Pour obtenir des instructions, consultez [Obtenir l’ID de locataire](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
    - ID d’application et clé d’application.  Pour obtenir des instructions, consultez [Obtenir un ID d’application et une clé d’authentification](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in).
- Ajoutez l’identité de l’utilisateur effectuant la demande d’accès aux données en tant que propriétaire de l’application web Azure AD (à partir de l’application web Azure AD > Paramètres > Propriétaires > Ajouter un propriétaire). 
    - L’identité de l’utilisateur doit être dans l’organisation Office 365 à partir de laquelle vous obtenez des données et ne doit pas être un utilisateur Invité.

## <a name="approving-new-data-access-requests"></a>Approbation de nouvelles demandes d’accès aux données

Si c’est la première fois que vous demandez des données pour ce contexte (une combinaison de la table de données consultée, du compte de destination dans lequel les données sont chargées et de l’identité de l’utilisateur à l’origine de la demande d’accès aux données), vous voyez l’activité de copie avec l’état « En cours d’exécution », et l’état ne devient « RequestingConsent » que quand vous cliquez sur le [lien « Détails » sous Actions](copy-activity-overview.md#monitoring).  Un membre du groupe d’approbateurs pour l’accès aux données doit approuver la requête dans Privileged Access Management pour permettre l’extraction de données.

Cliquez [ici](/graph/data-connect-tips#approve-pam-requests-via-office-365-admin-portal) pour obtenir des informations sur la façon dont l’approbateur peut approuver les demandes d’accès aux données et cliquez [ici](/graph/data-connect-pam) pour obtenir une explication sur l’intégration globale à Privileged Access Management, notamment comment configurer le groupe d’approbateurs d’accès aux données.

## <a name="policy-validation"></a>Validation de la stratégie

Si ADF est créé dans le cadre d’une application managée et que les affectations de stratégies Azure sont effectuées sur des ressources au sein du groupe de ressources de gestion, puis pour chaque exécution de l’activité de copie, ADF vérifie que les affectations de stratégie sont appliquées. Cliquez [ici](/graph/data-connect-policies#policies) pour obtenir la liste des stratégies prises en charge.

## <a name="getting-started"></a>Prise en main

>[!TIP]
>Pour obtenir une procédure pas à pas de l’utilisation du connecteur Office 365, consultez l’article [Charger des données depuis Office 365](load-office-365-data.md).

Vous pouvez créer un pipeline avec l’activité de copie à l’aide d’un des outils ou kits de développement logiciel suivants. Sélectionnez un lien pour accéder à un didacticiel contenant des instructions détaillées pour créer un pipeline avec une activité de copie. 

- [Azure portal](quickstart-create-data-factory-portal.md)
- [Kit de développement logiciel (SDK) .NET](quickstart-create-data-factory-dot-net.md)
- [Kit de développement logiciel (SDK) Python](quickstart-create-data-factory-python.md)
- [Azure PowerShell](quickstart-create-data-factory-powershell.md)
- [REST API](quickstart-create-data-factory-rest-api.md)
- [Modèle Azure Resource Manager](quickstart-create-data-factory-resource-manager-template.md) 

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur Office 365.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Office 365 sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **Office365** | Oui |
| office365TenantId | ID de locataire Azure auquel le compte Office 365 appartient. | Oui |
| servicePrincipalTenantId | Spécifiez les informations du locataire où se trouve votre application web Azure AD. | Oui |
| servicePrincipalId | Spécifiez l’ID client de l’application. | Oui |
| servicePrincipalKey | Spécifiez la clé de l’application. Marquez ce champ comme SecureString pour le stocker de façon sécurisée dans Data Factory. | Oui |
| connectVia | Runtime d’intégration à utiliser pour la connexion à la banque de données.  À défaut de spécification, le runtime d’intégration Azure par défaut est utilisé. | Non |

>[!NOTE]
> La différence entre **office365TenantId** et **servicePrincipalTenantId** , et la valeur correspondante à fournir :
>- Si vous êtes développeur en entreprise et que vous développez une application avec des données Office 365 pour l’utiliser au sein de votre propre organisation, vous devez fournir le même ID de locataire pour les deux propriétés, qui est l’ID de locataire AAD de votre organisation.
>- Si vous êtes développeur chez un éditeur de logiciels indépendant et que vous développez une application pour vos clients, office365TenantId est l’ID de locataire AAD (programme d’installation de l’application) de votre client et servicePrincipalTenantId est l’ID de locataire AAD de votre entreprise.

**Exemple :**

```json
{
    "name": "Office365LinkedService",
    "properties": {
        "type": "Office365",
        "typeProperties": {
            "office365TenantId": "<Office 365 tenant id>",
            "servicePrincipalTenantId": "<AAD app service principal tenant id>",
            "servicePrincipalId": "<AAD app service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<AAD app service principal key>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md). Cette section contient la liste des propriétés prises en charge par le jeu de données Office 365.

Pour copier des données depuis Office 365, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **Office365Table** | Oui |
| tableName | Nom du jeu de données à extraire d’Office 365. Cliquez [ici](/graph/data-connect-datasets#datasets) pour obtenir la liste des jeux de données Office 365 disponibles pour l’extraction. | Oui |

Si vous avez défini `dateFilterColumn`, `startTime`, `endTime` et `userScopeFilterUri` dans le jeu de données, il reste pris en charge tel quel, mais nous vous suggérons d’utiliser le nouveau modèle dans la source d’activité à l’avenir.

**Exemple**

```json
{
    "name": "DS_May2019_O365_Message",
    "properties": {
        "type": "Office365Table",
        "linkedServiceName": {
            "referenceName": "<Office 365 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [],
        "typeProperties": {
            "tableName": "BasicDataSet_v0.Event_v1"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section contient la liste des propriétés prises en charge par une source Office 365.

### <a name="office-365-as-source"></a>Office 365 comme source

Pour copier des données à partir d’Office 365, les propriétés prises en charge dans la section **source** de l'activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur : **Office365Source** | Oui |
| allowedGroups | Prédicat de sélection de groupe.  Cette propriété permet de sélectionner jusqu'à 10 groupes d’utilisateurs pour lesquels les données seront récupérées.  Si aucun groupe n’est spécifié, les données sont retournées pour toute l’organisation. | Non |
| userScopeFilterUri | Lorsque la propriété `allowedGroups` n’est pas spécifiée, vous pouvez utiliser une expression de prédicat appliquée sur l’ensemble du locataire pour filtrer les lignes spécifiques à extraire d’Office 365. Le format de prédicat doit correspondre au format de requête des API Microsoft Graph, par exemple, `https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'`. | Non |
| dateFilterColumn | Nom de la colonne de filtre de date/heure. Cette propriété permet de limiter l’intervalle de temps pendant lequel les données Office 365 sont extraites. | Oui, si le jeu de données comporte une ou plusieurs colonnes de date/heure. Reportez-vous [ici](/graph/data-connect-filtering#filtering) pour obtenir la liste des jeux de données qui nécessitent ce filtre de date/heure. |
| startTime | Valeur de date/heure de début sur laquelle filtrer. | Oui, si `dateFilterColumn` est spécifié |
| endTime | Valeur de date/heure de fin sur laquelle filtrer. | Oui, si `dateFilterColumn` est spécifié |
| outputColumns | Tableau des colonnes à copier dans le récepteur. | Non |

**Exemple :**

```json
"activities": [
    {
        "name": "CopyFromO365ToBlob",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Office 365 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "Office365Source",
                "dateFilterColumn": "CreatedDateTime",
                "startTime": "2019-04-28T16:00:00.000Z",
                "endTime": "2019-05-05T16:00:00.000Z",
                "userScopeFilterUri": "https://graph.microsoft.com/v1.0/users?$filter=Department eq 'Finance'",
                "outputColumns": [
                    {
                        "name": "Id"
                    },
                    {
                        "name": "CreatedDateTime"
                    },
                    {
                        "name": "LastModifiedDateTime"
                    },
                    {
                        "name": "ChangeKey"
                    },
                    {
                        "name": "Categories"
                    },
                    {
                        "name": "OriginalStartTimeZone"
                    },
                    {
                        "name": "OriginalEndTimeZone"
                    },
                    {
                        "name": "ResponseStatus"
                    },
                    {
                        "name": "iCalUId"
                    },
                    {
                        "name": "ReminderMinutesBeforeStart"
                    },
                    {
                        "name": "IsReminderOn"
                    },
                    {
                        "name": "HasAttachments"
                    },
                    {
                        "name": "Subject"
                    },
                    {
                        "name": "Body"
                    },
                    {
                        "name": "Importance"
                    },
                    {
                        "name": "Sensitivity"
                    },
                    {
                        "name": "Start"
                    },
                    {
                        "name": "End"
                    },
                    {
                        "name": "Location"
                    },
                    {
                        "name": "IsAllDay"
                    },
                    {
                        "name": "IsCancelled"
                    },
                    {
                        "name": "IsOrganizer"
                    },
                    {
                        "name": "Recurrence"
                    },
                    {
                        "name": "ResponseRequested"
                    },
                    {
                        "name": "ShowAs"
                    },
                    {
                        "name": "Type"
                    },
                    {
                        "name": "Attendees"
                    },
                    {
                        "name": "Organizer"
                    },
                    {
                        "name": "WebLink"
                    },
                    {
                        "name": "Attachments"
                    },
                    {
                        "name": "BodyPreview"
                    },
                    {
                        "name": "Locations"
                    },
                    {
                        "name": "OnlineMeetingUrl"
                    },
                    {
                        "name": "OriginalStart"
                    },
                    {
                        "name": "SeriesMasterId"
                    }
                ]
            },
            "sink": {
                "type": "BlobSink"
            }
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).