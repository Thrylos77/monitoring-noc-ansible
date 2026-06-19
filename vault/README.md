# Vault

Les secrets du projet sont chiffrés avec ansible-vault.

## Fichier concerné
`vault/secrets.yml`

## Variables contenues
- `vault_grafana_admin_password`
- `vault_pg_exporter_password`
- `vault_wazuh_enrollment_key`

## Commandes utiles

# Créer le fichier la première fois
ansible-vault create vault/secrets.yml

# Editer
ansible-vault edit vault/secrets.yml

# Chiffrer un fichier existant
ansible-vault encrypt vault/secrets.yml

# Déchiffrer (temporairement)
ansible-vault decrypt vault/secrets.yml

## Mot de passe vault
Stocker le mot de passe dans `.vault_pass` à la racine du projet.
Ce fichier est dans .gitignore — ne jamais le commiter.
