# `vendor/three/` — рантайм three.js

**Единственная копия three.js на весь репозиторий.** Сцены, игры и студия импортируют
отсюда. Второй копии быть не должно: расхождение версий между сценами — это баги,
которые ищут в своём коде.

## Версия — r186, и это не случайно

`REVISION = '186'` в `three.core.js`. Ровно та же версия, что у RAG-корпуса:
`tools/fetch_threejs_docs.py` пиннится на тег `r186` (см. `--ref`), поэтому
`knowledge/official_docs` и `knowledge/official_source` описывают **тот же** API, который
исполняется. Если поднять рантайм до другой ревизии — документация в базе начнёт врать:
модель будет писать код по докам одной версии, а падать на другой.

Поэтому версия меняется **только вместе** с корпусом:

```bash
python tools/fetch_threejs_docs.py --ref r187
# затем — те же файлы из того же тега в vendor/three/
```

## Что лежит

| файл | зачем |
|---|---|
| `three.module.js` | точка входа; реэкспортирует ядро |
| `three.core.js` | само ядро (1.4 МБ) — **обязателен**, без него `three.module.js` не загрузится |
| `addons/controls/OrbitControls.js` | орбитальная камера мышью |
| `addons/loaders/GLTFLoader.js` | загрузка `.glb` / `.gltf` |
| `addons/utils/BufferGeometryUtils.js` | операции над геометрией; нужен `GLTFLoader` |
| `addons/utils/SkeletonUtils.js` | клонирование скинованных моделей; нужен `GLTFLoader` |

Файлы взяты из официального репозитория (`raw.githubusercontent.com/mrdoob/three.js/r186`),
не из npm и не с CDN — сборка воспроизводима и не зависит от чужого хостинга.

## Как подключить — importmap обязателен

Аддоны импортируют `from 'three'` — это **голый спецификатор**, браузер его не разрешает
без карты импортов. Поэтому в HTML **до** основного модуля:

```html
<script type="importmap">
{
  "imports": {
    "three": "/vendor/three/three.module.js",
    "three/addons/": "/vendor/three/addons/"
  }
}
</script>

<script type="module">
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
</script>
```

Пути в `imports` — от **корня сервера**, а не от файла страницы. Если страница открыта
не с корня репозитория, а из подпапки, используйте относительные пути
(`../../vendor/three/three.module.js`) или поднимайте сервер от корня.

## Как посмотреть сцену

Нужен HTTP-сервер: `file://` не работает — модули и importmap требуют origin.

```cmd
cd /d F:\Pets\Projects\local_tools\3GS_Master && .venv\Scripts\python.exe -m http.server 8000
```

Дальше — `http://127.0.0.1:8000/scenes/demo/`.

## Чего здесь нет и почему

Здесь **только библиотека**. Ни `scene.js`, ни `levels.json`, ни `engine.js`, ни
`newgame` — это не часть рантайма, а надстройка под конкретную игру. Если задача —
просто 3D-сцена, файлы сцены пишутся с нуля под задачу; ничего копировать из чужих
папок не нужно.
