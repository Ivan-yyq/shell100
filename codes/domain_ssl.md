#!/bin/bash
#监控公司域名证书到期发送钉钉消息通知
#作者：小杨
#日期：2018/12/06
#版本：v0.2

#自定义机器人参考文档说明：https://open-doc.dingtalk.com/docs/doc.htm?spm=a219a.7629140.0.0.aS1CPQ&treeId=257&articleId=105735&docType=1
#定义艾特的人
PHONE="15xxxxxxxxx"
#定义钉钉群机器人的Token
TOKEN="https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxx"


#域名存放路径
dir="/tmp/domain_ssl.txt"
for yuming in `cat $dir/domain_ssl.txt` #读取存储了需要监测的域名的文件
do
    END_TIME=$(echo | openssl s_client -servername $yuming  -connect $yuming:443 2>/dev/null | openssl x509 -noout -dates |grep 'After'| awk -F '=' '{print $2}'| awk -F ' +' '{print $1,$2,$4 }')
    #使用openssl获取域名的证书情况，然后获取其中的到期时间
    END_TIME1=$(date +%s -d "$END_TIME") #将日期转化为时间戳
    NOW_TIME=$(date +%s -d "$(date | awk -F ' +'  '{print $2,$3,$6}')") #将目前的日期也转化为时间戳

    NEW_TIME=$(($(($END_TIME1-$NOW_TIME))/(60*60*24))) #到期时间减去目前时间再转化为天数

    if [ $NEW_TIME -lt 30 ];  #当到期时间小于30天时，发钉钉群消息告警
    then
        curl -H "Content-Type:application/json" -X POST --data '{"msgtype":"text","text":{"content":"域名：'$yuming'   赛门铁克免费证书到期日期，剩余：'$NEW_TIME' 天"} , "at": {"atMobiles": ['${PHONE}'], "isAtAll": false}}' ${TOKEN} > /dev/null 2>&1
    fi
done