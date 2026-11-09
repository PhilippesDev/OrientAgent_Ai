# OrientAgent

Agent IA conversationnel d'aide à la décision pour l'orientation, l'admission et l'inscription des nouveaux étudiants d'un établissement d'enseignement supérieur.

## Le problème

Pendant les périodes d'inscription, le secrétariat et le service d'orientation sont submergés de demandes répétitives :

- choix de la filière (Systèmes informatiques et réseaux, Génie logiciel, Intelligence artificielle, etc.) ;
- conditions d'admission ;
- calcul des frais d'études ;
- calendrier académique et démarches d'inscription.

Les futurs étudiants reçoivent souvent des réponses tardives ou incomplètes. OrientAgent automatise ces réponses de manière fiable, à partir de documents officiels et de règles de calcul vérifiables.

## Utilisateurs cibles

- Nouveaux diplômés de l'enseignement secondaire
- Futurs étudiants de Licence et de Master
- Agents du secrétariat et du service d'orientation

## Fonctionnalités

- **Orientation personnalisée** : recommande les filières adaptées au profil du candidat (option suivie au secondaire, pourcentage obtenu à l'examen d'État).
- **Calcul exact des frais** : détermine les frais d'inscription et d'études, par tranche ou en totalité, à partir d'une grille tarifaire structurée.
- **Questions-réponses documentaires** : répond aux questions sur l'admission et le règlement grâce à une recherche sémantique (RAG) dans les documents publics.
- **Guidage pas à pas** : indique l'itinéraire administratif et physique de l'inscription (retrait du formulaire, validation du diplôme, paiement de l'acompte, enrôlement et carte d'étudiant).
- **Pré-inscription assistée** : prépare la fiche de pré-inscription et délivre un numéro de dossier temporaire, uniquement après confirmation explicite d'un humain.

## Outils de l'agent

| Outil | Rôle |
|---|---|
| `verifier_eligibilite_et_filiere(section_secondaire, pourcentage_exetat)` | Vérifie l'éligibilité aux conditions d'admission et recommande les filières adaptées. |
| `calculer_frais_etudes(filiere, niveau, statut_tranche)` | Interroge la grille tarifaire pour calculer les frais exacts à payer. |
| `rechercher_faq_orientations(question_utilisateur)` | Recherche sémantique (RAG) dans le guide d'admission et le règlement académique. |
| `guider_parcours_inscription(etape_actuelle, statut_dossier)` | Fournit l'itinéraire administratif et physique exact de l'inscription. |

## Architecture

- **Langage** : Python standard
- **Modèle** : modèle local via Ollama (`qwen2.5-coder:3b`), sans dépendance à un service cloud
- **Boucle agentique** : *Planifier → Agir → Observer*
- **Format des réponses du modèle** : JSON strict
- **Configuration sensible** : lue depuis des variables d'environnement, jamais écrite dans le code

```
Question de l'utilisateur
        │
        ▼
   Planifier ──► choisir un outil
        │
        ▼
     Agir ──► exécuter l'outil (lecture seule, sauf pré-inscription)
        │
        ▼
   Observer ──► analyser le résultat
        │
        ▼
 Réponse finale (ou nouvelle itération, 6 étapes maximum)
```

## Risques et garde-fous

| Risque | Garde-fou |
|---|---|
| Hallucination sur les montants des frais | Passage obligatoire par `calculer_frais_etudes` : le modèle n'invente aucun chiffre |
| Mauvaise orientation vers une filière inadaptée | Vérification d'éligibilité par règles explicites avant toute recommandation |
| Injection de prompt visant à modifier les frais | Consignes et données isolées par des délimiteurs XML ; format JSON strict imposé |
| Boucle infinie ou dérive de l'agent | Boucle limitée à 6 étapes au maximum |
| Action irréversible non voulue | Confirmation humaine obligatoire avant la génération du dossier |
| Écriture de données non désirée | Moindre privilège : outils de calcul et de recherche en lecture seule ; seule la pré-inscription écrit des données |
| Manque de traçabilité | Journal horodaté de chaque décision, de chaque appel d'outil et de son résultat |

## Données

- Grille tarifaire officielle et publique
- Guide des programmes (descriptif des filières)
- Guide d'admission et règlement académique publics
- Base de profils d'étudiants **entièrement fictifs et anonymisés**

Aucune donnée personnelle réelle n'est utilisée. La pré-inscription s'effectue dans une base de démonstration qui simule celle de l'établissement.

## Installation

```bash
# 1. Cloner le dépôt
git clone <url-du-depot>
cd orient-agent

# 2. Créer un environnement virtuel et installer les dépendances
python -m venv .venv
source .venv/bin/activate        # Windows : .venv\Scripts\activate
pip install -r requirements.txt

# 3. Installer Ollama puis télécharger le modèle
ollama pull qwen2.5-coder:3b

# 4. Configurer les variables d'environnement
cp .env.example .env

# 5. Lancer l'agent
python main.py
```

## Feuille de route

- Interface web accessible par lien ou QR code
- Tableau de bord d'administration pour mettre à jour les documents et la grille tarifaire
- Extension à d'autres établissements et à d'autres langues

## Licence

À définir.
