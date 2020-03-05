# check_unix_password_age
nagios check for password age on UNIX-like operating systems (AIX, Linux, *BSD, etc)

 NOTES
 -----
  This script is intended to be the all-singing all-dancing nagios check for password age on UNIX-like systems.

 This script exists to warn you about expired passwords, based on your existing password policies.
 For example, if your passwords are configured to expire every 90 days, 
 this script will start alerting when passwords are 91+ days old.

 This script does not make value judgements about your password policy.  
 If your max password age is set to 9999 days, this script is cool with your decision.
 If your passwords are set to never expire, this script will never give you any warnings, unless you use the --maxage=## parameter.

 If you are not certain what the password rules are on each individual machine, but you have an overall corporate policy similar
 to something like "passwords must expire every 90 days", use this syntax:
    check_unix_password_age --maxage=90

 

 SUPPORTED OPERATING SYSTEMS
 ---------------------------
  Tested on AIX 6.1 and 7.1, IBM VIOS (based on AIX 6.1 but config file is at /home/padmin/config/ntp.conf)  
  Tested on CentOS 7, Ubuntu 20.04, Raspbian  
  Tested on FreeBSD 12.1, OpenBSD 6.6, NetBSD 9.0  
  Not yet tested on MacOS, SunOS, HP-UX, MacOS/Darwin (patches welcome)  


 USAGE 
 -----
  This script is executed remotely on a monitored system by the NRPE or check_by_ssh
  methods available in nagios.

  If you are using the check_by_ssh method, you will need a section in the services.cfg
  file on the nagios server that looks similar to the following.
  This assumes that you already have ssh key pairs configured.
    
       define service{
           use                             generic-24x7-service
           hostgroup_name                  all_linux,all_freebsd,all_aix
           service_description             password age
           check_command                   check_by_ssh!"/usr/local/nagios/libexec/check_unix_password_age"
           }

  If you are using the check_nrpe method, you will need a section in the services.cfg
  file on the nagios server that looks similar to the following.
  This assumes that you already have ssh key pairs configured.
  
       define service{
           use                             generic-24x7-service
           host_name                       unix11,unix12,unix13
           service_description             password age
           check_command                   check_nrpe!check_unix_password_age 
           }

  If using NRPE, you will also need a section defining the NRPE command in the /usr/local/nagios/nrpe.cfg file that looks like this:
     command[check_unix_password_age]=/usr/local/nagios/libexec/check_unix_password_age


 Schedule this script to run every 12 hours from the root crontab, which will update a file at /tmp/nagios.check_unix_password_age.tmp
 When this script runs as the low-privileged nagios user, the script will read the contents of /tmp/nagios.check_unix_password_age.tmp 
 Create cron entries in the root user crontab similar to the following:  (one for each vmware host)
   1 1,13 * * * /usr/local/nagios/libexec/check_unix_password_age  1>/dev/null 2>/dev/null





 ASSUMPTIONS
 -----------
  It is assumed that perl is installed on the machine running this script.
     For RHEL / CentOS     yum install perl
     For Debian / Ubuntu   apt install perl
     For FreeBSD           pkg install perl5
     For OpenBSD           (perl should already be in base install)
     For NetBSD            pkg_add install perl5
     For AIX               (perl should already be in base install)

  It is assumed that passwords do have an expiry date (ie 60 days, 90 days, etc)

  It is assumed that this script is being run as a low-privileged user (typically nagios)







 TROUBLESHOOTING
 ---------------
   The first line of this script is #!/usr/bin/perl, which is fine for most UNIX-like operating systems.
   However, FreeBSD puts the perl binary at /usr/local/bin/perl, so please create a symlink on FreeBSD:
      ln -s /usr/local/bin/perl /usr/bin/perl

   This script requires root privileges to run the following commands:
       lsuser ; pwdadm         (AIX)
       cat /etc/shadow         (Linux)
       cat /etc/master.passwd  (FreeBSD,OpenBSD,NetBSD)

   Rather than give the nagios user and additional privileges, this script will run from the root crontab every 12 hours,
   generating a /tmp/nagios.check_unix_password.age.tmp file.  This file will be read by the low-privileged nagios user
   when the check is run from nagios.
   Ensure the root crontab has an entry similar to the following:
   1 1,13 * * * /usr/local/nagios/libexec/check_unix_password_age >/dev/null 2>&1 #nagios helper script 




