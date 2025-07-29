# Jenkins 완전 가이드

## 목차
1. [Jenkins란 무엇인가?](#jenkins란-무엇인가)
2. [Jenkins 설치 및 설정](#jenkins-설치-및-설정)
3. [Jenkins Pipeline](#jenkins-pipeline)
4. [실습 예제](#실습-예제)
5. [보안 및 Best Practices](#보안-및-best-practices)
6. [2025년 트렌드](#2025년-트렌드)

---

## Jenkins란 무엇인가?

### 개념
Jenkins는 자바 기반의 **오픈소스 자동화 서버**입니다. 개발자들이 소프트웨어를 자동으로 빌드, 테스트, 배포할 수 있게 도와주는 CI/CD 도구입니다.

### 실생활 비유
Jenkins를 **자동화된 공장 라인**이라고 생각해보세요:
- **전통적 방식**: 각 공정마다 사람이 직접 작업 → 실수 가능성, 비효율성
- **Jenkins 방식**: 컨베이어 벨트처럼 자동으로 각 단계를 처리 → 빠르고 정확한 생산

### 핵심 특징

#### 1. 플러그인 생태계 🔌
- **수백 개의 플러그인**: 거의 모든 개발 도구와 연동 가능
- **확장성**: 필요에 따라 기능 추가 가능
- **예시**: Git, Docker, AWS, Kubernetes 등과 연동

#### 2. Pipeline as Code 📜
- **Jenkinsfile**: 파이프라인을 코드로 정의
- **버전 관리**: Git으로 파이프라인 자체도 관리
- **재사용성**: 다른 프로젝트에서도 활용 가능

#### 3. 분산 빌드 🏗️
- **Master-Agent 구조**: 여러 서버에서 동시 작업 실행
- **확장성**: 필요에 따라 빌드 서버 추가
- **효율성**: 작업 부하 분산

### 시장 현황 (2025년)
- **점유율**: CI/CD 시장의 44% 차지
- **사용자**: 전 세계 100만 명 이상의 활성 사용자
- **설치**: 20만 개 이상의 설치 인스턴스

---

## Jenkins 설치 및 설정

### 설치 방법

#### 1. 시스템 요구사항
- **Java**: OpenJDK 11 이상 권장
- **메모리**: 최소 256MB, 권장 1GB+
- **디스크**: 최소 1GB

#### 2. 설치 옵션

##### Windows 설치 🪟
```bash
# 1. Java 설치 확인
java -version

# 2. Jenkins MSI 설치 파일 다운로드
# jenkins.io/download 에서 Windows 설치 파일 다운로드

# 3. 설치 마법사 실행
# - 설치 경로 선택
# - 서비스로 실행 설정
# - 포트 설정 (기본: 8080)
```

##### Linux 설치 🐧
```bash
# Ubuntu/Debian 기준
# 1. Java 설치
sudo apt update
sudo apt install openjdk-11-jdk

# 2. Jenkins 저장소 추가
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# 3. Jenkins 설치
sudo apt update
sudo apt install jenkins

# 4. Jenkins 서비스 시작
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

##### Docker 설치 🐳
```bash
# Jenkins 컨테이너 실행
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

### 초기 설정

#### 1. 첫 접속
```bash
# 브라우저에서 접속
http://localhost:8080

# 초기 비밀번호 확인 (Linux 기준)
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### 2. 플러그인 설치
- **추천 플러그인 설치**: 일반적인 사용 사례에 필요한 플러그인
- **선택적 설치**: 특정 요구사항에 맞는 플러그인만 선택

#### 3. 관리자 계정 생성
- 사용자명, 비밀번호, 이메일 설정
- 보안을 위해 강력한 비밀번호 사용 권장

---

## Jenkins Pipeline

### Pipeline 개념
Pipeline은 Jenkins에서 CI/CD 워크플로우를 정의하는 방법입니다. **Jenkinsfile**이라는 파일에 코드로 작성합니다.

### Pipeline 종류

#### 1. Declarative Pipeline (추천) 📋
**특징**: 구조화되고 읽기 쉬운 문법

```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'myapp:latest'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
                sh 'docker push $DOCKER_IMAGE'
            }
        }
    }
    
    post {
        failure {
            mail to: 'team@company.com',
                 subject: 'Build Failed',
                 body: 'The build has failed. Please check Jenkins.'
        }
    }
}
```

#### 2. Scripted Pipeline 🔧
**특징**: Groovy 기반으로 더 유연하지만 복잡함

```groovy
node {
    try {
        stage('Checkout') {
            git 'https://github.com/user/repo.git'
        }
        
        stage('Build') {
            sh 'npm install'
            sh 'npm run build'
        }
        
        stage('Test') {
            sh 'npm test'
        }
        
        if (env.BRANCH_NAME == 'main') {
            stage('Deploy') {
                sh 'docker build -t myapp:latest .'
                sh 'docker push myapp:latest'
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        mail to: 'team@company.com',
             subject: 'Build Failed',
             body: "Build failed: ${e.message}"
        throw e
    }
}
```

### Pipeline 핵심 구성요소

#### Agent 🤖
```groovy
// 모든 사용 가능한 에이전트에서 실행
agent any

// 특정 레이블을 가진 에이전트에서 실행
agent { label 'linux' }

// Docker 컨테이너에서 실행
agent {
    docker {
        image 'node:16'
    }
}
```

#### Stages & Steps 📊
```groovy
stages {
    stage('Build') {
        steps {
            sh 'make build'
            archiveArtifacts artifacts: 'build/**', fingerprint: true
        }
    }
}
```

#### When 조건 🎯
```groovy
stage('Deploy to Production') {
    when {
        allOf {
            branch 'main'
            not { changeRequest() }
        }
    }
    steps {
        sh 'deploy-to-prod.sh'
    }
}
```

---

## 실습 예제

### 1단계: 준비물 챙기기 🎒

본격적인 실습 전에 몇 가지 프로그램을 먼저 설치해야 합니다.

1. **Git 설치:** 내 컴퓨터에 Git이 없다면 [여기](https://git-scm.com/download/win)에서 다운로드하여 설치합니다.
2. **Docker Desktop 설치:** 젠킨스를 가장 깔끔하고 쉽게 설치하는 방법입니다. [Docker Desktop 공식 사이트](https://www.docker.com/products/docker-desktop/)에서 다운로드하여 설치하고 실행해 주세요. (설치 후 컴퓨터 재시작이 필요할 수 있습니다.)
3. **GitHub 계정:** 실습용 코드를 올릴 GitHub 계정이 필요합니다.
4. **IntelliJ:** 이미 준비되셨군요!

---

### 2단계: 젠킨스 설치 및 실행 🐳

이제 Docker를 이용해 젠킨스를 설치(실행)하겠습니다.

1. Windows에서 **PowerShell** 또는 **터미널(cmd)**을 엽니다.

2. 아래 명령어를 복사하여 붙여넣고 실행합니다. 젠킨스 최신 안정화(LTS) 버전 이미지를 다운로드하고 실행하는 명령어입니다.

   ```bash
   docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   ```

3. **젠킨스 초기 설정:**

   - 웹 브라우저에서 `http://localhost:8080` 으로 접속합니다.
   - 'Unlock Jenkins' 화면이 나오면, 초기 비밀번호를 입력해야 합니다. 아래 명령어를 터미널에 입력하여 비밀번호를 확인하고 복사합니다.
     ```bash
     docker logs jenkins
     ```
     (로그 중간에 긴 문자열로 된 비밀번호가 보입니다.)
   - 비밀번호를 붙여넣고 **[Continue]** 를 누릅니다.
   - 'Customize Jenkins' 화면에서는 **[Install suggested plugins]** 를 선택합니다. 자동으로 추천 플러그인들이 설치됩니다.
   - 설치가 끝나면, 관리자 계정을 생성하는 화면이 나옵니다. 사용할 아이디, 비밀번호, 이름, 이메일 주소를 입력하고 **[Save and Continue]** → **[Save and Finish]** → **[Start using Jenkins]** 를 차례로 누릅니다.

이제 젠킨스 대시보드가 보이면 설치와 초기 설정이 모두 끝난 것입니다!

---

### 3단계: 실습용 프로젝트 생성 및 GitHub 연동 💻

이제 젠킨스가 빌드할 간단한 프로젝트를 인텔리제이에서 만들고 GitHub에 올리겠습니다.

1. **인텔리제이에서 새 프로젝트 생성:**

   - **[File]** > **[New]** > **[Project]** 를 선택합니다.
   - 'Node.js' 프로젝트를 선택하고, 프로젝트 이름을 `jenkins-practice` 와 같이 정한 후 생성합니다.

2. **간단한 파일 2개 만들기:**

   - `package.json` 파일을 열어 아래와 같이 `scripts` 부분을 수정합니다. 테스트 명령어를 정의하는 것입니다.
     ```json
     "scripts": {
       "test": "echo 'Test successful!' && exit 0"
     },
     ```
   - 프로젝트 루트에 `Jenkinsfile` 이라는 새 파일을 만들고 아래의 간단한 파이프라인 코드를 붙여넣습니다.
     ```groovy
     pipeline {
         agent any

         stages {
             stage('Hello') {
                 steps {
                     echo 'Hello World from Jenkins Pipeline!'
                 }
             }
             stage('Test') {
                 steps {
                     // package.json에 정의한 test 스크립트 실행
                     // bat: 윈도우 터미널 명령어 실행
                     bat 'npm test'
                 }
             }
         }
     }
     ```

3. **GitHub에 프로젝트 올리기:**

   - GitHub 사이트에서 `jenkins-practice` 라는 이름의 새 저장소(New repository)를 만듭니다. (Public으로 만드세요.)
   - 인텔리제이 터미널 또는 Git Bash를 열어 아래 명령어를 순서대로 실행하여 코드를 GitHub에 푸시(push)합니다.
     ```bash
     git init
     git add .
     git commit -m "Initial commit"
     git branch -M main
     git remote add origin https://github.com/YOUR_USERNAME/jenkins-practice.git
     git push -u origin main
     ```
     (`YOUR_USERNAME` 부분은 본인의 GitHub 아이디로 변경하세요.)

---

### 4단계: 젠킨스 파이프라인 연결 및 실행 🚀

이제 젠킨스에게 우리가 만든 GitHub 프로젝트를 알려주고 실행시켜 보겠습니다.

1. 젠킨스 대시보드(`http://localhost:8080`)로 돌아와 왼쪽 메뉴에서 **[New Item]** 을 클릭합니다.

2. **'Enter an item name'** 에 `jenkins-practice` 라고 입력하고, **[Pipeline]** 을 선택한 후 **[OK]** 를 누릅니다.

3. 설정 페이지가 나오면 스크롤을 맨 아래로 내려 **'Pipeline'** 섹션으로 갑니다.

   - **Definition:** 드롭다운 메뉴를 **[Pipeline script from SCM]** 으로 변경합니다.
   - **SCM:** **[Git]** 을 선택합니다.
   - **Repository URL:** 방금 만든 GitHub 저장소의 주소 (`https://github.com/YOUR_USERNAME/jenkins-practice.git`)를 붙여넣습니다.
   - **Branch Specifier:** `*/main` 이라고 되어 있는지 확인합니다.

4. **[Save]** 버튼을 눌러 설정을 저장합니다.

5. 프로젝트 페이지 왼쪽 메뉴에서 **[Build Now]** 를 클릭합니다.

---

### 5단계: 결과 확인하기 ✅

잠시 후 'Build History' 아래에 `#1` 빌드가 나타납니다. 빌드 번호 옆에 초록색 체크 표시가 뜨면 성공입니다!

- **Stage View:** 대시보드에서 파이프라인의 각 단계(`Hello`, `Test`)가 초록색으로 표시되는 것을 볼 수 있습니다.
- **로그 확인:** 빌드 번호 **`#1`** 을 클릭한 후, 왼쪽 메뉴에서 **[Console Output]** 을 클릭하면 파이프라인이 실행된 상세한 로그를 확인할 수 있습니다. `Hello World...` 와 `Test successful!` 메시지가 보일 겁니다.

여기까지 따라오셨다면, 윈도우 환경에서 젠킨스를 설치하고 GitHub 프로젝트와 연동하여 첫 파이프라인을 성공적으로 실행한 것입니다! 축하합니다!

---

## 보안 및 Best Practices

### 보안 Best Practices (2025년)

#### 1. 인증 및 권한 관리 🔐

```groovy
// Matrix 기반 권한 설정 예시
// Jenkins → Manage Jenkins → Configure Global Security

// 사용자별 권한 매트릭스
// - Admin: 모든 권한
// - Developer: Job 관련 권한
// - Viewer: 읽기 전용 권한
```

#### 2. 자격 증명 관리 🗝️
```groovy
// Jenkinsfile에서 자격증명 사용
pipeline {
    agent any
    
    environment {
        // 자격증명을 환경 변수로 안전하게 사용
        DOCKER_CREDS = credentials('docker-hub-credentials')
        AWS_CREDS = credentials('aws-credentials')
    }
    
    stages {
        stage('Deploy') {
            steps {
                // 자격증명은 마스킹되어 로그에 표시되지 않음
                sh 'docker login -u $DOCKER_CREDS_USR -p $DOCKER_CREDS_PSW'
                sh 'aws configure set aws_access_key_id $AWS_CREDS_USR'
            }
        }
    }
}
```

#### 3. 플러그인 보안 ⚠️
- **최소 권한 원칙**: 필요한 플러그인만 설치
- **정기 업데이트**: 보안 취약점 해결
- **신뢰할 수 있는 소스**: 공식 플러그인 사용

#### 4. 네트워크 보안 🌐
```bash
# 방화벽 설정 (Linux 예시)
sudo ufw allow 8080/tcp  # Jenkins 웹 인터페이스
sudo ufw allow 50000/tcp # 에이전트 통신
sudo ufw enable
```

### 개발 Best Practices

#### 1. Pipeline 설계 원칙 📐
- **단순성**: 복잡한 Groovy 스크립트 지양
- **재사용성**: 공유 라이브러리 활용
- **가독성**: 명확한 단계 이름과 주석

#### 2. 에러 처리 🚨
```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                script {
                    try {
                        sh 'npm run build'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Build failed: ${e.message}"
                    }
                }
            }
        }
    }
    
    post {
        failure {
            // 실패 시 알림
            emailext (
                subject: "Build Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Build failed. Check Jenkins for details.",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

#### 3. 성능 최적화 ⚡
```groovy
pipeline {
    agent any
    
    options {
        // 병렬 실행 최적화
        parallelsAlwaysFailFast()
        // 빌드 히스토리 관리
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // 타임아웃 설정
        timeout(time: 30, unit: 'MINUTES')
    }
    
    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }
    }
}
```

---

## 2025년 트렌드

### 현재 상황
- **여전히 강력**: CI/CD 시장의 44% 점유율 유지
- **커뮤니티 활발**: 2025년 커뮤니티 어워드 수상
- **새로운 도전**: 클라우드 네이티브 환경에서의 경쟁

### 주요 트렌드

#### 1. 클라우드 통합 ☁️
```groovy
// Kubernetes 배포 예시
pipeline {
    agent {
        kubernetes {
            yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: kubectl
                    image: bitnami/kubectl
                    command:
                    - sleep
                    args:
                    - "3600"
            """
        }
    }
    
    stages {
        stage('Deploy to K8s') {
            steps {
                container('kubectl') {
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }
}
```

#### 2. AI/ML 파이프라인 지원 🤖
```groovy
pipeline {
    agent any
    
    stages {
        stage('Train Model') {
            steps {
                sh 'python train_model.py'
                archiveArtifacts artifacts: 'model/**', fingerprint: true
            }
        }
        
        stage('Model Validation') {
            steps {
                sh 'python validate_model.py'
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'model_performance.html',
                    reportName: 'Model Performance Report'
                ])
            }
        }
    }
}
```

#### 3. 보안 강화 🔒
- **컨테이너 보안 스캔**
- **의존성 취약점 검사**
- **SAST/DAST 통합**

### Jenkins의 미래
- **장점**: 풍부한 플러그인, 강력한 커뮤니티, 유연성
- **과제**: 클라우드 네이티브 환경 적응, 설정 복잡성
- **대안**: GitHub Actions, GitLab CI, CircleCI 등과의 경쟁

---

## 마무리

### Jenkins를 선택해야 하는 경우
- 🏢 **복잡한 엔터프라이즈 환경**
- 🔧 **높은 커스터마이징 요구사항**
- 💰 **예산 제약** (오픈소스)
- 🔌 **다양한 도구 통합 필요**

### 주의사항
- 초기 설정과 학습 곡선이 steep
- 플러그인 관리와 보안 주의 필요
- 클라우드 네이티브 환경에서는 다른 대안 고려

### 핵심 포인트
Jenkins는 2025년에도 여전히 강력한 CI/CD 도구입니다. 하지만 현대적인 클라우드 환경에서는 설정의 복잡성과 관리 부담을 고려하여 선택해야 합니다. 팀의 요구사항과 인프라 환경에 맞는 최적의 도구를 선택하는 것이 중요합니다.