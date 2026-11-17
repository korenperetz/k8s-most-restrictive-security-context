# k8s-most-restrictive-security-context

The most restrictive Pod: every stable (GA, Kubernetes 1.37) security field declared. Accepted by a 1.37 API server.

```yaml
# The most restrictive Pod: every stable (GA, Kubernetes 1.37) security field declared.
# Not included: supplementalGroupsPolicy (still beta).
apiVersion: v1
kind: Pod
metadata:
  name: pause
spec:
  automountServiceAccountToken: false
  enableServiceLinks: false
  hostNetwork: false
  hostPID: false
  hostIPC: false
  # User namespaces: root in the container is an unprivileged uid on the node.
  hostUsers: false
  shareProcessNamespace: false
  # Pod-only fields. The rest are declared per container, which overrides the pod.
  securityContext:
    fsGroup: 65535
    fsGroupChangePolicy: Always
    supplementalGroups: []
    sysctls: []
    seLinuxChangePolicy: MountOption
  containers:
    - name: pause
      image: registry.k8s.io/pause:3.10
      imagePullPolicy: IfNotPresent
      securityContext:
        runAsNonRoot: true
        runAsUser: 65535
        runAsGroup: 65535
        privileged: false
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        procMount: Default
        capabilities:
          drop: ["ALL"]
        seLinuxOptions: {}
        seccompProfile:
          type: RuntimeDefault
        appArmorProfile:
          type: RuntimeDefault
        windowsOptions:
          hostProcess: false
      resources:
        requests:
          cpu: 10m
          memory: 16Mi
          ephemeral-storage: 16Mi
        limits:
          cpu: 100m
          memory: 64Mi
          ephemeral-storage: 64Mi
      volumeMounts:
        - name: tmp
          mountPath: /tmp
  volumes:
    # The only writable path, on disk, capped.
    - name: tmp
      emptyDir:
        sizeLimit: 64Mi
```
