# vault-backup

Dump your Hashicorp Vault to a file. Not guaranteed to be consistent.
Dump is a form of commands to inject keys into vault, so it is convenient to
use it later on to restore to different vault, for example

```bash
vault kv put /secret/data/dev/ \
  test1_user='test_pass'
vault kv put /secret/data/dev/bash-org-244321 \
  AzureDiamond='hunter2'
```

# Environment variables

The following environment variables are used:
 - PYTHONIOENCODING is used to ensure your keys are exported in valid encoding, make sure to use the same during import/export
 - VAULT_ADDR - vault address enpoint to use, default is http://localhost:8200
 - VAULT_TOKEN - after succesful authorization, should be detected by script itself
 - VAULT_CACERT - cert if using https:// with client cert, but actually not tested
 - TOP_VAULT_META_PREFIX - path to dump, optional, useful when dumping only a fraction of the strucutre of the vault

# Known limitations

You still need vault client.
You must have right to list keys for TOP_VAULT_PREFIX, otherwise you' will get error with trace.

# Preparing python environment

Under Ubuntu you need below packages:

* python-pip
* python-virtualenv

```bash
virtualenv .venv
. .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

# Example usage
Activate virtualenv.

Export some vars:

```bash
export PYTHONIOENCODING=utf-8
export VAULT_ADDR=https://vault.tld:12345
export TOP_VAULT_META_PREFIX=/secret/metadata/dev/
```

Authorize in vault, I use here ldap auth:

```bash
vault auth -method=ldap username=your-username
```

After succesful authorization you can run dump script and then tou need to replace single quotes to double quotes
to address issue with escape caracter in windows style paths.
```bash
python vault-dump.py > vault.dump
sed -i "s/'/\"/g" vault.dump
```
Optionally encrypt the dump.
```bash
gpg vault.dump -e -r GPG_KEY_ID > vault.dump.gpg

```


# Importing to new vault

Warning, all corresponding keys will be overwritten.

Activate virtualenv.
Export some vars:

```bash
export PYTHONIOENCODING=utf-8
export VAULT_ADDR=https://vault.tld:9001
```

Authorize in vault:

```bash
vault auth -method=ldap username=user-in-new-vault
```

Disable bash history, decrypt encrypted file and execute commands stored inside:

```bash
set +o history
. <(gpg -qd vault.dump.gpg)
or
. vault.dump
```
