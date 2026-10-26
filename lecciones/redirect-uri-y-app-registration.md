# Login con Azure: redirect URI incompleto o en el registro equivocado

**Síntoma 1:** `AADSTS50011: The redirect URI ... does not match`.

**Causa:** la solicitud pedía solo el dominio (`https://app.ejemplo.cl`) y la app usa `https://app.ejemplo.cl/auth/callback`. El admin cargó exactamente lo que se pidió.

**Síntoma 2:** después de "aplicado", el error seguía mencionando el Client ID de otro registro.

**Causa:** había dos App Registrations (uno para ambientes no productivos y otro de producción) y la URI se cargó en el que no correspondía.

**Para no repetirlo:** en cada solicitud indicar:

- La URI **completa** con su ruta de callback.
- El **nombre** del App Registration y su Client ID (y en qué ambiente lo usa la app: el `AZURE_CLIENT_ID` del `ENV`).
- La plataforma ("Aplicación de página única" / SPA vs Web).

Validar con un login real después de aplicado.
