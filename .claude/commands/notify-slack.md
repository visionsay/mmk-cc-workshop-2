# Slack 증시 요약 알림 전송

YouTube 증시 요약 결과를 Slack #증시정보 채널에 전송합니다.

## 인자
- `$ARGUMENTS`: 전송할 요약 내용 (또는 /summarize-video 결과를 참조)

## 실행 절차

1. `config.json`에서 Slack 채널 정보를 확인합니다:
   - 채널: `#증시정보`
   - 채널 ID: `C0ASLTU8GQG`

2. Slack MCP 도구 `slack_send_message`를 사용하여 메시지를 전송합니다.

3. 메시지 형식:

```
📊 *증시 요약 알림*

*[채널명] 영상 제목*
📅 업로드: YYYY-MM-DD
🔗 영상 링크

---
*핵심 요약*
- 요약 내용 1
- 요약 내용 2
- 요약 내용 3

*주요 포인트*
1. 포인트 1
2. 포인트 2

*언급 종목/섹터*
종목1, 종목2 (해당 시)

*전망*
전망 내용
```

## MCP 도구 사용

```
mcp__slack__slack_send_message:
  channel_id: "C0ASLTU8GQG"
  message: <위 형식의 메시지>
```

## 주의사항
- Slack 메시지는 5000자 제한이 있으므로, 요약이 길면 핵심만 추려서 전송합니다.
- 여러 영상이 있으면 각 영상별로 별도 메시지로 전송합니다.
- 메시지 전송 후 결과 링크를 사용자에게 보여줍니다.
- Slack markdown 형식을 사용합니다: *bold*, _italic_, `code`, ~strikethrough~
