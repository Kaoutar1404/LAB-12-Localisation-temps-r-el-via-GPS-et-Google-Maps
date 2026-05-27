🧩 Partie 1 — MySQL (Base de données)

Étape 1.1 — Créer la base

CREATE DATABASE localisation;

⸻

Étape 1.2 — Créer la table position

CREATE TABLE position (
  id int(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
  latitude double NOT NULL,
  longitude double NOT NULL,
  date datetime NOT NULL,
  imei varchar(20) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

✔️ Vérification :

SELECT * FROM position;

➡️ Doit être vide au départ

⸻

🧠 Partie 2 — Backend PHP (API)

📁 Étape 2.1 — Structure du projet

localisation/
  classe/Position.php
  connexion/Connexion.php
  dao/IDao.php
  service/PositionService.php
  createPosition.php
  showPositions.php

⸻

🧱 Étape 2.2 — classe/Position.php

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
    function getId() { return $this->id; }
    function getLatitude() { return $this->latitude; }
    function getLongitude() { return $this->longitude; }
    function getDate() { return $this->date; }
    function getImei() { return $this->imei; }
    function setId($id) { $this->id = $id; }
    function setLatitude($latitude) { $this->latitude = $latitude; }
    function setLongitude($longitude) { $this->longitude = $longitude; }
    function setDate($date) { $this->date = $date; }
    function setImei($imei) { $this->imei = $imei; }
}

⸻

🔌 Étape 2.3 — connexion/Connexion.php

<?php
class Connexion {
    private $connexion;
    public function __construct() {
        $host = 'localhost';
        $dbname = 'localisation';
        $login = 'root';
        $password = '';
        try {
            $dsn = "mysql:host=$host;dbname=$dbname;charset=utf8";
            $this->connexion = new PDO($dsn, $login, $password, [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
            ]);
        } catch (Exception $e) {
            die('Erreur : ' . $e->getMessage());
        }
    }
    function getConnexion() {
        return $this->connexion;
    }
}

⸻

📌 Étape 2.4 — dao/IDao.php

<?php
interface IDao {
    public function create($obj);
    public function update($obj);
    public function delete($obj);
    public function getById($obj);
    public function getAll();
}

⸻

⚙️ Étape 2.5 — service/PositionService.php

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
        return $stmt->fetchAll();
    }
    public function update($obj) {}
    public function delete($obj) {}
    public function getById($obj) {}
}

⸻

📡 Étape 2.6 — createPosition.php

<?php
header('Content-Type: application/json; charset=utf-8');
if ($_SERVER["REQUEST_METHOD"] != "POST") {
    http_response_code(405);
    echo json_encode(["ok" => false, "error" => "POST required"]);
    exit;
}
include_once _DIR_ . '/service/PositionService.php';
include_once _DIR_ . '/classe/Position.php';
$latitude = $_POST['latitude'] ?? null;
$longitude = $_POST['longitude'] ?? null;
$date = $_POST['date'] ?? null;
$imei = $_POST['imei'] ?? null;
$ip = $_SERVER['REMOTE_ADDR'];
if (!$latitude || !$longitude || !$date || !$imei) {
    http_response_code(400);
    echo json_encode(["ok" => false, "error" => "Missing params", "ip" => $ip]);
    exit;
}
try {
    $service = new PositionService();
    $service->create(new Position(null, $latitude, $longitude, $date, $imei));
    echo json_encode(["ok" => true, "ip" => $ip]);
} catch (Exception $e) {
    http_response_code(500);
    echo json_encode(["ok" => false, "error" => $e->getMessage()]);
}

⸻

🗺️ Étape 2.7 — showPositions.php

<?php
include_once _DIR_ . '/service/PositionService.php';
header('Content-Type: application/json; charset=utf-8');
$service = new PositionService();
echo json_encode(["positions" => $service->getAll()]);

⸻

📱 Partie 3 — Android (GPS + Volley)

🧾 Permissions (AndroidManifest.xml)

<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<application
    android:usesCleartextTraffic="true">

⸻

📦 Dependency Volley

implementation 'com.android.volley:volley:1.2.1'

⸻

📍 Layout activity_main.xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:id="@+id/tvLat"
        android:text="Latitude: -"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <TextView
        android:id="@+id/tvLon"
        android:text="Longitude: -"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <Button
        android:id="@+id/btnMap"
        android:text="Afficher Map"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>

⸻

📡 MainActivity (résumé logique)

✔️ GPS → récupère lat/lon
✔️ Volley → envoie vers PHP
✔️ Affiche données

👉 Endpoint :

http://IP_PC/localisation/createPosition.php

⸻

🗺️ Partie 4 — Google Maps

MapsActivity

✔️ récupère JSON
✔️ affiche markers

JSONArray positions = response.getJSONArray("positions");
for (int i = 0; i < positions.length(); i++) {
    JSONObject p = positions.getJSONObject(i);
    double lat = p.getDouble("latitude");
    double lon = p.getDouble("longitude");
    mMap.addMarker(new MarkerOptions()
        .position(new LatLng(lat, lon))
        .title("Position"));
}

