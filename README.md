# Stap 05 - Heroku PHP Deploy

In deze stap zullen we een PHP project via git op heroku deployen. Je zal de database op combell draaien, de PHP draait op de heroku servers. Via een git push kun je dan makkelijk / snel updates pushen naar de server.

## Nieuwe bestanden / wijzigingen

Er is een Procfile, composer.json en composer.lock file toegevoegd aan het project. Deze files zijn nodig om het project als PHP project uit te voeren op Heroku.

De .gitignore file is uitgebreid: hier staat nu ook een vendor/ mapje vermeld (dit is een map die gegenereerd wordt wanneer je het composer commando zou uitvoeren). Composer is een beetje de npm/yarn, maar dan voor PHP. Je zal dit momenteel nog niet lokaal gebruiken; Heroku zal wél composer gebruiken bij het opstarten van je project.

Aan de devDependencies is mamp-cli toegevoegd. Voer dus opnieuw `yarn` of `npm install` uit om deze dependency te installeren. Mamp-cli maakt het makkelijker om je mamp te starten / stoppen in je projectmap.

## Database connectie

Het kan vervelend zijn om telkens je database instellingen aan te passen wanneer je een project online zet. Om dit makkelijker te maken, hebben we enkele aanpassingen gedaan aan de PHP projectstructuur:

`src/dao/DAO.php` : Hier zie je dat we nu de connectiegegevens ophalen uit "environment" variabelen:

```php
$dbHost = getenv('PHP_DB_HOST') ?: "localhost";
$dbName = getenv('PHP_DB_DATABASE') ?: "todos";
$dbUser = getenv('PHP_DB_USERNAME') ?: "todos";
$dbPass = getenv('PHP_DB_PASSWORD') ?: "todos";
```

`src/index.php`: hier is logica bijgekomen om die environment variabelen in te stellen vanuit een config file:

```php
// basic .env file parsing
if (file_exists("../.env")) {
  $variables = parse_ini_file("../.env", true);
  foreach ($variables as $key => $value) {
    putenv("$key=$value");
  }
}
```

Je ziet in bovenstaande code dat we een .env bestand inlezen (indien het bestaat). Dit env bestand kun je in de root van je project zetten (dus in dezelfde map als je package.json). Maak zo'n .env bestand aan, en plaats hier je lokale database gegevens in:

```php
PHP_DB_DATABASE=todos
PHP_DB_HOST=localhost
PHP_DB_USERNAME=todos
PHP_DB_PASSWORD=todos
```

Online zullen we die variabelen instellen via een configuratiepaneel. Het .env bestand dient enkel voor je lokaal project, en zal niet mee gepushed worden (want: het staat in je .gitignore).

## Heroku Deployment

Installeer heroku-cli: https://devcenter.heroku.com/articles/heroku-cli#download-and-install

Zorg dat je project reeds een git project is. Daarna voer je het heroku create command uit (met extra optie om een Europese server te gebruiken):

```
heroku create --region eu
```

Vervolgens zul je instellen dat we zowel php als nodejs (voor de webpack build) willen gebruiken:

```
heroku buildpacks:set heroku/php
heroku buildpacks:add heroku/nodejs
```

Doe nu een push van je master branch naar heroku (na het maken van een commit uiteraard):

```
git push heroku master
```

Je zou een heleboel logs moeten krijgen, die starten met `remote: `, dit zijn meldingen van de heroku servers. Als alles goed gaat zou je volgende moeten zien (met jouw url uiteraard):

```
remote: -----> Launching...
remote:        Released v17
remote:        https://safe-bayou-12345.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy... done.
To https://git.heroku.com/safe-bayou-12345.git
   0e65821..a290685  master -> master
```

Wanneer je surft naar de URL, zul je waarschijnlijk nog een database exception zien. We moeten ervoor zorgen dat ons PHP project op Heroku connecteert naar onze Combell database.

### Database configuratie Heroku

Kijk eerst en vooral om een database aan te maken en te importeren in je Combell account. Eens dit gebeurt is, kunnen we de connectie instellen op ons Heroku project.

Door de eerdere aanpassingen (gebruik van environment variabelen ipv hardcoded database gegevens in onze code), kunnen we dit in het configuratiepaneel van heroku doen.

Log in op je heroku account en klik op de applicatie die overeenstemt met je php project. Je komt op een overzichtspagina van dit project:

![overzichtspagina heroku php project](../images/heroku-db-01.png)

Klik door naar settings. Je ziet er een knop "Reveal Config vars" staan:

![settings heroku php project](../images/heroku-db-02.png)

Klik op reveal config vars, en voeg 4 variabelen (PHP_DB_DATABASE, PHP_DB_HOST, PHP_DB_USERNAME en PHP_DB_PASSWORD) toe die je database connectie instellen. Gebruik hier je combell database settings:

![db settings in heroku](../images/heroku-db-03.png)

