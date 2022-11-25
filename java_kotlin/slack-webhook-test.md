```shell
curl --location --request POST 'https://hooks.slack.com/services/your-slack-webhook-url' \
--header 'Content-Type: application/json' \
--data-raw '{
  "text": "test hello"
}'
```