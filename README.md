Voici le TP propre en Markdown prêt à copier-coller (version propre, corrigée et bien structurée) :

⸻

📍 TP — Système de localisation (Android + PHP + MySQL)

⸻

🗄️ Partie 1 — MySQL

1.1 Créer la base de données

CREATE DATABASE localisation;
USE localisation;

⸻

1.2 Créer la table position

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

⸻

🧠 Partie 2 — Backend PHP

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
└── showPositions.php

⸻

📌 2.1 Position.php (Modèle)

<?php
class Position {
    private $id;
    private $latitude;
    private $longitude;
    private $date;
    private $imei;
    public function __construct($id, $latitude, $longitude, $date, $imei) {
        $this->id = $id;
        $this->latitude = $latitude;
        $this->longitude = $longitude;
        $this->date = $date;
        $this->imei = $imei;
    }
    public function getId() { return $this->id; }
    public function getLatitude() { return $this->latitude; }
    public function getLongitude() { return $this->longitude; }
    public function getDate() { return $this->date; }
    public function getImei() { return $this->imei; }
    public function setId($id) { $this->id = $id; }
    public function setLatitude($latitude) { $this->latitude = $latitude; }
    public function setLongitude($longitude) { $this->longitude = $longitude; }
    public function setDate($date) { $this->date = $date; }
    public function setImei($imei) { $this->imei = $imei; }
}

⸻

📌 2.2 Connexion.php (PDO)

<?php
class Connexion {
    private $connexion;
    public function __construct() {
        $host = "localhost";
        $dbname = "localisation";
        $login = "root";
        $password = "";
        try {
            $dsn = "mysql:host=$host;dbname=$dbname;charset=utf8";
            $this->connexion = new PDO($dsn, $login, $password);
            $this->connexion->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (Exception $e) {
            die("Erreur : " . $e->getMessage());
        }
    }
    public function getConnexion() {
        return $this->connexion;
    }
}

⸻

📌 2.3 IDao.php

<?php
interface IDao {
    public function create($obj);
    public function update($obj);
    public function delete($obj);
    public function getById($obj);
    public function getAll();
}

⸻

📌 2.4 PositionService.php

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
        $sql = "INSERT INTO position(latitude, longitude, date, imei) VALUES (?, ?, ?, ?)";
        $stmt = $this->connexion->getConnexion()->prepare($sql);
        $stmt->execute([
            $position->getLatitude(),
            $position->getLongitude(),
            $position->getDate(),
            $position->getImei()
        ]);
        return true;
    }
    public function getAll() {
        $sql = "SELECT * FROM position";
        $stmt = $this->connexion->getConnexion()->prepare($sql);
        $stmt->execute();
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }
    public function update($obj) {}
    public function delete($obj) {}
    public function getById($obj) {}
}

⸻

📌 2.5 createPosition.php

<?php
header('Content-Type: application/json');
if ($_SERVER["REQUEST_METHOD"] !== "POST") {
    http_response_code(405);
    echo json_encode(["ok" => false, "error" => "POST required"]);
    exit;
}
include_once "service/PositionService.php";
include_once "classe/Position.php";
$latitude = $_POST['latitude'] ?? null;
$longitude = $_POST['longitude'] ?? null;
$date = $_POST['date'] ?? null;
$imei = $_POST['imei'] ?? null;
if (!$latitude || !$longitude || !$date || !$imei) {
    http_response_code(400);
    echo json_encode(["ok" => false, "error" => "Missing params"]);
    exit;
}
$service = new PositionService();
$service->create(new Position(null, $latitude, $longitude, $date, $imei));
echo json_encode(["ok" => true]);

⸻

📌 2.6 showPositions.php

<?php
include_once "service/PositionService.php";
header('Content-Type: application/json');
$service = new PositionService();
echo json_encode([
    "positions" => $service->getAll()
]);

⸻

📱 Partie 3 — Android (résumé)

Permissions

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.INTERNET"/>

⸻

Volley dependency

implementation 'com.android.volley:volley:1.2.1'

⸻

URL API

String insertUrl = "http://YOUR_IP/localisation/createPosition.php";
String showUrl = "http://YOUR_IP/localisation/showPositions.php";

⸻

🗺️ Partie 4 — Google Maps

Objectif

* récupérer positions JSON
* afficher markers sur la carte

⸻

JSON attendu

{
  "positions": [
    {
      "latitude": 33.5,
      "longitude": -7.6
    }
  ]
}



