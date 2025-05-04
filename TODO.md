2. Cifrar el manifiesto completo con SOPS / KSOPS

Si no quieres que ningún valor quede a la vista, puedes cifrar todo el Issuer con Mozilla SOPS (que soporta age como backend) y usar el plugin KSOPS para Kustomize. El flujo sería:

    Instala SOPS y asegúrate de tener tu par de claves age.

    Crea issuer.sops.yaml en tu repo:

apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
name: argocd-test-issuer
spec:
acme:
server: https://acme-v02.api.letsencrypt.org/directory
email: mi-correo@example.com
privateKeySecretRef:
name: letsencrypt-prod
solvers: - selector: {}
http01:
ingress: {}

Cífralo in‑place con SOPS / age:

sops --encrypt --age <tu–public–key> issuer.sops.yaml

Tras esto, Git sólo verá el bloque sops: con datos cifrados
GitHub
.

Configura tu kustomization.yaml (o tu helm chart) para invocar KSOPS en el punto de render:

    apiVersion: kustomize.config.k8s.io/v1beta1
    kind: Kustomization
    resources:
      - issuer.sops.yaml
    generators:
      - secret-generator.yaml  # si usas SecretGenerator + KSOPS, o bien sólo resources

    Con el plugin instalado en el repo‑server de Argo CD, en tiempo de sincronización se descifra el Issuer y se aplica con el email ya en claro.

    🔐 Ventaja: ningún dato sensible (ni tu correo) queda en claro en Git.
    ⚠️ Inconveniente: necesitas extender Argo CD con el plugin KSOPS y manejar claves age en el cluster.

¿Qué opción elegir?

    Si prefieres simplicidad y consideras que tu correo no es crítico, déjalo directo en el Issuer.

    Si tu política corporativa exige cifrado absoluto incluso de metadatos, ve por SOPS + KSOPS.

    En cualquier caso, tus privateKeySecretRef y demás secretos 🔑 siguen gestionados por Sealed Secrets (o el método que ya tengas), que desencripta sólo recursos
