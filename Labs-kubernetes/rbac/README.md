Kubernetes no gestiona ninguna base de datos de usuarios. Cualquier BBDD de usuarios debe ser externa, como por ejemplo un LDAP, htpasswd, Azure Active Directory...

Con la API de Kubernetes se pueden autenticar usuarios "normales" y ServiceAccounts. Los métodos de autenticación pueden ser:

* Certificados
* Token
* Basic auth (htpasswd)
* OpenID
...

TODO

Roles
RoleBindings
ClusterRoleBindings
Ver Roles
Certificados para autenticar

Más [info](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
