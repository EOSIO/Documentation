## Дорожная карта программного обеспечения EOS.IO

This document outlines the development plan from a high level and will be updated as progress is made toward version 1.0. It should be noted that this roadmap applies only to the blockchain software and not to the other tools and utilities such as wallets and block explorers which will have their own teams and dedicated roadmaps once Phase 1 is complete.

***Everything contained in this document is in draft form and subject to change at any time and provided for information purposes only. block.one does not guarantee the accuracy of the information contained in this roadmap and the information is provided “as is” with no representations or warranties, express or implied.***

# Этап 1 - Минимальная необходимая тестовая среда - Лето 2017

The goal of this phase is to establish the APIs that developers will require to start building and testing applications on EOS.IO. In order for developers to start testing their applications they will require the following to be implemented:

### Standalone Node (Dan & Nathan)

A standalone node operates a test blockchain and produces blocks while exposing an API. This node does not need to concern itself with any P2P networking code.

### Нативные контракты (Нэйтан)

The EOS.IO software has a number of native contracts. These are contracts that manage the core operations of the blockchain and exist outside the Web Assembly interface. These contracts include:

  1. @eos - управляет переводами токена EOS
  2. @stake - управляет удерживаемыми EOS, голосованием и выборами Производителей
  3. @system - управляет разрешениями, сообщениями и обновлениями контактного кода

### Virtual Machine API (Dan)

Contracts are compiled to WebAssembly (WASM) and WASM must interface with the blockchain via a defined API. This API is what developers depend upon to build applications and be relatively stable before developers can really start to build on EOS.

### Интерфейс RPC (Архаг, Нэйтан)

A simple JSON RPC over HTTP interface will be provided that enables developers to broadcast transactions and query application state. Это крайне важно как для публикации, так и для взаимодействия с тестовыми приложениями.

### Инструменты командной строки (Архаг)

Инструменты командной строки упрощают интеграцию интерфейса RPC в созданные разработчиками среды.

### Базовая документация для разработчиков (Джош)

Documents that teach developers how to get started with building on EOS.IO blockchains. This includes documentations of the WASM API, RPC Interface, and Command Line Tools.

# Этап 2 - Минимальная необходимая Тестовая сеть - Осень 2017

Everything in Phase 1 assumes a trusted environment that only runs the developer's own code. Before a test network can be deployed several additional features need to be implemented and tested.

### Сетевой код P2P (Фил)

Это плагин, отвечающий за синхронизацию состояния блокчейна между двумя автономными узлами.

### WASM чистка и CPU песочница (Брайан)

The WASM code needs to be sanitized to check for non-deterministic behavior such as floating point operations and infinite loops.

### Resource Usage Tracking & Rate Limiting (Arhag)

В целях предотвращения злоупотреблений мониторинг ресурсов и отслеживание использования ограничивают пользователей по скорости в соответствии с их долей в EOS.

### Тестирование импорта создания (DappHub)

Необходимо разработать инструменты для экспорта данных из состояния распределения токенов EOS и создания файла исходной конфигурации. This will enable anyone participating in the Token Distribution to acquire some initial test EOS (TEOS).

### Межблокчейновая связь (Нэйтан)

Эта функция включает в себя проверку правильности хэширования Меркла транзакций.

# Этап 3 - Тестирование, Проверка безопасности - Зима 2017 / весна 2018

На этом этапе платформа будет проходить интенсивные испытания, сфокусированные на поиске уязвимостей в безопасности и багов. В конце 3-го этапа будет выпущена версия 1.0.

### Разработка примеров приложений

Примеры приложений необходимы для доказательства того, что платформа предоставляет реальным разработчикам все необходимые функции.

### Баунти за успешную атаку сети

Attacking the network with spam, virtual machine exploits, and bug crashes, and non-deterministic behavior will be a heavily involved process but necessary to ensure that version 1.0 is stable.

### Поддержка языков

Добавление поддержки нескольких дополнительных языков, которые будут компилироваться в WASM: C++, Rust и т.п.

### Документация и инструкции

# Phase 4 - Parallel Optimization Summer / Fall 2018

После выпуска стабильной версии продукта 1.0 мы перейдем к оптимизации кода для параллельного выполнения.

# Phase 5 - Cluster Implementation The Future