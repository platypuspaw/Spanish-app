# HolaTuki Spanish App — CLAUDE.md

## 프로젝트 개요

- **앱 이름**: HolaTuki (스페인어 학습 PWA)
- **배포 주소**: https://platypuspaw.github.io/Spanish-app
- **파일 구조**: `index.html` 단일 파일 (HTML + CSS + JS 전부 포함)
- **목적**: 과테말라 주재 한국인을 위한 실무 중심 스페인어 학습

---

## 절대 규칙

1. **원본 파일(`index.html`) 직접 수정 금지** — 반드시 복사본에서 작업
2. **작업 완료 파일은 항상 `index.html` 단일 파일로 출력**
3. **JS 문법 체크 필수**: 작업 후 반드시 실행 (PowerShell 사용)
   ```powershell
   python -c @"
   import re
   with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
       content = f.read()
   scripts = re.findall(r'<script[^>]*>(.*?)</script>', content, re.DOTALL)
   with open('C:/Users/mino/AppData/Local/Temp/check.js', 'w', encoding='utf-8') as f:
       f.write('\n'.join(scripts))
   "@
   node --check "C:/Users/mino/AppData/Local/Temp/check.js"
   ```
4. **단어 중복 검사 필수**: 단어 추가/삭제 후 반드시 실행 (PowerShell 사용)
   ```powershell
   # verbs[] / nouns[] 중복 검사 (공백 없는 패턴: { es:'...' )
   python -c @"
   import re
   with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
       c = f.read()
   words = re.findall(r\"{ es:'([^']+)'\", c)
   dups = {w for w in words if words.count(w) > 1}
   print(len(dups), list(dups) if dups else 'none')
   "@

   # vocabBook[] 중복 검사 (공백 있는 패턴: { es: '...' )
   python -c @"
   import re
   with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
       c = f.read()
   words = re.findall(r\"{ es: '([^']+)'\", c)
   dups = {w for w in words if words.count(w) > 1}
   print(len(dups), list(dups) if dups else 'none')
   "@
   ```
5. **JS 문자열 내 줄바꿈 금지** — `sentence:'...'` 같은 single-quote 문자열 안에 실제 줄바꿈 절대 금지 (SyntaxError 원인)

---

## 파일 구조

```
index.html
├── <style>         CSS (CSS 변수, 컴포넌트 스타일)
├── <nav>           탭 네비게이션
├── <main>          각 탭 콘텐츠 (tab-* div들)
└── <script>        데이터 배열 + 앱 로직 전체
```

### 데이터 배열 (script 내부)

| 배열 | 항목 수 | 설명 |
|------|--------|------|
| `verbs[]` | 225개 | 동사 (활용형 포함) |
| `nouns[]` | 686개 | 명사/형용사/부사 |
| `b1Words[]` | 78개 | B1 DELE 전용 단어 — **중복 제거 시 범위에서 반드시 제외** |

---

## 탭 구성

| 탭 이름 | ID | 설명 |
|---------|-----|------|
| 📇 단어 카드 | `cards` | 플래시카드 학습 |
| 🎧 받아쓰기 | `dictation` | 듣고 받아쓰기 |
| 👂 듣기 | `listening` | 문장 듣기 |
| 🔀 동사 변화 | `conjugation` | 동사 활용형 + 검색 |
| ✍️ 퀴즈 | `quiz` | 단어 퀴즈 |
| 📖 단어장 | `wordlist` | 전체 단어 목록 |
| 💬 문장 | `sentences` | 상황별 예문 |
| 📐 B1표현 | `patterns` | B1 문법 패턴 |
| 📊 진도 | `progress` | 학습 진도 |
| 🧩 문법 | `grammar` | 문법 설명 + 연습문제 |

---

## 데이터 형식

### 동사 엔트리 (verbs[])

```javascript
{ 
  es: 'hablar',           // 스페인어 원형
  ko: '말하다',            // 한국어 뜻
  pron: '아블라르',        // 한국어 발음
  type: 'verb',
  cat: 'general',         // 카테고리 (아래 목록 참조)
  verbType: 'ar',         // 'ar' | 'er' | 'ir'
  irregular: false,       // 불규칙 여부
  conjugations: {
    present:     ['hablo','hablas','habla','hablamos','habláis','hablan'],
    preterite:   ['hablé','hablaste','habló','hablamos','hablasteis','hablaron'],
    progressive: ['estoy hablando', ...],   // estar 현재형 + 현재분사
    subjunctive: ['hable','hables', ...],
    imperative:  ['—','habla','hable','hablemos','hablad','hablen'],  // yo는 '—'
    imperfect:   ['hablaba','hablabas', ...],
    perfect:     ['he hablado','has hablado', ...],  // haber 현재형 + 과거분사
    future:      ['hablaré','hablarás', ...],
    conditional: ['hablaría','hablarías', ...]
    // ※ pluperfect(과거완료)는 별도 데이터 없이 perfect에서 자동 생성됨
    //   (he → había, has → habías, ha → había, hemos → habíamos, habéis → habíais, han → habían)
  },
  conjPron: {             // 각 시제 한국어 발음 (present/preterite/progressive/subjunctive/imperative만 필수)
    present:     ['아블로','아블라스','아블라','아블라모스','아블라이스','아블란'],
    preterite:   ['아블레','아블라스떼', ...],
    progressive: ['에스또이 아블란도', ...],
    subjunctive: ['아블레','아블레스', ...],
    imperative:  ['—','아블라','아블레','아블레모스','아블라드','아블렌']
  },
  example:    '문장1 스페인어',
  exPron:     '문장1 한국어 발음',
  exampleKo:  '문장1 한국어 해석',
  example2:   '문장2 스페인어',
  ex2Pron:    '문장2 한국어 발음',
  example2Ko: '문장2 한국어 해석'
}
```

### 명사/형용사 엔트리 (nouns[])

```javascript
{
  es: 'trabajo',
  ko: '일, 직업',
  pron: '뜨라바호',
  type: 'noun',           // 'noun' | 'adj' | 'adv' | 'phrase'
  cat: 'general',
  example:    '문장1 스페인어',
  exPron:     '문장1 발음',
  exampleKo:  '문장1 해석',
  example2:   '문장2 스페인어',
  ex2Pron:    '문장2 발음',
  example2Ko: '문장2 해석'
}
```

---

## 카테고리 목록

### 동사/명사 공통
- `general` — 일반
- `accounting` — 회계/업무
- `work` — 직장
- `daily` — 일상생활
- `emotion` / `feeling` — 감정
- `body` — 신체
- `time` — 시간
- `travel` / `hotel` / `transport` — 여행
- `restaurant` / `shopping` — 식당/쇼핑
- `polite` / `request` — 공손/요청
- `opinion` / `reason` / `purpose` — 의견/이유/목적
- `compare` / `condition` / `suggest` — 비교/조건/제안
- `emergency` — 응급

### 공장/제조 전용 (명사)
- `factory` — 공장 일반
- `factory_sewing` — 봉제
- `factory_equipment` — 장비
- `factory_cleaning` — 세정
- `factory_laundry` — 세탁
- `factory_hardware` — 부자재
- `factory_accounting` — 공장 회계

---

## 동사 변화 탭 시제 키

| 키 | 표시명 | 설명 |
|----|--------|------|
| `present` | 현재 | |
| `preterite` | 단순과거 | |
| `imperfect` | 불완료과거 | |
| `perfect` | 현재완료 | haber 현재 + 과거분사 |
| `pluperfect` | 과거완료 | perfect에서 자동 생성 (데이터 불필요) |
| `future` | 미래 | |
| `conditional` | 조건법 | |
| `progressive` | 진행 | |
| `subjunctive` | 접속법 | |
| `imperative` | 명령 | yo = '—' |

---

## 작업 유형별 주의사항

### 단어 추가 시
- `verbs[]`에 추가할 경우: 9개 시제 `conjugations` + 최소 5개 시제 `conjPron` 필수
- `nouns[]`에 추가할 경우: example/example2 두 개 필수
- 추가 후 중복 검사 실행
- **영어 단어(rack, overlock 등) 절대 추가 금지**

### 중복 제거 시
- `b1Words[]` 배열은 범위에서 **반드시 제외** (verbs와 중복이어도 별도 학습 목적)
- 같은 배열 내 중복: 첫 번째 등장 유지, 이후 제거
- 다른 배열 간 중복(verbs vs nouns): 첫 번째 유지
- **vocabBook 중복 패턴 주의**: `verbs[]`/`nouns[]`는 `{ es:'...'` (공백 없음), `vocabBook[]`는 `{ es: '...'` (공백 있음) — 중복 검사 시 두 패턴 **각각 별도로** 실행해야 함

### 탭/기능 추가 시
- 탭 추가: `<nav>`에 버튼 + `<main>`에 `<div id="tab-{name}">` + `showTab()`에 `init{Name}()` 연결
- 새 문법 섹션: `showGrammarSection()` 함수가 `.g-section` 클래스로 동작

### JS 데이터 작성 시
- single-quote 문자열 안에 실제 줄바꿈 금지 → 한 줄로 작성
- 긴 문장은 공백으로 이어쓰기: `'문장1 → 문장2'`

---

## 자주 쓰는 검증 명령

> **Windows 환경 주의사항**
> - `python3` 아닌 `python` 사용
> - `/tmp/` 경로 없음 → `C:/Users/mino/AppData/Local/Temp/` 사용
> - PowerShell에서 `&&` 연산자 사용 불가 → `;` 또는 `if ($?) { ... }` 사용
> - 복잡한 Python 코드는 PowerShell here-string(`@"..."@`) 으로 전달

```powershell
# JS 문법 체크
python -c @"
import re
with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
    content = f.read()
scripts = re.findall(r'<script[^>]*>(.*?)</script>', content, re.DOTALL)
with open('C:/Users/mino/AppData/Local/Temp/check.js', 'w', encoding='utf-8') as f:
    f.write('\n'.join(scripts))
"@
node --check "C:/Users/mino/AppData/Local/Temp/check.js"

# 전체 단어 수 확인 (verbs/nouns)
python -c @"
import re
with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
    c = f.read()
verbs = re.findall(r\"{ es:'([^']+)'.*?type:'verb'\", c, re.DOTALL)
nouns = re.findall(r\"{ es:'([^']+)'.*?type:'noun'\", c, re.DOTALL)
print('verbs:', len(set(verbs)), 'nouns:', len(set(nouns)))
"@

# 중복 검사 — verbs[]/nouns[] (공백 없는 형식: { es:'...' )
python -c @"
import re
with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
    c = f.read()
words = re.findall(r\"{ es:'([^']+)'\", c)
dups = {w for w in words if words.count(w) > 1}
print(len(dups), list(dups) if dups else 'none')
"@

# 중복 검사 — vocabBook[] (공백 있는 형식: { es: '...' )
python -c @"
import re
with open('C:/Users/mino/Desktop/Spanish-app/index.html', encoding='utf-8') as f:
    c = f.read()
words = re.findall(r\"{ es: '([^']+)'\", c)
dups = {w for w in words if words.count(w) > 1}
print(len(dups), list(dups) if dups else 'none')
"@

# git 배포
git add index.html
git commit -m "update"
git push -u origin main
```

---

## 기타 메모

- localStorage에 학습 진도 저장 → 파일 교체해도 진도 유지됨
- TTS: 브라우저 내장 `SpeechSynthesis` 사용 (스페인어 `es-MX` 우선)
- PWA 설정 없음 (manifest/service worker 미적용)
- CSS 변수: `--accent`(노랑), `--accent3`(파랑), `--correct`(초록), `--wrong`(빨강), `--surface2`(카드 배경)
