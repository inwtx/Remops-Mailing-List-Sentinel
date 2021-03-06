# Remops-Mailing-List-Sentinel
Linux script to notify by email when changes occur to the Remops Mailing List.  
v1.1
```
#!/bin/bash
#
#====================================================================================#
# RemopsMailingList.sh v1.1 (beta)                                                   #
#                                                                                    #
# This script is used to check the Remops Mailing List for new messages.             #
# It is run by a cronjob and will send an email to notify you that the               #
# current month's mailing list has a new entry.                                      #
#                                                                                    #
# Place a notification email address in the 'emailaddress' field below               #
# You will need have wget installed.  Installation: 'apt-get install wget'           #
#                                                                                    #
# Cronjob examples:                                                                  #
# */1 * * * * /path/to/RemopsMailingList.sh &> /dev/null  # cron to run every minute #
# 0 6 * * * /path/to/RemopsMailingList.sh &> /dev/null  # cron to run once a day @ 6 #
#                                                                                    #
#====================================================================================#


filePath=${0%/*}  # current file path


emailaddress=""   # your email address  -  example: emailaddress="you@your.net"


rm $filePath/index.html     # remove any leftover index.html

if [ ! -e $filePath/RemopsMailingList.sav ]; then
   cat /dev/null > $filePath/RemopsMailingList.sav
fi

wget --no-check-certificate http://lists.mixmin.net/pipermail/remops/index.html   # get Remops Mailing List

RMLvar=$(grep -e '\.txt\"' $filePath/index.html)                                  # get top text line  =  <td><A href="2021-March.txt">[ Text 1 KB ]</a></td>
echo "RMLvar = $RMLvar"                                                           # display it
RMLvarMo=$(echo $RMLvar | sed -n 's/.*\-//;s/\..*//p')                            # get month  =  March
echo "RMLvarMo = $RMLvarMo"                                                       # display it

if [[ ! $RMLvarMo == $(date +"%B") ]]; then                                       # if list not current month, have new month
   cat /dev/null > $filePath/RemopsMailingList.sav                                # clear remop text for updated data comparison
fi

RMLvarAddr=$(echo $RMLvar | cut -d'"' -f 2)                                       # get html address  =  2021-March.txt
echo "RMLvarAddr = $RMLvarAddr"                                                   # display it
RMLvarAddr2="http://lists.mixmin.net/pipermail/remops/"$RMLvarAddr                # get Mailing List address  =  http://lists.mixmin.net/pipermail/remops/2021-March.txt
echo "RMLvarAddr2 = $RMLvarAddr2"                                                 # display it

wget --no-check-certificate $RMLvarAddr2                                          # get Remops Mailing List data
sed -i -e :a -e '/^\n*$/{$d;N;ba' -e '}' $RMLvarAddr                              # delete any blank lines at file end  =  2021-March.txt

if [[ $(cksum $RMLvarAddr | awk -F" " '{print $1}') == $(cksum RemopsMailingList.sav | awk -F" " '{print $1}') ]]; then
   echo "wget retrieved Remops Mailing List data = to saved Remops Mailing List data! no email sent."
   else
   echo "wget retrieved Remops Mailing List data not equal to saved Remops Mailing List data! email sent."
   echo "http://lists.mixmin.net/pipermail/remops/index.html" | mail -s "Notice: Remops Mailing List updated!" $emailaddress -r me@myserver.net
   cat $filePath/$RMLvarAddr >> RemopsMailingList.sav                            # save current Remops Mailing List data
fi

rm $filePath/$RMLvarAddr
rm $filePath/index.html

exit 0
  
```

  
