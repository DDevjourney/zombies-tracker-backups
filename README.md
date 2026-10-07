# zombies-tracker-backups

Copia diaria de la tabla `sessions` de Supabase de [zombies-tracker](https://github.com/DDevjourney/zombies-tracker).

- `.github/workflows/backup.yml` se ejecuta cada día a las 04:30 UTC (o a mano desde Actions → Backup Supabase → Run workflow).
- Exporta la tabla a `backups/sessions.json` y hace commit solo si ha cambiado. El historial de git es el versionado: para ver los datos de un día concreto, mira ese commit.
- Si la exportación falla o viene vacía, el job falla (GitHub avisa por email) y el último backup bueno no se toca.

## Secrets necesarios

Settings → Secrets and variables → Actions:

| Secret | Valor |
|---|---|
| `SUPABASE_URL` | `https://xxxxxxxxxxxx.supabase.co` |
| `SUPABASE_SERVICE_ROLE_KEY` | clave `service_role` del proyecto |

## Restaurar

Con el JSON de la versión que quieras recuperar (`git show <commit>:backups/sessions.json > sessions.json`):

```js
import { createClient } from '@supabase/supabase-js'
import { readFileSync } from 'node:fs'

const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_SERVICE_ROLE_KEY)
const rows = JSON.parse(readFileSync('sessions.json', 'utf8'))
const { error } = await supabase.from('sessions').upsert(rows)
if (error) throw error
console.log(`Restauradas ${rows.length} partidas`)
```

`upsert` usa el `id`, así que recupera las filas borradas y revierte las editadas sin duplicar nada.

> La API REST devuelve como máximo 1000 filas por petición. Si la tabla supera ese tamaño, hay que paginar la exportación.
