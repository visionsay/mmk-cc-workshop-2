# YouTube 증시 모니터링 (메인 오케스트레이터)

한국 증시 관련 YouTube 채널을 모니터링하여 시황 분석 영상을 자동으로 요약하고, Slack 알림과 Notion 저장을 수행합니다.

이 스킬은 전체 파이프라인을 순서대로 실행합니다. `/loop 1h /youtube-monitor` 명령으로 1시간마다 자동 실행할 수 있습니다.

## 전체 파이프라인

### STEP 1: 최신 영상 확인 (check-youtube)

1. `config.json`을 읽어 채널 목록을 확인합니다.
2. `state.json`을 읽어 이미 처리된 영상 ID를 확인합니다.
3. 각 채널의 RSS 피드를 WebFetch로 가져옵니다:
   - 삼프로TV: `https://www.youtube.com/feeds/videos.xml?channel_id=UChlv4GSd7OQl3js-jkLOnFA`
   - 증시각도: `https://www.youtube.com/feeds/videos.xml?channel_id=UCdOjVxkj5JA0iDu3_xcsTyQ`
4. WebFetch 프롬프트: "최근 15개 영상을 JSON 배열로 추출: [{\"title\": \"...\", \"url\": \"https://www.youtube.com/watch?v=VIDEO_ID\", \"videoId\": \"VIDEO_ID\", \"published\": \"YYYY-MM-DD\"}]"
5. `state.json`의 `processed_videos`에 없는 새 영상만 추출합니다.

→ 새 영상이 없으면 "새로운 영상이 없습니다. 모니터링 완료." 출력 후 종료합니다.

### STEP 2: 시황 분석 콘텐츠 필터링 (filter-content)

1. `config.json`의 필터 설정을 읽습니다.
2. 새 영상 목록에서 필터링합니다:
   - **포함**: 시황, 시장분석, 장마감, 마감시황, 증시전망, 장전시황, 미국시황, 주간전망, 데일리, 마켓, 인사이트, 여의도
   - **제외**: 크립토, 비트코인, 이더리움, 코인, NFT
3. 제목에 포함 키워드가 있고, 제외 키워드가 없는 영상만 선택합니다.

→ 매칭 영상이 없으면 "시황 분석 콘텐츠 없음. 모니터링 완료." 출력 후, 처리된 영상 ID를 state.json에 추가하고 종료합니다.

### STEP 3: 각 영상에 대해 (반복)

필터링된 각 영상에 대해 아래 3a~3c를 순서대로 실행합니다.

#### STEP 3a: 자막 추출 및 요약 (summarize-video)

1. `mmk youtube transcript <url>` 명령으로 자막을 추출합니다.
2. 자막 기반으로 한국어 요약을 생성합니다:

```
## 📊 [채널명] 영상 제목

### 핵심 요약
- 3~5문장으로 핵심 내용 요약

### 주요 포인트
1. 포인트 1
2. 포인트 2
3. 포인트 3

### 언급된 종목/섹터
- (해당 시)

### 전망/시사점
- 향후 전망 요약
```

3. 자막 실패 시 `mmk youtube metadata <url>`로 메타데이터 기반 간략 요약을 생성합니다.

#### STEP 3b: Slack 알림 전송 (notify-slack)

1. Slack MCP `slack_send_message`를 사용합니다:
   - channel_id: `C0ASLTU8GQG`
   - message: 아래 형식

```
📊 *증시 요약 알림*

*[채널명] 영상 제목*
📅 업로드: YYYY-MM-DD
🔗 영상 링크

---
핵심 요약 내용

*주요 포인트*
• 포인트 1
• 포인트 2

*전망*
전망 내용
```

#### STEP 3c: Notion DB 저장 (save-notion)

1. Notion MCP `notion-create-pages`를 사용합니다:
   - parent: `{"type": "data_source_id", "data_source_id": "59ad4fd8-441b-4091-a5bb-ab65715855d0"}`
   - pages: 아래 형식

```json
{
  "properties": {
    "제목": "영상 제목",
    "userDefined:URL": "https://www.youtube.com/watch?v=VIDEO_ID",
    "상태": "완료",
    "date:업로드일:start": "YYYY-MM-DD",
    "date:업로드일:is_datetime": 0,
    "요약": "AI 요약 내용 (500자 이내)",
    "채널": "삼프로TV 또는 증시각도",
    "카테고리": "[\"시황\"]"
  }
}
```

카테고리 자동 분류:
- "시황", "마감", "장전", "마켓" → `시황`
- "전망", "예측", "방향" → `증시전망`
- "뉴스", "이슈", "속보" → `경제뉴스`
- 종목명/기업명 포함 → `종목분석`

### STEP 4: 상태 업데이트

1. `state.json` 파일을 업데이트합니다:
   - `processed_videos` 배열에 처리된 모든 새 영상의 videoId를 추가합니다 (필터링 통과 여부 무관).
   - `last_check`를 현재 시각(ISO 8601)으로 업데이트합니다.
2. 처리 결과 요약을 출력합니다:

```
## ✅ 모니터링 완료
- 확인 시각: YYYY-MM-DD HH:MM
- 새 영상: N건
- 시황 분석: M건 (요약/알림/저장 완료)
- 건너뜀: K건 (필터 미매칭)
```

## 스케줄 실행

이 스킬을 1시간마다 자동 실행하려면:
```
/loop 1h /youtube-monitor
```

## 주의사항
- 모든 단계에서 에러가 발생하면 해당 영상을 건너뛰고 다음 영상을 처리합니다.
- state.json은 항상 최신 상태로 유지합니다.
- Slack 메시지는 5000자 제한이 있으므로 요약을 간결하게 유지합니다.
- Notion 저장 시 채널명은 "삼프로TV" 또는 "증시각도" 정확히 일치해야 합니다.
