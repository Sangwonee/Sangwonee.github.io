---
title: "Jekyll 블로그 404 페이지 오류 해결 과정"
date: 2025-12-03
categories: [Troubleshooting]
tags: [jekyll, error, troubleshooting]
---

## 문제 발생

Minimal Mistakes 테마로 Jekyll 블로그를 만들고, `_pages` 폴더에 `about.md`와 `categories.md` 페이지를 추가했다. 하지만 로컬 서버에서 `/about/`이나 `/categories/` 경로로 접속하면 404 Not Found 오류가 발생했다. 포스트는 정상적으로 보이지만, 새로 추가한 페이지만 보이지 않는 문제였다.

## 해결 과정

이 문제를 해결하기 위해 거의 반나절 동안 수많은 시도를 했다. 그 길고 험난했던 여정을 기록으로 남긴다.

### 1. Jekyll 서버 실행 문제

가장 처음에는 Jekyll 서버 자체가 실행되지 않았다.

- **오류**: `bundler: command not found: jekyll`
- **원인**: 프로젝트에 필요한 Gem(라이브러리)들이 설치되지 않음.
- **해결**: `bundle install` 명령어로 `Gemfile`에 명시된 의존성 설치.

### 2. Gemfile 설정 오류

`bundle install`을 실행하자 `Gemfile` 자체에 문제가 있었다.

- **오류**: `There are no gemspecs at ...`
- **원인**: `Gemfile`에 웹사이트용이 아닌 Gem 개발용 설정인 `gemspec`이 포함되어 있었음.
- **해결**: `gemspec` 라인을 삭제하고, `minimal-mistakes-jekyll` 테마와 Jekyll 플러그인 등 필요한 gem 목록으로 교체.

### 3. Gem 설치 권한 오류

다시 `bundle install`을 시도했지만, 이번에는 macOS 시스템 폴더에 gem을 설치하려다 권한 문제에 부딪혔다.

- **오류**: `Bundler::PermissionError`
- **원인**: 시스템 Ruby 환경에 gem을 설치할 쓰기 권한이 없음. (시스템 환경을 직접 건드리는 것은 좋지 않다.)
- **해결**: Bundler 설정을 변경하여 gem을 시스템이 아닌 프로젝트 폴더 내(`vendor/bundle`)에 설치하도록 변경.
  ```bash
  bundle config set --local path 'vendor/bundle'
  bundle install
  ```

### 4. 페이지가 렌더링되지 않는 문제 (1) - `include` 설정

서버는 드디어 실행되었지만, 여전히 404 오류는 계속되었다. `_site` 폴더를 확인해보니 `_pages` 폴더의 `.md` 파일들이 HTML로 변환되지 않고 그대로 복사되고 있었다.

- **원인**: `_config.yml`의 `include` 설정에 `_pages`가 포함되어 있었다. 이 설정은 Jekyll에게 해당 폴더를 변환하지 말고 그대로 복사하라고 지시하는 것과 같다.
- **해결**: `_config.yml`의 `include` 목록에서 `_pages` 항목을 제거.

### 5. 페이지가 렌더링되지 않는 문제 (2) - `collections` 설정

`include` 문제를 해결했지만, 여전히 페이지는 렌더링되지 않았다.

- **원인**: Jekyll은 `_posts` 외에 `_`로 시작하는 커스텀 폴더를 자동으로 페이지로 인식하지 않는다. `_pages` 폴더를 페이지 묶음(컬렉션)으로 사용하려면 이를 명시적으로 선언해야 한다.
- **해결**: `_config.yml`에 `collections` 설정을 추가하여 `_pages` 폴더를 정식 페이지 컬렉션으로 등록.
  ```yaml
  collections:
    pages:
      output: true
  ```

### 6. 최종 원인: Front Matter 이전의 주석

위의 모든 문제를 해결했음에도 불구하고, 페이지는 여전히 렌더링되지 않았다. 마지막으로 `test.md`라는 새로운 파일을 만들어 테스트해보니 이 파일은 정상적으로 렌더링되는 것을 확인했다. 이로써 문제는 `about.md`와 `categories.md` 파일 자체에 있다는 것이 확실해졌다.

- **진짜 원인**: **파일의 맨 첫 줄 시작이 `---`가 아니었다.** `about.md`와 `categories.md` 파일의 맨 앞에 `<!-- ... -->` 형태의 HTML 주석이 포함되어 있었다. Jekyll은 파일의 가장 첫 부분이 `---`로 시작하는 Front Matter 블록이어야만 해당 파일을 특별한 문서(페이지, 포스트 등)로 인식하고 HTML로 변환한다. 그 앞에 단 하나의 문자라도 있으면 그냥 일반 파일로 취급하여 복사만 할 뿐이다.
- **최종 해결**: 각 파일 맨 앞에 있던 주석을 모두 삭제하여 파일이 `---`로 시작하도록 수정했다.

## 결론 및 교훈

결국 서버를 재시작하자 모든 페이지가 정상적으로 보였다. 간단한 404 오류인 줄 알았지만, Jekyll의 빌드 환경부터 설정, 그리고 파일 내용의 미묘한 차이까지 모두 얽혀있는 복합적인 문제였다.

이번 트러블슈팅을 통해 얻은 교훈은 다음과 같다.

1.  **Jekyll 문서는 반드시 Front Matter(`---`)로 시작해야 한다.** 그 앞에 주석이나 공백 등 어떤 문자도 있어서는 안 된다.
2.  Jekyll 빌드에 문제가 생기면 `_site` 폴더의 내용물을 확인하여 파일이 어떻게 처리되고 있는지 직접 눈으로 보는 것이 가장 확실하다.
3.  `_config.yml`은 Jekyll의 모든 동작을 제어하는 핵심이다. `include`, `exclude`, `collections` 등의 설정을 유심히 살펴보자.
4.  문제가 해결되지 않을 때는, 아주 간단한 테스트용 파일을 만들어 '통제된 실험'을 해보는 것이 원인을 좁히는 데 큰 도움이 된다.
