---
title: Validation des entrées - Outil Microsoft de modélisation des menaces - Azure | Microsoft Docs
description: En savoir plus sur la validation des entrées dans l’outil Threat Modeling Tool. Voir les informations d’atténuation et les exemples de code.
services: security
documentationcenter: na
author: jegeib
manager: jegeib
editor: jegeib
ms.assetid: na
ms.service: security
ms.subservice: security-develop
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/07/2017
ms.author: jegeib
ms.custom: devx-track-csharp
ms.openlocfilehash: c816fbad05831c83c891c70849986b38cb7fdbeb
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589540"
---
# <a name="security-frame-input-validation--mitigations"></a>Infrastructure de sécurité : validation des entrées | Mesures d’atténuation 
| Produit/Service | Article |
| --------------- | ------- |
| **Application Web** | <ul><li>[Désactiver les scripts XSLT pour toutes les transformations à l’aide de feuilles de style non approuvées](#disable-xslt)</li><li>[S’assurer que chaque page susceptible de comporter du contenu contrôlable par l’utilisateur refuse la détection MIME automatique](#out-sniffing)</li><li>[Renforcer ou désactiver la résolution d’entité XML](#xml-resolution)</li><li>[Les applications utilisant http.sys doivent procéder à la vérification de la canonisation des URL](#app-verification)</li><li>[S’assurer que les contrôles appropriés sont en place lors de l’acceptation de fichiers provenant d’utilisateurs](#controls-users)</li><li>[S’assurer que des paramètres de type sécurisé sont utilisés dans une application web pour l’accès aux données](#typesafe)</li><li>[Utiliser des classes de liaison de modèle ou des listes de filtre de liaison distinctes pour empêcher une vulnérabilité d’attribution collective MVC](#binding-mvc)</li><li>[Encoder une sortie web non approuvée avant le rendu](#rendering)</li><li>[Procéder à la validation et au filtrage des entrées sur les propriétés de modèle de tous types de chaînes](#typemodel)</li><li>[Les champs de formulaire acceptant tous les caractères, par exemple dans un éditeur de texte enrichi, doivent être nettoyés](#richtext)</li><li>[Ne pas attribuer d’éléments DOM à des récepteurs sans codage intégré](#inbuilt-encode)</li><li>[Valider la sécurité de la fermeture ou de l’exécution de toutes les redirections au sein de l’application](#redirect-safe)</li><li>[Implémenter la validation des entrées sur tous les paramètres de type de chaîne acceptés par les méthodes de contrôleur](#string-method)</li><li>[Définir le seuil supérieur du délai d’expiration pour le traitement des expressions régulières afin d’empêcher un déni de service en raison d’expressions régulières incorrectes](#dos-expression)</li><li>[Éviter l’utilisation de Html.Raw dans les vues Razor](#html-razor)</li></ul> | 
| **Sauvegarde de la base de données** | <ul><li>[Ne pas utiliser les requêtes dynamiques dans les procédures stockées](#stored-proc)</li></ul> |
| **API Web** | <ul><li>[S’assurer que la validation du modèle est effectuée sur les méthodes d’API Web](#validation-api)</li><li>[Implémenter la validation des entrées sur tous les paramètres de type de chaîne acceptés par les méthodes d’API Web](#string-api)</li><li>[S’assurer que les paramètres de type sécurisé sont utilisés dans une API Web pour l’accès aux données](#typesafe-api)</li></ul> | 
| **Azure Document DB** | <ul><li>[Utiliser des requêtes SQL paramétrables pour Azure Cosmos DB](#sql-docdb)</li></ul> | 
| **WCF** | <ul><li>[Validation des entrées WCF par le biais de la liaison de schéma](#schema-binding)</li><li>[WCF - Validation des entrées par le biais des inspecteurs de paramètres](#parameters)</li></ul> |

## <a name="disable-xslt-scripting-for-all-transforms-using-untrusted-style-sheets"></a><a id="disable-xslt"></a>Désactiver les scripts XSLT pour toutes les transformations à l’aide de feuilles de style non approuvées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Sécurité XSLT](/previous-versions/windows/desktop/ms763800(v=vs.85)), [Propriété XsltSettings.EnableScript](/dotnet/api/system.xml.xsl.xsltsettings.enablescript) |
| **Étapes** | XSLT prend en charge les scripts dans les feuilles de style à l’aide de l’élément `<msxml:script>`. Cela permet d’utiliser des fonctions personnalisées dans une transformation XSLT. Le script est exécuté dans le cadre du processus exécutant la transformation. Le script XSLT doit être désactivé dans les environnements non approuvés afin d’empêcher l’exécution de code non approuvé. *Si vous utilisez .NET :* les scripts XSLT sont désactivés par défaut. Cependant, vous devez vous assurer qu’ils n’ont pas été explicitement activés par le biais de la propriété `XsltSettings.EnableScript`.|

### <a name="example"></a> Exemple 

```csharp
XsltSettings settings = new XsltSettings();
settings.EnableScript = true; // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>Exemple
Si vous utilisez MSXML 6.0, les scripts XSLT sont désactivés par défaut. Cependant, vous devez vous assurer qu’ils n’ont pas été explicitement activés par le biais de la propriété d’objet DOM XML AllowXsltScript. 

```csharp
doc.setProperty("AllowXsltScript", true); // WRONG: THIS SHOULD BE SET TO false
```

### <a name="example"></a>Exemple
Si vous utilisez MSXML 5 ou une version antérieure, les scripts XSLT sont activés par défaut et vous devez les désactiver explicitement. Définissez la propriété de l’objet XML DOM AllowXsltScript sur false. 

```csharp
doc.setProperty("AllowXsltScript", false); // CORRECT. Setting to false disables XSLT scripting.
```

## <a name="ensure-that-each-page-that-could-contain-user-controllable-content-opts-out-of-automatic-mime-sniffing"></a><a id="out-sniffing"></a>S’assurer que chaque page susceptible de comporter du contenu contrôlable par l’utilisateur refuse la détection MIME automatique

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [IE8 Security Part V - Comprehensive Protection (Sécurité IE8 Partie V - Protection complète)](/archive/blogs/ie/ie8-security-part-v-comprehensive-protection)  |
| **Étapes** | <p>Pour chaque page susceptible de comporter du contenu contrôlable par l’utilisateur, vous devez utiliser l’en-tête HTTP `X-Content-Type-Options:nosniff`. Pour satisfaire cette exigence, vous pouvez définir l’en-tête requis page par page uniquement pour les pages susceptibles de comporter du contenu contrôlable par l’utilisateur, ou vous pouvez définir un en-tête global pour toutes les pages de l’application.</p><p>Chaque type de fichier provenant d’un serveur web est associé à un [type MIME](https://en.wikipedia.org/wiki/Mime_type) (également appelé *type de contenu*) qui décrit la nature du contenu (image, texte, application, etc.)</p><p>L’en-tête X-Content-Type-Options est un en-tête HTTP permettant aux développeurs de spécifier que leur contenu ne doit pas être détecté par MIME. Cet en-tête est conçu pour limiter les attaques par détection MIME. La prise en charge de cet en-tête a été ajoutée dans Internet Explorer 8 (IE8).</p><p>Seuls les utilisateurs d’Internet Explorer 8 (IE8) bénéficient de l’en-tête X-Content-Type-Options. À l’heure actuelle, les versions antérieures d’Internet Explorer ne prennent pas en charge l’en-tête X-Content-Type-Options</p><p>Internet Explorer 8 (et les versions ultérieures) constitue le seul navigateur majeur permettant d’implémenter une fonctionnalité de refus de la détection MIME. Si d’autres navigateurs majeurs (Firefox, Safari, Chrome) implémentent des fonctionnalités similaires, cette recommandation sera mise à jour afin d’inclure également la syntaxe de ces navigateurs</p>|

### <a name="example"></a>Exemple
Pour activer l’en-tête requis sur toutes les pages de l’application, vous pouvez effectuer l’une des opérations suivantes : 

* Ajoutez l’en-tête dans le fichier web.config si l’application est hébergée par Internet Information Services (IIS) 7 

```
<system.webServer> 
  <httpProtocol> 
    <customHeaders> 
      <add name=""X-Content-Type-Options"" value=""nosniff""/>
    </customHeaders>
  </httpProtocol>
</system.webServer> 
```

* Ajoutez l’en-tête par le biais de la méthode Global.Application\_BeginRequest 

``` 
void Application_BeginRequest(object sender, EventArgs e)
{
  this.Response.Headers[""X-Content-Type-Options""] = ""nosniff"";
} 
```

* Implémentez le module HTTP personnalisé 

``` 
public class XContentTypeOptionsModule : IHttpModule 
  {
    #region IHttpModule Members 
    public void Dispose() 
    { 

    } 
    public void Init(HttpApplication context)
    { 
      context.PreSendRequestHeaders += newEventHandler(context_PreSendRequestHeaders); 
    } 
    #endregion 
    void context_PreSendRequestHeaders(object sender, EventArgs e) 
      { 
        HttpApplication application = sender as HttpApplication; 
        if (application == null) 
          return; 
        if (application.Response.Headers[""X-Content-Type-Options ""] != null) 
          return; 
        application.Response.Headers.Add(""X-Content-Type-Options "", ""nosniff""); 
      } 
  } 

``` 

* Vous pouvez activer l’en-tête requis uniquement pour des pages spécifiques en l’ajoutant à des réponses individuelles : 

```
this.Response.Headers[""X-Content-Type-Options""] = ""nosniff""; 
``` 

## <a name="harden-or-disable-xml-entity-resolution"></a><a id="xml-resolution"></a>Renforcer ou désactiver la résolution d’entité XML

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [XML Entity Expansion (Extension d’entité XML)](https://capec.mitre.org/data/definitions/197.html), [Attaques par déni de service XML et moyens de défense](/archive/msdn-magazine/2009/november/xml-denial-of-service-attacks-and-defenses), [Vue d’ensemble de la sécurité MSXML](/previous-versions/windows/desktop/ms754611(v=vs.85)), [Meilleures pratiques pour la sécurisation du Code MSXML](/previous-versions/windows/desktop/ms759188(v=vs.85)), [Référence de protocole NSXMLParserDelegate](https://developer.apple.com/library/ios/#documentation/cocoa/reference/NSXMLParserDelegate_Protocol/Reference/Reference.html), [Résolution des ressources externes](/dotnet/standard/data/xml/resolving-external-resources) |
| **Étapes**| <p>Bien qu’elle soit peu utilisée, une fonctionnalité XML permet à l’analyseur XML d’étendre des entités de macro avec des valeurs définies dans le document lui-même ou à partir de sources externes. Par exemple, le document peut définir une entité « nomentreprise » avec la valeur « Microsoft ». Ainsi, chaque fois que le texte « &companyname; » apparaît dans le document, il est automatiquement remplacé par le texte « Microsoft ». Autre solution : le document peut définir une entité « StockMSFT » qui fait référence à un service web externe permettant d’extraire la valeur actuelle du stock Microsoft.</p><p>Ainsi, à chaque fois que « &MSFTStock; » apparaît dans le document, il est automatiquement remplacé par le prix actuel du stock. Cependant, cette fonctionnalité peut être utilisée abusivement afin de générer un déni de service (DoS). Une personne malveillante peut imbriquer plusieurs entités pour créer une bombe XML à extension exponentielle, qui consomme toute la mémoire disponible sur le système. </p><p>Elle peut également créer une référence externe qui diffuse en continu une quantité infinie de données ou bloque simplement le thread. Par conséquent, toutes les équipes doivent entièrement désactiver la résolution d’entité XML interne et/ou externe si leur application ne l’utilise pas, ou limiter manuellement la quantité de mémoire et de temps que l’application peut utiliser pour la résolution d’entité si cette fonctionnalité est indispensable. Si la résolution d’entité n’est pas requise par votre application, désactivez-la. </p>|

### <a name="example"></a>Exemple
Pour le code .NET Framework, vous pouvez utiliser les méthodes suivantes :

```csharp
XmlTextReader reader = new XmlTextReader(stream);
reader.ProhibitDtd = true;

XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);

// for .NET 4
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
XmlReader reader = XmlReader.Create(stream, settings);
```
La valeur par défaut de `ProhibitDtd` dans `XmlReaderSettings` est true, mais elle est false dans `XmlTextReader`. Si vous utilisez XmlReaderSettings, vous n’êtes pas obligé de définir explicitement ProhibitDtd sur true, mais ce paramétrage est recommandé pour des raisons de sécurité. La classe XmlDocument autorise également la résolution d’entité par défaut. 

### <a name="example"></a>Exemple
Pour désactiver la résolution d’entité relative à XmlDocument, utilisez la surcharge `XmlDocument.Load(XmlReader)` de la méthode Load et définissez les propriétés concernées dans l’argument XmlReader, comme illustré dans le code suivant : 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = true;
XmlReader reader = XmlReader.Create(stream, settings);
XmlDocument doc = new XmlDocument();
doc.Load(reader);
```

### <a name="example"></a>Exemple
S’il est impossible de désactiver la résolution d’entité pour votre application, définissez la propriété XmlReaderSettings.MaxCharactersFromEntities sur une valeur raisonnable en fonction des besoins de votre application. Cela limitera l’impact des attaques DoS potentielles d’extension exponentielle. Le code suivant propose un exemple d’approche : 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
XmlReader reader = XmlReader.Create(stream, settings);
```

### <a name="example"></a>Exemple
Si vous devez résoudre les entités incorporées, mais pas les entités externes, définissez la propriété XmlReaderSettings.XmlResolver sur null. Par exemple : 

```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.ProhibitDtd = false;
settings.MaxCharactersFromEntities = 1000;
settings.XmlResolver = null;
XmlReader reader = XmlReader.Create(stream, settings);
```
Dans MSXML6, la valeur ProhibitDTD est définie sur true (ce qui désactive le traitement DTD) par défaut. Pour le code Apple OSX/iOS, vous pouvez utiliser deux analyseurs XML : NSXMLParser et libXML2. 

## <a name="applications-utilizing-httpsys-perform-url-canonicalization-verification"></a><a id="app-verification"></a>Les applications utilisant http.sys doivent procéder à la vérification de la canonisation des URL

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Toutes les applications utilisant http.sys doivent suivre ces instructions :</p><ul><li>Limitez la longueur de l’URL à 16 384 caractères maximum (Unicode ou ASCII). Il s’agit de la longueur URL maximale absolue selon le paramètre Internet Information Services (IIS) 6 par défaut. Les sites web doivent, dans la mesure du possible, proposer une longueur inférieure</li><li>Utilisez les classes d’E/S de fichier .NET Framework standard (par exemple, FileStream) qui se serviront des règles de canonisation dans .NET FX</li><li>Créez explicitement une liste verte des noms de fichiers connus</li><li>Refusez explicitement les types de fichiers connus qui ne vous serviront pas et qu’UrlScan refuse : exe, bat, cmd, com, htw, ida, idq, htr, idc, shtm[l], stm, printer, ini, pol et dat</li><li>Interceptez les exceptions suivantes :<ul><li>System.ArgumentException (pour les noms d’appareils)</li><li>System.NotSupportedException (pour les flux de données)</li><li>System.IO.FileNotFoundException (pour les noms de fichiers avec séquence d’échappement non valide)</li><li>System.IO.FileNotFoundException (pour les noms de répertoires avec séquence d’échappement non valide)</li></ul></li><li>*N’appelez pas* d’API E/S de fichier Win32. Sur une URL non valide, renvoyez de manière appropriée une erreur 400 vers l’utilisateur et enregistrez l’erreur réelle.</li></ul>|

## <a name="ensure-appropriate-controls-are-in-place-when-accepting-files-from-users"></a><a id="controls-users"></a>S’assurer que les contrôles appropriés sont en place lors de l’acceptation de fichiers provenant d’utilisateurs

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Unrestricted File Upload (Chargement de fichiers sans restriction)](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload), [File Signature Table (Table de signatures de fichier)](https://www.garykessler.net/library/file_sigs.html) |
| **Étapes** | <p>Les fichiers téléchargés constituent un risque significatif pour les applications.</p><p>La première étape de nombreuses attaques consiste à obtenir un code permettant d’attaquer le système. Il suffit ensuite que l’attaque trouve un moyen d’exécuter le code. Un téléchargement de fichier permet aux personnes malveillantes d’accomplir la première étape. Les conséquences d’un téléchargement de fichier sans restriction peuvent varier, de la prise de contrôle totale du système à la surcharge d’un système de fichiers ou d’une base de données, en passant par le transfert d’attaques vers les systèmes back-end ou la simple dégradation.</p><p>Tout dépend de ce que l’application fait du fichier téléchargé, et surtout de l’emplacement de stockage du fichier. Il manque la validation des téléchargements de fichiers côté serveur. Le suivi des contrôles de sécurité doit être implémenté pour la fonctionnalité de téléchargement de fichiers :</p><ul><li>Vérification de l’extension du fichier (seul un ensemble valide de types de fichiers autorisés doit être accepté)</li><li>Taille maximale du fichier</li><li>Le fichier ne doit pas être téléchargé sur webroot : l’emplacement choisi doit être un répertoire présent sur un lecteur autre que le lecteur système</li><li>La convention d’affectation de noms doit être suivie : par exemple, le nom du fichier téléchargé doit avoir un caractère aléatoire afin d’éviter l’écrasement du fichier</li><li>Les fichiers doivent être analysés par un antivirus avant d’être écrits sur le disque</li><li>Assurez-vous que le nom de fichier et toute autre métadonnée (par exemple, chemin d’accès) ne comporte pas de caractère nuisible</li><li>La signature de format de fichier doit être vérifiée afin que les utilisateurs ne puissent pas télécharger de fichier déguisé (par exemple, téléchargement d’un fichier exe en remplaçant l’extension par .txt)</li></ul>| 

### <a name="example"></a>Exemple
Concernant la validation de la signature de format de fichier, reportez-vous à la classe ci-dessous pour plus d’informations : 

```csharp
        private static Dictionary<string, List<byte[]>> fileSignature = new Dictionary<string, List<byte[]>>
                    {
                    { ".DOC", new List<byte[]> { new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 } } },
                    { ".DOCX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".PDF", new List<byte[]> { new byte[] { 0x25, 0x50, 0x44, 0x46 } } },
                    { ".ZIP", new List<byte[]> 
                                            {
                                              new byte[] { 0x50, 0x4B, 0x03, 0x04 },
                                              new byte[] { 0x50, 0x4B, 0x4C, 0x49, 0x54, 0x55 },
                                              new byte[] { 0x50, 0x4B, 0x53, 0x70, 0x58 },
                                              new byte[] { 0x50, 0x4B, 0x05, 0x06 },
                                              new byte[] { 0x50, 0x4B, 0x07, 0x08 },
                                              new byte[] { 0x57, 0x69, 0x6E, 0x5A, 0x69, 0x70 }
                                                }
                                            },
                    { ".PNG", new List<byte[]> { new byte[] { 0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A } } },
                    { ".JPG", new List<byte[]>
                                    {
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE1 },
                                              new byte[] { 0xFF, 0xD8, 0xFF, 0xE8 }
                                    }
                                    },
                    { ".JPEG", new List<byte[]>
                                        { 
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
                                            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 }
                                        }
                                        },
                    { ".XLS", new List<byte[]>
                                            {
                                              new byte[] { 0xD0, 0xCF, 0x11, 0xE0, 0xA1, 0xB1, 0x1A, 0xE1 },
                                              new byte[] { 0x09, 0x08, 0x10, 0x00, 0x00, 0x06, 0x05, 0x00 },
                                              new byte[] { 0xFD, 0xFF, 0xFF, 0xFF }
                                            }
                                            },
                    { ".XLSX", new List<byte[]> { new byte[] { 0x50, 0x4B, 0x03, 0x04 } } },
                    { ".GIF", new List<byte[]> { new byte[] { 0x47, 0x49, 0x46, 0x38 } } }
                };

        public static bool IsValidFileExtension(string fileName, byte[] fileData, byte[] allowedChars)
        {
            if (string.IsNullOrEmpty(fileName) || fileData == null || fileData.Length == 0)
            {
                return false;
            }

            bool flag = false;
            string ext = Path.GetExtension(fileName);
            if (string.IsNullOrEmpty(ext))
            {
                return false;
            }

            ext = ext.ToUpperInvariant();

            if (ext.Equals(".TXT") || ext.Equals(".CSV") || ext.Equals(".PRN"))
            {
                foreach (byte b in fileData)
                {
                    if (b > 0x7F)
                    {
                        if (allowedChars != null)
                        {
                            if (!allowedChars.Contains(b))
                            {
                                return false;
                            }
                        }
                        else
                        {
                            return false;
                        }
                    }
                }

                return true;
            }

            if (!fileSignature.ContainsKey(ext))
            {
                return true;
            }

            List<byte[]> sig = fileSignature[ext];
            foreach (byte[] b in sig)
            {
                var curFileSig = new byte[b.Length];
                Array.Copy(fileData, curFileSig, b.Length);
                if (curFileSig.SequenceEqual(b))
                {
                    flag = true;
                    break;
                }
            }

            return flag;
        }
```

## <a name="ensure-that-type-safe-parameters-are-used-in-web-application-for-data-access"></a><a id="typesafe"></a>S’assurer que des paramètres de type sécurisé sont utilisés dans une application web pour l’accès aux données

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Si vous utilisez la collection Parameters, SQL traite l’entrée comme une valeur littérale et non comme du code exécutable. La collection Parameters peut être utilisée pour appliquer des contraintes de type et de longueur sur les données d’entrée. Les valeurs situées en dehors de la plage déclenchent une exception. En l’absence de paramètres SQL de type sécurisé, des personnes malveillantes peuvent exécuter des attaques par injection, incorporées dans les entrées non filtrées.</p><p>Utilisez des paramètres de type sécurisé lors de la création de requêtes SQL afin d’éviter des attaques par injection de code SQL susceptibles de se produire avec des entrées non filtrées. Vous pouvez utiliser des paramètres de type sécurisé avec des procédures stockées et des instructions SQL dynamiques. Les paramètres sont traités comme des valeurs littérales par la base de données, et non comme du code exécutable. Le type et la longueur sont également vérifiés pour les paramètres.</p>|

### <a name="example"></a>Exemple 
Le code suivant montre comment utiliser les paramètres de type sécurisé avec SqlParameterCollection lors de l’appel d’une procédure stockée. 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
Dans l’exemple de code précédent, la valeur d’entrée ne peut pas être supérieure à 11 caractères. Si les données ne sont pas conformes au type ou à la longueur définis par le paramètre, la classe SqlParameter renvoie une exception. 

## <a name="use-separate-model-binding-classes-or-binding-filter-lists-to-prevent-mvc-mass-assignment-vulnerability"></a><a id="binding-mvc"></a>Utiliser des classes de liaison de modèle ou des listes de filtre de liaison distinctes pour empêcher une vulnérabilité d’attribution collective MVC

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Metadata Attributes (Classe MetadataTypeAttribute)](/dotnet/api/system.componentmodel.dataannotations.metadatatypeattribute), [Public Key Security Vulnerability And Mitigation (Failles de sécurité des clés publiques et mesures de correction)](https://github.com/blog/1068-public-key-security-vulnerability-and-mitigation), [Complete Guide to Mass Assignment in ASP.NET MVC (Guide complet d’attribution collective dans ASP.NET MVC)](https://odetocode.com/Blogs/scott/archive/2012/03/11/complete-guide-to-mass-assignment-in-asp-net-mvc.aspx), [Getting Started with EF using MVC (Prise en main d’EF à l’aide de MVC)](https://www.asp.net/mvc/tutorials/getting-started-with-ef-using-mvc/implementing-basic-crud-functionality-with-the-entity-framework-in-asp-net-mvc-application#overpost) |
| **Étapes** | <ul><li>**Quand dois-je rechercher les vulnérabilités de survalidation ? -** Les vulnérabilités de survalidation peuvent se produire partout où vous liez des classes de modèle à partir d’entrées utilisateur. Les infrastructures telles que MVC peuvent représenter des données utilisateur dans des classes .NET personnalisées, y compris les objets CLR traditionnels (OCT). MVC renseigne automatiquement ces classes de modèle avec des données à partir de la requête, ce qui offre une représentation pratique pour la gestion des entrées utilisateur. Lorsque ces classes incluent des propriétés qui ne doivent pas être définies par l’utilisateur, l’application peut être vulnérable aux attaques de survalidation, qui permettent un contrôle des données utilisateur que l’application n’avait pas prévu. Tout comme la liaison de modèle MVC, les technologies d’accès aux bases de données, telles que les mappeurs d’objets/relationnels du type Entity Framework, prennent aussi souvent en charge l’utilisation d’objets OCT pour représenter les données de la base de données. Ces classes de modèle de données fournissent la même fonctionnalité dans le traitement des données de base de données que MVC dans le traitement des entrées utilisateur. Étant donné que MVC et la base de données prennent en charge des modèles similaires, comme les objets OCT, il semble assez facile de réutiliser les mêmes classes pour les deux fonctions. Cette pratique ne parvient pas à préserver la séparation des intérêts. Il s’agit d’une zone commune où des propriétés inattendues sont exposées à la liaison de modèle, ce qui offre une vulnérabilité aux attaques de survalidation.</li><li>**Pourquoi ne puis-je pas utiliser mes classes de modèle de base de données non filtrées en tant que paramètres pour mes actions MVC ? -** Parce que la liaison de modèle MVC établit une liaison avec tout ce qui se trouve dans cette classe. Même si vous ne voyez pas les données, un utilisateur malveillant peut envoyer une requête HTTP incluant ces données, et MVC établira une liaison, car votre action indique que la classe de base de données est conforme à la forme des données acceptable pour les entrées utilisateur.</li><li>**Pourquoi dois-je faire attention à la forme utilisée pour la liaison de modèle ? -** L’utilisation d’une liaison de modèle ASP.NET MVC avec des modèles trop étendus expose une application aux attaques de survalidation. La survalidation permet aux personnes malveillantes de modifier les données d’application au-delà de ce que le développeur escomptait, comme le remplacement du prix d’un article ou les privilèges de sécurité d’un compte. Les applications doivent utiliser des modèles de liaison spécifiques en fonction des actions (ou des listes de filtres de propriétés autorisés) afin d’indiquer explicitement les entrées non approuvées qui sont autorisées par le biais de la liaison de modèle.</li><li>**Le fait de disposer de modèles de liaison distincts équivaut-il simplement à une duplication de code ? -** Non, c’est une question de séparation des intérêts. Si vous réutilisez un modèle de base de données dans les méthodes d’action, vous indiquez que toute propriété (ou sous-propriété) de cette classe peut être définie par l’utilisateur dans une requête HTTP. Si ce n’est pas ce que vous voulez, vous avez besoin d’une liste de filtrage ou d’une forme de classe distincte afin d’indiquer à MVC les données pouvant provenir d’entrées utilisateur.</li><li>**Si j’utilise des modèles de liaison distincts pour les entrées utilisateur, dois-je dupliquer tous les attributs d’annotation de mes données ? -** Pas forcément. Vous pouvez utiliser MetadataTypeAttribute sur la classe de modèle de base de données afin d’établir un lien vers les métadonnées sur une classe de liaison de modèle. Mais n’oubliez pas que le type référencé par MetadataTypeAttribute doit être un sous-ensemble du type de référencement (il peut comporter moins de propriétés, mais pas plus).</li><li>**Le déplacement des données entre les modèles d’entrée utilisateur et les modèles de base de données s’avère fastidieux. Puis-je simplement copier toutes les propriétés par réflexion ? -** Oui. Les seules propriétés apparaissant dans les modèles de liaison sont celles que vous avez déterminées comme étant sécurisées pour les entrées utilisateur. Il n’existe aucune raison de sécurité qui empêche l’utilisation de la réflexion pour copier toutes les propriétés communes aux deux modèles.</li><li>**Qu’en est-il de l’attribut [Bind(Exclude ="â&euro;¦")] ? Puis-je l’utiliser à la place de modèles de liaison distincts ? -** Cette approche n’est pas recommandée. L’utilisation de l’attribut [Bind(Exclude ="â&euro;¦")] signifie que toute nouvelle propriété peut être liée par défaut. Lorsqu’une nouvelle propriété est ajoutée, une étape supplémentaire permet de rappeler les principes de base de sécurité, au lieu de présenter une conception sécurisée par défaut. Il est risqué de s’appuyer sur le fait que le développeur vérifie cette liste à chaque ajout de propriété.</li><li>**L’attribut [Bind(Include ="â&euro;¦")] est-il utile pour les opérations de modification ? -** Non. L’attribut [Bind(Include ="â&euro;¦")] ne convient que pour les opérations de style INSERT (ajout de nouvelles données). Pour les opérations de style UPDATE (vérification des données existantes), utilisez une autre approche, comme les modèles de liaison séparés ou la transmission d’une liste explicite de propriétés autorisées à UpdateModel ou TryUpdateModel. L’ajout d’un attribut [Bind(Include ="â&euro;¦")] sur une opération de modification a pour effet que MVC crée une instance d’objet et définit uniquement les propriétés répertoriées, en conservant les valeurs par défaut pour toutes les autres propriétés. Lorsque les données sont conservées, l’entité existante est entièrement remplacée, ce qui réinitialise les valeurs par défaut de toutes les propriétés omises. Par exemple, si IsAdmin a été omis de l’attribut [Bind(Include ="â&euro;¦")] sur une opération de modification, tout utilisateur dont le nom a été modifié par cette action est réinitialisé à IsAdmin = false (tout utilisateur modifié perd le statut d’administrateur). Pour empêcher la mise à jour de certaines propriétés, utilisez l’une des autres approches ci-dessus. Notez que certaines versions des outils MVC génèrent des classes de contrôleur avec l’attribut [Bind(Include ="â&euro;¦")] sur des actions de modification et ont pour effet que la suppression d’une propriété de cette liste empêche les attaques par survalidation. Toutefois, comme décrit ci-dessus, cette approche ne fonctionne pas comme prévu et réinitialise toutes les valeurs par défaut des propriétés omises.</li><li>**Pour les opérations de création, existe-t-il des restrictions à l’utilisation de l’attribut [Bind(Include ="â&euro;¦")] au lieu de modèles de liaison séparés ? -** Oui. Tout d’abord, cette approche ne fonctionne pas pour les scénarios de modification, qui nécessitent le maintien de deux approches distinctes afin de limiter toutes les vulnérabilités de survalidation. Deuxièmement, des modèles de liaison séparés impliquent une séparation des problèmes entre la forme utilisée pour l’entrée d’utilisateur et celle utilisée pour la persistance, ce que l’attribut [Bind(Include ="â&euro;¦")] ne fait pas. Troisièmement, l’attribut [Bind(Include ="â&euro;¦")] ne peut gérer que des propriétés de niveau supérieur. Vous ne pouvez pas autoriser des portions de sous-propriétés (par exemple, « Details.Name ») dans l’attribut. Enfin, et c’est peut-être le plus important, l’utilisation de l’attribut [Bind(Include ="â&euro;¦")] ajoute une étape qui doit être mémorisée chaque fois que la classe est utilisée pour une liaison de données. Si une nouvelle méthode d’action lie directement à la classe de données et oublie d’inclure un attribut [Bind(Include ="â&euro;¦")], elle peut être vulnérable aux attaques par survalidation. L’approche [Bind(Include ="â&euro;¦")] est donc un peu moins sécurisée par défaut. Si vous utilisez l’attribut [Bind(Include ="â&euro;¦")], n’oubliez jamais de le spécifier chaque fois que vos classes de données apparaissent en tant que paramètres de méthode d’action.</li><li>**Pour les opérations de création, pourquoi ne pas placer l’attribut [Bind(Include ="â&euro;¦")] sur la classe de modèle elle-même ? Cette approche n’évite-t-elle pas d’avoir à se souvenir de placer l’attribut sur chaque méthode d’action ? -** Cette approche fonctionne dans certains cas. L’utilisation de l’attribut [Bind(Include ="â&euro;¦")] sur le type de modèle (plutôt que sur les paramètres d’action utilisant cette classe) évite d’avoir à inclure l’attribut [Bind(Include ="â&euro;¦")] sur chaque méthode d’action. L’utilisation directe de l’attribut sur la classe crée une zone distincte optimale dans cette classe pour la liaison de modèle. Toutefois, cette approche ne permet qu’une seule forme de liaison de modèle par classe de modèle. Si une méthode d’action doit autoriser la liaison de modèle d’un champ (par exemple, action réservée à l’administrateur pour mettre à jour les rôles d’utilisateur) et si d’autres actions doivent éviter la liaison de modèle de ce champ, cette approche ne fonctionne pas. Chaque classe ne peut avoir qu’une seule forme de liaison de données. Si différentes actions ont besoin de formes de liaison de données différentes, elles doivent représenter ces formes distinctes à l’aide de deux classes de liaison de données distinctes ou d’attributs [Bind(Include ="â&euro;¦")] distincts sur les méthodes d’action.</li><li>**Ce que sont les modèles de liaison ? Sont-ils identiques aux modèles de vue ? -** Ce sont deux concepts connexes. Le terme « modèle de liaison » désigne une classe de modèle utilisée dans la liste des paramètres d’une action (forme transmise de la liaison de modèle MVC à la méthode d’action). Le terme « modèle de vue » fait référence à une classe de modèle transmise d’une méthode d’action vers une vue. L’utilisation d’un modèle spécifique de la vue est une approche courante pour faire passer des données d’une méthode d’action vers une vue. Souvent, cette forme est également adaptée à la liaison de modèles, et le terme « modèle de vue » peut faire référence au même modèle utilisé aux deux emplacements. Plus précisément, cette procédure traite spécifiquement des modèles de liaison, et notamment de la forme transmise vers l’action, ce qui importe le plus dans l’attribution collective.</li></ul>| 

## <a name="encode-untrusted-web-output-prior-to-rendering"></a><a id="rendering"></a>Encoder une sortie web non approuvée avant le rendu

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, Web Forms, MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [How to prevent Cross-site scripting in ASP.NET (Comment éviter les scripts intersites dans ASP.NET)](/previous-versions/msp-n-p/ff649310(v=pandp.10)), [Cross-site Scripting (Scripts intersites)](https://cwe.mitre.org/data/definitions/79.html), [XSS (Cross Site Scripting) Prevention Cheat Sheet (Aide-mémoire sur la prévention des scripts intersites)](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) |
| **Étapes** | Les scripts intersites (XSS, Cross Site Scripting) sont un vecteur d’attaque pour les services en ligne ou toute application/tout composant qui utilise des entrées tirées du web. Les vulnérabilités XSS permettent à une personne malveillante d’exécuter un script sur l’ordinateur d’un autre utilisateur par le biais d’une application web vulnérable. Les scripts malveillants peuvent servir à dérober des cookies ou à manipuler un ordinateur cible par le biais de JavaScript. Pour empêcher les scripts intersites, il faut procéder à la validation des entrées utilisateur, en s’assurant que la forme et l’encodage sont corrects avant le rendu sur une page web. La validation des entrées et l’encodage des sorties peuvent se faire au moyen de Web Protection Library. Pour le code managé (C\#, VB.NET, etc.), utilisez une ou plusieurs méthodes d’encodage appropriées, tirées de Web Protection (Anti-XSS) Library, en fonction du contexte dans lequel les entrées utilisateur se manifestent :| 

### <a name="example"></a>Exemple

```csharp
* Encoder.HtmlEncode 
* Encoder.HtmlAttributeEncode 
* Encoder.JavaScriptEncode 
* Encoder.UrlEncode
* Encoder.VisualBasicScriptEncode 
* Encoder.XmlEncode 
* Encoder.XmlAttributeEncode 
* Encoder.CssEncode 
* Encoder.LdapEncode 
```

## <a name="perform-input-validation-and-filtering-on-all-string-type-model-properties"></a><a id="typemodel"></a>Procéder à la validation et au filtrage des entrées sur les propriétés de modèle de tous types de chaînes

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Adding Validation (Ajout d’une validation)](https://www.asp.net/mvc/overview/getting-started/introduction/adding-validation), [Validating Model Data in an MVC Application (Validation des données de modèle dans une application MVC)](/previous-versions/dd410404(v=vs.90)), [Principes directeurs pour vos applications ASP.NET MVC](/archive/msdn-magazine/2009/brownfield/extreme-asp-net-guiding-principles-for-your-asp-net-mvc-applications) |
| **Étapes** | <p>Tous les paramètres d’entrée doivent être validés avant d’être utilisés dans l’application afin de vous assurer que l’application est protégée contre les entrées utilisateur malveillantes. Validez les valeurs d’entrée en recourant à la validation d’expressions régulières côté serveur au moyen d’une stratégie de validation par liste autorisée. Les entrées utilisateur/paramètres non nettoyés et transmis aux méthodes peuvent engendrer des vulnérabilités par injection de code.</p><p>Pour les applications web, les points d’entrée peuvent également inclure des champs de formulaire, des chaînes de requête, des cookies, des en-têtes HTTP et des paramètres de service web.</p><p>Les contrôles de validation des entrées suivants doivent être effectués lors de la liaison de modèle :</p><ul><li>Les propriétés de modèle doivent être annotées au moyen d’une annotation RegularExpression, ce qui permet d’accepter les caractères autorisés et la longueur maximale autorisée</li><li>Les méthodes de contrôleur doivent procéder à la validation de ModelState</li></ul>|

## <a name="sanitization-should-be-applied-on-form-fields-that-accept-all-characters-eg-rich-text-editor"></a><a id="richtext"></a>Les champs de formulaire acceptant tous les caractères, par exemple dans un éditeur de texte enrichi, doivent être nettoyés

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Encode Unsafe Input (Encodage d’entrées à risque)](/previous-versions/msp-n-p/ff647397(v=pandp.10)#paght000003_step3), [HTML Sanitizer](https://github.com/mganss/HtmlSanitizer) |
| **Étapes** | <p>Identifiez toutes les balises statiques que vous souhaitez utiliser. Une pratique courante consiste à limiter la mise en forme des éléments HTML sécurisés, comme `<b>` (gras) et `<i>` (italique).</p><p>Avant d’écrire les données, encodez-les au format HTML. Cela sécurise n’importe quel script malveillant en le traitant comme du texte, et non comme du code exécutable.</p><ol><li>Désactivez la validation de requête ASP.NET en ajoutant l’attribut ValidateRequest="false" à la directive \@ Page</li><li>Encodez les données de chaîne avec la méthode HtmlEncode</li><li>Utilisez StringBuilder et appelez sa méthode de remplacement pour supprimer de manière sélective l’encodage sur les éléments HTML que vous souhaitez autoriser</li></ol><p>La page dans les références désactive la validation de requête ASP.NET en définissant `ValidateRequest="false"`. Elle encode les entrées au format HTML et autorise de manière sélective `<b>` et `<i>`. Une bibliothèque .NET pour le nettoyage HTML peut également être utilisée.</p><p>HtmlSanitizer est une bibliothèque .NET destinée au nettoyage des documents et fragments HTML provenant de constructions susceptibles d’aboutir à des attaques XSS. Elle utilise AngleSharp pour analyser, manipuler et réaliser le rendu HTML et CSS. HtmlSanitizer peut être installée en tant que package NuGet, et les entrées utilisateur peuvent être transmises par le biais de méthodes de nettoyage HTML ou CSS appropriées, le cas échéant, côté serveur. Dans le cadre du contrôle de sécurité, le nettoyage doit être envisagé uniquement en dernier recours.</p><p>La validation des entrées et l’encodage des sorties sont considérés comme de meilleurs contrôles de sécurité.</p> |

## <a name="do-not-assign-dom-elements-to-sinks-that-do-not-have-inbuilt-encoding"></a><a id="inbuilt-encode"></a>Ne pas attribuer d’éléments DOM à des récepteurs sans codage intégré

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | De nombreuses fonctions JavaScript ne procèdent pas à l’encodage par défaut. L’attribution d’entrées non approuvées à des éléments DOM par le biais de ces fonctions risque d’entraîner l’exécution de scripts intersites (XSS).| 

### <a name="example"></a>Exemple
Voici quelques exemples non sécurisés : 

```
document.getElementByID("div1").innerHtml = value;
$("#userName").html(res.Name);
return $('<div/>').html(value)
$('body').append(resHTML);   
```
N’utilisez pas `innerHtml`. Utilisez plutôt `innerText`. De même, au lieu de `$("#elm").html()`, utilisez`$("#elm").text()` 

## <a name="validate-all-redirects-within-the-application-are-closed-or-done-safely"></a><a id="redirect-safe"></a>Valider la sécurité de la fermeture ou de l’exécution de toutes les redirections au sein de l’application

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [The OAuth 2.0 Authorization Framework - Open Redirectors (Infrastructure d’autorisation OAuth 2.0 : Redirecteurs ouverts)](https://tools.ietf.org/html/rfc6749#section-10.15) |
| **Étapes** | <p>La conception d’applications nécessitant une redirection vers un emplacement fourni par l’utilisateur doit limiter les cibles possibles de redirection à une liste prédéfinie « sécurisée » de sites ou de domaines. Toutes les redirections au sein de l’application doivent être fermées/sécurisées.</p><p>Pour ce faire :</p><ul><li>Identifiez toutes les redirections</li><li>Implémentez une mesure de correction appropriée pour chaque redirection. Les mesures d’atténuation appropriées incluent la redirection de la confirmation par utilisateur ou liste autorisée. Si un site web ou un service avec vulnérabilité par redirection ouverte utilise les fournisseurs d’identité Facebook/OAuth/OpenID, une personne malveillante peut voler le jeton d’ouverture de session d’un utilisateur et emprunter l’identité de cet utilisateur. Il s’agit d’un risque inhérent de l’utilisation d’OAuth, décrit dans le document RFC 6749 « The OAuth 2.0 Authorization Framework » (Infrastructure d’autorisation OAuth 2.0), Section 10.15 « Open Redirects » (Redirecteurs ouverts). De même, les informations d’identification des utilisateurs peuvent être compromises par des attaques par hameçonnage ciblé se servant de redirections ouvertes</li></ul>|

## <a name="implement-input-validation-on-all-string-type-parameters-accepted-by-controller-methods"></a><a id="string-method"></a>Implémenter la validation des entrées sur tous les paramètres de type de chaîne acceptés par les méthodes de contrôleur

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Validating Model Data in an MVC Application (Validation des données de modèle dans une application MVC)](/previous-versions/dd410404(v=vs.90)), [Principes directeurs pour vos applications ASP.NET MVC](/archive/msdn-magazine/2009/brownfield/extreme-asp-net-guiding-principles-for-your-asp-net-mvc-applications) |
| **Étapes** | Pour les méthodes qui acceptent uniquement un type de données primitif, et non les modèles en tant qu’argument, la validation des entrées doit être effectuée à l’aide d’expressions régulières. Dans le cas présent, Regex.IsMatch doit être utilisé avec un modèle d’expression régulière valide. Si l’entrée ne correspond pas à l’expression régulière spécifiée, le contrôle ne doit pas se poursuivre, et un avertissement approprié indiquant l’échec de la validation doit s’afficher.| 

## <a name="set-upper-limit-timeout-for-regular-expression-processing-to-prevent-dos-due-to-bad-regular-expressions"></a><a id="dos-expression"></a>Définir le seuil supérieur du délai d’expiration pour le traitement des expressions régulières afin d’empêcher un déni de service en raison d’expressions régulières incorrectes

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, Web Forms, MVC5, MVC6  |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Propriété DefaultRegexMatchTimeout](/dotnet/api/system.web.configuration.httpruntimesection.defaultregexmatchtimeout) |
| **Étapes** | Pour empêcher les attaques par déni de service contre les expressions régulières incorrectes, qui suscitent de nombreux retours sur trace, définissez le délai d’attente global par défaut. Si le temps de traitement excède la limite supérieure définie, une exception d’expiration est générée. Si rien n’est configuré, le délai d’attente est infini.| 

### <a name="example"></a>Exemple
Par exemple, la configuration suivante génère une exception RegexMatchTimeoutException, si le traitement prend plus de 5 secondes : 

```csharp
<httpRuntime targetFramework="4.5" defaultRegexMatchTimeout="00:00:05" />
```

## <a name="avoid-using-htmlraw-in-razor-views"></a><a id="html-razor"></a>Éviter l’utilisation de Html.Raw dans les vues Razor

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Application Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| Étape | Pages Web ASP.NET (Razor) procède à un encodage HTML automatique. Toutes les chaînes imprimées par des pépites, ou nuggets, de code (blocs @) sont automatiquement encodées au format HTML. Toutefois, lorsque la méthode `HtmlHelper.Raw` est appelée, elle renvoie un balisage qui n’est pas encodé au format HTML. Si la méthode d’assistance `Html.Raw()` est utilisée, elle ignore la protection par encodage automatique fournie par Razor.|

### <a name="example"></a>Exemple
Voici un exemple non sécurisé : 

```csharp
<div class="form-group">
            @Html.Raw(Model.AccountConfirmText)
        </div>
        <div class="form-group">
            @Html.Raw(Model.PaymentConfirmText)
        </div>
</div>
```
N’utilisez pas `Html.Raw()` sauf si vous devez afficher le balisage. Cette méthode n’effectue pas de codage implicite en sortie. Utilisez d’autres aides ASP.NET, comme `@Html.DisplayFor()` 

## <a name="do-not-use-dynamic-queries-in-stored-procedures"></a><a id="stored-proc"></a>Ne pas utiliser les requêtes dynamiques dans les procédures stockées

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Base de données | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Une attaque par injection de code SQL exploite les vulnérabilités de la validation des entrées afin d’exécuter des commandes arbitraires dans la base de données. Cela peut se produire lorsque votre application utilise des entrées pour créer des instructions SQL dynamiques afin d’accéder à la base de données. Cela peut également se produire si votre code utilise des procédures stockées contenant des chaînes transmises qui comportent des entrées utilisateur brutes. Grâce aux attaques par injection de code SQL, une personne malveillante peut exécuter des commandes arbitraires dans la base de données. Toutes les instructions SQL (y compris les instructions SQL des procédures stockées) doivent être paramétrables. Les instructions SQL paramétrables acceptent les caractères ayant une signification particulière pour SQL (par exemple, les guillemets simples) sans problème, car ils sont fortement typés. |

### <a name="example"></a>Exemple
Voici un exemple de procédure stockée dynamique non sécurisée : 

```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteria]
(
  @productName nvarchar(200) = NULL,
  @startPrice float = NULL,
  @endPrice float = NULL
)
AS
 BEGIN
  DECLARE @sql nvarchar(max)
  SELECT @sql = ' SELECT ProductID, ProductName, Description, UnitPrice, ImagePath' +
       ' FROM dbo.Products WHERE 1 = 1 '
       PRINT @sql
  IF @productName IS NOT NULL
     SELECT @sql = @sql + ' AND ProductName LIKE ''%' + @productName + '%'''
  IF @startPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice > ''' + CONVERT(VARCHAR(10),@startPrice) + ''''
  IF @endPrice IS NOT NULL
     SELECT @sql = @sql + ' AND UnitPrice < ''' + CONVERT(VARCHAR(10),@endPrice) + ''''

  PRINT @sql
  EXEC(@sql)
 END
```

### <a name="example"></a>Exemple
Voici la même procédure stockée, implémentée de façon sécurisée : 
```csharp
CREATE PROCEDURE [dbo].[uspGetProductsByCriteriaSecure]
(
             @productName nvarchar(200) = NULL,
             @startPrice float = NULL,
             @endPrice float = NULL
)
AS
       BEGIN
             SELECT ProductID, ProductName, Description, UnitPrice, ImagePath
             FROM dbo.Products where
             (@productName IS NULL or ProductName like '%'+ @productName +'%')
             AND
             (@startPrice IS NULL or UnitPrice > @startPrice)
             AND
             (@endPrice IS NULL or UnitPrice < @endPrice)         
       END
```

## <a name="ensure-that-model-validation-is-done-on-web-api-methods"></a><a id="validation-api"></a>S’assurer que la validation du modèle est effectuée sur les méthodes d’API Web

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | MVC5, MVC6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Validation du modèle dans l’API web ASP.NET](https://www.asp.net/web-api/overview/formats-and-model-binding/model-validation-in-aspnet-web-api) |
| **Étapes** | Lorsqu’un client envoie des données vers une API Web, les données doivent être validées avant tout traitement. Pour les API Web ASP.NET qui acceptent les modèles en tant qu’entrée, utilisez des annotations de données sur les modèles afin de définir des règles de validation sur les propriétés du modèle.|

### <a name="example"></a>Exemple
Le code suivant illustre le même cas : 

```csharp
using System.ComponentModel.DataAnnotations;

namespace MyApi.Models
{
    public class Product
    {
        public int Id { get; set; }
        [Required]
        [RegularExpression(@"^[a-zA-Z0-9]*$", ErrorMessage="Only alphanumeric characters are allowed.")]
        public string Name { get; set; }
        public decimal Price { get; set; }
        [Range(0, 999)]
        public double Weight { get; set; }
    }
}
```

### <a name="example"></a>Exemple
Dans la méthode d’action des contrôleurs d’API, la validité du modèle doit être vérifiée explicitement comme indiqué ci-dessous : 

```csharp
namespace MyApi.Controllers
{
    public class ProductsController : ApiController
    {
        public HttpResponseMessage Post(Product product)
        {
            if (ModelState.IsValid)
            {
                // Do something with the product (not shown).

                return new HttpResponseMessage(HttpStatusCode.OK);
            }
            else
            {
                return Request.CreateErrorResponse(HttpStatusCode.BadRequest, ModelState);
            }
        }
    }
}
```

## <a name="implement-input-validation-on-all-string-type-parameters-accepted-by-web-api-methods"></a><a id="string-api"></a>Implémenter la validation des entrées sur tous les paramètres de type de chaîne acceptés par les méthodes d’API Web

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, MVC 5, MVC 6 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Validating Model Data in an MVC Application (Validation des données de modèle dans une application MVC)](/previous-versions/dd410404(v=vs.90)), [Principes directeurs pour vos applications ASP.NET MVC](/archive/msdn-magazine/2009/brownfield/extreme-asp-net-guiding-principles-for-your-asp-net-mvc-applications) |
| **Étapes** | Pour les méthodes qui acceptent uniquement un type de données primitif, et non les modèles en tant qu’argument, la validation des entrées doit être effectuée à l’aide d’expressions régulières. Dans le cas présent, Regex.IsMatch doit être utilisé avec un modèle d’expression régulière valide. Si l’entrée ne correspond pas à l’expression régulière spécifiée, le contrôle ne doit pas se poursuivre, et un avertissement approprié indiquant l’échec de la validation doit s’afficher.|

## <a name="ensure-that-type-safe-parameters-are-used-in-web-api-for-data-access"></a><a id="typesafe-api"></a>S’assurer que les paramètres de type sécurisé sont utilisés dans une API Web pour l’accès aux données

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | API Web | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | N/A  |
| **Étapes** | <p>Si vous utilisez la collection Parameters, SQL traite l’entrée comme une valeur littérale et non comme du code exécutable. La collection Parameters peut être utilisée pour appliquer des contraintes de type et de longueur sur les données d’entrée. Les valeurs situées en dehors de la plage déclenchent une exception. En l’absence de paramètres SQL de type sécurisé, des personnes malveillantes peuvent exécuter des attaques par injection, incorporées dans les entrées non filtrées.</p><p>Utilisez des paramètres de type sécurisé lors de la création de requêtes SQL afin d’éviter des attaques par injection de code SQL susceptibles de se produire avec des entrées non filtrées. Vous pouvez utiliser des paramètres de type sécurisé avec des procédures stockées et des instructions SQL dynamiques. Les paramètres sont traités comme des valeurs littérales par la base de données, et non comme du code exécutable. Le type et la longueur sont également vérifiés pour les paramètres.</p>|

### <a name="example"></a>Exemple
Le code suivant montre comment utiliser les paramètres de type sécurisé avec SqlParameterCollection lors de l’appel d’une procédure stockée. 

```csharp
using System.Data;
using System.Data.SqlClient;

using (SqlConnection connection = new SqlConnection(connectionString))
{ 
DataSet userDataset = new DataSet(); 
SqlDataAdapter myCommand = new SqlDataAdapter("LoginStoredProcedure", connection); 
myCommand.SelectCommand.CommandType = CommandType.StoredProcedure; 
myCommand.SelectCommand.Parameters.Add("@au_id", SqlDbType.VarChar, 11); 
myCommand.SelectCommand.Parameters["@au_id"].Value = SSN.Text; 
myCommand.Fill(userDataset);
}  
```
Dans l’exemple de code précédent, la valeur d’entrée ne peut pas être supérieure à 11 caractères. Si les données ne sont pas conformes au type ou à la longueur définis par le paramètre, la classe SqlParameter renvoie une exception. 

## <a name="use-parameterized-sql-queries-for-cosmos-db"></a><a id="sql-docdb"></a>Utiliser des requêtes SQL paramétrables pour Azure Cosmos DB

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | Azure Document DB | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique |
| **Attributs**              | N/A  |
| **Informations de référence**              | [Annonce de paramétrage SQL dans Azure Cosmos DB](https://azure.microsoft.com/blog/announcing-sql-parameterization-in-documentdb/) |
| **Étapes** | Bien qu’Azure Cosmos DB ne prenne en charge que les requêtes en lecture seule, l’injection de code SQL reste possible si les requêtes sont construites par concaténation avec les entrées utilisateur. Un utilisateur peut parfois accéder à des données auxquelles il ne devrait pas accéder dans la même collection en créant des requêtes SQL malveillantes. Utilisez des requêtes SQL paramétrables si les requêtes sont créées à partir d’entrées utilisateur. |

## <a name="wcf-input-validation-through-schema-binding"></a><a id="schema-binding"></a>Validation des entrées WCF par le biais de la liaison de schéma

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff647820(v=pandp.10)) |
| **Étapes** | <p>Une absence de validation peut entraîner différentes attaques par injection de code.</p><p>La validation des messages représente une ligne de défense spécifique dans la protection de votre application WCF. Avec cette approche, vous validez des messages à l’aide de schémas pour protéger les opérations de service WCF contre les attaques de clients malveillants. Validez tous les messages reçus par le client pour protéger le client contre des attaques effectuées par un service malveillant. La validation des messages permet de valider les messages lorsque les opérations utilisent des contrats de messages ou de données, ce qui ne peut pas être fait avec la validation des paramètres. La validation des messages vous permet de créer une logique de validation dans les schémas, offrant ainsi plus de flexibilité et réduisant le temps de développement. Les schémas peuvent être réutilisés dans plusieurs applications de l’organisation, ce qui crée des normes de représentation des données. En outre, la validation des messages vous permet de protéger les opérations lorsqu’elles utilisent des types de données plus complexes impliquant des contrats de logique métier.</p><p>Pour procéder à la validation des messages, commencez par créer un schéma qui représente les opérations de votre service et les types de données utilisées par ces opérations. Créez ensuite une classe .NET qui implémente un inspecteur de messages client personnalisé et un inspecteur de messages de répartition personnalisé afin de valider les messages envoyés/reçus à destination/en provenance du service. Puis implémentez un comportement de point de terminaison personnalisé afin d’activer la validation des messages sur le client et le service. Enfin, implémentez un élément de configuration personnalisé sur la classe afin d’exposer le comportement de point de terminaison personnalisé étendu dans le fichier de configuration du service ou le client.</p>|

## <a name="wcf--input-validation-through-parameter-inspectors"></a><a id="parameters"></a>WCF - Validation des entrées par le biais des inspecteurs de paramètres

| Intitulé                   | Détails      |
| ----------------------- | ------------ |
| **Composant**               | WCF | 
| **Phase SDL**               | Build |  
| **Technologies applicables** | Générique, NET Framework 3 |
| **Attributs**              | N/A  |
| **Informations de référence**              | [MSDN](/previous-versions/msp-n-p/ff647875(v=pandp.10)) |
| **Étapes** | <p>La validation des données et des entrées représente une ligne de défense importante de la protection de votre application WCF. Vous devez valider tous les paramètres exposés dans les opérations de service WCF pour protéger le service des attaques menées par un client malveillant. Inversement, vous devez aussi valider toutes les valeurs de retour reçues par le client pour protéger le client contre les attaques menées par un service malveillant</p><p>WCF fournit différents points d’extension qui vous permettent de personnaliser le comportement d’exécution WCF en créant des extensions personnalisées. Les inspecteurs de messages et les inspecteurs de paramètres sont deux mécanismes d’extension facilitant le contrôle des données échangées entre un client et un service. Vous devez utiliser les inspecteurs de paramètres pour la validation des entrées, et les inspecteurs de messages uniquement lorsque vous devez inspecter l’intégralité des messages entrant et sortant d’un service.</p><p>Pour procéder à la validation des entrées, créez une classe .NET et implémentez un inspecteur de paramètres personnalisé afin de valider les paramètres sur les opérations dans votre service. Implémentez ensuite un comportement de point de terminaison personnalisé afin d’activer la validation sur le client et le service. Enfin, implémentez un élément de configuration personnalisé sur la classe afin d’exposer le comportement de point de terminaison personnalisé étendu dans le fichier de configuration du service ou le client</p>|