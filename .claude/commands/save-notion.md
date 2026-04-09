# Notion 데이터베이스에 영상 요약 저장

YouTube 증시 요약 결과를 Notion "YouTube 증시 요약" 데이터베이스에 저장합니다.

## 인자
- `$ARGUMENTS`: 저장할 영상 정보 (또는 /summarize-video 결과를 참조)

## Notion DB 정보
- DB ID: `ace11b03-e8ab-4bc6-95a7-29f2e6709673`
- Data Source: `collection://59ad4fd8-441b-4091-a5bb-ab65715855d0`

## DB 스키마
| 필드 | 타입 | 설명 |
|------|------|------|
| 제목 | title | 영상 제목 |
| URL | url | YouTube 영상 URL |
| 상태 | status | "시작 전", "진행 중", "완료" |
| 업로드일 | date | 영상 게시일 |
| 요약 | text | AI 요약 내용 |
| 채널 | select | "삼프로TV" 또는 "증시각도" |
| 카테고리 | multi_select | "증시전망", "경제뉴스", "종목분석", "시황" 중 선택 |

## 실행 절차

1. Notion MCP 도구 `notion-create-pages`를 사용하여 새 페이지를 생성합니다.

2. 페이지 생성 시 다음 정보를 포함합니다:

```
notion-create-pages:
  database_id: "ace11b03-e8ab-4bc6-95a7-29f2e6709673"
  pages:
    - properties:
        제목: "영상 제목"
        userDefined:URL: "https://www.youtube.com/watch?v=VIDEO_ID"
        상태: "완료"
        업로드일: "YYYY-MM-DD"
        요약: "AI 요약 내용"
        채널: "삼프로TV" 또는 "증시각도"
        카테고리: ["시황"]
```

3. 저장 완료 후 Notion 페이지 URL을 사용자에게 보여줍니다.

## 카테고리 자동 분류 규칙
- 제목에 "시황", "마감", "장전", "마켓" → `시황`
- 제목에 "전망", "예측", "방향" → `증시전망`
- 제목에 "뉴스", "이슈", "속보" → `경제뉴스`
- 제목에 종목명/기업명 포함 → `종목분석`
- 복수 카테고리 가능 (multi_select)

## 주의사항
- 동일 URL의 영상이 이미 존재하는지 확인하지 않습니다 (state.json으로 중복 방지).
- 요약 텍스트가 너무 길면 500자 이내로 축약합니다.
- 채널명은 config.json의 채널 이름과 정확히 일치해야 합니다.
