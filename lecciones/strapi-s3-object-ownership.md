# Strapi 5 + S3 con Object Ownership "enforced"

**Contexto:** subidas desde el panel de Strapi a S3 con `@strapi/provider-upload-aws-s3` v5.

**Punto clave:** el bucket tenía Object Ownership en "Bucket owner enforced" (ACL deshabilitadas), así que el provider no debe enviar ACL.

**Solución en `config/plugins.ts`:**

```ts
upload: {
  config: {
    provider: 'aws-s3',
    providerOptions: {
      rootPath: 'proyecto/',
      s3Options: { region: 'us-east-1', params: { Bucket: '<bucket>' } },
    },
    actionOptions: {
      upload: { ACL: null },
      uploadStream: { ACL: null },
    },
  },
},
```

Y agregar el dominio del bucket al CSP en `config/middlewares.ts` (`img-src`, `media-src`) para que el panel muestre las imágenes.

**Otros:**

- El build del panel admin de Strapi necesitó `NODE_OPTIONS=--max-old-space-size=4096`.
- Política IAM del usuario: solo `s3:GetObject`/`PutObject`/`DeleteObject` sobre el bucket (y `ListBucket` sobre el bucket mismo), nada más.
