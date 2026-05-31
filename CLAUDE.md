# orange-build 릴리스 규칙

버전을 올릴 때는 **반드시 아래 두 파일의 `version` 필드를 함께 수정**한다.

- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`

두 파일의 버전이 어긋나면 Claude 마켓플레이스가 옛 버전을 최신으로 인식해 사용자에게 업데이트가 노출되지 않는다. 커밋 전 두 파일의 `version`이 일치하는지 확인할 것.
