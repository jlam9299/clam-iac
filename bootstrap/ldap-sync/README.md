# LDAP Group Sync (Manual)

Use this when you need to force a sync immediately rather than waiting for the next scheduled run.

## Prerequesites

- There is a `ocp-cluster-admin` group in FreeIPA.
- `bind-ocp-password` FreeIPA vault exists and is populated.

## Manual procedure

### 1. Retrieve bind user password

```bash
ipa vault-retrieve bind-ocp-password \
  --user bind-ocp \
  --out /tmp/bind-ocp-password
```

### 2. Dry run (no changes applied)

To preview what the sync would do without making any changes, run `oc adm groups sync` directly without `--confirm`:

```bash
oc adm groups sync \
  --sync-config=bootstrap/ldap-sync/ldap-sync-config.yaml \
  --confirm=false
```

### 3. Confirm the discovered groups and users from previous command

Example output:

```yaml
apiVersion: v1
items:
- apiVersion: user.openshift.io/v1
  kind: Group
  metadata:
    annotations:
      ...
    labels:
      ...
    name: ocp-cluster-admin
  users: []
kind: List
metadata: {}
```

### 4. Run LDAP sync (changes applied)

```bash
oc adm groups sync \
  --sync-config=bootstrap/ldap-sync/ldap-sync-config.yaml \
  --confirm=true
```

### 5. Clean up password file

```bash
rm /tmp/bind-ocp-password
```

### 6. Apply GroupRoleBinding

```bash
oc apply -f bootstrap/ldap-sync/cluster-role-binding.yaml
```