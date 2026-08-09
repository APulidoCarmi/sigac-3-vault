<!--
  Generado por mi-ide-claude. No editar aquí: la fuente de verdad es
  template/Reglas/ en el repositorio del cascarón, y esta nota se
  re-siembra (sobreescribe) en cada arranque del servidor.
-->

# Regla: el contrato entre capas se fija en el plan

Cuando un requerimiento cruza capas —el front consume una API que el back
construye en el mismo plan, dos servicios se hablan, un worker lee lo que
otro escribe—, **la forma de esa frontera se decide en `/plan`, antes de
implementar nada**, y queda escrita en el plan.

## Por qué existe esta regla

`/implementa-paraguas` paraleliza hijos que no comparten archivos. Ese
chequeo es correcto para evitar conflictos de edición, pero **no detecta
divergencias de contrato**: el hijo del front y el hijo del back pueden no
compartir ni una sola línea de código y aun así construir mitades que no
encajan. Uno espera `{ user: { id } }` y el otro devuelve `{ userId }`.

El fallo aparece tarde —al integrar, con las dos mitades ya escritas— y su
arreglo no es un ajuste: es rehacer una de las dos. Es el modo de fallo
clásico de repartir trabajo entre contextos aislados, y el único antídoto
barato es fijar la frontera antes de repartir.

## Qué cuenta como contrato

Todo lo que una pieza asume de otra sin poder verlo en su propio código:

- **Endpoints HTTP**: ruta, método, forma exacta del request y del response,
  códigos de error y qué significa cada uno.
- **Firmas públicas**: funciones, módulos o clases que otra capa importa.
- **Formas persistidas**: schema de base de datos, forma de un documento o
  de una entrada de caché.
- **Eventos y mensajes**: nombre del evento y payload.
- **Claves de configuración** y variables de entorno nuevas.

## Cómo se fija

- **Valores concretos, no descripciones.** "Devuelve el usuario" no es un
  contrato; el JSON exacto con sus campos y tipos sí lo es.
- **Prefiere un artefacto ejecutable** cuando el proyecto ya tenga uno:
  tipos compartidos, OpenAPI, el `schema.prisma`. El plan enlaza al
  artefacto en vez de duplicarlo, y así el compilador vigila por ti.
  (Si el contrato toca Prisma, la migración se genera con
  `npx prisma migrate dev --name <descripcion>`, nunca a mano.)
- **Vive en la sección `## Contrato`** del plan. Si es una iniciativa con
  sub-planes, vive en el **paraguas**, no repetido en cada hijo, y el
  orquestador se lo pasa a cada subagente al lanzarlo.

## Regla dura: de solo lectura para quien implementa

Quien implementa **no cambia el contrato**. Si al escribir código descubre
que la forma acordada no sirve —falta un campo, el tipo era otro, el error
no se puede representar—, se detiene y lo reporta. No lo "ajusta un poco"
por su cuenta, aunque el cambio parezca trivial y aunque su parte quede
mejor: al otro lado hay otra pieza escrita contra la forma vieja, y muy
probablemente en otra sesión que ya terminó.

Un contrato que hay que cambiar es un **replaneo** (ver
[[Planes paraguas y replaneo]]): se corrige en el plan, con su porqué y su
fecha, se valida con el humano, y solo entonces siguen los pasos que
dependían de él.

## Cuándo NO aplica

Si el requerimiento vive entero en una capa, no inventes una sección
`## Contrato` vacía. La regla es para fronteras reales entre piezas que se
implementan por separado, no un trámite de todo plan.
