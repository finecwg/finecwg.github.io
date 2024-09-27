## 1. 사전 준비

### 1.1 Git 설치
```bash
sudo apt update
sudo apt install git
```

### 1.2 GitHub 계정 설정
1. GitHub 계정이 없다면 생성합니다.
2. GitHub에서 새 리포지토리를 생성합니다.

### 1.3 SSH 키 생성 및 GitHub에 등록
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
cat ~/.ssh/id_ed25519.pub
```
생성된 공개키를 GitHub 계정 설정의 SSH keys 섹션에 추가합니다.

## 2. 백업 설정

### 2.1 백업할 디렉토리 선택
백업하려는 디렉토리 경로를 확인합니다. 예: `/path/to/backup`

### 2.2 Git 리포지토리 초기화
```bash
cd /path/to/backup
git init
```

### 2.3 .gitignore 파일 설정
```bash
nano .gitignore
```
백업에서 제외할 파일이나 디렉토리를 추가합니다.

### 2.4 GitHub 리포지토리 연결
```bash
git remote add origin git@github.com:username/repository.git
```

## 3. 백업 프로세스

### 3.1 변경 사항 스테이징
```bash
git add .
```

### 3.2 변경 사항 커밋
```bash
git commit -m "Backup: $(date +"%Y-%m-%d %H:%M:%S")"
```

### 3.3 GitHub에 푸시
```bash
git push -u origin main
```

## 4. 자동화

### 4.1 백업 스크립트 생성
```bash
nano /path/to/backup_script.sh
```

스크립트 내용:
```bash
#!/bin/bash
cd /path/to/backup
git add .
git commit -m "Automated backup: $(date +"%Y-%m-%d %H:%M:%S")"
git push origin main
```

### 4.2 스크립트 실행 권한 부여
```bash
chmod +x /path/to/backup_script.sh
```

### 4.3 Cron 작업 설정
```bash
crontab -e
```

다음 줄을 추가하여 매일 자정에 백업 실행:
```
0 0 * * * /path/to/backup_script.sh
```

## 5. 보안 고려사항

- 민감한 정보는 .gitignore에 추가하여 백업에서 제외합니다.
- 필요한 경우 git-crypt나 비슷한 도구를 사용하여 중요 파일을 암호화합니다.
- GitHub 리포지토리를 private으로 설정합니다.

## 6. 복원 프로세스

백업된 데이터를 복원해야 할 경우:

```bash
git clone git@github.com:username/repository.git /path/to/restore
```

## 7. 주의사항

- 대용량 파일이나 빈번하게 변경되는 파일은 Git 성능에 영향을 줄 수 있으므로 주의가 필요합니다.
- 정기적으로 백업 상태를 확인하고 필요한 경우 수동으로 백업을 실행합니다.

이 매뉴얼을 따라 Ubuntu 서버의 특정 디렉토리를 GitHub에 백업할 수 있습니다. 정기적인 백업은 데이터 손실 방지와 버전 관리에 도움이 됩니다.