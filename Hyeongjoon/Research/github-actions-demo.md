# GitHub Actions 실습 가이드

## 🎯 실습 목표
- GitHub Actions의 기본 개념 이해
- 간단한 워크플로우 작성 및 실행
- 자동화된 CI/CD 파이프라인 체험

## 📁 프로젝트 구조
```
github-actions-demo/
├── .github/
│   └── workflows/
│       ├── hello-world.yml
│       ├── test-workflow.yml
│       └── deploy-workflow.yml
├── src/
│   ├── index.html
│   ├── style.css
│   └── script.js
├── tests/
│   └── test.js
├── package.json
└── README.md
```

## 🚀 실습 1: Hello World 워크플로우

### 1단계: GitHub 저장소 생성
1. GitHub에서 새 저장소 생성: `github-actions-demo`
2. 로컬에서 저장소 클론

### 2단계: 기본 파일 생성

**package.json**
```json
{
  "name": "github-actions-demo",
  "version": "1.0.0",
  "description": "GitHub Actions 실습 프로젝트",
  "main": "index.js",
  "scripts": {
    "test": "node tests/test.js",
    "build": "echo '빌드 완료!'",
    "start": "echo '서버 시작!'"
  },
  "keywords": ["github-actions", "ci-cd"],
  "author": "Your Name",
  "license": "MIT"
}
```

**src/index.html**
```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GitHub Actions Demo</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>🎉 GitHub Actions 실습 성공!</h1>
        <p>이 페이지는 GitHub Actions로 자동 배포되었습니다.</p>
        <button id="testBtn">테스트 버튼</button>
        <div id="result"></div>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

**src/style.css**
```css
body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    margin: 0;
    padding: 20px;
    min-height: 100vh;
}

.container {
    max-width: 600px;
    margin: 0 auto;
    background: white;
    padding: 40px;
    border-radius: 10px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.2);
    text-align: center;
}

h1 {
    color: #333;
    margin-bottom: 20px;
}

button {
    background: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
    margin: 10px;
}

button:hover {
    background: #0056b3;
}

#result {
    margin-top: 20px;
    padding: 10px;
    border-radius: 5px;
}
```

**src/script.js**
```javascript
document.getElementById('testBtn').addEventListener('click', function() {
    const result = document.getElementById('result');
    result.innerHTML = '✅ 버튼이 정상적으로 작동합니다!';
    result.style.backgroundColor = '#d4edda';
    result.style.color = '#155724';
});
```

**tests/test.js**
```javascript
// 간단한 테스트 함수
function testFunction() {
    return "테스트 성공!";
}

// 테스트 실행
console.log("🧪 테스트 시작...");
const result = testFunction();
console.log("결과:", result);

if (result === "테스트 성공!") {
    console.log("✅ 모든 테스트가 통과했습니다!");
    process.exit(0);
} else {
    console.log("❌ 테스트가 실패했습니다!");
    process.exit(1);
}
```

### 3단계: GitHub Actions 워크플로우 생성

**.github/workflows/hello-world.yml**
```yaml
name: Hello World Workflow

# 언제 실행할지 정의
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# 실행할 작업들
jobs:
  hello-world:
    runs-on: ubuntu-latest
    
    steps:
    # 1단계: 코드 체크아웃
    - name: Checkout code
      uses: actions/checkout@v3
      
    # 2단계: Node.js 설정
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    # 3단계: 의존성 설치
    - name: Install dependencies
      run: npm install
      
    # 4단계: 테스트 실행
    - name: Run tests
      run: npm test
      
    # 5단계: 빌드
    - name: Build project
      run: npm run build
      
    # 6단계: 성공 메시지 출력
    - name: Success message
      run: |
        echo "🎉 Hello World Workflow가 성공적으로 실행되었습니다!"
        echo "현재 시간: $(date)"
        echo "브랜치: ${{ github.ref }}"
        echo "커밋: ${{ github.sha }}"
```

## 🧪 실습 2: 고급 워크플로우

**.github/workflows/advanced-workflow.yml**
```yaml
name: Advanced Workflow

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  # 코드 품질 검사
  code-quality:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm install
      
    - name: Run linting
      run: |
        echo "🔍 코드 품질 검사 중..."
        # ESLint가 있다면: npm run lint
        echo "✅ 코드 품질 검사 완료!"
        
    - name: Run tests
      run: npm test
      
  # 빌드 및 배포
  build-and-deploy:
    needs: code-quality
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm install
      
    - name: Build project
      run: npm run build
      
    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./src
```

## 🎮 실습 3: 조건부 워크플로우

**.github/workflows/conditional-workflow.yml**
```yaml
name: Conditional Workflow

on:
  push:
    branches: [ main, develop, feature/* ]
  pull_request:
    branches: [ main ]

jobs:
  detect-environment:
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.env.outputs.environment }}
    steps:
    - name: Detect environment
      id: env
      run: |
        if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
          echo "environment=production" >> $GITHUB_OUTPUT
        elif [[ "${{ github.ref }}" == "refs/heads/develop" ]]; then
          echo "environment=staging" >> $GITHUB_OUTPUT
        else
          echo "environment=development" >> $GITHUB_OUTPUT
        fi
        
    - name: Show environment
      run: echo "현재 환경: ${{ steps.env.outputs.environment }}"
      
  deploy:
    needs: detect-environment
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    if: needs.detect-environment.outputs.environment == matrix.environment
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Deploy to ${{ matrix.environment }}
      run: |
        echo "🚀 ${{ matrix.environment }} 환경에 배포 중..."
        echo "브랜치: ${{ github.ref }}"
        echo "커밋: ${{ github.sha }}"
        
        # 실제 배포 명령어들
        if [[ "${{ matrix.environment }}" == "production" ]]; then
          echo "⚠️ 프로덕션 배포 - 주의 필요!"
        elif [[ "${{ matrix.environment }}" == "staging" ]]; then
          echo "🔧 스테이징 배포 - 테스트 환경"
        else
          echo "🛠️ 개발 배포 - 개발 환경"
        fi
```

## 📊 실습 4: 워크플로우 모니터링

**.github/workflows/monitoring.yml**
```yaml
name: Workflow Monitoring

on:
  workflow_run:
    workflows: ["Hello World Workflow", "Advanced Workflow"]
    types: [completed]

jobs:
  notify:
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion != 'skipped' }}
    steps:
    - name: Notify workflow result
      run: |
        echo "📊 워크플로우 모니터링 결과:"
        echo "워크플로우: ${{ github.event.workflow_run.name }}"
        echo "결과: ${{ github.event.workflow_run.conclusion }}"
        echo "시작 시간: ${{ github.event.workflow_run.created_at }}"
        echo "완료 시간: ${{ github.event.workflow_run.updated_at }}"
        
        if [[ "${{ github.event.workflow_run.conclusion }}" == "success" ]]; then
          echo "✅ 워크플로우가 성공적으로 완료되었습니다!"
        else
          echo "❌ 워크플로우가 실패했습니다."
        fi
```

## 🎯 실습 실행 방법

### 1. 저장소 설정
```bash
# 로컬에서 실행
git clone https://github.com/your-username/github-actions-demo.git
cd github-actions-demo
```

### 2. 파일 생성
위의 모든 파일들을 생성하고 저장소에 추가

### 3. GitHub에 푸시
```bash
git add .
git commit -m "Add GitHub Actions workflows"
git push origin main
```

### 4. 결과 확인
1. GitHub 저장소 페이지로 이동
2. "Actions" 탭 클릭
3. 워크플로우 실행 상태 확인

## 🔍 실습 결과 확인

### 성공적인 실행 시:
- ✅ 모든 단계가 녹색으로 표시
- 📊 실행 시간 및 로그 확인 가능
- 🎉 "Success message" 단계에서 커스텀 메시지 확인

### 실패 시 디버깅:
- ❌ 실패한 단계 확인
- 📝 로그 파일에서 오류 메시지 확인
- 🔧 코드 수정 후 재실행

## 🎓 학습 포인트

1. **워크플로우 구조 이해**: `on`, `jobs`, `steps` 개념
2. **조건부 실행**: `if`, `needs` 활용
3. **환경별 배포**: 브랜치별 다른 동작
4. **모니터링**: 워크플로우 실행 결과 추적
5. **실제 CI/CD 파이프라인**: 코드 → 테스트 → 빌드 → 배포

## 🚀 다음 단계

실습 완료 후 시도해볼 것들:
- [ ] Slack 알림 추가
- [ ] Docker 이미지 빌드 및 푸시
- [ ] AWS/Google Cloud 배포
- [ ] 자동 버전 태깅
- [ ] 보안 스캔 통합 