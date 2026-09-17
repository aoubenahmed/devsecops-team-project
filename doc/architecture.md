# Architecture du projet DevSecOps

> Schéma d'architecture de **référence**, adapté au travail d'équipe à 3.
> Chaque module correspond à un membre de l'équipe (voir README).

## Vue globale

```
                               ┌─────────────────────────────────────────────┐
                               │  DEV (3 étudiants)                          │
                               │  branches git + PR + review obligatoire     │
                               └──────────────┬──────────────────────────────┘
                                              │ push / PR sur GitHub
                                              ▼
┌───────────────────────────────────────────────────────────────────────────────────┐
│                         CI/CD  (CloudFormation - membre B)                        │
│                                                                                   │
│   AWS CodePipeline  ──►  CodeBuild: build client                                  │
│        │                          │ buildspecs/                                  │
│        │                          ▼                                               │
│        ├──► CodeBuild SECURITE ──► SAST (Snyk) + secrets scan (git-secrets) + SCA │
│        │                          ▼                                               │
│        └──► Artefacts ──► S3 (versionné, chiffré KMS)                             │
└──────────────────────────────┬────────────────────────────────────────────────────┘
                               │ images Docker poussées
                               ▼
┌───────────────────────────────────────────────────────────────────────────────────┐
│                     PLATEFORME  (Terraform - membre A)                            │
│                                                                                   │
│   VPC /16 multi-AZ ────► subnets publics (ALB + NAT EC2)                          │
│                        │                                                          │
│                        └─► subnets privés ──► ECS staging + ECS prod (EC2/ASG)    │
│                                                │                                  │
│                                                ▼                                  │
│                              Blue/Green via ALB + target groups                   │
└──────────────────────────────┬────────────────────────────────────────────────────┘
                               │
                               ▼
                    Utilisateurs  (HTTP/HTTPS 80/443 → ALB)
```

## Couches de sécurité & ops (transversales - membre C)

```
   CloudTrail (audit API)  ─┐
   AWS Config (drift)       ─┼──►  Security Hub  ─► Lambda normalisation ─► SNS ─► alertes
   CloudWatch (logs/métriques)┘                         (membre C)         (email/Slack)
   Falco (RASP prod) ─────────────────► détection runtime containers
   OWASP ZAP (DAST) ──────────────► scans sur l'app déployée
```

## Explication du flux

1. **Développement** - chaque étudiant travaille sa branche ; merge sur `main`
   uniquement via PR validée (protection activée sur le repo).

2. **Pipeline CI/CD (membre B)** - CodePipeline détecte le push GitHub et orchestre
   les étapes CodeBuild : construction du client, scans de sécurité
   (Snyk/SAST/git-secrets), puis déploiement. Artefacts stockés dans S3 chiffré.

3. **Images** - poussées dans un **ECR** (scan de vulnérabilités intégré),
   consommées par ECS.

4. **Plateforme (membre A)** - création via Terraform du **VPC multi-AZ**, des
   **subnets publics** (ALB + NAT EC2 gratuits) et privés (clusters **ECS
   staging/prod** sur EC2), avec **Blue/Green** : zéro interruption des déploiements.

5. **Entrée utilisateur** - seul l'**ALB** est publié (DNS régional, ports
   80/443) ; les ECS derrière restent privés, egress via NAT (compatible Free Tier).

6. **Sécurité & ops (membre C)** - chaque étape est tracée et auditable :
   SAST/SCA/DAST/RASP, CloudTrail/Config, Security Hub centralise les findings,
   une **Lambda** normalise les alertes et les pousse via **SNS**.

## Découpage des tâches

| Sprint | Tâche | Domaine | Membre |
|--------|-------|---------|--------|
| 1 | Provisionner le VPC et le réseau | Terraform | A |
| 2 | Cluster ECS EC2 (staging + prod) | Terraform | A |
| 3 | Stack CI/CD (CodePipeline + CodeBuild) | CloudFormation | B |
| 3 | Guide de déploiement & README | Docs | C |
| 4 | Lambda normalisation & agrégation | Lambda | C |
| 5 | SAST + secrets scanning dans les buildspecs | Pipeline/Sécurité | B |

> Calendrier : 8 sprints hebdomadaires (Sprint 1 - Sprint 8), milestones visibles
> sur le GitHub Project. Ajustez le découpage selon la charge réelle.