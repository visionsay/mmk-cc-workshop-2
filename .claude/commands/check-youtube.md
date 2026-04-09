# YouTube 채널 최신 영상 확인

config.json에 설정된 YouTube 채널들의 최신 영상을 확인합니다.

## 실행 절차

1. `config.json` 파일을 읽어 채널 목록을 확인합니다.
2. `state.json` 파일을 읽어 이미 처리된 영상 ID 목록을 확인합니다.
3. 각 채널의 RSS 피드 URL을 WebFetch로 가져옵니다:
   - 삼프로TV: `https://www.youtube.com/feeds/videos.xml?channel_id=UChlv4GSd7OQl3js-jkLOnFA`
   - 증시각도: `https://www.youtube.com/feeds/videos.xml?channel_id=UCdOjVxkj5JA0iDu3_xcsTyQ`
4. WebFetch prompt로 다음 정보를 추출합니다:
   - 각 영상의 제목(title), URL(link), 게시일(published), video ID
5. `state.json`의 `processed_videos` 목록에 없는 새로운 영상만 필터링합니다.
6. 새로운 영상 목록을 다음 형식으로 출력합니다:

```
## 새로운 영상 발견
- [채널명] 제목 | URL | 게시일
```

7. 새 영상이 없으면 "새로운 영상이 없습니다."를 출력합니다.

## WebFetch 프롬프트 예시

```
최근 15개 영상 엔트리를 추출해주세요. 각 영상마다: title, video URL (yt:videoId 또는 link href), published date를 반환해주세요. JSON 배열 형태로 출력해주세요: [{"title": "...", "url": "https://www.youtube.com/watch?v=VIDEO_ID", "videoId": "VIDEO_ID", "published": "..."}]
```

## 주의사항
- RSS 피드는 최신 15개 영상만 포함합니다.
- state.json의 processed_videos에 이미 있는 videoId는 건너뜁니다.
- 결과를 사용자에게 명확하게 보여주세요.
