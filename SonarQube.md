---
tags:
  - Sonar
  - Configuration
---
Это называется **“Decorate MR/PR with Sonar Issues”** — и это **очень мощная практика для улучшения качества кода**.

---

# 🚀 Часть 1: GitHub — через GitHub Actions + SonarCloud

## 📄 Шаг 1: Настрой `sonar-project.properties`

В корне репозитория:

```properties
sonar.projectKey=your-org_your-repo
sonar.organization=your-org
sonar.host.url=https://sonarcloud.io

sonar.sources=.
sonar.exclusions=**/bin/**,**/obj/**,**/*.min.js
sonar.tests=.
sonar.test.inclusions=**/*Test*/
sonar.coverage.exclusions=**/Program.cs,**/Startup.cs
```

---

## 📄 Шаг 2: Создай `.github/workflows/sonar.yml`

```yaml
name: SonarQube Scan

on:
  pull_request:
    branches: [ main ]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # важно для анализа изменений

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cache SonarScanner
        uses: actions/cache@v3
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar

      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: https://sonarcloud.io

      - name: SonarQube Decorate PR
        uses: sonarsource/sonarqube-quality-gate-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

> 🔑 `SONAR_TOKEN` — создай в SonarCloud → My Account → Security → Generate Token → добавь в GitHub Secrets.

---

# 🚀 Часть 2: GitLab — через GitLab CI + SonarQube

> ⚠️ **Важно**: Inline-комментарии в MR поддерживаются **только в GitLab EE/Ultimate** (не в Community Edition).

## 📄 Шаг 1: `.gitlab-ci.yml`

```yaml
stages:
  - test
  - sonar

variables:
  SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
  GIT_DEPTH: "0"  # важно для анализа изменений

cache:
  key: "${CI_JOB_NAME}"
  paths:
    - .sonar/cache

sonarqube-check:
  stage: sonar
  image: 
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  script:
    - sonar-scanner
      -Dsonar.host.url=$SONAR_HOST_URL
      -Dsonar.login=$SONAR_TOKEN
      -Dsonar.projectKey=$CI_PROJECT_NAME
      -Dsonar.gitlab.project_id=$CI_PROJECT_PATH
      -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA
      -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME
  allow_failure: true
  only:
    - merge_requests
```

## 📄 Шаг 2: Переменные в GitLab CI/CD Settings → Variables

```
SONAR_HOST_URL = https://your-sonarqube.com
SONAR_TOKEN = your-token-here
```

## 📌 Что нужно для inline-комментариев в MR?

- GitLab **EE или Ultimate**.
- Настроить **Integration → SonarQube** в **Project Settings → Integrations**.
- Указать URL и токен SonarQube.
- Включить опцию **“Enable comments inside merge requests”**.

→ После этого SonarQube будет **автоматически публиковать комментарии в MR**.

---

# ✅ Общие рекомендации

## 🧩 1. Всегда используй `fetch-depth: 0` / `GIT_DEPTH: 0`

Чтобы Sonar мог анализировать **изменения относительно target-ветки** — нужна полная история.

## 🧩 2. Кэшируй `.sonar/cache`

Ускоряет повторные запуски.

## 🧩 3. Используй Quality Gate

Настрой в SonarQube — чтобы pipeline падал, если качество ниже порога.

## 🧩 4. Не публикуй чувствительные данные

Никогда не коммить токены — только через Secrets.

---

# ✅ Итог — как добавить вывод ошибок Sonar в MR

| Платформа        | Что сделать                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------------- |
| **GitHub**       | Используй `sonarsource/sonarqube-scan-action` + `sonarqube-quality-gate-action` → автоматически комментирует PR |
| **GitLab**       | Настрой `sonar-scanner` + включи интеграцию в Project Settings → работает в EE/Ultimate                         |

---

## 💡 Бонус: Что делать, если inline-комментарии не поддерживаются?

Можно **публиковать отчёт как артефакт** или **оставлять комментарий с линком на SonarQube**:

```yaml
- name: Comment PR with SonarQube Link
  if: always()
  uses: actions/github-script@v7
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      await github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: '📊 SonarQube Report: [View Report](${{ env.SONAR_URL }}/dashboard?id=your-project-key)'
      });
```

---
