# Guide de Tests Postman - Architecture Microservices

## Services disponibles
- **Eureka Server**: http://localhost:8761
- **Service Client**: http://localhost:8081
- **Service Car**: http://localhost:8082

---

## Scénario de test complet

### Étape 1 : Créer un client

**Méthode**: `POST`  
**URL**: `http://localhost:8081/api/clients`  
**Headers**: 
```
Content-Type: application/json
```

**Body** (raw JSON):
```json
{
  "nom": "Salma",
  "age": 22
}
```

**Résultat attendu**: 
```json
{
  "id": 1,
  "nom": "Salma",
  "age": 22.0
}
```

**Note**: Notez l'`id` retourné (ex: 1) pour les étapes suivantes.

---

### Étape 2 : Récupérer tous les clients

**Méthode**: `GET`  
**URL**: `http://localhost:8081/api/clients`

**Résultat attendu**: 
```json
[
  {
    "id": 1,
    "nom": "Salma",
    "age": 22.0
  }
]
```

---

### Étape 3 : Récupérer un client par ID

**Méthode**: `GET`  
**URL**: `http://localhost:8081/api/clients/1`

**Résultat attendu**: 
```json
{
  "id": 1,
  "nom": "Salma",
  "age": 22.0
}
```

---

### Étape 4 : Tester WebClient (appel inter-service)

**Méthode**: `GET`  
**URL**: `http://localhost:8082/api/test/client/1`

**Description**: Ce endpoint teste que service-car peut appeler service-client via WebClient et Eureka.

**Résultat attendu**: 
```json
{
  "id": 1,
  "nom": "Salma",
  "age": 22.0
}
```

**Si ça fonctionne**: WebClient résout correctement "SERVICE-CLIENT" via Eureka ! ✅

---

### Étape 5 : Créer une voiture liée au client

**Méthode**: `POST`  
**URL**: `http://localhost:8082/api/cars`  
**Headers**: 
```
Content-Type: application/json
```

**Body** (raw JSON):
```json
{
  "marque": "Toyota",
  "modele": "Yaris",
  "clientId": 1
}
```

**Résultat attendu**: 
```json
{
  "id": 1,
  "marque": "Toyota",
  "modele": "Yaris",
  "clientId": 1,
  "client": null
}
```

**Note**: `client` est `null` car il n'est pas enrichi lors de la création.

---

### Étape 6 : Récupérer toutes les voitures (avec enrichissement)

**Méthode**: `GET`  
**URL**: `http://localhost:8082/api/cars`

**Description**: Ce endpoint enrichit chaque voiture avec les données du client via WebClient.

**Résultat attendu**: 
```json
[
  {
    "id": 1,
    "marque": "Toyota",
    "modele": "Yaris",
    "clientId": 1,
    "client": {
      "id": 1,
      "nom": "Salma",
      "age": 22.0
    }
  }
]
```

**✅ Succès**: Le pattern d'enrichissement fonctionne ! La voiture contient maintenant les données du client.

---

### Étape 7 : Récupérer les voitures d'un client spécifique

**Méthode**: `GET`  
**URL**: `http://localhost:8082/api/cars/byClient/1`

**Résultat attendu**: 
```json
[
  {
    "id": 1,
    "marque": "Toyota",
    "modele": "Yaris",
    "clientId": 1,
    "client": {
      "id": 1,
      "nom": "Salma",
      "age": 22.0
    }
  }
]
```

---

## Tests supplémentaires

### Créer un deuxième client
```json
POST http://localhost:8081/api/clients
{
  "nom": "Ahmed",
  "age": 30
}
```

### Créer plusieurs voitures pour différents clients
```json
POST http://localhost:8082/api/cars
{
  "marque": "Renault",
  "modele": "Clio",
  "clientId": 1
}

POST http://localhost:8082/api/cars
{
  "marque": "BMW",
  "modele": "X5",
  "clientId": 2
}
```

---

## Vérification Eureka

Pour vérifier que les services sont bien enregistrés dans Eureka :
- Ouvrir: http://localhost:8761
- Section "Instances currently registered with Eureka" doit afficher:
  - SERVICE-CLIENT (UP)
  - SERVICE-CAR (UP)

---

## Dépannage

### Erreur: "No instances available for SERVICE-CLIENT"
- Vérifier que service-client est démarré et visible dans Eureka
- Vérifier que `@LoadBalanced` est présent sur WebClient.Builder
- Vérifier la dépendance Spring Cloud LoadBalancer

### Erreur: 404 Not Found
- Vérifier le port (8081 pour client, 8082 pour car)
- Vérifier le chemin exact: `/api/clients` ou `/api/cars`

### Erreur: MySQL connection
- Vérifier que MySQL est démarré
- Vérifier username/password dans application.yml

