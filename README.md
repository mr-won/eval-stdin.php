# eval-stdin.php
what is eval-stdin.php code injection
코드 분석해보니 php 단위 테스트의 eval-stdin.php 경로를 스캔한 뒤 body 값 data에 die 함수를 실행하는 것 그래서 die 스캔이라고도 하는듯
# eval-stdin.php 코드 인젝션 공격 분석 보고서

## 1. 개요
`eval-stdin.php` 코드 인젝션 공격은 PHP의 `eval()` 함수를 이용해 원격 코드 실행(RCE, Remote Code Execution)이 가능한 취약점을 악용하는 공격입니다. 이 공격은 특히 PHP가 직접 표준 입력(stdin)에서 코드를 받아 실행하도록 구성된 환경에서 발생할 수 있습니다.

본 보고서에서는 `eval-stdin.php` 코드 인젝션의 원리, 취약점 분석, 실제 공격 예제, 대응 방법 및 보안 권장 사항을 다룹니다.

## 2. 공격 원리
공격자는 `eval-stdin.php` 파일을 이용하여 PHP 코드를 원격으로 실행할 수 있습니다. 다음과 같은 코드가 존재한다고 가정합니다.

```php
<?php
eval(file_get_contents("php://stdin"));
?>
```

위 코드는 표준 입력(stdin)에서 데이터를 읽어와 `eval()`을 통해 실행합니다. 이 경우, 공격자가 악성 PHP 코드를 전송하면 서버에서 그대로 실행되어 심각한 보안 위협이 발생할 수 있습니다.

### 2.1. 취약점 요약
- **`eval()` 함수 사용**: 동적으로 코드 실행이 가능해 보안에 취약함.
- **표준 입력을 통한 코드 실행**: 외부 입력값이 검증 없이 실행됨.
- **원격 코드 실행 가능성**: 공격자가 원하는 코드를 주입하여 서버를 장악할 수 있음.

## 3. 공격 예제

공격자는 `curl` 또는 `nc`(netcat) 명령어를 이용하여 원격에서 악성 코드를 주입할 수 있습니다.

### 3.1. `curl`을 이용한 공격
```sh
echo '<?php system("ls -la"); ?>' | curl -X POST --data-binary @- http://target.com/eval-stdin.php
```
위 명령어는 `system("ls -la");` 코드를 `eval-stdin.php`에 전달하여 실행시키는 방식입니다.

### 3.2. `nc`(netcat)를 이용한 공격
```sh
echo '<?php system("cat /etc/passwd"); ?>' | nc target.com 80
```
이 코드는 서버에서 `/etc/passwd` 파일을 읽어 공격자에게 반환하도록 합니다.

## 4. 대응 방법

### 4.1. `eval()` 함수 제거
`eval()` 함수는 보안상 매우 위험하므로 사용하지 않는 것이 최선입니다. `eval()`을 대체할 수 있는 방법을 고려해야 합니다.

### 4.2. 표준 입력을 통한 실행 차단
PHP에서 표준 입력을 직접 실행하지 않도록 방어해야 합니다.

```php
<?php
// 입력 데이터 실행 금지
$input = file_get_contents("php://stdin");
if (strpos($input, '<?php') !== false) {
    die("실행이 차단되었습니다.");
}
?>
```

### 4.3. 웹 방화벽(WAF) 적용
웹 방화벽(WAF)을 설정하여 `eval()` 및 기타 위험한 함수의 사용을 차단합니다.

### 4.4. PHP 설정 강화
PHP 설정에서 위험한 함수 실행을 제한할 수 있습니다.

`php.ini` 설정:
```ini
disable_functions = "eval, system, exec, shell_exec, passthru, popen, proc_open"
```

## 5. 결론
`eval-stdin.php` 취약점은 `eval()` 함수와 표준 입력을 결합하여 원격 코드 실행을 가능하게 만드는 심각한 보안 위협입니다. 이와 같은 공격을 방지하려면 `eval()`을 사용하지 않는 코드 작성 습관을 들이고, 보안 설정을 강화해야 합니다. 웹 방화벽과 보안 패치를 적용하는 것도 중요한 대응 방법이 될 수 있습니다.
