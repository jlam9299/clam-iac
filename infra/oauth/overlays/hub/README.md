# OAuth — Hub Overlay

Configures LDAP authentication against FreeIPA (`idm.clamie.au`) on the hub cluster.

## Why this is not fully GitOps-managed

The LDAP bind password cannot be stored in Git and there is no external secrets manager configured. The `ldap-bind-password` Secret is generated at apply time from a local plaintext file (`ldap-bind-password.txt`) using a kustomize `secretGenerator`. This file is fetched from IDM manually before applying and must **never** be committed to Git.

## Prerequesites

The `bind-ocp` user must be created in FreeIPA. This user's password should be stored in a FreeIPA vault, the format of this vault content should be `bindPassword=PASSWORD`.

```bash
# Do not save echo command to bash history.
set +o history
echo -n 'bindPassword=PASSWORD' > /tmp/bindPassword
set -o history

# Create FreeIPA vault and archive content.
ipa vault-add --user bind-ocp --type standard bind-ocp-password
ipa vault-archive --user bind-ocp --in /tmp/bindPassword bind-ocp-password
```

## Applying

### 1. Authenticate to IDM

```bash
kinit admin@CLAMIE.AU
```

### 2. Retrieve the bind password from IDM Vault

```bash
ipa vault-retrieve bind-ocp-password \
  --user bind-ocp \
  --out infra/oauth/overlays/hub/ldap-bind-password.txt
```

***NOTE: The format of the password file should be `bindPassword=PASSWORD`.***

### 3. Apply the overlay

```bash
oc apply -k infra/oauth/overlays/hub/
```

### 4. Clean up

```bash
rm infra/oauth/overlays/hub/ldap-bind-password.txt
```

## Notes

- The `secretGenerator` uses `disableNameSuffixHash: true` so the Secret name remains stable and the OAuth CR reference does not need updating on each apply.
- If the bind password is rotated in IDM, repeat the steps above and re-apply.
