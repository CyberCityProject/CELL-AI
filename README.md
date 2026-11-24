# CELL-AI

# CellAI — CUDA avant CUDA, sur une PlayStation 3

> **General-purpose computing sur CELL + RSX.**  
> L’IA n’est pas le produit. C’est le banc d’essai.

À l’époque de CUDA, l’idée n’était pas « faire joli à l’écran » : c’était **détourner une architecture parallèle** pour du calcul généraliste — le GPGPU.

**CellAI** reprend la même réflexion, sur le silicium d’une autre époque :

| CUDA / GPGPU | CellAI sur PS3 |
|---|---|
| GPU massivement parallèle | **SPE** (SIMD + Local Store + DMA) + **RSX** détourné |
| Host CPU → kernels | **PPE** → jobs SPE / commandes GCM |
| Workloads (physique, vision, ML…) | **Inférence** miniature pour saturer et mesurer |

Pas un ChatGPT sur console. Une question plus dure :

**Jusqu’où peut-on faire du general-purpose computing sur CELL + RSX ?**

---

## Pourquoi c’est intéressant

Le SDK Sony le dit déjà à demi-mot :

- **CELL** — la puissance est côté SPE (`CBE Overview`) : Local Store **256 KB**, pas de load/store hors LS, tout passe par **DMA MFC**. **SPURS** orchestre jusqu’à 32 workloads. C’est un modèle *host / device / transfer* avant la lettre.
- **RSX** — pipeline graphique documenté (fragment shaders, textures, local mem ~249 Mo, `libgcm`). Pas d’API « compute ». Comme le GPGPU *pré-CUDA* : on détourne shaders et render-to-texture, et on **cartographie** ce qui tient vraiment.
- Pont documenté : génération de commandes RSX **depuis un SPU** (`libgcm_spu`) — CELL et RSX ne sont pas deux îles.

---

## Architecture cible

```
PPE (scheduler, I/O, HUD)
 ├── SPE0…SPE5  →  kernels SIMD (matmul / conv / act), tuilés dans 256 KB LS
 └── RSX        →  viz + expériences GPGPU-like (shaders / RTT)
```

Charge de travail : classifieur image toy (MNIST-like), poids **int8**, métriques à l’écran. Assez simple pour itérer, assez dense pour stresser DMA et occupancy.

---

## Phases

| Phase | Objectif |
|---|---|
| **0** | Socle PSL1GHT — écran RSX, FPS, `.pkg` sur console |
| **1** | Baseline PPE (MLP int8) — référence ms/inférence |
| **2** | Kernels SPE tuilés + multi-SPE (le cœur CELL) |
| **3** | RSX dans la boucle — viz puis détournement compute |
| **4** | Sweep des limites — OOM, DMA vs compute, mesures |

---

## Statut

Projet **en démarrage** (2026-11-24). Dossier code : ce repo.  
Journal de bord (hors Git si besoin) : `JOURNAL_CELLAI.md` sur la machine de lab.

Toolchain : **ps3dev** / PSL1GHT (homebrew), SDK 4.75 en référence docs.

---

## Ce que ce projet n’est pas

- Pas un port d’émulateur
- Pas un LLM moderne sur PS3
- Pas une démo « IA marketing »

C’est de la **archéologie active du compute hétérogène** : lire le silicium, pousser SPE + RSX, écrire ce que les manuels ne disent pas.

---

## Licence

À définir. Code homebrew — respectez les contraintes légales de votre région et du hardware.
