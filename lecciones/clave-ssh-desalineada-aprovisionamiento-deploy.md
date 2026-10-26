# La clave SSH del aprovisionamiento no era la del deploy

**Síntoma:** en los primeros deploys sobre servidores recién aprovisionados por pipeline, el job fallaba al entrar por SSH con el usuario de deploy. Además faltaba la carpeta de apps con el dueño correcto.

**Causa:** el pipeline de configuración escribía en `authorized_keys` una clave pública **en duro en el script**, distinta de la que correspondía a la `SSH_PRIVATE_KEY` que usan los templates de deploy.

**Parche (para destrabar):** dos acciones en el pipeline de lifecycle: una agregaba la clave correcta y otra creaba `/opt/apps/` con el dueño correcto.

**Arreglo de raíz (después):**

- La clave pública pasa como variable CI/CD al script de bootstrap, que la escribe al crear el usuario. Misma variable de grupo que usa el deploy.
- La creación de `/opt/apps/logs` (dueño el usuario de deploy) pasa a ser parte del bootstrap normal (se omite en servidores solo de BDD).
- Retiradas las acciones parche y la clave en duro.

**Lecciones:**

- Nada de claves ni credenciales en duro en scripts: una sola fuente (variable) para lo que tiene que coincidir.
- Un parche está bien para destrabar, pero dejar anotado el arreglo de raíz como pendiente y retirarlo cuando se haga.
- Los servidores parchados a mano ya quedaron bien; verificar si hace falta reaprovisionarlos antes de hacerlo.
