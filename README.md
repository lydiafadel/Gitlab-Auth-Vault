# Gitlab-Auth-Vault

Step 0: Setup environment variables

      export VAULT_ADDR=https://vault-cluster-prod-public-vault-fa7b3917.bc6c60bc.z1.hashicorp.cloud:8200

      export VAULT_TOKEN=''

      export VAULT_NAMESPACE=admin

Step 1 : Setup oidc application in Gitlab 

      Get the application ID and secret ID (cf documentation's link below)


Step 2 : Activate the oidc authentication on the namespace desired through UI or API/CLI

      vault auth enable  oidc

Step 3 : Setup the oidc configuration

       vault write auth/oidc/config \
         oidc_discovery_url="https://gitlab.com" \
         oidc_client_id="<gitlab application ID>" \
         oidc_client_secret="<gitlab secret ID>" \
         default_role="demo" \
         bound_issuer="localhost"
  
 
The oidc_discovery_url matches your instance gitlab URL.
My configuration is tied up to the "demo" role that I am going to create in the next step

Step 4 : Define a role and attach to it a policy (default), this policy was already created in the namespace admin Vault

      vault write auth/oidc/role/demo -<<EOF
      {
       "user_claim": "sub", 
       "allowed_redirect_uris": ["http://localhost:8250/oidc/callback", "https://vault-cluster-prod-public-vault-               fa7b3917.bc6c60bc.z1.hashicorp.cloud:8200/ui/vault/auth/oidc/oidc/callback"], 
       "bound_audiences": "<gitlab application ID>",
       "oidc_scopes": ["openid"],
       "role_type": "oidc", 
       "policies": ["default"],
       "ttl": "1h",
       "bound_claims": {"groups": ["vault20/developers"]}
      }       
   EOF

- allowed_redirect_uris" : The first URL allows you to connect to Vault locally in your machine, the second URL allows you to connect through the Web UI.
- Keep in mind that the TTL you defined affects the duration of the authentication token.

- Bound claims is tied up to gitlab groups/project & subgroups. Pay attention to the name of the group, you can find it in the advanced parameters of gitlab or by making an API Call using tool like Postman.


You can now connect to the Vault locally in your shell : 
      
      vault login -method=oidc port=8250 role=demo
   
Other option : when you connect to Vault locally, a token is created and this token can be used to connect to Vault through the UI without using the role of the OIDC gitlab provider.

Gitlab documentation :
   https://docs.gitlab.com/ee/integration/vault.html
