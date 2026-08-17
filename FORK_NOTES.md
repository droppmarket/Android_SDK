# FORK_NOTES — форк UseDesk Android SDK для ДропМаркет

Это форк `usedesk/Android_SDK` под организацией `droppmarket`. Содержит точечные
правки поверх релизных тегов апстрима. README намеренно не трогаем (чтобы
избежать конфликтов при будущих rebase/обновлениях с апстрима).

Схема зеркалирует iOS-форк `droppmarket/UseDeskSwift` (см. его FORK_NOTES.md).

## DMT-7189 (тег `4.5.3-dropp.1`)

Тикет: https://droppteam.atlassian.net/browse/DMT-7189

Баг: отправленный файл дублируется в чате после сворачивания/возврата в
приложение. Причина: `ChatImpl.onChatInited` дедуплицирует сообщения истории
только по `id`. Если socket-подтверждение отправки не успело прийти до
дисконнекта (сворачивание сразу после send), локальная копия сообщения
остаётся в модели с `id == localId`; при reconnect история приносит ту же
запись с серверным `id` — обе остаются в списке. Третьего дубля не
появляется (оба id уже в модели), рестарт процесса лечит (история строится
с нуля) — ровно как в репро тикета.

Фикс: перед мержем `chatInited.messages` из модели удаляются неподтверждённые
клиентские копии (`id == localId`), чей `localId` пришёл в истории echo-полем
(`payload.messageId` → `localId` в `MessageResponseConverter`).

## Схема веток и тегов

- Ветка `dropp/X.Y.Z` создаётся **от тега апстрима** `X.Y.Z` (не от `master`).
- Релизные теги форка: `X.Y.Z-dropp.N` (например, `4.5.3-dropp.1`).
- Теги **immutable**: на них завязаны сборки `droppmarket/mobile`
  (gradle тянет `com.github.droppmarket.Android_SDK:*` по тегу через JitPack).

## Политика апгрейда SDK

1. `git fetch upstream --tags`.
2. Создать ветку `dropp/X.Y.Z` от нового тега апстрима `X.Y.Z`.
3. `cherry-pick` коммитов фикса с предыдущей dropp-ветки.
4. Проверить, не починил ли апстрим баг штатно — тогда правка не нужна.
5. Новый тег `X.Y.Z-dropp.1`, обновить координаты в
   `expo-config-plugins/withAndroidBuildGradle.js` + `android/app/build.gradle`.
