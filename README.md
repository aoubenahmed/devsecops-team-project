# DevSecOps Team Project

Projet d'équipe — 3 étudiants ingénieurs (PFE). Plateforme DevSecOps hybride
sur AWS : infrastructure Terraform + orchestration CI/CD CloudFormation +
sécurité intégrée (SAST/SCA/DAST, secrets scanning, monitoring).

## Organisation du travail

- **Backlog & sprints** : voir le [GitHub Project](https://github.com/aoubenahmed/devsecops-team-project/).
- **Tâches** : chaque tâche est une Issue avec critères d'acceptation et labels
  (`priority/*`, `type/*`, `domain/*`).
- **Sprints** : une milestone par sprint (1 semaine), planifié depuis le backlog.
- **Workflow Git** : branches + Pull Requests, `main` protégé, 1 review minimum.

## Structure prévue

```
├── terraform-manifests/   # Infrastructure (VPC, ECS, stockage)
├── cloudformation/        # Stack CI/CD (CodePipeline, CodeBuild)
├── lambda-function/       # Lambda (normalisation, événements)
├── buildspecs/            # Buildspecs CodeBuild
├── scripts/               # Scripts de déploiement
└── doc/                   # Documentation & architecture
```

## Membres de l'équipe

| Rôle | Étudiant |
|------|----------|
| Terraform / Infrastructure | @aoubenahmed |
| CI/CD / CloudFormation | (à ajouter) |
| Sécurité / QA / Docs | (à ajouter) |