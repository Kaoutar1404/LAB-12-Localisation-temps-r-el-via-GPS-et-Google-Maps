📌 Description

Ce projet permet :

* récupérer la position GPS depuis Android ;
* envoyer latitude/longitude vers un serveur PHP avec Volley ;
* enregistrer les positions dans MySQL ;
* afficher toutes les positions sur une carte Google Maps.

Architecture :

* Android → GPS + HTTP POST
* PHP API → reçoit les données
* MySQL → stockage
* Google Maps → affichage des marqueurs

⸻

📁 Structure du projet

localisation/
│
├── classe/
│   └── Position.php
│
├── connexion/
│   └── Connexion.php
│
├── dao/
│   └── IDao.php
│
├── service/
│   └── PositionService.php
│
├── createPosition.php
│
└── showPositions.php

⸻

Partie 1 — MySQL

🔹 Création de la base

CREATE DATABASE localisation;
USE localisation;

⸻

🔹 Création de la table

CREATE TABLE position (
  id int(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
  latitude double NOT NULL,
  longitude double NOT NULL,
  date datetime NOT NULL,
  imei varchar(20) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

⸻

✅ Vérification

SELECT * FROM position;

Au début la table est vide.

⸻

Partie 2 — Backend PHP

🔹 classe/Position.php

<?php
class Position {
    private $id;
    private $latitude;
    private $longitude;
    private $date;
    private $imei;
    function __construct($id, $latitude, $longitude, $date, $imei) {
        $this->id = $id;
        $this->latitude = $latitude;
        $this->longitude = $longitude;
        $this->date = $date;
        $this->imei = $imei;
    }
    function getId() {
        return $this->id;
    }
    function getLatitude() {
        return $this->latitude;
    }
    function getLongitude() {
        return $this->longitude;
    }
    function getDate() {
        return $this->date;
    }
    function getImei() {
        return $this->imei;
    }
    function setId($id) {
        $this->id = $id;
    }
    function setLatitude($latitude) {
        $this->latitude = $latitude;
    }
    function setLongitude($longitude) {
        $this->longitude = $longitude;
    }
    function setDate($date) {
        $this->date = $date;
    }
    function setImei($imei) {
        $this->imei = $imei;
    }
}

⸻

🔹 connexion/Connexion.php

<?php
class Connexion {
    private $connextion;
    public function __construct() {
        $host = 'localhost';
        $dbname = 'localisation';
        $login = 'root';
        $password = '';
        try {
            $dsn = "mysql:host=$host;dbname=$dbname;charset=utf8";
            $this->connextion = new PDO($dsn, $login, $password, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
            ]);
        } catch (Exception $e) {
            die('Erreur : ' . $e->getMessage());
        }
    }
    function getConnextion() {
        return $this->connextion;
    }
}

⸻

🔹 dao/IDao.php

<?php
interface IDao {
    public function create($obj);
    public function update($obj);
    public function delete($obj);
    public function getById($obj);
    public function getAll();
}

⸻

🔹 service/PositionService.php

<?php
include_once _DIR_ . '/../dao/IDao.php';
include_once _DIR_ . '/../classe/Position.php';
include_once _DIR_ . '/../connexion/Connexion.php';
class PositionService implements IDao {
    private $connexion;
    public function __construct() {
        $this->connexion = new Connexion();
    }
    public function create($position) {
        $sql = "INSERT INTO position(latitude, longitude, date, imei)
                VALUES (?, ?, ?, ?)";
        $stmt = $this->connexion
                ->getConnextion()
                ->prepare($sql);
        $stmt->execute([
            $position->getLatitude(),
            $position->getLongitude(),
            $position->getDate(),
            $position->getImei()
        ]);
        return true;
    }
    public function getAll() {
        $query = "SELECT * FROM position";
        $req = $this->connexion
                ->getConnextion()
                ->prepare($query);
        $req->execute();
        return $req->fetchAll(PDO::FETCH_ASSOC);
    }
    public function update($obj) {}
    public function delete($obj) {}
    public function getById($obj) {}
}

⸻

🔹 createPosition.php

<?php
header('Content-Type: application/json; charset=utf-8');
if($_SERVER["REQUEST_METHOD"] != "POST") {
    http_response_code(405);
    echo json_encode([
        "ok" => false,
        "error" => "POST required"
    ]);
    exit;
}
include_once _DIR_ . '/service/PositionService.php';
include_once _DIR_ . '/classe/Position.php';
$latitude = $_POST['latitude'] ?? null;
$longitude = $_POST['longitude'] ?? null;
$date = $_POST['date'] ?? null;
$imei = $_POST['imei'] ?? null;
$ip = $_SERVER['REMOTE_ADDR'];
if ($latitude === null ||
    $longitude === null ||
    $date === null ||
    $imei === null) {
    http_response_code(400);
    echo json_encode([
        "ok" => false,
        "error" => "Missing params",
        "ip" => $ip
    ]);
    exit;
}
try {
    $ss = new PositionService();
    $ss->create(
        new Position(
            null,
            $latitude,
            $longitude,
            $date,
            $imei
        )
    );
    echo json_encode([
        "ok" => true,
        "ip" => $ip
    ]);
} catch(Exception $e) {
    http_response_code(500);
    echo json_encode([
        "ok" => false,
        "error" => $e->getMessage(),
        "ip" => $ip
    ]);
}

⸻

🔹 showPositions.php

<?php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    include_once _DIR_ . '/service/PositionService.php';
    showPositions();
}
function showPositions() {
    $cs = new PositionService();
    header('Content-type: application/json; charset=utf-8');
    echo json_encode(array(
        "positions" => $cs->getAll()
    ));
}

⸻

Partie 3 — Android

🔹 Permissions AndroidManifest.xml

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>

⸻

🔹 Autoriser HTTP

<application
    android:usesCleartextTraffic="true"
    ... >

⸻

🔹 Dépendance Volley

dependencies {
    implementation 'com.android.volley:volley:1.2.1'
}

⸻

🔹 strings.xml

<string name="provider_enabled">
    The provider %s is now enabled
</string>
<string name="provider_disabled">
    The provider %s is now disabled
</string>
<string name="provider_new_status">
    The provider %1$s has now a new status %2$s
</string>
<string name="new_location">
    New Location : Latitude = %1$s,
    Longitude = %2$s,
    Altitude = %3$s
    avec une précision de %4$s mètres
</string>

⸻

🔹 activity_main.xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/tvLat"
        android:text="Latitude: -"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <TextView
        android:id="@+id/tvLon"
        android:text="Longitude: -"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <Button
        android:id="@+id/btnMap"
        android:text="Afficher Map"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>

⸻

🔹 MainActivity.java

// Code MainActivity complet ici
// (reprendre exactement celui fourni dans l’énoncé)

⸻

Partie 4 — Google Maps

🔹 MapsActivity.java

// Code MapsActivity complet ici
// (reprendre exactement celui fourni dans l’énoncé)

⸻

✅ Fonctionnement global

📍 Étapes

1. Android récupère la position GPS
2. Volley envoie latitude/longitude vers PHP
3. PHP insère les données dans MySQL
4. Google Maps récupère les positions
5. Les marqueurs sont affichés sur la carte

⸻

✅ Tests

Vérifier insertion

SELECT * FROM position;

⸻

Tester API insertion

POST :

http://IP_PC/localisation/createPosition.php

⸻

Tester API JSON

POST :

http://IP_PC/localisation/showPositions.php

Réponse :

{
  "positions": [
    {
      "id": 1,
      "latitude": 31.63,
      "longitude": -8.00,
      "date": "2026-05-27 12:00:00",
      "imei": "ANDROID_DEVICE"
    }
  ]
}

⸻

