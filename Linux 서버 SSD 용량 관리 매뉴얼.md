
## 1. 디스크 사용량 모니터링

### 1.1 df 명령어 사용
```bash
df -h
```
- `-h` 옵션은 사람이 읽기 쉬운 형식으로 출력합니다.

### 1.2 du 명령어 사용
```bash
du -sh /path/to/directory
```
- `-s`: 요약 정보만 표시
- `-h`: 사람이 읽기 쉬운 형식으로 출력

## 2. 주요 디렉토리 관리

### 2.1 /var/log 관리
```bash
# 로그 파일 압축
find /var/log -type f -name "*.log" -mtime +7 -exec gzip {} \;

# 오래된 압축 로그 삭제
find /var/log -type f -name "*.gz" -mtime +30 -delete
```

### 2.2 /tmp 디렉토리 정리
```bash
# 7일 이상 된 파일 삭제
find /tmp -type f -atime +7 -delete
```

### 2.3 /var/cache 관리
```bash
# APT 캐시 정리 (Ubuntu/Debian)
apt-get clean

# YUM 캐시 정리 (CentOS/RHEL)
yum clean all
```

### 2.4 홈 디렉토리 정리
```bash
# 큰 파일 찾기
find /home -type f -size +100M
```

## 3. Virtual Environment 관리

### 3.1 Conda 환경 관리

#### 사용하지 않는 환경 제거
```bash
# 환경 목록 확인
conda env list

# 환경 제거
conda env remove --name 환경이름
```

#### 캐시 정리
```bash
conda clean --all
```

#### 패키지 정리
```bash
# 사용하지 않는 패키지 제거
conda clean --packages
```

### 3.2 Python venv 관리

#### 오래된 venv 정리
```bash
# 30일 이상 된 venv 디렉토리 찾기
find /path/to/venv/directory -maxdepth 1 -type d -mtime +30

# 필요없는 venv 삭제
rm -rf /path/to/venv/환경이름
```

### 3.3 Node.js nvm 환경 관리
```bash
# 설치된 버전 확인
nvm ls

# 사용하지 않는 버전 제거
nvm uninstall 버전번호
```

## 4. 디스크 공간 확보 팁

- 패키지 캐시 정리: `apt-get clean` (Ubuntu/Debian)
- 오래된 커널 제거: `apt-get autoremove` (Ubuntu/Debian)
- 압축: `tar -cvzf archive.tar.gz /path/to/directory`
- 중복 파일 찾기: `fdupes -r /path/to/search`

## 5. 자동화

### 5.1 cron job 설정
```bash
0 2 * * * /path/to/cleanup_script.sh
```
- 매일 새벽 2시에 정리 스크립트를 실행합니다.

## 6. 모니터링 도구

- iotop: I/O 사용량 모니터링
- htop: 시스템 리소스 모니터링
- ncdu: 디스크 사용량 분석

정기적으로 이 가이드를 따라 SSD 용량을 관리하면 서버의 성능과 안정성을 유지할 수 있습니다.

## 7. 정기적인 검토 및 정리

- 월간 검토: 대용량 파일 및 디렉토리 확인
- 분기별 검토: 사용하지 않는 virtual environment 및 패키지 정리
- 반기별 검토: 전체 시스템 스캔 및 대규모 정리

이 가이드를 따라 정기적으로 SSD 용량을 관리하면 서버의 성능과 안정성을 유지할 수 있습니다. Virtual environment와 주요 시스템 디렉토리를 꾸준히 관리하는 것이 중요합니다.