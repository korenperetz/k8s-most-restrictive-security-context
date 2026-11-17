# k8s-most-restrictive-security-context

Last updated: 1.37

```yaml
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
  # Requires user namespaces
  hostUsers: false
  shareProcessNamespace: false
  securityContext:
    # Requires ownership-capable volumes
    fsGroup: 65535
    fsGroupChangePolicy: Always
    supplementalGroups: []
    sysctls: []
    # Requires SELinux, CSI support
    seLinuxChangePolicy: MountOption
  containers:
    - name: pause
      image: registry.k8s.io/pause:3.10.2
      imagePullPolicy: IfNotPresent
      securityContext:
        runAsNonRoot: true
        # Requires image uid
        runAsUser: 65535
        runAsGroup: 65535
        privileged: false
        allowPrivilegeEscalation: false
        # Requires writable mounts
        readOnlyRootFilesystem: true
        procMount: Default
        capabilities:
          drop: ["ALL"]
        # Requires SELinux
        seLinuxOptions: {}
        # Requires seccomp
        seccompProfile:
          type: RuntimeDefault
        # Requires AppArmor
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
    - name: tmp
      emptyDir:
        sizeLimit: 64Mi
```
