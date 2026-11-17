# k8s-most-restrictive-security-context

The most restrictive Pod: every stable (GA, Kubernetes 1.37) security field declared. Accepted by a 1.37 API server. Fields that only work when the node supports them say so in a comment.

```yaml
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
  # Depends on: user namespaces on the node (Linux 6.3+, containerd 2.0+ or CRI-O 1.25+,
  # user.max_user_namespaces > 0). Bottlerocket disables them: the sandbox fails to start.
  hostUsers: false
  shareProcessNamespace: false
  # Pod-only fields. The rest are declared per container, which overrides the pod.
  securityContext:
    # Depends on: a volume type that supports ownership changes (emptyDir, most CSI drivers).
    fsGroup: 65535
    fsGroupChangePolicy: Always
    supplementalGroups: []
    sysctls: []
    # Depends on: SELinux enforcing on the node (Bottlerocket, RHEL); ignored otherwise.
    # MountOption also needs the volume's CSI driver to support it, else falls back to Recursive.
    seLinuxChangePolicy: MountOption
  containers:
    - name: pause
      image: registry.k8s.io/pause:3.10.2
      imagePullPolicy: IfNotPresent
      securityContext:
        runAsNonRoot: true
        # Depends on: the image working as this uid (pause runs as 65535).
        runAsUser: 65535
        runAsGroup: 65535
        privileged: false
        allowPrivilegeEscalation: false
        # Depends on: the app writing only to mounted volumes.
        readOnlyRootFilesystem: true
        procMount: Default
        capabilities:
          drop: ["ALL"]
        # Depends on: SELinux on the node; {} takes the runtime's default label. Ignored otherwise.
        seLinuxOptions: {}
        # Depends on: seccomp in the kernel and runtime (containerd, CRI-O).
        seccompProfile:
          type: RuntimeDefault
        # Depends on: AppArmor enabled on the node (Ubuntu, Debian). On nodes without it
        # (Bottlerocket, AL2023, RHEL) the kubelet rejects the pod.
        appArmorProfile:
          type: RuntimeDefault
        # Ignored on Linux; rejected if spec.os.name is linux.
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
