## 1. Compare authentication methods

### Exam Objectives
- 1a. Describe authentication methods
- 1b. Choose an authentication method based on use case
- 1c.	Differentiate human vs. system auth methods

### Notes

#### Vault Authentication
Before a client can interact with Vault, it must authenticate against an auth method. 

#### What happens during authentication
Upon authentication, a token is generated. This token is conceptually similar to a session ID on a website. The token may have attached policy, which is mapped at authentication time. 

#### auth methods
- Vault supports a number of auth methods. Some backends are targeted toward users while others are targeted toward machines. 
- Most authentication backends must be enabled before use. 
- Vault supports multiple auth methods simultaneously, and you can even mount the same type of auth method at different paths. 
- Only one authentication is required to gain access to Vault.

To enable an auth method:
```bash
$ vault write sys/auth/my-auth type=userpass
```

- `my-auth` is user-defined.
- `userpass` is the auth method

The above command is also equivalent below:
```bash
$ vault auth enable userpass --path=my-path
```

By default, auth methods are mounted to `auth/<type>`.

To disable an auth method:
```bash
$ vault auth disable userpass
```

- All tokens generated by logins using this authentication method are revoked.

#### Token authentication

- Token authentication is automatically enabled. 
- Tokens are the core method for authentication within Vault. 
- Tokens can be used directly or auth methods can be used to dynamically generate tokens based on external identities.
- `vault server -dev` (or `vault operator init` for a non-dev server) outputs an initial "root token." This is the first method of authentication for Vault. 
- It is also the only auth method that cannot be disabled.

#### Display list of auth methods

```bash
$ vault auth list
```
Display all the authentication methods that is enabled.

#### AppRole Auth Method
- The `approle` auth method allows machines or apps to authenticate with Vault-defined roles. 
- This auth method is oriented to automated workflows (machines and services), and is less useful for human operators.

##### AppRole Usage
1. Enable the approle auth method
```bash
$ vault auth enable approle
```
2. Create a named role
```bash
$ vault write auth/approle/role/my-role \
    secret_id_ttl=10m \
    token_num_uses=10 \
    token_ttl=20m \
    token_max_ttl=30m \
    secret_id_num_uses=40
```
3. Fetch the RoleID of the AppRole
```bash
$ vault read auth/approle/role/my-role/role-id
Key        Value
---        -----
role_id    016a08ab-5a46-bdf1-aeee-f60c6a31906e
```
4. Get a SecretID issued against the AppRole:
```bash
$ vault write -f auth/approle/role/my-role/secret-id
Key                   Value
---                   -----
secret_id             290ca7e4-d51c-5366-d47b-6d2400ff9511
secret_id_accessor    219e8f4c-9023-c8ee-4531-02a3412c522e
```
5. Use the `role_id` and `secret_id` to login
```bash
$ vault write auth/approle/login \
    role_id=016a08ab-5a46-bdf1-aeee-f60c6a31906e \
    secret_id=290ca7e4-d51c-5366-d47b-6d2400ff9511
```

#### RoleID

RoleID is an identifier that selects the AppRole against which the other credentials are evaluated.

#### SecretID

SecretID is a credential that is required by default for any login (via `secret_id`) and is intended to always be secret.

#### Pull And Push SecretID Modes

If the SecretID used for login is fetched from an AppRole, this is operating in **Pull** mode. If a "custom" SecretID is set against an AppRole by the client, it is referred to as a **Push** mode. 

#### AppRole Further Constraints

- `role_id` is a required credential at the login endpoint. AppRole pointed to by the role_id will have constraints set on it. This dictates other required credentials for login. 
- `bind_secret_id` constraint requires `secret_id` to be presented at the login endpoint.
- `secret_id_bound_cidrs` will only allow logins coming from IP addresses belonging to configured CIDR blocks on the AppRole.

### Resources
- [Authentication Concept](https://www.vaultproject.io/docs/concepts/auth)
- [Auth methods](https://www.vaultproject.io/docs/concepts/auth)
- [AppRole Tutorial](https://learn.hashicorp.com/tutorials/vault/approle)
