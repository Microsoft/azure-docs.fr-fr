---
title: Journalisation dans les applications MSAL | Azure
titleSuffix: Microsoft identity platform
description: En savoir plus sur la journalisation dans les applications Microsoft Authentication Library (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/11/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: b53a12db9203121d12a69c10aaa81bceab5c1754
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584253"
---
# <a name="logging-in-msal-applications"></a>Journalisation dans les applications MSAL

Les applications Microsoft Authentication Library (MSAL) génèrent des messages de journal qui contribuent à diagnostiquer des problèmes. Une application peut configurer la journalisation avec quelques lignes de code et contrôler de façon personnalisée le niveau de détail, ainsi que permettre de savoir si des données personnelles et d’organisation sont consignées ou non. Nous vous conseillons de créer un rappel de journalisation et de donner aux utilisateurs le moyen d’envoyer des journaux lorsqu’ils rencontrent des problèmes d’authentification.

## <a name="logging-levels"></a>Niveaux de journalisation

MSAL fournit plusieurs niveaux de détail de journalisation :

- Erreur : indique qu’un problème est survenu et qu’une erreur a été générée. À utiliser pour le débogage et l’identification des problèmes.
- Avertissement : Il n’y a pas nécessairement eu une erreur ou une défaillance, mais l’objectif est d’effectuer un diagnostic et de mettre en avant des problèmes éventuels.
- Information : MSAL enregistrera des événements destinés à fournir des informations et pas nécessairement destinés à être débogués.
- Verbose : Par défaut. MSAL enregistre les détails complets du comportement de la bibliothèque.

## <a name="personal-and-organizational-data"></a>Données personnelles et d’organisation

Par défaut, l’enregistreur d’événements MSAL ne capture pas de données personnelles ou d’organisation hautement sensibles. La bibliothèque permet d’activer la journalisation de données personnelles et d’organisation si vous décidez de le faire.

Pour des détails sur la journalisation MSAL dans un langage particulier, choisissez l’onglet correspondant à votre langage :

## <a name="net"></a>[.NET](#tab/dotnet)

## <a name="logging-in-msalnet"></a>Journalisation dans MSAL.NET

 > [!NOTE]
 > Pour des exemples de journalisation MSAL.NET et bien plus, consultez le [wiki MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki).

Dans la MSAL 3.x, la journalisation est définie par l’application sur l’application lors de la création du modificateur de générateur `.WithLogging`. Cette méthode accepte des paramètres optionnels :

- `Level` vous permet de définir le niveau de journalisation souhaité. Si vous le définissez sur Erreurs, vous obtiendrez seulement les erreurs.
- `PiiLoggingEnabled` vous permet de journaliser des données personnelles et d’organisation s’il est défini sur true. Par défaut, il est défini sur false. Votre application n’enregistre donc pas de données personnelles.
- `LogCallback` est défini sur un délégué qui effectue la journalisation. Si `PiiLoggingEnabled` a la valeur true, cette méthode reçoit les messages à deux reprises : une fois avec le paramètre `containsPii` égal à «false» et le message sans données personnelles, et une deuxième fois avec le paramètre `containsPii` égal à la valeur true et le message pouvant contenir des données personnelles. Dans certains cas (lorsque le message ne contient pas de données personnelles), le message doit être le même.
- `DefaultLoggingEnabled` active la journalisation par défaut pour la plateforme. La valeur par défaut est false. Si vous la définissez sur true elle utilise le suivi d’événements dans les applications de bureau/plateforme Windows universelle, NSLog sur iOS et logcat sur Android.

```csharp
class Program
 {
  private static void Log(LogLevel level, string message, bool containsPii)
  {
     if (containsPii)
     {
        Console.ForegroundColor = ConsoleColor.Red;
     }
     Console.WriteLine($"{level} {message}");
     Console.ResetColor();
  }

  static void Main(string[] args)
  {
    var scopes = new string[] { "User.Read" };

    var application = PublicClientApplicationBuilder.Create("<clientID>")
                      .WithLogging(Log, LogLevel.Info, true)
                      .Build();

    AuthenticationResult result = application.AcquireTokenInteractive(scopes)
                                             .ExecuteAsync().Result;
  }
 }
 ```

## <a name="android"></a>[Android](#tab/android)

## <a name="logging-in-msal-for-android-using-java"></a>Journalisation dans MSAL pour Android à l’aide de Java

Activez la journalisation lors de la création de l’application en créant un rappel de journalisation. Le rappel utilise les paramètres suivants :

- `tag` est une chaîne transmise au rappel par la bibliothèque. Il est associé à l’entrée de journal et peut être utilisé pour trier les messages de journalisation.
- `logLevel` vous permet de définir le niveau de journalisation souhaité. Les niveaux de journalisation pris en charge sont les suivants : `Error`, `Warning`, `Info` et `Verbose`.
- `message` correspond au contenu de l’entrée de journal.
- `containsPII` indique si les messages contenant des données personnelles ou organisationnelles sont journalisés. Par défaut, il est défini sur false. Votre application n’enregistre donc pas de données personnelles. Si `containsPII` a la valeur `true`, cette méthode reçoit les messages à deux reprises : une fois avec le paramètre `containsPII` défini sur `false` et le `message` sans données personnelles, et une deuxième fois avec le paramètre `containsPii` défini sur `true` et le message pouvant contenir des données personnelles. Dans certains cas (lorsque le message ne contient pas de données personnelles), le message doit être le même.

```java
private StringBuilder mLogs;

mLogs = new StringBuilder();
Logger.getInstance().setExternalLogger(new ILoggerCallback()
{
   @Override
   public void log(String tag, Logger.LogLevel logLevel, String message, boolean containsPII)
   {
      mLogs.append(message).append('\n');
   }
});
```

Par défaut, l’enregistreur d’événements MSAL ne capture pas les informations d’identification personnelles ou informations d’identification organisationnelles.
Pour activer la journalisation des informations d’identification personnelles et des informations d’identification organisationnelles :

```java
Logger.getInstance().setEnablePII(true);
```

Pour désactiver la journalisation des données personnelles et organisationnelles :

```java
Logger.getInstance().setEnablePII(false);
```

Par défaut, la journalisation dans logcat est désactivée. Pour activer :

```java
Logger.getInstance().setEnableLogcatLog(true);
```

## <a name="javascript"></a>[JavaScript](#tab/javascript)

 Activez la journalisation dans MSAL.js (JavaScript) en passant un objet enregistreur d’événements lors de la configuration pour la création d’une instance `UserAgentApplication`. Cet objet enregistreur d'événements a les propriétés suivantes :

- `localCallback` : une instance de rappel qui peut être fournie par le développeur pour consommer et publier des journaux de manière personnalisée. Implémentez la méthode localCallback en fonction de la façon dont vous souhaitez rediriger les journaux.
- `level` (facultatif) : le niveau de journalisation configurable. Les niveaux de journalisation pris en charge sont les suivants : `Error`, `Warning`, `Info` et `Verbose`. Par défaut, il s’agit de `Info`.
- `piiLoggingEnabled` (facultatif) : s'il est défini sur true, vous permet de journaliser des données personnelles et organisationnelles. Par défaut, il est défini sur false. Votre application n’enregistre donc pas de données personnelles. Les journaux de données personnelles ne sont jamais écrits dans les sorties par défaut telles que Console, Logcat ou NSLog.
- `correlationId` (facultatif) : un identificateur unique utilisé pour mapper la requête avec la réponse à des fins de débogage. Guide des valeurs par défaut de RFC4122, version 4 (128 bits).

```javascript
function loggerCallback(logLevel, message, containsPii) {
   console.log(message);
}

var msalConfig = {
    auth: {
        clientId: "<Enter your client id>",
    },
    system: {
        logger: new Msal.Logger(
            loggerCallback , {
                level: Msal.LogLevel.Verbose,
                piiLoggingEnabled: false,
                correlationId: '1234'
            }
        )
    }
}

var UserAgentApplication = new Msal.UserAgentApplication(msalConfig);
```

## <a name="objective-c"></a>[Objective-C](#tab/objc)

## <a name="msal-for-ios-and-macos-logging-objc"></a>Journalisation MSAL pour iOS et macOS - ObjC

Définissez un rappel pour capturer la journalisation MSAL et l’incorporer dans la journalisation de votre propre application. La signature du rappel se présente comme suit :

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Par exemple :

```objc
[MSALGlobalConfig.loggerConfig setLogCallback:^(MSALLogLevel level, NSString *message, BOOL containsPII)
    {
        if (!containsPII)
        {
#if DEBUG
            // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
            NSLog(@"MSAL log: %@", message);
#endif
        }
    }];
```

### <a name="personal-data"></a>Données à caractère personnel

Par défaut, MSAL ne capture et n’enregistre aucune information d’identification personnelle (PII). La bibliothèque permet aux développeurs d’applications d’activer cette fonctionnalité via une propriété de la classe MSALLogger. Lorsque vous activez `pii.Enabled`, l’application devient responsable de la gestion de façon sécurisé des données hautement sensibles et du suivi des exigences réglementaires.

```objc
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = YES;

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = NO;
```

### <a name="logging-levels"></a>Niveaux de journalisation

Pour définir le niveau de journalisation lorsque vous utilisez MSAL pour iOS et macOS, optez pour l’une des valeurs suivantes :

|Level  |Description |
|---------|---------|
| `MSALLogLevelNothing`| Désactiver la journalisation |
| `MSALLogLevelError` | Niveau par défaut, affiche uniquement les informations lorsque des erreurs se produisent |
| `MSALLogLevelWarning` | Avertissements |
| `MSALLogLevelInfo` |  Points d’entrée de bibliothèque, avec paramètres et diverses opérations de trousseau |
|`MSALLogLevelVerbose`     |  Suivi API |

Par exemple :

```objc
MSALGlobalConfig.loggerConfig.logLevel = MSALLogLevelVerbose;
 ```

 ### <a name="log-message-format"></a>Format de message de journal

La partie message des messages de journal MSAL se présente au format `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`

Par exemple :

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Les ID de corrélation et les timestamps facilitent le suivi des problèmes. Les informations de timestamp et d’ID de corrélation sont disponibles dans le message de journal. Seuls les messages de journalisation MSAL permettent de les récupérer de manière fiable.

## <a name="swift"></a>[Swift](#tab/swift)

## <a name="msal-for-ios-and-macos-logging-swift"></a>Journalisation MSAL pour iOS et macOS - Swift

Définissez un rappel pour capturer la journalisation MSAL et l’incorporer dans la journalisation de votre propre application. La signature (représentée en Objective-C) pour le rappel ressemble à ceci :

```objc
/*!
    The LogCallback block for the MSAL logger
 
    @param  level           The level of the log message
    @param  message         The message being logged
    @param  containsPII     If the message might contain Personally Identifiable Information (PII)
                            this will be true. Log messages possibly containing PII will not be
                            sent to the callback unless PIllLoggingEnabled is set to YES on the
                            logger.

 */
typedef void (^MSALLogCallback)(MSALLogLevel level, NSString *message, BOOL containsPII);
```

Par exemple :

```swift
MSALGlobalConfig.loggerConfig.setLogCallback { (level, message, containsPII) in
    if let message = message, !containsPII
    {
#if DEBUG
    // IMPORTANT: MSAL logs may contain sensitive information. Never output MSAL logs with NSLog, or print, directly unless you're running your application in debug mode. If you're writing MSAL logs to file, you must store the file securely.
    print("MSAL log: \(message)")
#endif
    }
}
```

### <a name="personal-data"></a>Données à caractère personnel

Par défaut, MSAL ne capture et n’enregistre aucune information d’identification personnelle (PII). La bibliothèque permet aux développeurs d’applications d’activer cette fonctionnalité via une propriété de la classe MSALLogger. Lorsque vous activez `pii.Enabled`, l’application devient responsable de la gestion de façon sécurisé des données hautement sensibles et du suivi des exigences réglementaires.

```swift
// By default, the `MSALLogger` doesn't capture any PII

// PII will be logged
MSALGlobalConfig.loggerConfig.piiEnabled = true

// PII will NOT be logged
MSALGlobalConfig.loggerConfig.piiEnabled = false
```

### <a name="logging-levels"></a>Niveaux de journalisation

Pour définir le niveau de journalisation lorsque vous utilisez MSAL pour iOS et macOS, optez pour l’une des valeurs suivantes :

|Level  |Description |
|---------|---------|
| `MSALLogLevelNothing`| Désactiver la journalisation |
| `MSALLogLevelError` | Niveau par défaut, affiche uniquement les informations lorsque des erreurs se produisent |
| `MSALLogLevelWarning` | Avertissements |
| `MSALLogLevelInfo` |  Points d’entrée de bibliothèque, avec paramètres et diverses opérations de trousseau |
|`MSALLogLevelVerbose`     |  Suivi API |

Par exemple :

```swift
MSALGlobalConfig.loggerConfig.logLevel = .verbose
 ```

### <a name="log-message-format"></a>Format de message de journal

La partie message des messages de journal MSAL se présente au format `TID = <thread_id> MSAL <sdk_ver> <OS> <OS_ver> [timestamp - correlation_id] message`

Par exemple :

`TID = 551563 MSAL 0.2.0 iOS Sim 12.0 [2018-09-24 00:36:38 - 36764181-EF53-4E4E-B3E5-16FE362CFC44] acquireToken returning with error: (MSALErrorDomain, -42400) User cancelled the authorization session.`

Les ID de corrélation et les timestamps facilitent le suivi des problèmes. Les informations de timestamp et d’ID de corrélation sont disponibles dans le message de journal. Seuls les messages de journalisation MSAL permettent de les récupérer de manière fiable.

## <a name="java"></a>[Java](#tab/java)

## <a name="msal-for-java-logging"></a>Journalisation MSAL pour Java

MSAL pour Java vous permet d’utiliser la bibliothèque de journalisation que vous utilisez déjà avec votre application, à condition qu’elle soit compatible avec SLF4J. MSAL pour Java utilise la [façade de journalisation simple pour Java](http://www.slf4j.org/) (SLF4J) comme façade ou abstraction simple pour différents frameworks de journalisation tels que [java.util.logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html), [Logback](http://logback.qos.ch/) et [log4j](https://logging.apache.org/log4j/2.x/). SLF4J permet à l’utilisateur de brancher le framework de journalisation souhaité au moment du déploiement.

Par exemple, pour utiliser Logback comme framework de journalisation dans votre application, ajoutez la dépendance Logback au fichier pom Maven pour votre application :

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```

Ajoutez ensuite le fichier de configuration Logback :

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">

</configuration>
```

SLF4J est automatiquement lié à Logback au moment du déploiement. Les journaux MSAL seront écrits dans la console.

Pour obtenir des instructions sur la liaison à d’autres frameworks de journalisation, consultez le [Manuel SLF4J](http://www.slf4j.org/manual.html).

### <a name="personal-and-organization-information"></a>Informations personnelles et propres à l’organisation

Par défaut, la journalisation MSAL ne capture et ne consigne dans un journal aucune donnée personnelle ou propres à une organisation. Dans l’exemple suivant, la journalisation des données personnelles ou organisationnelles est désactivée par défaut :

```java
    PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
            .authority(AUTHORITY)
            .build();
```

Activez la journalisation des données personnelles et organisationnelles en définissant `logPii()` sur le générateur d’applications clientes. Si vous activez la journalisation des informations d’identification personnelles ou propres à l’organisation, votre application doit garantir la gestion de façon sécurisée des données hautement sensibles, et respecter les exigences réglementaires.

Dans l’exemple suivant, la journalisation des informations d’identification personnelles ou organisationnelles est activée :

```java
PublicClientApplication app2 = PublicClientApplication.builder(PUBLIC_CLIENT_ID)
        .authority(AUTHORITY)
        .logPii(true)
        .build();
```

## <a name="python"></a>[Python](#tab/python)

## <a name="msal-for-python-logging"></a>Journalisation MSAL pour Python

La journalisation dans MSAL Python utilise le mécanisme de journalisation Python standard, par exemple `logging.info("msg")`. Vous pouvez configurer la journalisation MSAL comme suit (et la voir en action dans [username_password_sample](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.0.0/sample/username_password_sample.py#L31L32)) :

### <a name="enable-debug-logging-for-all-modules"></a>Activer la journalisation du débogage pour tous les modules

Par défaut, la journalisation dans n’importe quel script Python est désactivée. Si vous souhaitez activer la journalisation du débogage pour tous les modules dans l’intégralité de votre script Python, utilisez :

```python
logging.basicConfig(level=logging.DEBUG)
```

### <a name="silence-only-msal-logging"></a>Mettre au silence uniquement la journalisation MSAL

Pour mettre au silence uniquement la journalisation de la bibliothèque MSAL, tout en activant la journalisation du débogage dans tous les autres modules de votre script Python, désactivez le journaliseur utilisé par MSAL Python :

```Python
logging.getLogger("msal").setLevel(logging.WARN)
```

### <a name="personal-and-organizational-data-in-python"></a>Données personnelles et de l’organisation dans Python

MSAL pour Python ne journalise pas les données personnelles ni les données de l’organisation. Il n’existe aucune propriété pour activer ou désactiver la journalisation des données personnelles ou de l’organisation.

Vous pouvez utiliser la journalisation Python standard pour consigner ce que vous voulez, mais vous êtes responsable de la gestion des données sensibles et du respect des conditions réglementaires.

Pour plus d’informations sur la journalisation dans Python, consultez [Logging HOWTO](https://docs.python.org/3/howto/logging.html#logging-basic-tutorial) dans la documentation de Python.

---
