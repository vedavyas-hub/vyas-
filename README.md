#! /usr/bin/ksh
#set -x
#
#
function lastpwchg {
lastupdate=`lsuser -a lastupdate $1 2>/dev/null | awk -F= "{print \\$2}"`; [ -n "$lastupdate" ] && perl -e "print \"Last password change: \". scalar (localtime($lastupdate)) . \"\n\" " || echo "User does not exist or has no last password update attribute";
}

RSP1=0
while [ ${RSP1} != "5" ]
do
#
clear
echo
echo "Epic System RedHat Level 1 User Admin Main Menu for CSL"
echo
echo "1.        Display list of Epic Users "
echo
echo "2.        Re-Set failed login count for an Epic User "
echo
echo "3.        Re-Set failed login count AND password for an Epic User "
echo
echo "4.        Confirm Cache access for an Epic User "
echo
echo "5.        Exit this menu "
echo
echo
echo "Enter an option 1 thru 4, or option 5 Quit: "
echo
read RSP1 ; RSP1=${RSP1}
#
case ${RSP1} in
        1)      # Display list of users
                echo "Displaying list of Epic Users"
                echo
#
echo "Userid  User Name" > $HOME/user.list
echo "------  ---------" >> $HOME/user.list
#
grep ':209:' /etc/passwd | sort | cut -d ":" -f 1,5 | tr ':' '\t' >> $HOME/user.list
#
#
more $HOME/user.list
                ;;
        2)      # Re-Set login count for Epic User
                echo "Re-Set login count for Epic User"
USER1=0
#
while [ ${USER1} != "q" ]
do
#
echo ; echo "Enter the userid to Re-set Login Count for or q to quit: "
#
read USER1 ; USER1=${USER1}
#
if [ ${USER1} ] ; then echo ; else echo "You must type in a userid or q to quit"
USER1="z" ; fi
#
if [ ${USER1} = "z" ] ; then echo ; else
#
if [ ${USER1} = "q" ] ; then echo "Returning to Main Menu" ; else
#
USER2=`grep "^${USER1}:" /etc/passwd | cut -d ":" -f 1`
#
if [ ${USER2} ] ; then echo ; else echo " ${USER1} is not a valid userid"
 USER2="z" ; fi
#
if [ ${USER2} = "z" ] ; then echo ; else
#
USER2A=`grep "^${USER2}:" /etc/passwd | cut -d ":" -f 5 | cut -d " " -f 1`
#
if [ ${USER2A} ] ; then echo ; else USER2A="blank" ; fi
#
UGID=`grep "^${USER2}:" /etc/passwd | cut -d ":" -f 4`
#
if [ ${UGID} != "209" ] ; then echo "You cannot reset a non Epic user"
echo ; else
#
USER3=`grep "^${USER2}:" /etc/passwd | cut -d ":" -f 5`
#
#/usr/sbin/lsuser -a account_locked unsuccessful_login_count ${USER2} | sed "s/unsuccessful/failed/g" | perl -p -e "s/\s+/\n/g"
#lastpwchg ${USER2}  | sed "s/lastupdate/lastupdate (last password update)/g"
last ${USER2}
echo ""
chage -l ${USER2}
echo ""

echo "You entered ${USER2} - User Name: ${USER3}, verify intention "
echo "to reset this user's failed login count (y/n): "
#
read VFYRSP ; VFYRSP=${VFYRSP}
#
if [ ${VFYRSP} ] ; then echo ; else VFYRSP="n" ; fi
#
if [ ${VFYRSP} != "y" ] ; then echo ; else
#
echo "Performing steps to re-set user login count....Please wait"
#
# Resetting the login count - technically
#
CHSEC1="/usr/bin/chsec -f /etc/security/lastlog -a unsuccessful_login_count=0 -s"
HOME2="/home/"
#
#
echo "${CHSEC1} ${USER2}" > ${HOME2}$USER/user.reset
chmod 777 ${HOME2}$USER/user.reset
#
${HOME2}$USER/user.reset
echo  $basename | mailx -s "`hostname` user.seccsl login count reset - `whoami` - ${USER2}" carlos.madrid@inova.org,mustafa.denniz@inova.org,naser.mohammed@inova.org,Vyas.Devaguddi@inova.org

#
fi ; fi ; fi ; fi ; fi ; done
                ;;
        3)    # Re-setting a Password for a Epic User
              echo "Re-setting a Password for a Epic User"
              echo
#
echo "Enter the userid, or q to quit: "
#
read USER1 ; USER1=${USER1}
#
if [ ${USER1} ] ; then echo ; else echo "You must type in a userid"
 USER1="q" ; fi
if [ ${USER1} = "q" ] ; then echo ; echo "Returning to Main Menu" ; else
#
USER2=`grep "^${USER1}:" /etc/passwd | cut -d ":" -f 1`
#
if [ ${USER2} ] ; then echo ; else echo " ${USER1} is not a valid userid"
 USER2="q" ; fi
#
if [ ${USER2} = "q" ] ; then echo ; echo "Returning to Main Menu" ; else
#
UGID=`grep "^${USER2}:" /etc/passwd | cut -d ":" -f 4`
#
if [ ${UGID} != "209" ] ; then echo "You cannot reset a non Epic user"
echo ; echo "Returning to Main Menu" ; else
#
USER3=`grep "^${USER2}:" /etc/passwd | cut -d ":" -f 5`
#
#/usr/sbin/lsuser -a account_locked unsuccessful_login_count ${USER2} | sed "s/unsuccessful/failed/g" | perl -p -e "s/\s+/\n/g"
#lastpwchg ${USER2}  | sed "s/lastupdate/lastupdate (last password update)/g"
echo ""
last ${USER2}
echo ""
chage -l ${USER}
echo ""

#lastpwchg ${USER2}
#lastpwchg ${USER2}  | sed "s/lastupdate/lastupdate (last password update)/g"
echo "You entered ${USER2} - User Name: ${USER3}, verify intention "
echo "to reset this user's failed login count and password (y/n): "
#
read VFYRSP ; VFYRSP=${VFYRSP}
if [ ${VFYRSP} != "y" ] ; then VFYRSP="q" ; else echo ; fi
#
if [ ${VFYRSP} = "q" ] ; then echo ; echo "Returning to Main Menu" ; else
#
echo "Performing password reset on ${USER2} - ${USER3} "
#
/usr/bin/pwdadm ${USER2}
#hard-coding in the failed login count reset as part of password reset (c. madrid, consultant, 2018-10-01)
echo "Performing unsuccessful login count reset on ${USER2} - ${USER3} "
/usr/bin/chsec -f /etc/security/lastlog -a "unsuccessful_login_count=0" -s '${USER2}'
# build chsec command and run
CHSEC1="/usr/bin/chsec -f /etc/security/lastlog -a unsuccessful_login_count=0 -s"
HOME2="/home/"
echo "${CHSEC1} ${USER2}" > ${HOME2}$USER/user.reset
chmod 777 ${HOME2}$USER/user.reset
${HOME2}$USER/user.reset

echo  $basename | mailx -s "`hostname` user.seccsl login count reset & pw reset - `whoami` - ${USER2}" carlos.madrid@inova.org,mustafa.denniz@inova.org
#
fi ; fi ; fi ; fi
#
#
                ;;
        4)      # Display Cache user for environment
                echo "4-Displaying Cache user access for environment"
                echo
#
echo " This option should be used to confirm that an EU has access"
echo "  to a Cache instance."
echo ""
# debug - set -x
echo "Enter user environment (PRD): \c" ; read CSLENV
CSLENVLC=`echo $CSLENV | tr '[A-Z]' '[a-z]'`
echo "Enter user ID: \c" ; read USER2
CSLUSER2LC=`echo $USER2 | tr '[A-Z]' '[a-z]'`
echo ""
echo "Confirming ${CSLUSER2LC} in Epic instance $CSLENV.."
#/usr/local/bin/sudo -Eu epicadm /epic/$CSLENVLC/bin/cusers --list --users $CSLUSER2LC
/usr/local/bin/sudo /epic/$CSLENVLC/bin/cusers --list --users $CSLUSER2LC
echo $basename | mailx -s  "`hostname` user.seccsl cache check - `whoami` - $CSLENVLC - $CSLUSER2LC" carlos.madrid@inova.org,mustafa.denniz@inova.org
# debug off - set +x
echo ""
echo ""
echo "      * * * * * * Press [Enter] to continue... * * * * * * \c\n" ; read anykey
                ;;
        5)    # Exiting this menu
                echo "Exiting "
                RSP1=5
                ;;
        *)      # Invalid option entered
                echo "You must enter an option between 1 and 5 "
                echo
                RSP1=0
                ;;
esac
#
#
clear
done
