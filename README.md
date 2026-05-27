Localisation Android — GPS + PHP + MySQL + Google Maps

Application Android de géolocalisation en temps réel :
l'appareil envoie ses coordonnées GPS à un serveur PHP/MySQL via Volley,
et une activité Google Map les affiche sous forme de markers.

---

## Architecture

```
Android App
│
├── MainActivity       ← GPS listener + POST via Volley → createPosition.php
└── MapsActivity       ← POST via Volley → showPositions.php → markers

Serveur PHP (WAMP/XAMPP)
└── localisation/
    ├── classe/Position.php          ← Modèle objet
    ├── connexion/Connexion.php      ← PDO wrapper
    ├── dao/IDao.php                 ← Interface CRUD
    ├── service/PositionService.php  ← INSERT + SELECT
    ├── createPosition.php           ← API POST insertion
    └── showPositions.php            ← API POST lecture JSON

MySQL
└── localisation.position            ← Table stockage
```

---

## Partie 1 — MySQL

### 1.1 Créer la base

```sql
CREATE DATABASE localisation;
```

### 1.2 Créer la table

```sql
USE localisation;

CREATE TABLE position (
  id        INT(11)     NOT NULL PRIMARY KEY AUTO_INCREMENT,
  latitude  DOUBLE      NOT NULL,
  longitude DOUBLE      NOT NULL,
  date      DATETIME    NOT NULL,
  imei      VARCHAR(20) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

> ✅ **Checkpoint** : `SELECT * FROM position;` renvoie un résultat vide.

---

## Partie 2 — Backend PHP

### Structure des fichiers

```
localisation/
├── classe/
│   └── Position.php
├── connexion/
│   └── Connexion.php
├── dao/
│   └── IDao.php
├── service/
│   └── PositionService.php
├── createPosition.php
└── showPositions.php
```

Placer ce dossier dans le répertoire web de WAMP/XAMPP :
- Windows : `C:\wamp64\www\` ou `C:\xampp\htdocs\`

### 2.1 `classe/Position.php` — Modèle

Représente une ligne de la table. Encapsule les 5 champs avec getters/setters.

**Pourquoi une classe ?**
Manipuler des objets `new Position(...)` plutôt que des tableaux de chaînes
permet de valider, typer et accéder aux champs proprement.

### 2.2 `connexion/Connexion.php` — PDO

Ouvre une connexion unique vers MySQL.

**Options PDO importantes :**
- `ERRMODE_EXCEPTION` : toute erreur SQL lève une `PDOException` (pas de silence)
- `FETCH_ASSOC` : `fetchAll()` retourne des tableaux associatifs

> ⚠️ Adapter `$login` / `$password` selon votre configuration WAMP/XAMPP.

### 2.3 `dao/IDao.php` — Interface CRUD

Impose un contrat standard : `create`, `update`, `delete`, `getById`, `getAll`.
Dans ce TP seuls `create` et `getAll` sont implémentés.

### 2.4 `service/PositionService.php` — Couche données

- `create(Position)` → `INSERT` avec requête préparée (`?`)
- `getAll()` → `SELECT *` retourne un tableau associatif

**Pourquoi les requêtes préparées ?**
Protègent contre les injections SQL et gèrent correctement les formats
(virgule dans les doubles, caractères spéciaux dans l'IMEI).

> ✅ **Checkpoint** : un appel à `create()` insère une ligne visible dans phpMyAdmin.

### 2.5 `createPosition.php` — API insertion

| Paramètre | Type   | Description               |
|-----------|--------|---------------------------|
| latitude  | double | Coordonnée GPS            |
| longitude | double | Coordonnée GPS            |
| date      | string | Format `YYYY-MM-DD HH:mm:ss` |
| imei      | string | Identifiant appareil      |

**Réponse JSON :**
```json
{ "ok": true, "ip": "192.168.x.x" }
```

Le champ `ip` renvoie `$_SERVER['REMOTE_ADDR']` (utile pour debug NAT).

### 2.6 `showPositions.php` — API lecture

**Réponse JSON :**
```json
{
  "positions": [
    { "id": 1, "latitude": 33.58, "longitude": -7.59, "date": "2024-01-01 12:00:00", "imei": "abc123" }
  ]
}
```

> ✅ **Checkpoint** : Postman → POST `showPositions.php` → JSON avec tableau `positions`.

---

## Partie 3 — Android

### 3.1 Créer le projet

Android Studio → **Empty Activity** → Nom : `Localisation`

### 3.2 `AndroidManifest.xml` — Permissions

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```

Et dans `<application>` pour autoriser HTTP en TP :
```xml
android:usesCleartextTraffic="true"
```

| Permission | Utilité |
|---|---|
| `ACCESS_FINE_LOCATION` | GPS précis |
| `ACCESS_COARSE_LOCATION` | Localisation réseau |
| `INTERNET` | Requêtes Volley |
| `READ_PHONE_STATE` | IMEI (Android < 10) |

### 3.3 `build.gradle` — Dépendances

```gradle
implementation 'com.android.volley:volley:1.2.1'
implementation 'com.google.android.gms:play-services-maps:18.2.0'
```

### 3.4 `MainActivity.java`

**Logique :**
1. Initialiser `RequestQueue` Volley
2. Vérifier + demander la permission `ACCESS_FINE_LOCATION` au runtime
3. `locationManager.requestLocationUpdates(GPS_PROVIDER, 60000, 150, listener)`
4. `onLocationChanged` → afficher lat/lon + POST via Volley

**Paramètres GPS :**
- `minTime = 60 000 ms` (1 minute entre deux mises à jour)
- `minDistance = 150 m`

**Identifiant appareil :**
- Android récent → `ANDROID_ID` (stable, pas de permission spéciale)
- Android < 10 → `TelephonyManager.getDeviceId()` (IMEI, nécessite `READ_PHONE_STATE`)

**Format date envoyé :**
```java
new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault()).format(new Date())
```

### 3.5 `MapsActivity.java`

1. `SupportMapFragment.getMapAsync(this)` → callback `onMapReady`
2. POST vers `showPositions.php` via `JsonObjectRequest`
3. Parser `response.getJSONArray("positions")` → `addMarker()` par position

> ✅ **Checkpoint** : markers visibles sur la carte si la table n'est pas vide.

---

## Partie 4 — Google Maps

### Obtenir une clé API

1. [Google Cloud Console](https://console.cloud.google.com/) → créer un projet
2. Activer **Maps SDK for Android**
3. Créer une clé API → restreindre à votre `package name` + `SHA-1`
4. Coller dans `res/values/google_maps_api.xml` :

```xml
<resources>
    <string name="google_maps_key" templateMergeStrategy="preserve" translatable="false">
        VOTRE_CLE_API_ICI
    </string>
</resources>
```

---

## Configuration réseau

> ⚠️ Android et le PC serveur doivent être sur le **même réseau Wi-Fi**.

1. Trouver l'IP du PC :
   - Windows : `ipconfig` → chercher l'IP sous l'adaptateur Wi-Fi
   - Linux/Mac : `ip addr` ou `ifconfig`
2. Remplacer dans **MainActivity.java** et **MapsActivity.java** :
   ```java
   "http://192.168.43.228/localisation/createPosition.php"
   //        ^^^^^^^^^^^^ remplacer par votre IP
   ```
3. Vérifier que WAMP/XAMPP est démarré (icône verte).
4. Tester dans le navigateur du PC : `http://localhost/localisation/showPositions.php`

---

## Checkpoints résumés

| Étape | Vérification |
|---|---|
| MySQL | `SELECT * FROM position;` → table vide |
| PHP create | Postman POST `createPosition.php` → `{"ok":true}` |
| PHP show | Postman POST `showPositions.php` → `{"positions":[]}` |
| Android GPS | Toast "New Location" à chaque position |
| Android Volley | Ligne ajoutée dans phpMyAdmin après chaque toast |
| Google Map | Markers visibles sur la carte |

---

## Dépannage courant

| Problème | Cause probable | Solution |
|---|---|---|
| `Erreur réseau` (Android) | IP incorrecte ou serveur éteint | Vérifier IP + WAMP actif |
| `cleartext traffic` | HTTP bloqué par Android | `usesCleartextTraffic="true"` dans Manifest |
| Pas de mise à jour GPS | Émulateur sans GPS | Utiliser un vrai appareil ou envoyer des coordonnées fictives via l'émulateur |
| `Permission denied` | Runtime permission manquante | Vérifier `onRequestPermissionsResult` |
| Markers absents | Table vide ou URL showPositions incorrecte | Vérifier en Postman + phpMyAdmin |
| `PDOException` | Mauvais login/password MySQL | Corriger `Connexion.php` |
