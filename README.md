# aur-orchard

Paquete AUR `orchard-git` de [Orchard](https://github.com/SFG5453/Orchard), el cliente de
YouTube Music para usuarios avanzados. La fuente es el repo oficial del autor: cada
`paru -S orchard-git` compila el último commit de `main`.

## Instalación

```sh
paru -S orchard-git
```

Depende de `electron` (repo extra). Instalado en `/opt/orchard`, lanzador `/usr/bin/orchard`.

## Diseño del paquete

- `pkgver()` deriva de `git describe` sobre el repo oficial, así que el paquete siempre
  refleja el HEAD de `main`.
- Build: `npm ci` → `npm run build` (addons Rust napi + renderer Vite) → `npm prune --omit=dev`.
- `options=('!lto')`: con el `lto` de `makepkg.conf` la lib C++ de `signalsmith-stretch`
  queda como bitcode y el `.node` falla con `undefined symbol` al cargar.
- Se recortan los binarios de ONNX Runtime que no son `linux/x64` (ahorra ~240 MB).

## Mantenimiento automático

| Workflow | Frecuencia | Qué hace |
| --- | --- | --- |
| `AUR sync` | diario | Recalcula `pkgver` desde upstream, regenera `.SRCINFO` y hace push a este repo y a AUR |
| `Canary build` | semanal | Compila `main` de upstream en un contenedor Arch para detectar roturas del build |

### Secret requerido

`AUR_SSH_KEY`: clave privada ed25519 autorizada en la cuenta AUR (Apartado SSH Keys de
`aur.archlinux.org/account/edit/`). Generar con:

```sh
ssh-keygen -t ed25519 -f aur_deploy -C aur-orchard-ci
gh secret set AUR_SSH_KEY < aur_deploy
```

## Estructura

```
PKGBUILD            Fuente del paquete
.SRCINFO            Generado con makepkg --printsrcinfo
orchard.desktop     Entrada de menú (WM_CLASS dev.sfg.orchard)
orchard.sh          Lanzador: exec /usr/bin/electron /opt/orchard
```
