# Con un caso en rojo no se publicaba ningún resultado

**Síntoma:** el job de certificación terminaba en rojo y en la herramienta de gestión de pruebas no aparecía nada, ni los casos que pasaron.

**Causa:** `pytest` devolvía código distinto de 0 y el script del job (con `set -e` implícito) cortaba antes de los pasos que publican resultados y evidencia.

**Solución:** guardar el código de salida, publicar, y devolverlo al final:

```bash
pytest ... || PYTEST_RC=$?
python resultados_a_herramienta.py
python evidencia_pdf.py
python publicar.py
exit ${PYTEST_RC:-0}
```

El job sigue quedando en rojo, pero los resultados quedan publicados.

**Patrón general:** cualquier job que "prueba y reporta" debe reportar siempre, también cuando falla la prueba.
