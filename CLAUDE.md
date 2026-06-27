# CLAUDE.md — hosting.xplaya.com

Landing de servicios técnicos: hosting, desarrollo, IA, asesoría para negocios pequeños.
Desplegada en **https://hosting.xplaya.com**

Stack: HTML estático · CSS propio · nginx:alpine · ARM64 k3s vía ArgoCD.
Sin frameworks JS, sin DB, sin backend.

## Estructura

```
hosting/
├── index.html          # one-pager principal
├── css/styles.css      # estilos (no usa Bulma — CSS propio)
├── img/
│   ├── portfolio/      # webps optimizadas (900×837 desktop, 380×1035 mobile)
│   └── og-hosting.jpg  # imagen Open Graph 1200×630
├── favicon/            # copiado de xplaya/static/img/favicon/
├── robots.txt
├── sitemap.xml
├── nginx.conf
├── Containerfile       # nginx:alpine, copia archivos estáticos
├── .dockerignore       # excluye hosting/, src/, .git, CLAUDE.md, etc.
├── .gitignore          # excluye src/img/*.png (PNGs fuente, pesados)
├── .github/workflows/
│   └── build.yaml      # push a ghcr.io/rogithub/hosting:latest — ARM64
├── hosting/            # capturas fuente originales (NO modificar)
└── src/img/            # copias de trabajo de PNGs (gitignoreadas)
    └── portfolio/      # vacío — webps ya movidas a img/portfolio/
```

## Imágenes

Las 18 capturas originales (3288×1152 PNG) están en `hosting/` sin tocar.
Las copias de trabajo están en `src/img/` (gitignoreadas).
Las webps procesadas están en `img/portfolio/` — son las que usa el sitio.

Pipeline de procesado (ya ejecutado, documentado por si se rehace):
```bash
# Desktop: recorta OS bar (y=41) + wallpaper (x>914), resize 900px, q82 webp
convert input.png -crop 914x850+0+41 +repage -resize 900x -strip -define webp:method=6 -quality 82 output.webp

# Mobile (kiosko1, mobil1): panel estrecho del teléfono
convert input.png -crop 312x850+0+41 +repage -resize 380x -strip -define webp:method=6 -quality 82 output.webp
```

## Analytics

La página tiene el script de Umami pero con `data-website-id="REEMPLAZAR-CON-ID-DE-HOSTING"`.
Pasos para activar:
1. Entrar a analytics.xplaya.com → Configuración → Añadir sitio web → `hosting.xplaya.com`
2. Copiar el website-id que genere
3. Reemplazar en `index.html` la cadena `REEMPLAZAR-CON-ID-DE-HOSTING`

## Despliegue

1. Crear repo GitHub: `rogithub/hosting`
2. Push a `main` → GitHub Actions compila imagen ARM64 → la sube a `ghcr.io/rogithub/hosting:latest`
3. En `k3s-manifests/`: ya existen `apps/hosting-app.yaml` + `workloads/hosting/` → ArgoCD sincroniza
4. Marcar el paquete GHCR `hosting` como **público** (o crear sealed secret para pull credentials)
5. Añadir ruta en Cloudflare Tunnel: `hosting.xplaya.com` → `http://hosting-svc.hosting.svc.cluster.local:80`

## Contacto en la página

- WhatsApp: `wa.me/524522018336`
- Correo: `contacto@xplaya.com`
- Mismo número que papelería xplaya — no cambiar sin actualizar ambos sitios

## Repos relacionados

| Repo | Ruta | Rol |
|------|------|-----|
| xplaya | `/mnt/storage/data/code/xplaya` | Identidad visual de referencia (colores, favicon, FAB WhatsApp) |
| k3s-manifests | `/mnt/storage/data/code/k3s-manifests` | Despliegue GitOps |
| pdfeitor | `/mnt/storage/data/code/pdfeitor` | Fuente del kiosko (screenshot kiosko1) |
| inventario_papeleria | `/mnt/storage/data/code/inventario_papeleria` | Fuente del POS (screenshots pos*, fidelidad*, etc.) |
