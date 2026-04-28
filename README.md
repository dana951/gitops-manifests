# gitops-manifests

GitOps repository that defines the desired Kubernetes workload state for the [`podinfo`](https://github.com/dana951/app-source.git) application.

This repository is part of the larger [`eks-gitops-platform`](https://github.com/dana951/eks-gitops-platform) portfolio.  
Its responsibility is to hold deployable Helm configuration that Argo CD continuously reconciles to EKS.

## What This Repo Does

- Acts as the **source of truth** for application deployment state.
- Stores a reusable Helm chart for `podinfo`.
- Stores environment overlays (`dev`, `qa`, `staging`, `prod`) with per-environment values.
- Receives image tag updates from the CI pipeline and exposes them to Argo CD for rollout.

## Repository Layout

```text
gitops-manifests/
├── charts/
│   └── podinfo/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── _helpers.tpl
│           ├── configmap.yaml
│           ├── deployment.yaml
│           └── service.yaml
└── environments/
    └── podinfo/
        ├── dev/values.yaml
        ├── qa/values.yaml
        ├── staging/values.yaml
        └── prod/values.yaml
```

## Helm Chart Scope

The `podinfo` chart currently templates:
- `Deployment`
- `Service`
- `ConfigMap`

Key configurable settings:
- Image repository/tag
- Replica count
- Service ports
- App environment variables (`APP_ENV`, `THEME_COLOR`, `APP_VERSION`, `GIT_SHA`)
- `nodeSelector` for workload placement

## Environment Model

Each environment has a dedicated `values.yaml` under `environments/podinfo/`:

- `dev` - early validation
- `qa` - test verification
- `staging` - pre-production validation
- `prod` - production release

In this setup, environments mainly override:
- `env.APP_ENV`
- `env.THEME_COLOR`
- `image.tag`

## Local Validation

From the repository root:

```bash
# Render manifests for an environment
helm template podinfo ./charts/podinfo \
  -f ./environments/podinfo/dev/values.yaml

# Optional: validate chart structure
helm lint ./charts/podinfo
```

## Change and Promotion Guidance

- Keep chart defaults in `charts/podinfo/values.yaml` environment-agnostic.
- Apply environment-specific values only in `environments/podinfo/<env>/values.yaml`.
- Promote by updating the target environment image tag through pull requests.

## Related Repositories

- Platform overview: [`eks-gitops-platform`](https://github.com/dana951/eks-gitops-platform)
- Infrastructure (EKS, Jenkins, Argo CD): [`infra-aws`](https://github.com/dana951/infra-aws)
- Application source and CI pipeline: [`app-source`](https://github.com/dana951/app-source)
- Test automation gates: [`tests-repo`](https://github.com/dana951/tests-repo)
- Argo CD bootstrap (App of Apps): [`argocd-apps`](https://github.com/dana951/argocd-apps)

## License

See `LICENSE` in this repository.

