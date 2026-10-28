# Compte rendu : TP API LogiAI

**Noms :** BAYONNE MANGA   **Groupe :** 3

## 1. Environnement et premiers tests
- Sortie de `python -m pytest` au départ (collez-la) :
 baseline_accuracy : 0.726
          accuracy : 0.774
            rappel : 0.464
         precision : 0.617
               auc : 0.769
- Pourquoi certains tests qui attendent un 422 passent-ils déjà, alors que rien n'est écrit ?
 Si le corps reçu est invalide, FastAPI renvoie 422 sans que tu aies à écrire de vérification manuelle.

## 2. Mon schéma
- Mes bornes et leur justification physique (une ligne par variable) :

| Variable | Min | Max | Pourquoi |
|---|---|---|---|
| distance_km | 1 | 1000 | |
| poids_kg | | | |
| heure_depart | | | |
| pluie_mm | | | |
| trafic_index | | | |
| nb_arrets | | | |

## 3. Ma fonction predict()
def predict(livraison: Livraison):
    predictor = state.get("predictor")
    if predictor is None:
        raise HTTPException(
            status_code=503,
            detail="Modèle absent. Générez les données puis entraînez le modèle avec "
            "`python -m logiai.dataset` et `python -m logiai.train`.",
        )
    return PredictionOut(**predictor.predict(livraison.model_dump()))

## 4. Quatre requêtes
| Requête | Code obtenu | Pourquoi |
|---|---|---|
| livraison normale | 200 | La requête est valide, le modèle renvoie une prédiction. |
| distance_km = -5 | 422 | La valeur est sous la borne minimale `distance_km >= 1`, donc Pydantic rejette la requête. |
| type_vehicule = "avion" | 422 | La valeur n'appartient pas aux choix autorisés (`moto`, `fourgon`, `camion`). |
| modèle absent (supprimez models/ et relancez) | 503 | Le modèle n'est pas chargé ; le service est temporairement indisponible jusqu'à la génération des données et l'entraînement du modèle. |

## 5. Questions de compréhension
1. On charge le modèle au démarrage pour éviter de relire le fichier et de reconstruire le prédicteur à chaque requête. Les prédictions sont ainsi plus rapides.
2. Le développeur définit la borne dans le schéma Pydantic `Livraison`, dans `logiai/api/schemas.py`, avec `Field`. Pydantic vérifie ensuite automatiquement la valeur et FastAPI renvoie 422 si elle est invalide. Dans ce projet, la borne actuelle est `pluie_mm <= 60` (et non 100).
3. Le code 422 indique que les données de la requête sont invalides (par exemple une valeur hors limites). Le code 503 indique que le service ne peut pas traiter la requête pour le moment, ici parce que le modèle n'est pas chargé.
4. Le préfixe `/v1` indique la version du contrat de l'API. Il permet de publier plus tard une version incompatible, comme `/v2/predict`, sans casser les clients qui utilisent encore la v1.
5. Une distance décimale, par exemple `12.5 km`, ne serait pas une valeur entière et serait rejetée par la validation, avec une réponse 422. Le type `float` permet de représenter les distances fractionnaires.

## 6. Décision : modèle absent
Je choisis de démarrer l'API et de répondre 503 sur `/v1/predict` tant que le modèle manque. Cela laisse accessibles la documentation et la route `/health`, et explique clairement comment rétablir les prédictions. En contrepartie, l'application démarre sans pouvoir fournir son service principal, et chaque prédiction échoue jusqu'à ce que le modèle soit généré. Refuser de démarrer évite de lancer une API inutilisable et signale immédiatement le problème, mais rend aussi inaccessibles la documentation et les autres routes ; le diagnostic peut être moins pratique pour un utilisateur ou un système de supervision.

## 7. Lien de la pull request
La pull request n'a pas encore été créée. Ouvrir ce lien pour la créer depuis la branche `TP-API` : https://github.com/joshBM7/TP-API/pull/new/TP-API
