# South Admin — Backoffice interno

Panel de administración interno para monitorear South Sales Intelligence.

## Estructura

```
south-admin/
├── index.html      → Login con validación contra Edge Function
├── dashboard.html  → Panel completo de métricas
├── vercel.json     → Config de rutas + headers de seguridad
└── README.md
```

## Deploy en Vercel (proyecto separado)

### 1. Instalá Vercel CLI (si no lo tenés)
```bash
npm install -g vercel
```

### 2. Deploy
```bash
cd south-admin
vercel deploy --prod
```

Cuando te pregunte:
- **Project name**: `south-admin` (o lo que quieras)
- **Framework**: Other (ninguno)
- **Root**: `./`

La URL quedará como `south-admin.vercel.app` o podés configurar un dominio custom como `admin.southsales.io`.

### 3. Verificar que el EDGE_URL es correcto

En `index.html` y `dashboard.html` hay una constante:
```js
const EDGE_URL = 'https://iquzgpkrrvtsnvaynyqj.supabase.co/functions/v1/admin-metrics';
```

Asegurate de que la Edge Function `admin-metrics` esté deployada en Supabase con el `ADMIN_SECRET` configurado.

## Flujo de autenticación

1. Usuario entra a `/` → ve el login
2. Ingresa el password → se valida contra la Edge Function con `x-secret-key` header
3. Si la Edge Function responde 200 → se guarda en `sessionStorage` y redirige a `/dashboard`
4. Si responde 401 → error de contraseña incorrecta
5. Al cerrar el tab → sesión expirada automáticamente (sessionStorage)

## Seguridad

- `vercel.json` incluye `X-Robots-Tag: noindex` para que no aparezca en Google
- `X-Frame-Options: DENY` para prevenir clickjacking
- Sesión en `sessionStorage` (no persiste entre tabs/sesiones)
- La contraseña nunca queda en el código — siempre viene del `ADMIN_SECRET` de la Edge Function

## Personalizar el límite de tokens

En la Edge Function `admin-metrics`, cambiá:
```ts
limit: 10_000_000 // tokens por mes
```

La alerta se activa automáticamente al 70% (amarilla) y 90% (roja).