---
title: Comprendre les concepts du référentiel de modèles d’appareil | Microsoft Docs
description: En tant que développeur de solutions ou professionnel de l’informatique, découvrez les concepts de base du référentiel de modèles d’appareil.
author: rido-min
ms.author: rmpablos
ms.date: 11/17/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: b567efe2541bb33c905def73bb78398799b4ed69
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920540"
---
# <a name="device-model-repository"></a>Référentiel de modèles d’appareil

Le référentiel de modèles d’appareil (DMR) permet aux fabricants d’appareils de gérer et de partager les modèles d’appareil IoT Plug-and-Play. Les modèles d’appareil sont des documents LD JSON définis à l’aide du [langage DTDL (Digital Twins Modeling Language)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

Le système DMR définit un modèle pour stocker les interfaces DTDL dans une structure de dossiers en fonction de l’identificateur de modèle de jumeau d’appareil (DTMI). Vous pouvez localiser une interface dans le DMR en convertissant le DTMI en chemin d’accès relatif. Par exemple, le DTMI `dtmi:com:example:Thermostat;1` se traduit par `/dtmi/com/example/thermostat-1.json`.

## <a name="public-device-model-repository"></a>Référentiel de modèles d’appareil public

Microsoft héberge un DMR public avec les caractéristiques suivantes :

- Modèles organisés. Microsoft révise et approuve toutes les interfaces disponibles à l’aide d’un workflow de validation des demandes de tirage (PR, pull request) GitHub.
- Immuabilité.  Une fois publiée, une interface ne peut pas être mise à jour.
- Hyperscale. Microsoft fournit l’infrastructure nécessaire pour créer un point de terminaison sécurisé et évolutif où vous pouvez publier et utiliser des modèles d’appareil.

## <a name="custom-device-model-repository"></a>Référentiel de modèles d’appareil personnalisé

Utilisez le même modèle DMR pour créer un DMR personnalisé dans n’importe quel support de stockage, comme un système de fichiers local ou des serveurs web HTTP personnalisés. Vous pouvez récupérer des modèles d’appareil à partir du DMR personnalisé de la même façon que vous le faites à partir du DRM public, en modifiant l’URL de base utilisée pour accéder au DMR.

> [!NOTE]
> Microsoft fournit des outils pour valider les modèles d’appareil dans le DMR public. Vous pouvez réutiliser ces outils dans des référentiels personnalisés.

## <a name="public-models"></a>Modèles publics

Les modèles d’appareil publics stockés dans le référentiel de modèles sont librement accessibles pour que tous les utilisent et les intègrent à leurs applications. Les modèles d’appareil publics permettent aux fabricants d’appareils et aux développeurs de solutions de partager et de réutiliser leurs modèles d’appareils IoT Plug-and-Play dans un écosystème ouvert.

Pour obtenir des instructions sur la publication d’un modèle dans le référentiel de modèles afin de le rendre public, reportez-vous à la section [Publier un modèle](#publish-a-model).

Les utilisateurs peuvent parcourir, rechercher et afficher des interfaces publiques à partir du [référentiel GitHub](https://github.com/Azure/iot-plugandplay-models) officiel.

Toutes les interfaces dans les dossiers `dtmi` sont également disponibles à partir du point de terminaison public [https://devicemodels.azure.com](https://devicemodels.azure.com)

### <a name="resolve-models"></a>Résoudre des modèles

Pour accéder à ces interfaces par programmation, vous devez convertir un DTMI en un chemin d’accès relatif que vous pouvez utiliser pour interroger le point de terminaison public.

Pour convertir un DTMI en chemin d’accès absolu, utilisez la fonction `DtmiToPath` avec `IsValidDtmi` :

```cs
static string DtmiToPath(string dtmi)
{
    if (!IsValidDtmi(dtmi))
    {
        return null;
    }
    // dtmi:com:example:Thermostat;1 -> dtmi/com/example/thermostat-1.json
    return $"/{dtmi.ToLowerInvariant().Replace(":", "/").Replace(";", "-")}.json";
}

static bool IsValidDtmi(string dtmi)
{
    // Regex defined at https://github.com/Azure/digital-twin-model-identifier#validation-regular-expressions
    Regex rx = new Regex(@"^dtmi:[A-Za-z](?:[A-Za-z0-9_]*[A-Za-z0-9])?(?::[A-Za-z](?:[A-Za-z0-9_]*[A-Za-z0-9])?)*;[1-9][0-9]{0,8}$");
    return rx.IsMatch(dtmi);
}
```

Avec le chemin d’accès obtenu et l’URL de base du référentiel, nous pouvons obtenir l’interface :

```cs
const string _repositoryEndpoint = "https://devicemodels.azure.com";

string dtmiPath = DtmiToPath(dtmi.ToString());
string fullyQualifiedPath = $"{_repositoryEndpoint}{dtmiPath}";
string modelContent = await _httpClient.GetStringAsync(fullyQualifiedPath);
```

## <a name="publish-a-model"></a>Publier un modèle

> [!Important]
> Vous devez disposer d’un compte GitHub pour pouvoir envoyer des modèles au DMR public.

1. Dupliquez le référentiel GitHub public : [https://github.com/Azure/iot-plugandplay-models](https://github.com/Azure/iot-plugandplay-models).
1. Clonez le référentiel dupliqué. Facultativement, vous pouvez créer une branche pour que vos modifications restent isolées de la branche `main`.
1. Ajoutez les nouvelles interfaces au dossier `dtmi` à l’aide de la convention nom de dossier/nom de fichier. Pour plus d’informations, consultez [Importer un modèle dans le `dtmi/`dossier](#import-a-model-to-the-dtmi-folder).
1. Validez les modèles localement à l’aide de l’outil `dmr-client`. Pour plus d’informations, consultez [Valider les modèles](#validate-models).
1. Validez les modifications localement et envoyez vers votre duplication.
1. À partir de votre duplication (fork), créez une PR qui cible la branche `main`. Consultez les documents relatifs à la [création d’une requête de problème ou d’une demande de tirage (pull request)](https://docs.github.com/free-pro-team@latest/desktop/contributing-and-collaborating-using-github-desktop/creating-an-issue-or-pull-request).
1. Passez en revue les [conditions requises pour une demande de tirage](https://github.com/Azure/iot-plugandplay-models/blob/main/pr-reqs.md).

La PR déclenche un ensemble d’actions GitHub qui valident les interfaces envoyées et s’assurent que votre PR est conforme à toutes les exigences.

Microsoft répond à une PR avec toutes les vérifications dans les trois jours ouvrables.

### <a name="dmr-client-tools"></a>Outils `dmr-client`

Les outils utilisés pour valider les modèles lors des vérifications de PR permettent également d’ajouter et de valider les interfaces DTDL localement.

> [!NOTE]
> Cet outil requiert le [kit de développement logiciel (SDK) .NET](https://dotnet.microsoft.com/download) version 3.1 ou ultérieure.

### <a name="install-dmr-client"></a>Installer `dmr-client`

```bash
curl -L https://aka.ms/install-dmr-client-linux | bash
```

```powershell
iwr https://aka.ms/install-dmr-client-windows -UseBasicParsing | iex
```

### <a name="import-a-model-to-the-dtmi-folder"></a>Importer un modèle dans le dossier `dtmi/`

Si votre modèle est déjà stocké dans des fichiers JSON, vous pouvez utiliser la commande `dmr-client import` pour les ajouter au dossier `dtmi/` avec les noms de fichiers qui conviennent :

```bash
# from the local repo root folder
dmr-client import --model-file "MyThermostat.json"
```

> [!TIP]
> Vous pouvez utiliser l’argument `--local-repo` pour spécifier le dossier racine du référentiel local.

### <a name="validate-models"></a>Valider les modèles

Vous pouvez valider vos modèles à l’aide de la commande `dmr-client validate` :

```bash
dmr-client validate --model-file ./my/model/file.json
```

> [!NOTE]
> La validation utilise la dernière version de l’analyseur DTDL afin de garantir la compatibilité de toutes les interfaces avec la spécification du langage DTDL.

Pour valider les dépendances externes, celles-ci doivent être présentes dans le référentiel local. Pour valider les modèles, utilisez l’option `--repo` afin de spécifier un dossier `local` ou `remote` et résoudre les dépendances :

```bash
# from the repo root folder
dmr-client validate --model-file ./my/model/file.json --repo .
```

### <a name="strict-validation"></a>Validation stricte

Le DMR contient des [exigences](https://github.com/Azure/iot-plugandplay-models/blob/main/pr-reqs.md) supplémentaires. Utilisez l’indicateur `stict` pour valider votre modèle par rapport à celles-ci :

```bash
dmr-client validate --model-file ./my/model/file.json --repo . --strict true
```

Vérifiez la sortie de la console pour tout message d’erreur.

### <a name="export-models"></a>Exporter des modèles

Les modèles peuvent être exportés à partir d’un référentiel donné (local ou distant) vers un fichier unique à l’aide d’un tableau JSON :

```bash
dmr-client export --dtmi "dtmi:com:example:TemperatureController;1" -o TemperatureController.expanded.json
```

## <a name="next-steps"></a>Étapes suivantes

Nous vous recommandons maintenant de consulter [Architecture IoT Plug-and-Play](concepts-architecture.md).
