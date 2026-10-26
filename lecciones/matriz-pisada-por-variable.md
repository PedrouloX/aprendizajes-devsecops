# `parallel:matrix` ignorado: todos los jobs corrían el mismo proyecto

**Contexto:** el pipeline de certificación QA pasó a correr una vez por proyecto con `parallel:matrix` sobre `PROYECTO`.

**Síntoma:** todos los jobs de la matriz ejecutaban el mismo proyecto.

**Causa:** `PROYECTO` seguía definida como variable en Settings → CI/CD del repo (antes era obligatoria). Las variables del proyecto tienen más precedencia que las de la matriz.

**Solución:** borrar la variable del repo. La matriz es la única fuente.

**Para no repetirlo:** al pasar una variable a la matriz, revisar que no exista en el repo ni en el grupo, y anotarlo en el README del template ("esta variable NO debe existir en el repo").
