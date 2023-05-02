# Description

This chart is used to bootstrap a kubernetes cluster with argocd. 
You can use this chart to deploy argocd through tools like terraform.

Usage example:

```hcl
resource "helm_release" "argocd" {
  name                  = "argocd"
  repository            = "https://charts.iits.tech"
  chart                 = "argocd"
  version               = "5.22.1"
  namespace             = "argocd"
  create_namespace      = true
  wait                  = true
  atomic                = true
  timeout               = 900 // 15 Minutes
  render_subchart_notes = true
  dependency_update     = true
  wait_for_jobs         = true
  values                = [
    yamlencode({
      projects = {
        infrastructure-charts = {
          projectValues = {
            # Set this to enable stage $STAGE-values.yaml
            stage        = var.stage
            # Example values which are handed down to the project. Like this you can give over informations from terraform to argocd
            traefikElbId = module.terraform_secrets_from_encrypted_s3_bucket.secrets["elb_id"]
            adminDomain  = "admin.${var.domain_name}"
          }

          git = {
            password = var.git_token
            repoUrl  = "https://github.com/victorgetz/infrastructure-charts.git"
          }
        }
      }
    }
    )
  ]
}
```

In the project https://github.com/victorgetz/infrastructure-charts.git it expects a helm chart
named infrastructure-charts and will install everything from there.