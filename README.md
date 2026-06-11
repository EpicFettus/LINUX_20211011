7주차

기본 디스크 상태와 사용량  
- df -Th

블록 디바이스 트리 구조와 섹터  
- lsblk -f

실제 inode 사용량  
- ls -i 또는 df -i
 
<img width="466" height="153" alt="image" src="https://github.com/user-attachments/assets/ea5163a6-7a43-4e5a-bb5e-3875bf62361a" />

하드 링크 생성  
- ln 원본파일 만들하드링크

심볼릭 링크 생성  
- ln -s 원본파일 만들심볼릭링크

<img width="544" height="210" alt="image" src="https://github.com/user-attachments/assets/a091d921-3c6b-4645-978d-e4d77e934e4e" />

폴더 공간 사용량 (/home)  
- du -ah --max-depth=1 /home | sort -hr

도커 전체 디스크 점유 크기  
- du -sh /var/lib/docker/

도커 컨테이너 폴더 분석  
- ls -F /var/lib/docker/

도커 파일 시스템  
- docker info | grep -q "storage driver"
- Storage Driver: overlay2

마운트 네임스페이스  
- /proc/mounts

컨테이너 로그  
- docker logs 컨테이너이름

<img width="681" height="662" alt="화면 캡처 2026-06-09 234639" src="https://github.com/user-attachments/assets/8208e5e4-2ffb-4b76-a58f-549ceb8e8172" />


9주차  

디스크 추가/LVM 구성  
1. 도커 엔진이 직접 관리

가상 디스크 파일 생성  
- dd if=/dev/zero of=disk_a.img bs=1M count=1024  

루프 디바이스  
- 파일을 물리 디스크로 인식해 마운트 가능

루프 디바이스 생성(연결)  
- losetup -fP --show 가상디스크

루프 디바이스 연결 목록  
- losetup -a
- lsblk (디스크 전체 구조)

PV(디스크장치 파티션된 상태) 생성  
- pvcreate /dev/loop0
- pvs

VG(볼륨 그룹) 생성  
- vgcreate vg_docker /dev/loop0
- vgs

LV(논리적 볼륨) 생성  
- lvcreate -l 100%FREE -n lv_docker vg_docker
- lvs

EXT4 포멧 및 마운트  
- mkfs.ext4 /dev/vg_docker/lv_docker

도커 서비스 중지 및 데이터 이관  
- systemctl stop docker
- mount /dev/vg_docker/lv_docker /mnt/docker_new

기존 데이터 이동 (정밀복사)
- rsync -aHAX /var/lib/docker /mnt/docker_new

가상 디스크 파일 생성 -> 루프 디바이스 생성 -> PV VG LV 생성 -> ext4 포멧 및 마운트 -> 도커 서비스 중지와 데이터 이관 -> 기존 데이터 이동 -> 새 위치로 마운트 지점 변경 -> LV 할당 및 도커 확인 -> 다시 연결

2. fstab 직접 등록  

영구 마운트 (fstab 등록)  
1. mkdir /mnt/...
2. blkid /dev/vg.../lv....
3. uuid 복사
4. nano /etc/fstab
5. UUID=uuid /mnt/... ext4 defaults,noatime 0 2

LV 온라인 확장
lvextend -L +500M /dev/vg.../lv...  
resize2fs /dev/vg.../lv...  

디스크 B 생성 및 PV VG LV 생성 -> 영구 마운트(fstab) 등록 -> MySQL 컨테이너 볼륨 연결 -> 도커 실행 -> LV 온라인 확장  

<img width="338" height="591" alt="image" src="https://github.com/user-attachments/assets/bb1ca600-2675-456c-b7f4-c2b83566c2ee" />


10주차

디스크 연결 확인  
-losetup -a

LVS 상태 확인  
- lvs

도커 상태 확인  
- docker ps

세부 마운트 확인 / 디스크 연결 상태  
- df -h

db 컨테이너 단독 실행/상태  
- docker compose -f dbyaml파일 up -d
- docker compose -f dbyaml파일 ps

db와 wp를 병합 실행/확인/wp초기화로그/정지  
- docker compose -f dbyaml파일 -f wpyaml파일 up -d
- docker compose -f dbyaml파일 -f wpyaml파일 ps
- docker compose -f dbyaml파일 -f wpyaml파일 log -f wordpress
- docker compose -f dbyaml파일 -f wpyaml파일 down


11주차

Portainer  
- 컨테이너 관리

Portainer 접속  
- docker compose -f .yaml파일 up -d portainer

Portainer 사이트  
- Inspect 탭: 컨테이너 상세 JSON
- Stats 탭: CPU/메모리 실시간 그래프

netdata  
- 웹 서버 모니터링
- CPU 사용량 / 메모리 / Disk I/O / 네트워크

nginx stub_status 활성화  
<img width="400" height="156" alt="image" src="https://github.com/user-attachments/assets/38f2cc0a-8c3b-4897-908c-2f00bf187404" />

일반 페이지 부하  
- ab -n 2000 -c 50

이미지 접근 부하  
- ab -t 30 -c 50 이미지주소 &


12주차

<img width="483" height="297" alt="image" src="https://github.com/user-attachments/assets/2ef92faa-6edb-4362-854a-328479e524cf" />

listen 80   
- http

return 301  
- 해당 주소로 보냄
  
listen 443  
- https

<img width="509" height="361" alt="image" src="https://github.com/user-attachments/assets/8065ae5c-c788-45d6-b1f6-7d31a52a1ec7" />

443 포트 추가  
- ports 안 - "8443:443"

인증서 마운트 추가  
- volumes 안 - ./nginx/certs:/etc/nginx/certs:ro

http -> https 리다이렉트  
- curl -I http주소

TLS 1.2 차단 확인  
- openssl s_client -connect localhost:8443 -tls1_2 2>/dev/null | grep "handshake"

TLS 1.3 동작 및 차단 검증  
- openssl s_client -connect localhost:8443 -tls1_3 2>/dev/null | grep "Protocol“

협상된 암호 스위트 확인  
- openssl s_client -connect localhost:8443 -tls1_3 2>/dev/null | grep "Cipher"

lynis
- 시스템 감사

전체 시스템 감사  
- lynis audit system

warning 취약점  
- grep "^warning" /var/log/lynis-report.dat

suggestion 취약점  
- grep "^suggestion" /var/log/lynis-report.dat | head -10

trivy  
- 컨테이너 이미지 취약점 스캔

스캔 (nginx:alpine 이미지)  
- trivy image nginx:alpine


13주차


- 
