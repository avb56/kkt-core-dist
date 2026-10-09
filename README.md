# kkt-core

Драйверы онлайн-касс — **АТОЛ** (протокол v3, без драйвера ДТО) и **Вики Принт** —
и эмулятор ККТ, с одним интерфейсом: **JSON-задания АТОЛа** (`sell`,
`openShift`, `getDeviceStatus`, …), ответы — в том же формате. Тот же код
работает в браузере (Web Serial, WebUSB) и в Node.

Код обфусцирован. Файл подключается **как есть**: его нельзя пропускать через
сборщик или минификатор — после любой переделки он перестаёт запускаться.
Обфускация затрудняет чтение и правку, но не защищает от целенаправленного
разбора.

| файл | для чего |
| --- | --- |
| `kkt-core.mjs` | браузер, ES-модуль (es2020; Chrome 109 и новее) |
| `kkt-core.cjs` | Node 12 и новее, CommonJS |
| `SHA256SUMS` | контрольные суммы |

## Подключение

Браузер — файл рядом со страницей, модулем:

```js
const { KKT, configure } = await import('/vendor/kkt-core.mjs');
```

Node — npm-архивом выпуска (`npm i https://github.com/avb56/kkt-core-dist/releases/download/vX.Y.Z/kkt-core-X.Y.Z.tgz`):

```js
const { KKT, configure } = require('kkt-core');
```

Собираете своё приложение сборщиком — объявите `kkt-core` внешним
(esbuild `external`, rollup `external`), чтобы файл не пересобирался.

## Настройки

`configure({...})` — один раз, до работы с ККТ. Всё необязательно.

| ключ | по умолчанию | что это |
| --- | --- | --- |
| `log` | молчит | `{ info, warn, error }(текст, значение)` — журнал |
| `fetch` | `globalThis.fetch` | для отправки на серверы (в Node 12 — свой) |
| `sOfdProxyUrl` | — | прокси ОФД для браузера: POST, тело и ответ — base64, адрес ОФД — в `X-Attr-OFD-Host` |
| `pSendToOfd` | через `sOfdProxyUrl` | `(хост:порт, байты) → Promise<байты>` — свой путь в ОФД (в Node — прямой TCP) |
| `pConfirm` | «нет» | `(текст) → Promise<boolean>` — прервать долгое ожидание результата сервера? |
| `fOnResults` | — | `(результаты, ккт)` — после каждого удачного задания |
| `sTelePortUrl` | — | WebSocket «телепорта»: порт ККТ на другой машине |

## ККТ

```js
// Web Serial (выбор порта браузер показывает по нажатию пользователя)
const oKKT = new KKT({ sConnectionType: 'comPort', sVendorOrUrl: 'auto', id: 'РНМ' });
// Web Requests АТОЛа или совместимый сервер; ?deviceID — какая ККТ сервера
const oSrv = new KKT({ sConnectionType: 'atolSrv', sVendorOrUrl: 'http://логин:пароль@127.0.0.1:16732/api/v2/?deviceID=1', id: 'РНМ' });
// эмулятор
const oEmu = new KKT({ sConnectionType: 'emulate', sVendorOrUrl: 'auto', id: '0000000000000000' });

const [oStatus] = await oKKT.pTaskSender({ type: 'getDeviceStatus' });
```

Задания к одной ККТ на своём порту выполняются по очереди. Ошибка задания —
исключение с текстом для человека («ККТ …: Сервер не отвечает на адресе …»).

Типы подключения: `comPort` (Web Serial), `usbPort` (WebUSB), `emulate`,
`telePort`, `atolSrv` (Web Requests / агент kktsite), `kkmSrv` (KKM Server).

## Лицензия

См. `LICENSE`. Не MIT: использование — на условиях правообладателя.
