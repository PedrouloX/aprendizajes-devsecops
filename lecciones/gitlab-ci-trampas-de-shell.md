# Trampas de shell en jobs de GitLab CI

| Síntoma | Causa | Solución |
|---|---|---|
| `grep: unrecognized option -P` | Imágenes Alpine usan BusyBox, sin `-P` | `grep -E` en scripts que corren en Alpine |
| `bad substitution` con `${!var}` | Expansión indirecta es de bash; Alpine usa `sh` | `$(eval echo "\$$var")` |
| `ssh-add` falla con la clave | La variable es tipo **File**: contiene la ruta, no el contenido | `ssh-add "$SSH_PRIVATE_KEY"`, no `echo "$SSH_PRIVATE_KEY" \| ssh-add -` |
| GitLab no parsea el `.gitlab-ci.yml` | Un heredoc dentro de `script:` rompe el YAML | SSH inline con comillas dobles, o mover la lógica a un script `.sh` |
| `sudo docker` pide password | El chequeo usaba `sudo` sin necesidad | El usuario de deploy está en el grupo `docker`; no usar `sudo` para Docker |
| `sudo: a password is required` en otra operación | Sudoers acotado no incluye esa ruta | Agregar la ruta puntual al sudoers, no sudo genérico |
| Error de un script remoto no aparece en el log | `set -e` corta antes de imprimir | Capturar la salida y mostrarla antes de salir: `out=$(cmd 2>&1) \|\| { echo "$out"; exit 1; }` |
