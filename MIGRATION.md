# Migration Guide: kube-ns-suspender v2.x → v3.0

## Overview

This guide helps you migrate from kube-ns-suspender v2.x (Kubernetes 1.21-1.24) to v3.0 (Kubernetes 1.31-1.34).

## Breaking Changes

### 1. Kubernetes Version Requirements

**v2.x**: Supports Kubernetes 1.21 - 1.24
**v3.0**: Supports Kubernetes 1.31 - 1.34

**Action Required**: You must upgrade your Kubernetes cluster to at least version 1.31 before upgrading kube-ns-suspender.

### 2. Removed: batch/v1beta1 CronJob Support

**What Changed**: Support for the deprecated `batch/v1beta1` CronJob API has been completely removed.

**Why**: The `batch/v1beta1` CronJob API was deprecated in Kubernetes 1.21 and removed in Kubernetes 1.25.

**Impact**: If your cluster is already on Kubernetes 1.25+, you're already using `batch/v1` CronJobs, and this won't affect you. The migration was automatic when you upgraded your cluster.

### 3. Updated Dependencies

- **KEDA**: v2.8.1 → v2.18.1
- **client-go**: v0.24.3 → v0.33.5
- **Go**: 1.18 → 1.24.7

**Impact**: If you're using KEDA ScaledObjects, ensure your KEDA installation is v2.15+ (recommended: v2.18+) for full compatibility with Kubernetes 1.31-1.34.

## Pre-Migration Checklist

Before upgrading kube-ns-suspender to v3.0:

- [ ] Verify your Kubernetes cluster version is 1.31 or newer: `kubectl version --short`
- [ ] Verify KEDA version (if using ScaledObjects): `kubectl get deployment -n keda keda-operator -o jsonpath='{.spec.template.spec.containers[0].image}'`
- [ ] Verify no batch/v1beta1 CronJobs exist: `kubectl get cronjobs.v1beta1.batch --all-namespaces` (should return error on K8s 1.25+)
- [ ] Back up your current configuration: `kubectl get deployment -n kube-ns-suspender kube-ns-suspender -o yaml > backup-kns-v1.yaml`
- [ ] Document currently suspended namespaces: `kubectl get namespaces -l kube-ns-suspender/suspended=true -o name > suspended-namespaces.txt`

## Migration Paths

### Path A: Clean Kubernetes Cluster Upgrade (Recommended)

If you haven't upgraded your Kubernetes cluster yet:

1. **Upgrade Kubernetes cluster from 1.24 → 1.31+ following your platform's guidance**:
   - AWS EKS: Follow [EKS upgrade guide](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html)
   - GKE: Follow [GKE upgrade guide](https://cloud.google.com/kubernetes-engine/docs/how-to/upgrading-a-cluster)
   - AKS: Follow [AKS upgrade guide](https://learn.microsoft.com/en-us/azure/aks/upgrade-cluster)

2. **During cluster upgrade**: kube-ns-suspender v2.x will stop working after Kubernetes 1.25 due to v1beta1 removal

3. **After cluster is on 1.31+**: Upgrade kube-ns-suspender to v3.0

### Path B: Already on Kubernetes 1.31+

If your cluster is already on Kubernetes 1.31 or newer but running kube-ns-suspender v2.x:

1. **Verify current installation**:
   ```bash
   kubectl get deployment -n kube-ns-suspender kube-ns-suspender -o jsonpath='{.spec.template.spec.containers[0].image}'
   ```

2. **Upgrade to v3.0** (see upgrade steps below)

### Path C: Stuck Between 1.25-1.30

If your cluster is on Kubernetes 1.25-1.30:

**Problem**: kube-ns-suspender v2.x doesn't work (v1beta1 removed), and v3.0 requires 1.31+

**Solution**:
1. Upgrade Kubernetes cluster to 1.31+ first
2. Then upgrade kube-ns-suspender to v3.0

## Upgrade Steps

### Step 1: Verify Prerequisites

```bash
# Check Kubernetes version (must be 1.31+)
kubectl version --short

# Check current kube-ns-suspender version
kubectl get deployment -n kube-ns-suspender kube-ns-suspender \
  -o jsonpath='{.spec.template.spec.containers[0].image}'

# Check KEDA version if using ScaledObjects
kubectl get deployment -n keda keda-operator \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

### Step 2: Backup Current State

```bash
# Backup deployment configuration
kubectl get deployment -n kube-ns-suspender kube-ns-suspender \
  -o yaml > backup-kns-deployment-v1.yaml

# Backup all kube-ns-suspender resources
kubectl get all -n kube-ns-suspender -o yaml > backup-kns-all-v1.yaml

# Document suspended namespaces
kubectl get namespaces -l kube-ns-suspender/suspended=true \
  -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' \
  > suspended-namespaces.txt
```

### Step 3: Update Image Version

```bash
# Update the deployment to use v3.0.0 (GitHub Container Registry)
kubectl set image deployment/kube-ns-suspender \
  kube-ns-suspender=ghcr.io/adnilim/kube-ns-suspender:v3.0.0 \
  -n kube-ns-suspender

# Or update your manifests/helm values and re-apply
kubectl apply -k manifests/run/overlays/default/
```

### Step 4: Verify Upgrade

```bash
# Check pod is running with new version
kubectl get pods -n kube-ns-suspender

# Check pod logs for any errors
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender --tail=50

# Verify version in logs (should show v3.0.0)
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender | grep "Version:"
```

### Step 5: Test Functionality

```bash
# Check that suspended namespaces are still suspended
kubectl get namespaces -l kube-ns-suspender/suspended=true

# Test suspension on a test namespace
kubectl create namespace kns-upgrade-test
kubectl label namespace kns-upgrade-test kube-ns-suspender/enabled=true
kubectl label namespace kns-upgrade-test kube-ns-suspender/suspended=true

# Wait 60 seconds for kube-ns-suspender to process
sleep 60

# Verify test namespace deployments are scaled to 0
kubectl get deployments -n kns-upgrade-test

# Cleanup test namespace
kubectl delete namespace kns-upgrade-test
```

## Rollback Procedures

### Rollback to v2.x (Only if Kubernetes < 1.25)

**Important**: You can only rollback to v2.x if your Kubernetes cluster is version 1.24 or older. If your cluster is 1.25+, v2.x will NOT work due to v1beta1 API removal.

#### Prerequisites for Rollback:
- Kubernetes cluster version ≤ 1.24
- Backup files from Step 2 above

#### Rollback Steps:

```bash
# 1. Verify Kubernetes version allows v2.x
kubectl version --short
# If server version is 1.25+, DO NOT proceed - v2.x won't work

# 2. Restore previous deployment configuration
kubectl apply -f backup-kns-deployment-v1.yaml

# 3. Verify rollback
kubectl get pods -n kube-ns-suspender
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender --tail=50

# 4. Verify functionality
kubectl get namespaces -l kube-ns-suspender/suspended=true
```

### Emergency: Delete and Unsuspend Everything

If kube-ns-suspender is causing issues and you need to emergency unsuspend all namespaces:

```bash
# 1. Scale kube-ns-suspender to 0 to stop processing
kubectl scale deployment/kube-ns-suspender --replicas=0 -n kube-ns-suspender

# 2. Remove suspended labels from all namespaces
kubectl get namespaces -l kube-ns-suspender/suspended=true -o name | \
  xargs -I {} kubectl label {} kube-ns-suspender/suspended-

# 3. Manually unsuspend all resources in affected namespaces
# For each namespace that was suspended, run:

NAMESPACE="your-namespace-here"

# Scale Deployments back up (from annotation)
kubectl get deployments -n $NAMESPACE -o json | \
  jq -r '.items[] | select(.metadata.annotations["kube-ns-suspender/originalReplicas"]) | .metadata.name' | \
  while read deploy; do
    replicas=$(kubectl get deployment -n $NAMESPACE $deploy -o jsonpath='{.metadata.annotations.kube-ns-suspender/originalReplicas}')
    kubectl scale deployment/$deploy --replicas=$replicas -n $NAMESPACE
  done

# Un-suspend CronJobs
kubectl get cronjobs -n $NAMESPACE -o name | \
  xargs -I {} kubectl patch {} -p '{"spec":{"suspend":false}}' --type=merge

# Remove KEDA paused-replicas annotations
kubectl get scaledobjects.keda.sh -n $NAMESPACE -o name | \
  xargs -I {} kubectl annotate {} autoscaling.keda.sh/paused-replicas-
```

## Troubleshooting

### Issue: Pod fails to start after upgrade

**Symptoms**: Pod in CrashLoopBackOff or Error state

**Diagnosis**:
```bash
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender
kubectl describe pod -n kube-ns-suspender -l app=kube-ns-suspender
```

**Common Causes**:
1. **RBAC permissions outdated**: Update RBAC from manifests: `kubectl apply -f manifests/run/base/rbac.yaml`
2. **Image pull error**: Verify image exists: `docker pull ghcr.io/adnilim/kube-ns-suspender:v3.0.0`
3. **Incompatible Kubernetes version**: Verify cluster is 1.31+: `kubectl version --short`
4. **Wrong image registry**: Ensure you're using the correct GitHub Container Registry image (`ghcr.io/adnilim/kube-ns-suspender:v3.0.0`)

### Issue: CronJobs not being suspended

**Symptoms**: CronJobs continue running even when namespace is labeled as suspended

**Diagnosis**:
```bash
# Check CronJob API version in use
kubectl get cronjobs -n <namespace> -o yaml | grep "apiVersion:"

# Check kube-ns-suspender logs
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender | grep -i cronjob
```

**Solution**: CronJobs must use `batch/v1` API (standard since K8s 1.21). If any CronJobs are still using `batch/v1beta1`, they should have been automatically migrated by Kubernetes during cluster upgrade. Recreate them if needed.

### Issue: KEDA ScaledObjects not working

**Symptoms**: ScaledObjects don't pause when namespace is suspended

**Diagnosis**:
```bash
# Check KEDA version
kubectl get deployment -n keda keda-operator -o jsonpath='{.spec.template.spec.containers[0].image}'

# Check kube-ns-suspender KEDA client logs
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender | grep -i keda
```

**Solution**: Upgrade KEDA to v2.18+ for full compatibility with Kubernetes 1.31-1.34:
```bash
helm upgrade keda kedacore/keda --namespace keda --version 2.18.1
```

## Post-Migration Validation

After successful migration, validate:

```bash
# 1. Check version
kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender | grep "Version: v3.0.0"

# 2. Verify all previously suspended namespaces are still suspended
cat suspended-namespaces.txt | while read ns; do
  echo "Checking namespace: $ns"
  kubectl get deployments -n $ns -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.replicas}{"\n"}{end}'
done

# 3. Test suspend/unsuspend cycle on test namespace
kubectl create namespace kns-migration-test
kubectl create deployment nginx --image=nginx --replicas=2 -n kns-migration-test

# Enable kube-ns-suspender
kubectl label namespace kns-migration-test kube-ns-suspender/enabled=true

# Suspend
kubectl label namespace kns-migration-test kube-ns-suspender/suspended=true
sleep 60
kubectl get deployment nginx -n kns-migration-test  # Should show 0/2 replicas

# Unsuspend
kubectl label namespace kns-migration-test kube-ns-suspender/suspended=false
sleep 60
kubectl get deployment nginx -n kns-migration-test  # Should show 2/2 replicas

# Cleanup
kubectl delete namespace kns-migration-test
```

## Getting Help

If you encounter issues during migration:

1. Check the [Troubleshooting](#troubleshooting) section above
2. Review logs: `kubectl logs -n kube-ns-suspender deployment/kube-ns-suspender --tail=100`
3. Open an issue at https://github.com/adnilim/kube-ns-suspender/issues with:
   - Your Kubernetes version: `kubectl version --short`
   - Your kube-ns-suspender version
   - Relevant logs and error messages
   - Steps taken during migration

## FAQ

**Q: Can I skip intermediate Kubernetes versions (e.g., 1.24 → 1.34 directly)?**

A: For Kubernetes cluster upgrades, follow your platform's guidance (most require upgrading one minor version at a time). For kube-ns-suspender, you can upgrade directly from v2.x to v3.0 once your cluster is on 1.31+.

**Q: Will my suspended namespaces remain suspended during the upgrade?**

A: Yes. The suspension state is stored in namespace labels and annotations on resources. After upgrading, kube-ns-suspender v3.0 will continue managing suspended namespaces normally.

**Q: Do I need to upgrade KEDA?**

A: If you're using KEDA ScaledObjects, yes. Upgrade to KEDA v2.18+ for full compatibility with Kubernetes 1.31-1.34.

**Q: What if I'm on Kubernetes 1.28-1.30?**

A: You must upgrade your Kubernetes cluster to 1.31+ before using kube-ns-suspender v3.0. There's no intermediate version that supports 1.28-1.30.

**Q: Can I run v2.x and v3.0 side-by-side?**

A: No. Running two instances of kube-ns-suspender will cause conflicts as they both try to manage the same namespaces. Choose one version based on your Kubernetes cluster version.
