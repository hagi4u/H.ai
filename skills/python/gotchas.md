# Python 주의사항

## Flask (kream 어드민)

### 실행
```bash
uv run flask run -p 8000
```

### 필수 환경변수 (.env)
- `FLASK_ENV=dev` — 프로젝트 설정 로드 필수 (없으면 설정 안 됨)
- `FLASK_DEBUG=1` — Flask 2.3+ 권장 (자동 리로드, 디버거)
- `KREAM_SECRET_KEY` — 앱 시크릿

### 주의사항
- **FLASK_ENV Deprecation (Flask 2.3+)**: deprecated 경고가 뜨지만 프로젝트 설정 로드를 위해 여전히 필요. `FLASK_DEBUG`만으로는 부족
- **ENCRYPTED_PRIVACY_ARCH_DATABASE_URI 누락 시 에러**: `.env`에 없으면 앱 실행 실패. `app/config.py` L119-135에서 조건부 처리로 해결됨
