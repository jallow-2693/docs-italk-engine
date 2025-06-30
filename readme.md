# Italk Engine

**Italk Engine** est un moteur logiciel modulaire écrit en Python, conçu pour faciliter la gestion d’utilisateurs, de groupes et de messages, avec prise en charge native des extensions (plugins).

## Qu’est-ce que le moteur Italk Engine ?

Italk Engine est le **noyau** logique du système :  
- Il gère les comptes utilisateurs, leur état de connexion, leurs groupes d’appartenance et tous les messages échangés.
- Il fournit un système d’événements (hooks) et de plugins pour personnaliser ou enrichir son comportement sans modifier le cœur du code.
- Il peut être utilisé en tant que backend pur, être exposé via une API web (Flask, FastAPI, etc.), ou intégré dans des applications desktop/bots/scripts.

## Fonctionnalités principales

- **Gestion d’utilisateurs**
  - Connexion/déconnexion, suivi d’état, métadonnées personnalisées
- **Gestion de groupes**
  - Création de groupes, ajout/retrait de membres, gestion de groupes privés/publics
- **Système de messages**
  - Envoi/réception de messages avec horodatage, gestion des historiques
- **Événements & hooks**
  - Actions automatiques déclenchées sur : connexion, déconnexion, message, erreur, etc.
- **Extensions dynamiques**
  - Plugins Python pour ajouter/modifier des fonctionnalités sans toucher au noyau
- **Persistance**
  - Sauvegarde/restauration de l’état des utilisateurs, groupes, etc.
- **Journalisation (logging)**
  - Système configurable de logs, extensible via plugins
- **Configuration centralisée**
  - Toutes les options sont paramétrables via fichier JSON

## Architecture (vue synthétique)

```
Italk-Engine/
├── Engine/
│   ├── core.py         # Le moteur principal (logique, hooks, persistance…)
│   ├── extensions.py   # Gestion avancée des extensions/plugins
├── extensions/
│   ├── config.json     # Configuration des plugins/extensions & options globales
│   └── ...             # Vos extensions Python (.py)
```

## Comment fonctionne le moteur (hors web)

1. **Initialisation**
   - Charge sa configuration (JSON), ses logs, et les extensions listées.
2. **Gestion des utilisateurs**
   - Ajoute, connecte, déconnecte, stocke les informations utilisateurs (ID, pseudo, métadonnées…)
3. **Groupes**
   - Permet de créer des groupes d’utilisateurs, d’y ajouter/retirer des membres, de gérer leur état.
4. **Messages**
   - Permet d’envoyer des messages de la part d’utilisateurs connectés, chaque message étant horodaté.
5. **Événements/hooks**
   - Déclenche automatiquement des fonctions sur les événements clés (ex : on_connect, on_message…)
6. **Extensions**
   - Charge dynamiquement les modules Python qui étendent ou modifient le comportement du moteur.
7. **Persistance**
   - Sauvegarde automatique (JSON) de l’état du moteur (utilisateurs, groupes, etc.)

## Exemple de code minimal (hors web)

```python
from Engine.core import ItalkEngine

def on_connect(user):
    print(f"{user.username} vient de se connecter !")

def on_message(user, message):
    print(f"{user.username} : {message.content}")

engine = ItalkEngine()
engine.on("on_connect", on_connect)
engine.on("on_message", on_message)

user = engine.connect_user("1", "Alice")
engine.send_message("1", "Bonjour à tous !")
engine.disconnect_user("1")
```

## Points importants

- **Le moteur ne fournit pas d’interface graphique par défaut** : il doit être branché sur une interface web (API, site, app) ou desktop si besoin.
- **La logique d’authentification, de gestion d’utilisateurs, de messages, etc., est centralisée dans le moteur**.
- **L’architecture “plugins” permet d’ajouter facilement des fonctionnalités** (modération, jeux, IA, analytics, etc.) sans toucher au code principal.
- **Il peut être utilisé comme backend autonome ou intégré dans une API web (comme dans le dossier `web/`)**.

## Pour quoi utiliser ce moteur ?

- Construire le backend d’un tchat, d’un forum, d’une plateforme collaborative
- Développer des bots ou assistants multi-utilisateurs
- Gérer une communauté (groupe, permissions, historiques…)
- Ajouter/expérimenter des extensions Python dynamiques
