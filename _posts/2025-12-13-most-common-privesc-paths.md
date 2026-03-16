---
title: 리눅스 시스템의 권한 상승 경로
date: 2025-12-13 13:30:00 +0900
tags: HTB CTF 시스템해킹
---

| 분류            | 체크 대상          | 확인 명령 / 위치                          | 타깃                   | 비고                          |
| ------------- | -------------- | ----------------------------------- | ------------------------- | ------------------------------ |
| **계정/권한**     | 현재 사용자         | `id`, `whoami`, `groups`            | 특수 그룹(docker, lxd, adm 등) | 소속된 그룹 확인             |
|               | 사용자 목록         | `/etc/passwd`                       | 중간 계정 존재                  | 홈 디렉터리 있는 계정 우선               |
|               | 비밀번호 재사용       | `su - user`, `ssh user@localhost`   | 웹/DB 패스워드 재사용             | 크레덴셜 스터핑                  |
| **sudo**      | sudo 권한        | `sudo -l`                           | NOPASSWD, 특정 바이너리         | 가장 먼저 실행해 볼 것                     |
|               | sudo PATH      | `sudo -l` 출력의 `secure_path`         | PATH Hijacking            | 절대경로를 사용하고 있지 않은 경우        |
|               | sudo 버전        | `sudo --version`                    | sudo 자체 취약점               | CVE 존재 여부 확인   |
| **cron**      | 시스템 cron       | `/etc/cron*`                        | root가 실행하는 스크립트           | 쓰기 가능한지 여부 확인               |
|               | 사용자 cron       | `crontab -l`                        | 자동 실행 로직                  | 환경변수/경로 오염                    |
|               | 로그             | `/var/log/syslog`, `grep CRON`      | 실제 실행 파일 추적               | 실행 주기 및 경로 확인                   |
| **SUID**      | SUID 바이너리      | `find / -perm -4000`                | GTFOBins 가능 여부            | 흔하지 않은 값 위주 확인              |
| **Capabilities** | 바이너리 Capabilities | `getcap -r / 2>/dev/null`        | cap_setuid, cap_dac_override 등 | SUID 대신 확인해 볼 만한 위치 |
| **서비스**       | systemd        | `/etc/systemd/system/*.service`     | ExecStart 경로              | 스크립트 쓰기 가능 여부                 |
|               | 실행 중 서비스       | `ps aux`, `systemctl list-units`    | root 서비스                  | 설정 파일 권한 확인                   |
| **파일 권한**     | 쓰기 가능한 파일      | `find / -writable -type f`          | root 실행 파일                | 백업/스크립트 주의                    |
|               | /etc/passwd 쓰기 | `ls -la /etc/passwd`                | 직접 root 계정 추가             | 해시 직접 삽입이 가능한지 확인            |
|               | 디렉터리           | `/opt`, `/srv`, `/var/backups`      | 관리 스크립트                   | 핵심 디렉터리                      |
| **환경 변수**     | PATH           | `echo $PATH`                        | 명령어 가로채기                  | 상대경로 호출 노림                    |
|               | LD 변수          | `env`                               | 라이브러리 하이재킹                | 환경 변수 노출                    |
| **패키지/업데이트**  | apt 기록         | `/var/log/apt/history.log`          | postinst 스크립트             | needrestart 확인            |
| **홈 디렉터리**    | 설정 파일          | `~/.bashrc`, `.profile`             | 자동 실행                     | 시스템 계정이 해당 파일을 읽는 경우            |
|               | SSH            | `~/.ssh/`                           | 키 재사용                     | 시스템 권한으로 재로그인 가능                 |
| **네트워크/컨테이너** | docker         | `groups`                            | docker.sock               | 컨테이너에서 시스템 권한 획득 가능         |
|               | lxd/lxc        | `groups`                            | privileged 컨테이너           | 컨테이너에서 시스템 권한 획득 가능         |
|               | 내부 서비스         | `ss -tlnp`, `netstat -tlnp`         | localhost 전용 서비스           | MySQL/Redis/Jenkins 무인증 여부 확인 |
|               | NFS            | `cat /etc/exports`                  | no_root_squash 설정         | 공격자 측에서 SUID 바이너리 심기 가능       |
| **커널**        | 커널 버전          | `uname -a`                          | 로컬 익스플로잇                  | CVE 존재 여부 확인                       |
| **로그 기반 힌트**  | auth/syslog    | `/var/log/auth.log`                 | sudo/cron 흔적              | 실제 실행 사용자 확인              |