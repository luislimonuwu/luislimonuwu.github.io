<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Seguimiento en Vivo</title>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>
    <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyAFn9WFZcuHxOKm6y10GoSyflcxCZDo168&callback=initMap" async defer></script>
    <style>
        #map {
            height: 100vh;
            width: 100%;
        }
    </style>
</head>
<body>
    <div id="map"></div>

    <script>
        // Configuración de Firebase
        const firebaseConfig = {
            apiKey: "AIzaSyAFn9WFZcuHxOKm6y10GoSyflcxCZDo168",
            authDomain: "seguimiento-en-vivo-fcf6a.firebaseapp.com",
            databaseURL: "https://seguimiento-en-vivo-fcf6a-default-rtdb.firebaseio.com/",
            projectId: "seguimiento-en-vivo-fcf6a",
            storageBucket: "seguimiento-en-vivo-fcf6a.appspot.com",
            messagingSenderId: "1234567890",
            appId: "1:1234567890:web:abcdefg"
        };

        // Inicializar Firebase
        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        let map;
        let marker;
        let path = [];

        function initMap() {
            map = new google.maps.Map(document.getElementById("map"), {
                center: { lat: 20.97, lng: -89.62 }, // Coordenadas iniciales (Mérida)
                zoom: 15,
            });

            marker = new google.maps.Marker({
                position: { lat: 20.97, lng: -89.62 },
                map: map,
                title: "Ubicación Actual",
            });

            const recorridoPath = new google.maps.Polyline({
                path: path,
                geodesic: true,
                strokeColor: "#FF0000",
                strokeOpacity: 1.0,
                strokeWeight: 2,
            });
            recorridoPath.setMap(map);

            // SI EL USUARIO ESTÁ EN SU TELÉFONO, COMPARTIRÁ SU UBICACIÓN
            if ("geolocation" in navigator) {
                navigator.geolocation.watchPosition(
                    (position) => {
                        const lat = position.coords.latitude;
                        const lng = position.coords.longitude;

                        marker.setPosition({ lat, lng });
                        path.push({ lat, lng });
                        recorridoPath.setPath(path);
                        map.setCenter({ lat, lng });

                        // Guardar en Firebase
                        database.ref("ubicacion").set({ lat, lng });
                    },
                    (error) => {
                        console.error("Error al obtener ubicación:", error);
                    },
                    { enableHighAccuracy: true }
                );
            }

            // SI SE ABRE EN UNA COMPUTADORA, SOLO MOSTRARÁ EL RECORRIDO
            database.ref("ubicacion").on("value", (snapshot) => {
                if (snapshot.exists()) {
                    const data = snapshot.val();
                    const latLng = { lat: data.lat, lng: data.lng };
                    marker.setPosition(latLng);
                    path.push(latLng);
                    recorridoPath.setPath(path);
                    map.setCenter(latLng);
                }
            });
        }
    </script>
</body>
</html>
