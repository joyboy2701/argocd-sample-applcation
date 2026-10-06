# NGINX with Argo Rollouts

The dev manifests use an Argo Rollout with a canary strategy. Updates move to
50% canary weight, pause for 30 seconds, then promote automatically to 100%.
The first deployment goes directly to the desired replica count.

The existing ClusterIP Service selects both stable and canary pods. Without a
traffic router, weights are approximated using pod counts; one desired replica
does not provide precise traffic percentages. Updates can temporarily use two
pods because `maxSurge` is 1 and `maxUnavailable` is 0.

## Cluster prerequisites

Install the Argo Rollouts controller and CRDs before syncing this application.
Argo CD alone does not install them. Follow the official installation guide:
https://argoproj.github.io/argo-rollouts/installation/

If migrating an existing deployment, ensure Argo CD prunes the old
`Deployment/demo-nginx` after the Rollout is healthy. Both resources select the
same application labels, so leaving the old Deployment running would keep its
pods behind the Service. Review this migration before syncing with automatic
pruning enabled.

## Render and monitor

```sh
kubectl kustomize kustomize/overlays/dev
```

Keep the Argo CD application pointed at `kustomize/overlays/dev` and sync it.
With the Argo Rollouts kubectl plugin installed, monitor updates with:

```sh
kubectl argo rollouts get rollout demo-nginx -n demo --watch
```

Change the NGINX tag in `kustomize/overlays/dev/kustomization.yaml` to trigger
an image update through GitOps.
