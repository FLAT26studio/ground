# Design Tokens para la app de Ground

Este repositorio contiene los design tokens exportados desde Figma vía Supernova, listos para transformarse en código consumible por la app Flutter.

## 📦 Estructura del repo

- ground/
   - base/ # Tokens base (colores, spacing, tipografía, radios, etc.)
   - light/ # Tokens de color para el tema claro (en progreso)

## 🔄 Flujo del pipeline

1. **Figma** → los diseñadores definen/actualizan variables y estilos.
2. **Supernova** → sincroniza esas variables y las exporta como tokens (JSON) a este repo.
3. **Style Dictionary** → transforma esos JSON en código nativo (en este caso, Dart/Flutter).
4. **Flutter app** → consume los tokens generados como constantes/clases Dart.

## ⚙️ Sugerencia de configuración de Style Dictionary (Gernerado por Claude)

### 1. Instalar dependencias

En la raíz del proyecto Flutter (o en un paquete/módulo dedicado a design tokens):

```bash
npm install --save-dev style-dictionary
```

### 2. Crear el archivo de configuración

Crea `style-dictionary.config.js` en la raíz del proyecto:

```js
const StyleDictionary = require('style-dictionary');

module.exports = {
  source: [
    'ground/base/**/*.json',
    'ground/light/**/*.json'
  ],
  platforms: {
    flutter: {
      transformGroup: 'flutter',
      buildPath: 'lib/design_tokens/',
      files: [
        {
          destination: 'app_colors.dart',
          format: 'flutter/class.dart',
          filter: {
            attributes: { category: 'color' }
          },
          options: {
            className: 'AppColors'
          }
        },
        {
          destination: 'app_spacing.dart',
          format: 'flutter/class.dart',
          filter: {
            attributes: { category: 'spacing' }
          },
          options: {
            className: 'AppSpacing'
          }
        },
        {
          destination: 'app_typography.dart',
          format: 'flutter/class.dart',
          filter: {
            attributes: { category: 'typography' }
          },
          options: {
            className: 'AppTypography'
          }
        }
      ]
    }
  }
};
```

> ⚠️ Style Dictionary no trae el transform group `flutter` ni el formato `flutter/class.dart` por defecto. Se recomienda usar el paquete [`style-dictionary-flutter`](https://www.npmjs.com/search?q=style-dictionary-flutter) o registrar transforms/formats custom en `style-dictionary.config.js` con `StyleDictionary.registerTransform()` / `registerFormat()`. Confirmar con el dev qué approach se usa antes de correr el build.

### 3. Ajustar el nombre de las categorías

Revisa que los tokens exportados por Supernova usen los mismos nombres de `category`/`type` que se filtran en el config (`color`, `spacing`, `typography`, etc.). Si Supernova exporta otra convención (por ejemplo `$type` en vez de `category`), hay que ajustar los `filter` del config para que coincidan.

### 4. Generar el código

```bash
npx style-dictionary build --config style-dictionary.config.js
```

Esto genera los archivos Dart dentro de `lib/design_tokens/`, listos para importar en la app:

```dart
import 'package:tu_app/design_tokens/app_colors.dart';
import 'package:tu_app/design_tokens/app_spacing.dart';

Container(
  color: AppColors.primary,
  padding: EdgeInsets.all(AppSpacing.md),
)
```

## 🔁 Automatización (recomendado)

Para que los tokens se regeneren automáticamente cada vez que Supernova haga push a este repo, se puede configurar un GitHub Action:

```yaml
name: Build Design Tokens

on:
  push:
    branches: [main]
    paths:
      - 'base/**'
      - 'light/**'

jobs:
  build-tokens:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm install
      - run: npx style-dictionary build --config style-dictionary.config.js
      - run: |
          git config user.name "tokens-bot"
          git config user.email "tokens-bot@flat26.studio"
          git add lib/design_tokens/
          git commit -m "chore: update design tokens" || echo "No changes"
          git push
```

## ✅ Checklist para el developer

- [ ] Instalar `style-dictionary` (y el paquete de soporte para Flutter que se elija)
- [ ] Validar que las categorías de los tokens (`color`, `spacing`, `typography`...) coincidan con los filtros del config
- [ ] Correr el build local y verificar que los archivos `.dart` se generen sin errores
- [ ] Importar los tokens generados en el design system de la app (evitar hardcodear valores de color/spacing en widgets)
- [ ] (Opcional) Configurar el GitHub Action para automatizar la regeneración en cada push de Supernova
