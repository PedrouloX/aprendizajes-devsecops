# Reintentar un job no toma el arreglo recién commiteado

**Síntoma:** se corrigió el script, se hizo commit, se apretó "Retry" en el job fallido y falló igual.

**Causa:** "Retry" vuelve a ejecutar el job con el **commit y las variables del pipeline original**.

**Solución:** lanzar un **pipeline nuevo** (Run pipeline) en la rama.

**Aplica también** a cambios de variables del formulario: un retry conserva los valores con que se lanzó.
