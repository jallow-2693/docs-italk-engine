# Documentation Globale – Italk-Engine

Italk-Engine est un moteur modulaire de gestion de messagerie instantanée écrit en Python.  
Il permet la gestion d’utilisateurs, de groupes, de messages, d’événements et l’extension de ses fonctionnalités via un système de plugins dynamiques (extensions).

---

## Sommaire

- [Présentation et fonctionnalités](#présentation-et-fonctionnalités)
- [Installation & Pré-requis](#installation--pré-requis)
- [Configuration](#configuration)
- [Intégration dans une messagerie](#intégration-dans-une-messagerie)
- [Structure typique d’un serveur](#structure-typique-dun-serveur)
- [Gestion des tokens et sécurité](#gestion-des-tokens-et-sécurité)
- [Extensions](#extensions)
- [Exemple d’utilisation et d’intégration](#exemple-dutilisation-et-dintégration)
- [Déploiement & hébergement](#déploiement--hébergement)
- [FAQ et bonnes pratiques](#faq-et-bonnes-pratiques)
- [Licence](#licence)

---

## Présentation et fonctionnalités

- **Gestion des utilisateurs** : Connexion, déconnexion, métadonnées, état en ligne.
- **Groupes** : Création, gestion, ajout/suppression de membres.
- **Messagerie** : Envoi, réception, historisation des messages (avec timestamp).
- **Système d’événements** : Hooks personnalisables (`on_connect`, `on_disconnect`, `on_message`, `on_error`, etc.).
- **Logging** : Journalisation configurable, extensible par plugin.
- **Extensions/plugins** : Ajout dynamique de fonctionnalités (modération, quotas, logs avancés, etc.).
- **Persistance** : Sauvegarde et rechargement de l’état du moteur (utilisateurs, groupes).
- **Configuration JSON** : Paramétrage centralisé dans un fichier.
- **Tests unitaires fournis** (`pytest`).

---

## Installation & Pré-requis

- **Python 3.7+** recommandé
- Facultatif : environnement virtuel (`venv`)
- Cloner le dépôt :
  ```bash
  git clone https://github.com/jallow-2693/Italk-Engine.git
  cd Italk-Engine
  ```
- Installer les dépendances (standard Python, pas de requirements.txt par défaut)
- Pour certains plugins, installer les modules nécessaires (voir leur documentation)

---

## Configuration

Le fichier principal de configuration est `extensions/config.json`.  
Il contient :
- Les options globales (`logging`, options avancées…)
- La liste des extensions/plugins à charger
- Les paramètres des extensions (messages de bienvenue, limitations, etc.)

Exemple :
```json
{
  "logging": true,
  "advanced_log_file": "advanced_engine.log",
  "advanced_log_level": "INFO",
  "welcome_message": "Bienvenue sur le serveur !",
  "extensions": [
    "welcome",
    "users_management",
    "groups_management",
    "messages_management",
    "events_management",
    "advanced_logging",
    "message_rate_limiter"
  ]
}
```

---

## Intégration dans une messagerie

Vous pouvez importer le moteur dans n’importe quel projet Python (bot, application web, serveur chat personnalisé…).

### Exemple minimal d’import :

```python
from Engine.core import ItalkEngine

engine = ItalkEngine(config_path="extensions/config.json")
```

### Abonnement aux événements :

```python
def on_connect(user):
    print(f"{user.username} connecté !")

engine.on("on_connect", on_connect)
```

### Connexion utilisateur (ex : via un token JWT ou session) :

```python
# À vous de vérifier et décoder le token côté API avant !
user = engine.connect_user(user_id="123", username="Alice")
engine.send_message("123", "Hello !")
```

### Gestion des tokens
- **Italk-Engine ne gère pas la génération ou la validation des tokens d’authentification (JWT, OAuth, etc.)**.  
- **À vous de gérer l’authentification en amont** (API, serveur web, etc.) et de n’appeler `connect_user()` que pour les utilisateurs validés.
- Vous pouvez stocker des infos du token dans le champ `metadata` de l’utilisateur.

---

## Structure typique d’un serveur

```
Italk-Engine/
│
├── app.py (votre point d’entrée, API, bot…)
├── Engine/
│   ├── core.py
│   └── extensions.py
│
├── extensions/
│   ├── __init__.py
│   ├── ...vos extensions...
│   └── config.json
│
└── web/ (optionnel : API REST, site admin, etc.)
```

---

## Gestion des tokens et sécurité

- **Authentification** : Gérez-la côté serveur : API REST, FastAPI, Flask, Django, etc.
- **Session utilisateur** : Passez l’ID et l’username à `engine.connect_user()` après vérification du token.
- **Permissions/admin** : Gérez-les via le champ `metadata` ou via une extension personnalisée (par exemple : `is_admin`, `roles`, etc.).
- **Sécurité** : Ne stockez jamais de token brut dans le moteur, seulement des infos nécessaires à la session.

---

## Extensions

Le moteur charge dynamiquement toutes les extensions listées dans le fichier de configuration.  
Chaque extension doit fournir au minimum une fonction `setup(engine)`.

### Ajouter une extension :

1. Créer un fichier Python dans `extensions/` (ex : `anti_spam.py`)
2. Lister le nom du fichier (sans `.py`) dans `"extensions"` du `config.json`
3. (Optionnel) Gérer une config spécifique dans le même fichier ou via le `config.json`

---

## Exemple d’utilisation et d’intégration

```python
from Engine.core import ItalkEngine

def on_message(user, message):
    print(f"[{message.timestamp}] {user.username} : {message.content}")

def main():
    engine = ItalkEngine()
    engine.on("on_message", on_message)
    # Imaginons que vous avez validé le token côté API :
    user = engine.connect_user("456", "Bob", metadata={"role": "admin"})
    engine.send_message("456", "Coucou tout le monde !")

if __name__ == "__main__":
    main()
```

---

## Déploiement & hébergement

- Lancez simplement votre app Python (`python app.py`)
- Pour une API REST, utilisez un framework (FastAPI conseillé, Flask, etc.) et importez le moteur dans vos endpoints
- Hébergez sur n’importe quel serveur (VPS, cloud, Docker…)
- Pensez à sauvegarder régulièrement le fichier d’état (`engine_state.json` par défaut)

---

## FAQ et bonnes pratiques

- **Puis-je brancher Italk-Engine sur une appli web ou mobile ?**
  Oui, via une API adaptée ou WebSocket, en important le moteur côté backend.

- **Puis-je ajouter des fonctionnalités ?**
  Oui, via les extensions/plugins Python.

- **Comment gérer les droits/admin ?**
  Ajoutez vos contrôles dans les extensions ou via le champ `metadata` utilisateur.

- **Comment limiter les spams ?**
  Utilisez une extension telle que `message_rate_limiter`.

---

## Licence

Projet open-source sous licence MIT.  
Voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

**Auteur :** [jallow-2693](https://github.com/jallow-2693)
