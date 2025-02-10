Вот готовое решение для Telegram бота в GitHub Actions, который будет отправлять уведомления о релизах и пререлизаx в тему группового чата:

### 1. Создайте Telegram бота
1. Напишите [@BotFather](https://t.me/BotFather) в Telegram
2. Используйте команду `/newbot` и следуйте инструкциям
3. Сохраните полученный API-токен

### 2. Добавьте бота в групповой чат
1. Добавьте созданного бота в нужный чат
2. Получите chat_id и message_thread_id группы:
   - Отправьте любое сообщение в чат
   - Перейдите по ссылке: `https://api.telegram.org/bot<ВАШ_ТОКЕН>/getUpdates`
   - Найдите ID чата `chat.id` в ответе API
   - Найдите ID темы `message_thread_id` в ответе API


### 3. Настройте GitHub Secrets
В репозитории:
1. Settings → Secrets → New repository secret
2. Добавьте:
   - `TELEGRAM_BOT_TOKEN` (токен из шага 1)
   - `TELEGRAM_CHAT_ID` (ID чата из шага 2)
   - `TELEGRAM_TOPIC_ID` (ID темы из шага 2)

### 4. Создайте workflow файл
`.github/workflows/release-notifier.yml`:
```yaml
name: Release Notifier

on:
  release:
    types: [published]

jobs:
  notify-telegram:
    runs-on: ubuntu-latest
    steps:
      - name: Send Release Notification to Telegram
        env:
          BOT_TOKEN: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          CHAT_ID: ${{ secrets.TELEGRAM_CHAT_ID }}
          TOPIC_ID: ${{ secrets.TELEGRAM_TOPIC_ID }}
        run: |
          RELEASE_TYPE="Release"
          if [ "${{ github.event.release.prerelease }}" == "true" ]; then
            RELEASE_TYPE="Prerelease"
          fi

          RELEASE_TAG=${GITHUB_REF#refs/tags/}
          
          MESSAGE=$(cat <<EOF
          🔔 *${RELEASE_TYPE} Published* 🔔

          📦 *Repository:* [${{ github.repository }}](https://github.com/${{ github.repository }})
          🏷 *Version:* \`${RELEASE_TAG}\`
          📛 *Title:* ${{ github.event.release.name }}

          🔗 [View Release](https://github.com/${{ github.repository }}/releases/tag/${RELEASE_TAG})
          EOF
          )

          curl -s -X POST \
            -H "Content-Type: application/json" \
            -d "{
              \"chat_id\": \"$CHAT_ID\",
              \"message_thread_id\": \"$TOPIC_ID\",
              \"text\": \"$MESSAGE\",
              \"parse_mode\": \"Markdown\"
            }" \
            "https://api.telegram.org/bot$BOT_TOKEN/sendMessage"
```
