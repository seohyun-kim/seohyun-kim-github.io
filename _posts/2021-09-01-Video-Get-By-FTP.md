---
layout: single
title:  "NAS 서버에서 DELL 서버로 주기적으로 비디오 전송"
author: seohyun-kim
date: 2021-09-01 9:00
comments: true
---

## ftp로 NAS서버에서 .avi 파일을 받아와 저장하고, .m4v 파일로 변환  

1. (실시간 영상처리가 완료된) 모든 영상 **삭제**
2. 현재 시간으로부터 **5분 전에** 해당하는 영상을 가져옴
3. 저장 된 .avi 파일을 .m4v 확장자로 **변환**     
<br>  

#### 쉘 스크립트 코드  
```
#!/bin/sh

# 변수선언
MYHOME=/home/malab/workspace/ftp
SBIN_DIR=${MYHOME}/sbin
LOG_DIR=${MYHOME}/log
DOWNLOAD_DIR=${MYHOME}/video
ftp_script_file=${SBIN_DIR}/ftpscript.pcl

# 현재 시간 추출
DATE=`date +%Y-%m-%d`
TIME=`date +%H-%M-`
TIME_5M=`date -d '5 minute ago' +%H-%M-`
HOUR=`date +%H`

# 존재하는 비디오(5분이상 지난) 모두 삭제
rm  ${DOWNLOAD_DIR}/*

USER=
PASSWD=
TARGET_IP=
TARGET_DIR=교내_CCTV_20210107/record_nvr/channel1/${DATE}/${HOUR}
# (ex: TARGET_DIR=교내_CCTV_20210107/record_nvr/channel1/2021-07-26/14 )

# 파일에 따라 0초인 것도 있고 1초인 것도 있었음
FILE_NAME1="${DATE} ${TIME}00~${TIME_5M}00.avi"
FILE_NAME2="${DATE} ${TIME}01~${TIME_5M}01.avi"

# 파일명 예시
#FILE_NAME1="2021-07-26 14-00-00~14-05-00.avi"
#FILE_NAME2="2021-07-26 14-00-01~14-05-01.avi"


if [ -r $ftp_script_file ]
then
	rm -rf $ftp_script_file
fi



cat << FTPSCRIPT_EOF > ${ftp_script_file}
user ${USER} ${PASSWD}
prompt

# Write FTP command hear
bi
cd ${TARGET_DIR}
lcd ${DOWNLOAD_DIR}
get "${FILE_NAME1}" "$FILE_NAME1"
get "${FILE_NAME2}" "$FILE_NAME2"
# (ex. "2021-07=25 14-00-00~14-05-00.avi")
lcd ${MYHOME}

bye
FTPSCRIPT_EOF

echo "target system: ${USER}, ${TARGET_IP}"
ftp -n ${TARGET_IP} < ${ftp_script_file}
# if log command

# m4v파일로 변환
rm -rf $ftp_script_file
ffmpeg -i "${DOWNLOAD_DIR}/${FILE_NAME1}" -vcodec libx264 -an "${DOWNLOAD_DIR}/${FILE_NAME1%.*}.m4v" || ffmpeg -i "${DOWNLOAD_DIR}/${FILE_NAME2}" -vcodec libx264 -an "${DOWNLOAD_DIR}/${FILE_NAME2%.*}.m4v"
echo "${FILE_NAME%.*}.m4v"

# cat << complete > ${LOG_DIR}/log.txt

```

## crontab 으로 5분마다 실행 
10초 딜레이

```
*/5 8-18 * * * sleep 10; /home/malab/workspace/ftp/sbin/download.sh
```
<br>  


### 파일 위치  

` 비디오 저장 위치 ` : /home/malab/workspace/ftp/video  
` 쉘 스크립트 파일 ` : /home/malab/workspace/ftp/sbin/download.sh  
