---
title: "Héberger ASP.NET Core sur Linux avec Nginx"
author: rick-anderson
description: "Décrit comment le programme d’installation de Nginx comme un proxy inverse sur 16.04 Ubuntu pour transférer le trafic HTTP vers une application web ASP.NET Core sur Kestrel."
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 08/21/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/linux-nginx
ms.openlocfilehash: 1044a87a4dcc7636413078b0fc09ade206c97d0a
ms.sourcegitcommit: b83a5f731a9c02bdb1cc1e3f9a8bf273eb5b33e0
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/11/2018
---
# <a name="host-aspnet-core-on-linux-with-nginx"></a>Héberger ASP.NET Core sur Linux avec Nginx

De [Sourabh Shirhatti](https://twitter.com/sshirhatti)

Ce guide explique comment configurer un environnement ASP.NET Core prêt pour la production sur un serveur Ubuntu 16.04.

**Remarque :** pour Ubuntu 14.04, *supervisord* est recommandé d’utiliser une solution pour l’analyse du processus Kestrel. *systemd* n’est pas disponible sur Ubuntu 14.04. [Voir la version précédente de ce document](https://github.com/aspnet/Docs/blob/e9c1419175c4dd7e152df3746ba1df5935aaafd5/aspnetcore/publishing/linuxproduction.md)

Ce guide montre comment effectuer les opérations suivantes :

* Place une application ASP.NET Core existante derrière un serveur proxy inverse.
* Configure le serveur de proxy inverse pour transférer les demandes au serveur web Kestrel.
* Garantit que l’application web s’exécute au démarrage en tant que démon.
* Configure un outil de gestion des processus permettant de redémarrer l’application web.

## <a name="prerequisites"></a>Prérequis

1. Accès à un serveur Ubuntu 16.04 avec un compte d’utilisateur standard disposant de privilèges sudo
1. Une application ASP.NET Core existante

## <a name="copy-over-the-app"></a>Copier sur l’application

Exécutez `dotnet publish` à partir de l’environnement de développement pour empaqueter une application dans un répertoire autonome pouvant s’exécuter sur le serveur.

Copie de l’application ASP.NET Core pour le serveur à l’aide de n’importe quel outil s’intègre dans le flux de travail de l’organisation (par exemple, SCP, FTP). Testez l’application, par exemple :

* À partir de la ligne de commande, exécutez `dotnet <app_assembly>.dll`.
* Dans un navigateur, accédez à `http://<serveraddress>:<port>` pour vérifier que l’application fonctionne sur Linux. 
 
## <a name="configure-a-reverse-proxy-server"></a>Configurer un serveur proxy inverse

Un proxy inverse est une installation commune pour traiter les applications web dynamiques. Un proxy inverse met fin à la requête HTTP et le transmet à l’application ASP.NET Core.

### <a name="why-use-a-reverse-proxy-server"></a>Pourquoi utiliser un serveur proxy inverse ?

Kestrel est idéal pour héberger un contenu dynamique à partir de ASP.NET. Toutefois, les fonctionnalités de service web ne sont pas en tant que riche en tant que serveurs tels que IIS, Apache ou Nginx. Un serveur proxy inverse peut décharger du travail, tels que fournit du contenu statique, la mise en cache des demandes, la compression des demandes et l’arrêt SSL à partir du serveur HTTP. Un serveur proxy inverse peut résider sur un ordinateur dédié ou peut être déployé à côté d’un serveur HTTP.

Pour les besoins de ce guide, nous utilisons une seule instance de Nginx. Elle s’exécute sur le même serveur, en plus du serveur HTTP. Selon la configuration requise, un paramétrage différent peut être choisi.

Étant donné que les demandes sont transmises par le proxy inverse, utilisez l’intergiciel en-têtes transférés à partir de la [Microsoft.AspNetCore.HttpOverrides](https://www.nuget.org/packages/Microsoft.AspNetCore.HttpOverrides/) package. Les mises à jour de l’intergiciel (middleware) le `Request.Scheme`, à l’aide du `X-Forwarded-Proto` en-tête, afin qu’URI de redirection et d’autres stratégies de sécurité fonctionnent correctement.

Lorsque vous utilisez n’importe quel type d’intergiciel (middleware) d’authentification, les en-têtes transféré intergiciel (middleware) doit exécuter en premier. Cette commande garantit que l’intergiciel (middleware) d’authentification permettre consommer les valeurs d’en-tête et générer l’URI de redirection correcte.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Appeler le [UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders) méthode dans `Startup.Configure` avant d’appeler [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication) ou intergiciel (middleware) du schéma d’authentification similaire :

```csharp
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseAuthentication();
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Appeler le [UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders) méthode dans `Startup.Configure` avant d’appeler [UseIdentity](/dotnet/api/microsoft.aspnetcore.builder.builderextensions.useidentity) et [UseFacebookAuthentication](/dotnet/api/microsoft.aspnetcore.builder.facebookappbuilderextensions.usefacebookauthentication) ou le schéma d’authentification similaires intergiciel (middleware) :

```csharp
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});

app.UseIdentity();
app.UseFacebookAuthentication(new FacebookOptions()
{
    AppId = Configuration["Authentication:Facebook:AppId"],
    AppSecret = Configuration["Authentication:Facebook:AppSecret"]
});
```

---

Si aucun [ForwardedHeadersOptions](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersoptions) sont spécifiés à l’intergiciel (middleware), les en-têtes par défaut à transférer sont `None`.

### <a name="install-nginx"></a>Installer Nginx

```bash
sudo apt-get install nginx
```

> [!NOTE]
> Si des modules Nginx facultatifs sont installés, il peut être nécessaire de génération Nginx à partir de la source.

Utilisez `apt-get` pour installer Nginx. Le programme d’installation crée un script d’initialisation System V qui exécute Nginx en tant que démon au démarrage du système. Comme il s’agit de l’installation initiale de Nginx, vous devez le démarrer explicitement en exécutant :

```bash
sudo service nginx start
```

Vérifiez qu’un navigateur affiche la page d’accueil par défaut de Nginx.

### <a name="configure-nginx"></a>Configurer Nginx

Pour configurer Nginx comme un proxy inverse pour transférer les demandes à notre application ASP.NET Core, modifiez `/etc/nginx/sites-available/default`. Ouvrez-le dans un éditeur de texte et remplacez le contenu par ce qui suit :

```
server {
    listen 80;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $http_host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Ce fichier de configuration Nginx transfère le trafic public entrant du port `80` au port `5000`.

Une fois la configuration de Nginx est établie, exécutez `sudo nginx -t` pour vérifier la syntaxe des fichiers de configuration. Si le test de fichier de configuration a réussi, forcer Nginx pour appliquer les modifications en exécutant `sudo nginx -s reload`.

## <a name="monitoring-the-app"></a>Surveillance de l’application

Le serveur est configuré pour transférer les demandes faites à `http://<serveraddress>:80` une session sur l’application ASP.NET Core s’exécutant sur Kestrel à `http://127.0.0.1:5000`. Toutefois, Nginx n’est pas configuré pour gérer le processus Kestrel. *systemd* peut être utilisé pour créer un fichier de service pour démarrer et surveiller l’application web sous-jacente. *systemd* est un système d’initialisation qui fournit de nombreuses et puissantes fonctionnalités pour le démarrage, l’arrêt et la gestion des processus. 

### <a name="create-the-service-file"></a>Créer le fichier de service

Créez le fichier de définition de service :

```bash
sudo nano /etc/systemd/system/kestrel-hellomvc.service
```

Voici un exemple de fichier de service pour l’application :

```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/aspnetcore/hellomvc
ExecStart=/usr/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
Restart=always
RestartSec=10  # Restart service after 10 seconds if dotnet service crashes
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

**Remarque :** si l’utilisateur *www-données* n’est pas utilisé par la configuration, l’utilisateur défini ici doit être créé en premier et étant donné la propriété appropriée pour les fichiers.
**Remarque :** Linux possède un système de fichiers respectant la casse. La ASPNETCORE_ENVIRONMENT produit « Production » dans la recherche du fichier de configuration *appsettings. Production.JSON*, et non *appsettings.production.json*.

Enregistrez le fichier et activez le service.

```bash
systemctl enable kestrel-hellomvc.service
```

Démarrez le service et vérifiez qu’il s’exécute.

```
systemctl start kestrel-hellomvc.service
systemctl status kestrel-hellomvc.service

● kestrel-hellomvc.service - Example .NET Web API App running on Ubuntu
    Loaded: loaded (/etc/systemd/system/kestrel-hellomvc.service; enabled)
    Active: active (running) since Thu 2016-10-18 04:09:35 NZDT; 35s ago
Main PID: 9021 (dotnet)
    CGroup: /system.slice/kestrel-hellomvc.service
            └─9021 /usr/local/bin/dotnet /var/aspnetcore/hellomvc/hellomvc.dll
```

Avec la configuration de proxy inverse et le Kestrel gérés via systemd, l’application web est entièrement configurée et sont accessibles à partir d’un navigateur sur l’ordinateur local à `http://localhost`. Il est également accessible à partir d’un ordinateur distant, la restriction de pare-feu pouvant bloquer. Inspecter les en-têtes de réponse, le `Server` en-tête indique à l’application ASP.NET Core sont traitée par Kestrel.

```text
HTTP/1.1 200 OK
Date: Tue, 11 Oct 2016 16:22:23 GMT
Server: Kestrel
Keep-Alive: timeout=5, max=98
Connection: Keep-Alive
Transfer-Encoding: chunked
```

### <a name="viewing-logs"></a>Affichage des journaux

Depuis l’application web à l’aide de Kestrel est géré à l’aide de `systemd`, tous les processus et les événements sont enregistrés dans un journal centralisé. Toutefois, ce journal inclut toutes les entrées pour tous les services et les processus gérés par `systemd`. Pour afficher les éléments propres à `kestrel-hellomvc.service`, utilisez la commande suivante :

```bash
sudo journalctl -fu kestrel-hellomvc.service
```

Si vous voulez appliquer un filtrage supplémentaire, des options chronologiques, comme `--since today`, `--until 1 hour ago` ou une combinaison de ces options, peuvent réduire la quantité d’entrées retournées.

```bash
sudo journalctl -fu kestrel-hellomvc.service --since "2016-10-18" --until "2016-10-18 04:00"
```

## <a name="securing-the-app"></a>Sécurisation de l’application

### <a name="enable-apparmor"></a>Activer AppArmor

Modules de sécurité Linux (LSM) est une infrastructure qui fait partie du noyau Linux depuis Linux 2.6. LSM prend en charge différentes implémentations de modules de sécurité. [AppArmor](https://wiki.ubuntu.com/AppArmor) est un LSM qui implémente un système de contrôle d’accès obligatoire permettant de confiner le programme à un ensemble limité de ressources. Vérifiez qu’AppArmor est activé et configuré correctement.

### <a name="configuring-the-firewall"></a>Configuration du pare-feu

Fermez tous les ports externes qui ne sont pas en cours d’utilisation. Uncomplicated firewall (ufw) fournit une interface de ligne de commande pour `iptables` afin de configurer le pare-feu. Vérifiez que `ufw` est configuré pour autoriser le trafic sur les ports requis.

```bash
sudo apt-get install ufw
sudo ufw enable

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

### <a name="securing-nginx"></a>Sécurisation de Nginx

La distribution par défaut de Nginx n’active pas le protocole SSL. Pour activer des fonctionnalités de sécurité supplémentaires, générez à partir de la source.

#### <a name="download-the-source-and-install-the-build-dependencies"></a>Télécharger la source et installer les dépendances de build

```bash
# Install the build dependencies
sudo apt-get update
sudo apt-get install build-essential zlib1g-dev libpcre3-dev libssl-dev libxslt1-dev libxml2-dev libgd2-xpm-dev libgeoip-dev libgoogle-perftools-dev libperl-dev

# Download Nginx 1.10.0 or latest
wget http://www.nginx.org/download/nginx-1.10.0.tar.gz
tar zxf nginx-1.10.0.tar.gz
```

#### <a name="change-the-nginx-response-name"></a>Changer le nom de la réponse Nginx

Modifiez *src/http/ngx_http_header_filter_module.c* :

```c
static char ngx_http_server_string[] = "Server: Web Server" CRLF;
static char ngx_http_server_full_string[] = "Server: Web Server" CRLF;
```

#### <a name="configure-the-options-and-build"></a>Configurer les options et générer

La bibliothèque PCRE est obligatoire pour les expressions régulières. Les expressions régulières sont utilisées dans la directive d’emplacement pour ngx_http_rewrite_module. http_ssl_module ajoute la prise en charge du protocole HTTPS.

Envisagez d’utiliser un pare-feu d’application web comme *ModSecurity* pour renforcer l’application.

```bash
./configure
--with-pcre=../pcre-8.38
--with-zlib=../zlib-1.2.8
--with-http_ssl_module
--with-stream
--with-mail=dynamic
```

#### <a name="configure-ssl"></a>Configurer le protocole SSL

* Configurer le serveur pour écouter le trafic HTTPS sur le port `443` en spécifiant un certificat valide émis par une autorité de certificat approuvé (CA).

* Renforcer la sécurité en utilisant certaines des pratiques mentionnés dans l’exemple suivant */etc/nginx/nginx.conf* fichier. Vous pouvez par exemple choisir un chiffrement plus fort et rediriger tout le trafic sur HTTP vers HTTPS.

* L’ajout d’un en-tête `HTTP Strict-Transport-Security` (HSTS) garantit que toutes les requêtes ultérieures du client s’effectuent uniquement par le biais du protocole HTTPS.

* N’ajoutez pas l’en-tête de sécurité de Transport Strict ou avez choisi une `max-age` si SSL est désactivé dans le futur.

Ajoutez le fichier de configuration */etc/nginx/proxy.conf* :

[!code-nginx[Main](linux-nginx/proxy.conf)]

Modifiez le fichier de configuration */etc/nginx/nginx.conf*. Dans l’exemple, les sections `http` et `server` figurent dans un même fichier de configuration.

[!code-nginx[Main](linux-nginx/nginx.conf?highlight=2)]

#### <a name="secure-nginx-from-clickjacking"></a>Sécuriser Nginx contre le détournement de clic
Le détournement de clic, ou clickjacking, est une technique malveillante visant à recueillir les clics d’un utilisateur infecté. Elle pousse la victime (le visiteur) à cliquer sur un site infecté. Utilisez X-FRAME-OPTIONS pour sécuriser le site.

Modifiez le fichier *nginx.conf* :

```bash
sudo nano /etc/nginx/nginx.conf
```

Ajoutez la ligne `add_header X-Frame-Options "SAMEORIGIN";` et enregistrez le fichier, puis redémarrez Nginx.

#### <a name="mime-type-sniffing"></a>Détection de type MIME

Cet en-tête empêche la plupart des navigateurs de détourner le type MIME d’une réponse et de remplacer le type de contenu déclaré, car l’en-tête indique au navigateur qu’il ne doit pas substituer le type de contenu de la réponse. Avec l’option `nosniff`, si le serveur indique que le contenu est « text/html », le navigateur le restitue en tant que « text/html ».

Modifiez le fichier *nginx.conf* :

```bash
sudo nano /etc/nginx/nginx.conf
```

Ajoutez la ligne `add_header X-Content-Type-Options "nosniff";` et enregistrez le fichier, puis redémarrez Nginx.
