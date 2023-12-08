### Table of Contents
- **[README](../README.md)**
- **Ansible** (English/[Russian](../ru/ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** (English/[Russian](../ru/elastic-ru.md)): Search and data analysis.
- **Kubernetes** (English/[Russian](../ru/kubectl-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** (English/[Russian](../ru/regulars-ru.md)): A powerful tool for working with text.
- **Ubuntu** (English/[Russian](../ru/ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](vault-en.md)/Russian): Management of secrets and access.

# Install
1. Add PGP for the package signing key. 
    ```bash
    sudo apt update && sudo apt install gpg
    ```
2. Add the HashiCorp GPG key.
    ```bash
    wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg 
    ```
3. Verify the key's fingerprint. 
    ```bash
    gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint
    ```
4. Add the official HashiCorp Linux repository.
    ```bash
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    ```
5. Update and install.
    ```bash
    sudo apt update && sudo apt install vault
    ```
# Setup
1. Dev mode
    **NEVER** use Dev mode on production.
    ```bash
    vault server -dev
    ```
2. Set VAULT_ADDR by exporting to environment variable (bash command)
    ```bash
    export VAULT_ADDR='http://127.0.0.1:8200'
    ```    
3. Set Root Token by exporting to environment variable
    ```bash
    export VAULT_TOKEN="hvs.6j4cuewowBGit65rheNoceI7"
    ```
4. Check status
    ```bash
    vault status
    ```
# R/W/D Operations via Vault CLI

### READ SECRET 
```bash
vault kv get my/path 
```
### WRITE SECRET 
```bash
vault kv put my/path my-key-1=vaule-1 
```
### DELETE SECRET 
```bash
vault kv delete my/path
```
### READ the secrets in JSON format  
```bash
vault kv get -format=json my/path 
```
### Enable secret path in Hashi corp 
```bash
vault secrets enable -path=my kv/aws/...
```
### List all the secret engine available in vault
```bash
vault secrets list
```
### List all key values from secret engine
```bash
vault kv list my/
```
### Disable secret engine 
```bash
vault secrets disable my-custom-secret-engine-1
```
# Dynamic Secrets generation
1. Enable the secret engine path for AWS 
    ```bash
    vault secrets enable -path=aws aws
    ```
2. View the secret list
    ```bash
    vault secrets list
    ```
3. Write AWS root config inside your hashicorp vault
    ```bash
    vault write aws/config/root \
    access_key=YOUR_ACCESS_KEY \
    secret_key=YOUR_SECRET_KEY \
    region=eu-north-1
    ```
4. Setup role (bash command)
    ```bash
    vault write aws/roles/my-ec2-role \
        credential_type=iam_user \
        policy_document=-EOF
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1426528957000",
            "Effect": "Allow",
            "Action": [
                "ec2:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
    }
    EOF
    ```
5. Generate access key and secret key for that role
    ```bash
    vault read aws/creds/my-ec2-role
    ```
6. Revoke the secrets if you do not want it any longer
    ```bash
    vault lease revoke aws/creds/my-ec2-role/J8WHZJ5NItdH23KYYHdORv3K
    ```
# Policies
### List vault policies 
```bash
vault policy list
```
### Write your custom policy 
```bash
vault policy write my-policy - << EOF
path "secret/data/*" {
capabilities = ["create", "update"]
}

path "secret/data/foo" {
capabilities = ["read"]
}
EOF
```
### Read Vault policy details 
```bash
vault policy read my-policy
```
### Delete Vault policy by policy name 
```bash
vault policy delete my-policy
```
### Attach token to policy 
```bash
export VAULT_TOKEN="$(vault token create -field token -policy=my-policy)"
```
### Associate auth method with policy 
```bash
vault write auth/approle/role/my-role \
secret_id_ttl=10m \
token_num_uses=10 \
token_ttl=20m \
token_max_ttl=30m \
secret_id_num_uses=40 \
token_policies=my-policy
```
### Generate and Export Role ID
```bash
export ROLE_ID="$(vault read -field=role_id auth/approle/role/my-role/role-id)"
```
### Generate and Export Secret ID
```bash
export SECRET_ID="$(vault write -f -field=secret_id auth/approle/role/my-role/secret-id)"
```
# Token Auth
### Create token 
```bash
vault token create
```
### Vault login
```bash
vault login 
```
### Revoke token
```bash
vault token revoke YOUR_TOKEN_STRING
```
### List all authentication methods
```bash
vault auth list
```