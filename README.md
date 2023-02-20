# file-integrity-manager
이 Repository는 PCI-DSS에서 요구하는 일일 파일 무결성 점검을 수행하는 스크립트 저장소입니다.  
file-integrity.sh를 각 인스턴스에 배포하여 크론잡으로 실행하면 변경된 파일명과 내용을 Slack으로 실시간 alert을 줍니다.  

# Usage
Usage: [-a|d] [files..]  
-a 옵션은 무결성을 체크할 파일을 등록하는 옵션입니다. 해당 옵션으로 파일 등록 시 `file_list` 라는 전역변수에 기록됩니다.  
`./file-integrity.sh -a /etc/my.cnf` # append file  
`./file-integrity.sh -a "/etc/my.cnf`  /etc/passwd" # append multiple file  
  
-d 옵션은 `file_list` 전역변수에 등록된 파일을 제거하는 옵션입니다.(무결성 체크를 하지 않겠다는 의미)  
`./file-integrity.sh -d /etc/my.cnf`   
`./file-integrity.sh -d "/etc/my.cnf /etc/passwd"`  

따로 실행 옵션을 선언하지 않고, 스크립트 파일에서 직접 `file_list`변수에 무결성을 점검할 파일을 명시할 수 있습니다.  

### Slack Webhook 설정
스크립트 파일에서 `slack_webhook`변수에 alert을 받을 slack 채널의 Webhook URL을 지정해야 합니다.  

# 배포방법  
### **step 1**  
사용자를 root로 전환시키고 해당 스크립트를 /root/ 디렉토리로 이동시킵니다.  

### **step 2**  
해당 스크립트의 owner와 group을 root로 그리고 퍼미션은 700으로 변경합니다.  
이렇게 하는 이유는 스크립트를 실행해서 떨궈지는 결과들에 대해 실수로 인한 삭제 또는  
고의적인 손상 으로부터 보호하기 위해 chattr -i(immutable) 속성을 주입했기 때문입니다.    
chattr은 root 권한으로 실행 가능  

`chown root ./file_integrity_checker.sh`  
`chgrp root ./file_integrity_checker.sh`  
`chmod 700 ./file_integrity_checker.sh`  

### **step 3**  
root 권한으로 crontab -e 등록을 수행합니다.  
수행 주기는 일 1회를 권장하지만, 기호에 따라 시간별로 돌려도 됨.  

`0 2 * * * bash -c '/root/file_integrity_checker.sh 2>&1 | tee /root/result_log/result_log.$(date +\%Y-\%m-\%d\_\%H:\%M:\%S -d "9 hour")'`  


### **step 4**  
크론 재시작  
`systemctl restart cron.service`  