# OpenLDAP and Keycloak

Docker Compose deploys both services on the `openldap-keycloak` Docker network.

## Start with Docker Compose

From this directory, copy the environment template and replace both passwords with strong, different values:

```sh
cp .env.example .env
chmod 600 .env
```

Then start the containers:

```sh
docker compose up -d
```

The `.env` file is ignored by Git. OpenLDAP stores its data in `./openldap`; Keycloak uses the `keycloak_data` Docker volume.

To stop the services without deleting their data:

```sh
docker compose down
```

## Access

- Keycloak: `http://<server-address>:8080` — username is set by `KEYCLOAK_ADMIN_USERNAME`
- OpenLDAP: ports `389` (LDAP) and `636` (LDAPS)
- Default administrator DN: `cn=admin,dc=example,dc=org`

The LDAP domain and organization can be changed in `.env`. From Keycloak, OpenLDAP is reachable at hostname `openldap` on the `openldap-keycloak` Docker network. By default, published ports are bound to `127.0.0.1`. To allow access from the local network, set `BIND_ADDRESS=0.0.0.0` in `.env` and restrict access with the host firewall.

Keycloak starts with `start-dev`, which is intended for testing only. LDAP federation is not configured automatically. For production, configure TLS, an appropriate DNS name and certificate, and an external database for Keycloak.

## Deploy to a host with Ansible

The target host must have Docker Engine, the Docker Compose v2 plugin, and the Ansible `community.docker` collection. The local `.env` file is copied to `/opt/openldap-keycloak/.env` on the host with `0600` permissions; do not commit it to version control.

Run this command from the repository root:

```sh
set -a
. services/.env
set +a
ansible-playbook -i services/inventory/inventory.yml services/openldap-keycloak/deploy.yml
```

The playbook copies the Compose file and `.env` to the target, then starts the services. The `install.yml` playbook is also available if you prefer to create the containers directly with Ansible.
