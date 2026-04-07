# Penpot (Docker) et MCP

Ce dépôt contient une stack **Penpot** via Docker Compose. L’URL publique par défaut est **`http://localhost:9001`** (voir `PENPOT_PUBLIC_URI` dans `docker-compose.yaml`).

## Démarrer Penpot

```bash
docker compose up -d
```

Ouvrez ensuite **<http://localhost:9001>** dans le navigateur.

Documentation générale de configuration : [Penpot — Configuration](https://help.penpot.app/technical-guide/configuration/#penpot-configuration).

---

## MCP Penpot (assistant IA dans Cursor, etc.)

Penpot peut se brancher au [Model Context Protocol (MCP)](https://help.penpot.app/mcp/) pour que des outils comme Cursor interagissent avec le fichier de design ouvert.

Deux approches existent :

| Mode        | Où c’est configuré                                                                     | Quand l’utiliser                                                           |
| ----------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **Distant** | Compte Penpot → **Intégrations** → MCP Server (Beta), + URL `…/mcp/stream?userToken=…` | Instance Penpot **assez récente** qui expose cette page (voir ci‑dessous). |
| **Local**   | Terminal (`npx`) + plugin Penpot + URL `http://localhost:4401/mcp`                     | **Recommandé** pour une instance Docker stable **sans** menu Intégrations. |

### Pourquoi « Intégrations » peut être absent

La page **Compte → Intégrations → MCP Server** dépend de la **version** de l’image Docker Penpot. Les images stables sur Docker Hub peuvent être **en retard** par rapport au SaaS (**design.penpot.app**) ou à la doc. Ce n’est **pas** un réglage manquant dans `docker-compose.yaml` : soit votre build l’affiche, soit non.

Tant que ce menu n’existe pas, utilisez le **mode local** ci‑dessous.

---

## Mode local MCP (sans Intégrations)

### Prérequis

- [Node.js](https://nodejs.org/) (idéalement v20 ou v22).
- Penpot accessible (ex. **<http://localhost:9001>** après `docker compose up`).
- Chromium peut demander l’accès au **réseau local** pour joindre `localhost` depuis une app web ; en cas de blocage, essayez Firefox ou autorisez l’accès dans les paramètres du navigateur.

### Étapes

1. **Démarrer le serveur MCP Penpot** (laisser le terminal ouvert) :

   ```bash
   npx -y @penpot/mcp@latest
   ```

   Sur npm, le tag **`stable` n’existe pas** (d’où l’erreur `ETARGET`). Utilisez **`@latest`** pour la version stable publiée, ou **`@beta`** pour la préversion (`dist-tag` `beta`).

   Vous pouvez aussi épingler une version : `npx -y @penpot/mcp@2.14.1`.

2. **Ouvrir Penpot** dans le navigateur (ex. <http://localhost:9001>), puis ouvrir un **fichier de design**.

3. **Extensions → Charger depuis une URL** et indiquer :

   ```text
   http://localhost:4400/manifest.json
   ```

4. Ouvrir l’interface du plugin et cliquer sur **Connecter au serveur MCP**. Laisser la fenêtre du plugin ouverte pendant l’utilisation.

5. **Configurer Cursor** (ou un autre client MCP compatible HTTP) avec une entrée serveur pointant vers :

   ```text
   http://localhost:4401/mcp
   ```

   Exemple de fragment JSON (adapter au format exact de votre fichier de config Cursor) :

   ```json
   {
     "mcpServers": {
       "penpot": {
         "url": "http://localhost:4401/mcp",
         "type": "http"
       }
     }
   }
   ```

L’authentification en mode local repose sur votre **session navigateur** Penpot (pas de clé MCP dans l’URL).

Référence détaillée côté projet : [README MCP dans le dépôt Penpot](https://github.com/penpot/penpot/blob/main/mcp/README.md).

---

## Mode distant MCP (quand Intégrations est disponible)

1. **Compte → Intégrations → MCP Server (Beta)** : activer, générer la **clé MCP** (affichée une seule fois).
2. Copier l’**URL** fournie (contient `userToken=…`).
3. Dans Cursor, ajouter un serveur MCP HTTP avec cette URL (voir [Aide Penpot — MCP](https://help.penpot.app/mcp/)).
4. Dans Penpot : **Fichier → MCP Server → Connect** sur le fichier ouvert.

Le domaine dans l’URL doit correspondre à **`PENPOT_PUBLIC_URI`** de votre instance (ex. `http://localhost:9001` en local).

---

## Rappels sécurité

- La clé MCP (mode distant) est sensible : ne pas la commiter ni la partager.
- Un agent MCP connecté peut **modifier** la page actuellement focalisée dans Penpot : commencer par des actions en lecture seule pour valider le setup.

---

## Note

Le MCP **Penpot** (fichiers `.pen` / Plugin API) est distinct d’autres serveurs MCP nommés « pencil » ou similaires dans certains environnements.
