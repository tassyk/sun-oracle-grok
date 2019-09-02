# Grok patterns pour Sun [Oracle](https://docs.oracle.com/cd/E19566-01/819-4428/index.html)
# log format: https://docs.oracle.com/cd/E19566-01/819-4428/bgbfk/index.html ou https://msg.wikidoc.info/index.php/MTA_transaction_log_entry_format ou https://docs.oracle.com/cd/E19566-01/819-4428/bgbeu/index.html
# Doc: https://docs.oracle.com/cd/E63708_01/doc.801/e63711/toc.htm

#  Managing Message Store (IMAP/POP) (log file: imap/pop/imta)
##  log samples https://docs.oracle.com/cd/E63708_01/doc.801/e63711/msadm_conf_pop_imap.htm#MSVAG1256)

## Managing Message Store Patterns
MSGSTORE_DATETIMESTAMP %{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} [-+]%{INT}
LOGINDATE %{YEAR}/%{MONTHNUM}/%{MONTHDAY}
LOGINEMAIL (%{EMAILLOCALPART}|%{DATA})@%{HOSTNAME}
LOGID (?:[0-9]+)
LOGINUSER (%{WORD:loginuser}|\[%{WORD:loginuser}\]|%{USERNAME:loginuser}|\[%{USERNAME:loginuser}\]|%{HOSTNAME}|%{LOGINEMAIL})
PROCESS %{PROG}|PopProxy|ImapProxy|AService|(Imap|Pop)ProxyAService.cfg
CLIENTINFO (%{IP:clientip}|%{HOSTNAME:clienthost})(:%{INT:clientport})?
LOGLEVELS ([Dd]ebug|[Nn]otice|[Ii]nfo|[Ww]arn?(?:ing)?|[Ee]rr?(?:or)?|[Cc]rit?(?:ical)?|[Ff]atal|[Ii]nformation)
CATEGORIES ([Gg]eneral|[Aa]ccount|LDAP|[Ll]dap|[Nn]etwork|[Pp]rotocol|[Ss]tats|[Ss]tore)
IMAPEVENTS (close|badlogin|Log created|login|append|fetch|expunge|delete|cleanup|purge|repairing|cannot connect)
IMAPEVENTMESSAGES (Authentication failed|failed to connect|account disabled \(hold\)|starting up|shutting down|Connection refused|Error reading)
AUTHMETHODS plaintext
FUNC_ERROR function=%{DATA}\|port=%{INT}\|error=%{IMAPEVENTMESSAGES}
#1998/12/3 6:54:32 0:00:00 0 115 0 -> (logindate logintime loginduration clientbytes servbytes mailboxes_selected)
LOGINEVENTMSG %{LOGINDATE}\s+%{TIME}\s+(%{TIME:duration}|%{NOTSPACE:duration})\s+%{NUMBER:inbytes}\s+%{NUMBER:outbytes}\s+%{NUMBER:nbmailbox}

#[29/May/2019:17:05:33 +0200] vadar popd[17856]:
MSG_STORE_PREFIX \[?%{MSGSTORE_DATETIMESTAMP:datetime}\] %{SYSLOGHOST:loghost} %{PROCESS:process}(?:\[%{NUMBER:pid}\])?:
#Account Information: login
IMAP_PREFIX %{CATEGORIES:facility} %{LOGLEVELS:loglevel}:\s*(%{IMAPEVENTS:event}(:)?)?
#[29/May/2019:10:44:56 +0200] SiroePost imapd[17849]: General Information: close [192.168.87.4:40581] user3 2019/5/29 10:01:41 0:06:36 12483 52510 1
#[29/May/2019:10:44:56 +0200] hosttr14 imapd[17849]: General Information: Log created (1559119496)
#[04/May/1998:11:07:44 -0400] xyzmail cgi_service[343]: General Error: function=getserverhello|port=2500|error=failed to connect
#[12/Jun/2019:05:16:02 +0200] hoststr14 ims_master[11468]: General Notice: Oracle Communications Messaging Server 7u4-23.01(7.0.4.23.0) 64bit (built Aug 10 2011)) starting up
#[12/Jun/2019:05:13:12 +0200] hoststr14 ims_master[11442]: General Notice: iBiff plugin loaded: ms-internal
#[11/Jun/2019:23:01:19 +0200] hoststr14 imexpire[24489]: General Notice: Expire finished
#[11/Jun/2019:15:15:01 +0200] hoststr14 msprobe[21616]: General Warning: alarmid=diskavail|instance=part143|time=11/Jun/2019:15:15:01 +0200|value=8|low=0|high=94|threshold(below)=10|count below threshold=14|warning sent=272
#[24/Feb/2012:14:47:15 +1100] hostname impurge[17986]: General Error: Could not get purge session lock. Possibly another impurge is running
#[25/Feb/2012:09:55:58 +1100] hostname impurge[7263]: General Notice: repairing user/username
#[05/Jul/2019:16:14:09 +0200] hostmta4 httpd[31129]: General Notice: f0004279[192.168.87.4:48960] created message <29cef698311ec16a.5d1f7751@mydomaine.eu>
#[08/Jul/2019:14:40:56 +0200] hostmta4 httpd[31129]: General Error: Error reading from IMAP server: Timeout
IMAP_GENERAL %{IMAP_GENERAL_CREATED}|%{IMAP_GENERAL_ERROR}|%{IMTA_GENERAL_SRV_STATUS}|%{IMAP_GENERAL_PLUGIN}|%{IMAP_GENERAL_EXPIRATION}|%{IMAP_GENERAL_WARNING}
IMAP_GENERAL_CREATED %{IMAP_GENERAL_LOG_CREATED}|%{IMAP_GENERAL_MSG_CREATED}
IMAP_GENERAL_LOG_CREATED \(%{LOGID:envid}\)
IMAP_GENERAL_MSG_CREATED %{LOGINUSER:loginuser}\[%{CLIENTINFO}\] (?<event>created message) <%{GREEDYDATA:msgid}>
IMAP_GENERAL_ERROR %{IMAP_GENERAL_PURGE_ERROR}|%{IMAP_GENERAL_FUNC_ERROR}|%{IMAP_GENERAL_PURGE_REPAIRING}|%{IMAP_GENERAL_TIMEOUT_ERROR}|%{IMAP_GENERAL_STORE_MESSAGE}|%{IMAP_GENERAL_MAIL_EXCEED}
IMAP_GENERAL_TIMEOUT_ERROR (?<event>Error \w.*) from (?<eventmsg>\w.* Timeout)
IMAP_GENERAL_FUNC_ERROR %{FUNC_ERROR:eventmsg}
IMAP_GENERAL_PURGE_ERROR (?<event>Could not get purge\s* \w.+\b)\s*(?<eventmsg>Possibly\s* \w.+\b)
IMAP_GENERAL_PURGE_REPAIRING (?<event>repairing)\s* %{GREEDYDATA:folder}
IMTA_GENERAL_SRV_STATUS (?<loginfo>Oracle \w.+|Postfix \w.+) %{IMAPEVENTMESSAGES:eventmsg}
IMAP_GENERAL_PLUGIN (?<event>\b\w+ plugin\b loaded):\s*%{GREEDYDATA}
IMAP_GENERAL_EXPIRATION (?<event>Expire \w+\b)\s*(%{GREEDYDATA:eventmsg})?
IMAP_GENERAL_WARNING %{GREEDYDATA:eventmsg}\s* sent=%{INT:msgsize}
#[11/Jul/2019:10:47:25 +0200] hostmta3 ims_master[23402]: General Critical: message store is not enabled!
IMAP_GENERAL_STORE_MESSAGE (?<event>message store) (?<eventmsg>(is)? %{GREEDYDATA})
# [08/Jul/2019:16:14:55 +0200] hostmta4 httpd[31129]: General Error: [192.168.87.4:55773] Compose: 5850272 bytes exceeds maxmessagesize 5242880
IMAP_GENERAL_MAIL_EXCEED \[(%{IP:clientip}|%{HOSTNAME:clienthost})(:%{INT:clientport})?\] (?<event>Compose): (?<eventmsg>%{INT:inbytes} bytes exceeds maxmessagesize %{INT:msgsize})
#[03/Dec/1998:06:54:32 +0200] SiroePost imapd[232]: Account Notice: close [127.0.0.1] [unauthenticated] 1998/12/3 6:54:32 0:00:00 0 115 0
#[30/Aug/2004:16:53:31 -0700] vadar imapd[13027]: Account Notice: badlogin: [192.18.126.64:40720] plaintext user3 account disabled (hold)
#[29/May/2019:08:39:42 +0200] xyzmail imapd[17849]: Account Notice: badlogin: [192.168.87.6:45711] plaintext user1@mydomaine.fr Authentication failed
#[30/Aug/2004:16:53:13 -0700] vadar imapd[13027]: Account Information: login [192.18.126.64:40718] user1 plaintext
#[15/Jul/2019:14:04:56 +0200] messstr13 popd[25234]: Account Notice: close [192.168.87.5:46109] user1@mydomaine.fr 2019/7/15 14:04:56 0:00:00 32 185 1
IMAP_ACCOUNT %{IMAP_ACCOUNT_CLOSE}|%{IMAP_ACCOUNT_BADLOGIN}|%{IMAP_ACCOUNT_LOGIN}
IMAP_ACCOUNT_CLOSE \[(%{CLIENTINFO})?\]\s+%{LOGINUSER:loginuser}\s+%{LOGINEVENTMSG:eventmsg}
IMAP_ACCOUNT_BADLOGIN \[%{CLIENTINFO}\]\s+%{AUTHMETHODS:authmethod}\s+%{LOGINUSER:loginuser}\s+%{IMAPEVENTMESSAGES:eventmsg}
IMAP_ACCOUNT_LOGIN \[%{CLIENTINFO}\]\s+%{LOGINUSER:loginuser}\s+%{AUTHMETHODS:authmethod}
#[31/Aug/2004:16:33:14 -0700] vadar ims_master[13822]: Store Information:append:user1:user/user1:659:<Roam.SIMC.2.0.6.1093995286.11265.user1@vadar.siroe.com>
#[12/Jun/2019:02:00:02 +0200] hoststr14 imdbverify[1731]: Store Notice: Database snapshot finished, duration 3 seconds, destination /opt/sun/comms/messaging64/data/store/dbdata/snapshots/003/
#[25/Feb/2012:09:55:57 +1100] hostname imapd[7264]: Store Critical: Unable to open mailbox user/username: Mailbox does not exist
#[25/Feb/2012:09:55:57 +1100] hostname imapd[7264]: Store Error: user/username: Mailbox has an invalid format; repair requested
IMAP_STORE %{IMAP_STORE_APPEND}|%{IMAP_STORE_SNAPSHOT}|%{IMAP_SOTRE_ERROR}
IMAP_STORE_APPEND (%{USER:loginuser}:%{DATA:folder}:<%{GREEDYDATA:msgid}>|%{DATA:folder}:<%{GREEDYDATA:msgid}>)
IMAP_STORE_SNAPSHOT (?<event>\b\w+ snapshot \w+\b),\s* duration\s* %{SECOND:duration}\s* seconds,\s* destination\s* %{UNIXPATH:folder}
IMAP_SOTRE_ERROR %{IMAP_STORE_UNABLE_MAILBOX}|%{IMAP_STORE_INVALID_MAILBOX}|%{IMAP_STORE_ERROR_SIZE}
IMAP_STORE_UNABLE_MAILBOX (?<event>Unable to open mailbox)\s* %{DATA:mailbox}:\s* %{GREEDYDATA:eventmsg}
IMAP_STORE_INVALID_MAILBOX %{DATA:folder}:\s* (?<eventmsg>Mailbox has an invalid\s* \w.+\b)
#[10/Jul/2019:14:54:12 +0200] hoststr1 imapd[16678]: Store Error: Message file 356 in user/dmatelot/Sent Messages has wrong size
IMAP_STORE_ERROR_SIZE (?<event>Message)\s+(?<eventmsg>file %{INT} in %{DATA:folder} %{GREEDYDATA})

# IMAP/POP Proxy
IMAP_POP_PROXY %{IMAP_POP_PROXY_SOCKET_ERROR}|%{IMAP_POP_PROXY_BADGUY}|%{IMAP_POP_PROXY_EXITING}|%{IMAP_POP_PROXY_MAILSTATUS}|%{IMAP_POP_PROXY_SSL_ERROR}|%{IMAP_POP_PROXY_CONNEC_ERROR}|%{IMAP_POP_PROXY_NOT_FOUND}
IMAP_POP_PROXY_EVENTS client socket IO error|badguy|SSL negotiation failed|Connection limit reached
# [02/Jul/2019:11:41:29 +0200] hostmta5 PopProxy[12607]: General Error: (id 1022591) client socket IO error: Connection reset by peer (104)  
IMAP_POP_PROXY_SOCKET_ERROR \(id %{LOGID:id}\) %{IMAP_POP_PROXY_EVENTS:event}: %{GREEDYDATA:eventmsg}
#[02/Jul/2019:11:26:57 +0200] hostmta5 ImapProxy[12607]: General Notice: (id 1019831) badguy 192.12.87.25 now has 3 badness
IMAP_POP_PROXY_BADGUY \(id %{LOGID:id}\) %{IMAP_POP_PROXY_EVENTS:event} %{IP:clientip} %{GREEDYDATA:eventmsg}
#[24/Jun/2019:10:25:10 +0200] hostmta5 AService[23993]: General Notice: ImapProxy(ImapProxyAService.cfg): exiting
IMAP_POP_PROXY_EXITING (?<event>(Imap|Pop)Proxy\(%{DATA}\)): (?<eventmsg>exiting)
#[11/Jul/2019:10:51:43 +0200] hostamta3 ImapProxy[14580]: General Notice: (id 2067687) user 'mplumentelet@mydomaine.eu' mailUserStatus: 'disabled'
IMAP_POP_PROXY_MAILSTATUS \(id %{LOGID:id}\) user '%{LOGINUSER:loginuser}' (?<event>mailUserStatus): '%{GREEDYDATA:eventmsg}'
#[11/Jul/2019:10:35:40 +0200] hostamta1 ImapProxy[23598]: General Error: (id 1102203) SSL negotiation failed for IP 192.168.87.250: Cannot communicate securely with peer: no common encryption algorithm(s). (-12286)
IMAP_POP_PROXY_SSL_ERROR %{IMAP_POP_PROXY_SSL_NEG_ERROR}
IMAP_POP_PROXY_SSL_NEG_ERROR \(id %{LOGID:id}\) %{IMAP_POP_PROXY_EVENTS:event} for IP %{IP:relayip}: %{GREEDYDATA:eventmsg}
#[16/Jul/2019:08:39:24 +0200] hostamta1 ImapProxy[23598]: General Error: (id 1380833) Connection limit reached for client IP 192.168.87.250
IMAP_POP_PROXY_CONNEC_ERROR \(id %{LOGID:id}\) (?<event>%{IMAP_POP_PROXY_EVENTS}|Connection %{DATA}) for %{WORD} IP %{IP:clientip}
#[15/Jul/2019:20:09:18 +0200] hostamta1 ImapProxy[23598]: General Error: (id 1354640) domain 'syndicat.ac-rennes.fr' not found in LDAP (-1 )
IMAP_POP_PROXY_NOT_FOUND \(id %{LOGID:id}\) %{WORD:event} (?<eventmsg>'%{DATA:fromdomain}' not found %{GREEDYDATA})

IMAP %{MSG_STORE_PREFIX}\s*%{IMAP_PREFIX}\s*(?:%{IMAP_GENERAL}|%{IMAP_ACCOUNT}|%{IMAP_STORE}|%{IMAP_POP_PROXY})

# -------------------------------------------------------------------------------------------------------
# MTA transaction (log file: mail.log)
## log format https://docs.oracle.com/cd/E19566-01/819-4428/6n6j427ll/index.html
## log samples https://msg.wikidoc.info/index.php?title=MTA_transaction_log_entry_format#MTA_transaction_log_entry_format)

## MTA transaction patterns
SOURCE_EMAIL_ADDRESS (%{EMAILLOCALPART:fromuser}|%{DATA:fromuser})@%{HOSTNAME:fromdomain}|%{EMAILLOCALPART:fromuser}
DESTINATION_EMAIL_ADDRESS (%{EMAILLOCALPART:touser}|%{DATA:touser})@%{HOSTNAME:todomain}|%{EMAILLOCALPART:touser}
ORIGINAL_DESTINATION_EMAIL_ADDRESS (%{EMAILLOCALPART}|%{DATA})@%{HOSTNAME}|%{EMAILLOCALPART}
ENVELOPPE_ID (%{DATA}@%{HOSTNAME}|[0-9A-Z]{6,15})
RFC rfc[\d]{3,}
DATESTAMP_RFC2822X %{DAY},\s* %{MONTHDAY} %{MONTH} %{YEAR} %{TIME} %{ISO8601_TIMEZONE}
PROCESS_ID [\h\d]{3,}
THREAD_ID [a-z\d]{1,}
# 2e2d.5.1 -> format: process-id.thread-id.sequence
MTA_PROCESS %{PROCESS_ID}\.%{THREAD_ID}\.%{INT}
EXECUSERNAME mailsrv
SOURCE_CHANNELS tcp_local|tcp_intranet|tcp_submit|ims-ms|(re)?process|bitbucket|tcp_auth|tcp_submit|tcp_tas
DESTINATION_CHANNELS tcp_local|tcp_intranet|tcp_submit|ims-ms|(re)?process|bitbucket|tcp_auth|tcp_submit|tcp_tas
INBOUND_OUTBOUND_TYPE \s*(?<event>[\-\+])\s*
MTAEVENTS (Received|Stored|Delivered|Submitted|Attempted|Rejected|Failed|Too many failures)
ENTRYCODES [BDEJKQRVWZCOUXYIALS]*
SOURCE_SYSTEME (%{HOSTNAME:clienthost}\s*dns;%{HOSTNAME:clienthost}|%{HOSTNAME:clienthost}\s*(\((%{HOSTNAME:clienthost} )?(\[%{IP:clientip}\])?\))?)
MTA_RELAYHOST %{HOSTNAME:relayhost}\s*(\(\[%{IP:relayip}\]\)|\(%{HOSTNAME:relayhost}\s+\[%{IP:relayip}\]\))?
MTA_CLIENTHOST %{HOSTNAME:clienthost}\s*(\(\[%{IP:clientip}\]\)|\(%{HOSTNAME:clienthost}\s+\[%{IP:clientip}\]\))?
MTA_IPV6_CLIENTHOST %{HOSTNAME:clienthost}\s+(?<clientip>\(\[IPv6:::1\]\))
MTA_IPV6_RELAYHOST %{HOSTNAME:relayhost}\s+\(%{HOSTNAME:relayhost}\s+(?<relayip>\[IPv6:::1\])\)
MTA_PREFIX_LONG %{BIND9_TIMESTAMP:datetime}(\s*(%{HOSTNAME:loghost})?(\s*%{MTA_PROCESS:process})?)? (%{SOURCE_CHANNELS:srcchannel})?
MTA_PREFIX_SHORT %{BIND9_TIMESTAMP:datetime}(\s*%{SOURCE_CHANNELS:srcchannel})
MTA_TRANSACTION_PREFIX %{MTA_PREFIX_SHORT}|%{MTA_PREFIX_LONG}
FILTERS systemfilter|ldap_filter|sourcefilter|destinationfilter|disablesourcefilter|log_filter
#TCP|129.153.12.42|25|123.4.5.67|65228 -> Format TCP|MTA-IP|MTA-port|client-IP|client-port
TRANSPORT_INFORMATION \w.+\b\|%{IP}\|%{POSINT}\|%{IP}\|%{POSINT}
#SMTP/domain.com/mail.domain.com/TLS-192-DES-CBC3-SHA  -> Format: SMTP/initial-host/DNS-host/TLS-info
APPLICATION_INFORMATION %{WORD:proto}(\/%{HOSTNAME:clienthost}\/%{HOSTNAME:clienthost})?(\/%{GREEDYDATA:ciphers})?

#04-Sep-2002 01:00:06.49 host.domain.com 1f627.3.3 tcp_local    -            C TCP|129.153.12.42|4303|123.45.6.7|25 SMTP/domain.com/mail.domain.com/TLS-192-DES-CBC3-SHA
MTA_BOUNDS_CONNEXTION %{INBOUND_OUTBOUND_TYPE:event} %{ENTRYCODES:code} %{TRANSPORT_INFORMATION:transinfo} %{APPLICATION_INFORMATION}
MTA_TRANSACTION_CONNEXTION (?:%{MTA_BOUNDS_CONNEXTION})

#05-Jun-2019 10:08:29.74 > Received: from localhost (host11.mydomaine.fr [127.0.0.1]) by host11.mydomaine.fr (Postfix) with ESMTP id AD0BF87703    for <nashat.gnomy@mydomaine.fr>; Wed,  5 Jun 2019 10:08:29 +0200 (CEST)
MTA_RECEIVING %{MTA_RECEIVING_SHORT}|%{MTA_RECEIVING_LONG}|%{MTA_RECEIVING_WITHOUT_RELAY}|%{MTA_RECEIVING_WITH_RELAY}|%{MTA_RECEIVING_SUBMIT}|%{MTA_RECEIVING_IPV6}
MTA_RECEIVING_SHORT > (%{MTAEVENTS:event}:)\s+(from\s* %{MTA_CLIENTHOST})?\s*by\s* %{MTA_RELAYHOST}\s+(\(+%{GREEDYDATA:relayinfo}\)+)?\s+(with\s* %{WORD:proto}?)?\s*id\s* ((%{ENVELOPPE_ID:envid})(\s*for\s* <%{DESTINATION_EMAIL_ADDRESS}>)?;)\s*(%{GREEDYDATA:eventmsg})?
MTA_RECEIVING_LONG > (%{MTAEVENTS:event}:)\s+(from\s* %{MTA_CLIENTHOST})\s*by\s*(%{MTA_RELAYHOST})?\s+(\(+%{GREEDYDATA:relayinfo}\)+)?\s+(with %{WORD:proto})?\s*id\s* (<?%{ENVELOPPE_ID:envid}>?)?(;)?\s*(for\s* <?%{DESTINATION_EMAIL_ADDRESS}>?(\s*\(%{WORD:}\s* %{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\))?;)?\s*(%{GREEDYDATA:eventmsg})?
MTA_RECEIVING_WITHOUT_RELAY > (%{MTAEVENTS:event}:)\s+from\s+%{MTA_CLIENTHOST}\s*\(%{GREEDYDATA:clientinfo}\)\s+id\s+<%{ENVELOPPE_ID:envid}>(;\s*%{GREEDYDATA:eventmsg})?
MTA_RECEIVING_WITH_RELAY > (%{MTAEVENTS:event}:)\s+from\s*%{GREEDYDATA:clienthost}\s+by\s*%{MTA_RELAYHOST}\s*\(%{GREEDYDATA:relayinfo}\)\s+id\s+<%{ENVELOPPE_ID:envid}>(;\s*%{GREEDYDATA:eventmsg})?
#12-Jul-2019 18:00:02.38 > Received: (from root@localhost) by hotstr5.mydomaine.in.fr (8.13.8/8.13.8/Submit) id x6CG02Mj032479; Fri, 12 Jul 2019 18:00:02 +0200
MTA_RECEIVING_SUBMIT > (%{MTAEVENTS:event}:)\s+\(from\s+%{SOURCE_EMAIL_ADDRESS}\)\s+by\s+%{MTA_RELAYHOST}\s+(?<eventmsg>\(%{GREEDYDATA}\/Submit\)\s+id\s+%{DATA:id};\s+%{DATESTAMP_RFC2822}(\s+\(%{WORD}\))?)

MTA_RECEIVING_IPV6 %{MTA_RECEIVING_IPV6_LOCAL}
#12-Jul-2019 16:55:26.78 > Received: from smtps.mydomaine.fr ([IPv6:::1]) by localhost (smtps.mydomaine.fr [IPv6:::1]) (amavisd-new, port 10024) with LMTP id ps_yy8Hr-1vr; Fri, 12 Jul 2019 16:55:24 +0200 (CEST)
MTA_RECEIVING_IPV6_LOCAL > (%{MTAEVENTS:event}:)\s+from\s+%{MTA_IPV6_CLIENTHOST}\s+by\s+%{MTA_IPV6_RELAYHOST}\s+(?<eventmsg>\(%{GREEDYDATA:relayinfo}\)\s+with\s+%{WORD:proto}\s+id\s+%{GREEDYDATA:id};\s+%{DATESTAMP_RFC2822}\s+\(%{WORD}\))


#03-Jun-2019 10:53:22.43 > Message-id: <2.1.11.1101027887.8673165.52284868659@software.avanquest.com>
MTA_MESSAGE_ID_LINE > Message-id:\s* (<%{DATA:msgid}>|%{GREEDYDATA:msgid})

##11-Jun-2019 09:07:57.65 ims-ms                    D 5
MTA_ENQUEUE_ALWAYS_PREFIX (%{DESTINATION_CHANNELS:destchannel})?\s*%{ENTRYCODES:code}\s*%{INT:msgsize}
#sdubois2@ims-ms-daemon mailsrv hostatv11.my-domain.fr
MTA_ENQUEUE_ALWAYS_SUFFIX %{DESTINATION_EMAIL_ADDRESS}\s*%{EXECUSERNAME:execuser}\s*(%{SOURCE_SYSTEME})?
#huber 276 /opt/sun/comms/hostaging64/data/queue/tcp_intranet/ZZi0D4d9f5mwC.00 <01IWFVYLGTS499EC9W@innosoft.com> <01IWFVYLGTS499EC9Y@innosoft.com> mailsrv innosoft.com (innosoft.com [192.160.253.66])
MTA_ENQUEUE_OPTIONS_SUFFIX %{DESTINATION_EMAIL_ADDRESS}\s*(%{INT:dsn})?\s*(%{UNIXPATH:folder})?\s*(<%{DATA:envid}>)?\s*(<%{DATA:msgid}>)?\s*(%{EXECUSERNAME:execuser})?\s*%{SOURCE_SYSTEME}
#11-Jun-2019 09:07:57.65 ims-ms                    D 5  rfc822;pengouin.carl@my-domain.fr sdubois2@ims-ms-daemon mailsrv
#11-Jun-2019 09:07:57.62 tcp_local    ims-ms       EEQ 5  rfc822;pengouin.carl@my-domain.fr sdubois2@ims-ms-daemon mailsrv hostatv11.my-domain.fr ([195.221.67.130])
MTA_ENQUEUE_ORIGADDR %{MTA_ENQUEUE_ALWAYS_PREFIX}\s*%{RFC};%{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\s*%{MTA_ENQUEUE_ALWAYS_SUFFIX}
#11-Jun-2019 09:05:50.70 ims-ms                    D 60 bounces-fohfajbwoja3j-fonion.berly=my-domain.fr@bx.d.mailin.fr rfc822;fonion.berly@my-domain.fr ntalec@ims-ms-daemon mailsrv
#11-Jun-2019 08:59:34.42 tcp_local    ims-ms       EEQ 12 veron.bassey@bretagne.bzh rfc822;dimitar.novac@my-domain.fr dnovac@ims-ms-daemon mailsrv hostatv13.my-domain.fr ([195.221.67.140])
MTA_ENQUEUE_DESTADDR %{MTA_ENQUEUE_ALWAYS_PREFIX}\s*%{SOURCE_EMAIL_ADDRESS}\s*%{RFC};%{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\s*%{MTA_ENQUEUE_ALWAYS_SUFFIX}

#07-Jun-2019 17:07:01.82 tcp_local                 DEQ 70 bounce@newsletters.lerobert.com rfc822;Estelitae.Nebrun@my-domain.fr estellebrune@hotmail.com mailsrv hostatv9.my-domain.fr dns;hostatv9.my-domain.fr (TCP|172.29.208.120|39856|195.221.67.123|25) (hostatv9.my-domain.fr ESMTP Postfix) smtp;250 2.1.5 Ok
MTA_ENQUEUE_TRANSINFO %{MTA_ENQUEUE_ALWAYS_PREFIX}\s*%{SOURCE_EMAIL_ADDRESS}\s*%{RFC};%{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\s*%{MTA_ENQUEUE_ALWAYS_SUFFIX}\s*(\(%{TRANSPORT_INFORMATION:transinfo}\s*(%{APPLICATION_INFORMATION})?\))?\s*\((\[%{IP:clientip}\]|%{DATA:clientinfo})\)\s*%{GREEDYDATA:eventmsg}
#11-Jun-2019 09:33:28.00 tcp_intranet              Q 8  rfc822;foot@hoststr1.in.my-domain.fr @hostmtadmz.my-domain.fr:adminmtadmz@my-domain.fr mailsrv  Too many failures to this host during this run; skipping this host: Dead (unresponsive) host
MTA_ENQUEUE_FAILURES %{MTA_ENQUEUE_ALWAYS_PREFIX}\s*%{RFC};%{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\s*%{DATA}:%{MTA_ENQUEUE_ALWAYS_SUFFIX}\s*%{MTAEVENTS:event}\s*to this host during this run; skipping this host:\s*%{GREEDYDATA:eventmsg}
MTA_ENQUEUE_ERROR %{MTA_ENQUEUE_FAILURES}
#16-Feb-2007 15:41:32.36 tcp_intranet tcp_local    EE 1   adam@sesta.com rfc822;marlowe@siroe.com marlowe@siroe.com /opt/SUNWmsgsr/data/queue/tcp_local/002/ZZf0r4i0Wdy51.01 <0JDJ00D02IBWDX00@sesta.com> siroe.com (siroe.com [192.160.253.66])
#19-Jan-1998 13:13:27.10 hosta   2e2d.5.1 tcp_local   tcp_intranet E 1 service@innosoft.com rfc822;huber@domain.com huber 276 /opt/sun/comms/hostaging64/data/queue/tcp_intranet/ZZi0D4d9f5mwC.00 <01IWFVYLGTS499EC9W@innosoft.com> <01IWFVYLGTS499EC9Y@innosoft.com> mailsrv innosoft.com (innosoft.com [192.160.253.66]) 0 3
MTA_ENQUEUE_MULTI_OPTIONS %{MTA_ENQUEUE_ALWAYS_PREFIX}\s*%{SOURCE_EMAIL_ADDRESS}\s*%{RFC};%{ORIGINAL_DESTINATION_EMAIL_ADDRESS:origto}\s*%{MTA_ENQUEUE_OPTIONS_SUFFIX}\s*(%{INT:sensitivity})?\s*(%{INT:priority})?\s*(%{EMAILADDRESS})?\s*(%{EMAILADDRESS})?\s*(\(%{TRANSPORT_INFORMATION:transinfo}\s*(%{APPLICATION_INFORMATION})?\))?\s*(\(%{DATA:clientinfo}\))?\s*(%{GREEDYDATA:eventmsg})?

#10-Jul-2019 09:30:57.32 tcp_intranet - Y SMTP/hostmtadmz.mydomaine.eu/hostmtadmz.mydomaine.eu
MTA_ENQUEUE_APPINFO_SHORT %{INBOUND_OUTBOUND_TYPE}\s*%{ENTRYCODES:code}\s*%{APPLICATION_INFORMATION}
#%{MTA_TRANSACTION_PREFIX}\s*(%{MTA_ENQUEUE_ERROR}|%{MTA_ENQUEUE_TRANSINFO}|%{MTA_ENQUEUE_MULTI_OPTIONS}|%{MTA_ENQUEUE_DESTADDR}|%{MTA_ENQUEUE_ORIGADDR})
MTA_ENQUEUE_TRANSACTION %{MTA_ENQUEUE_ERROR}|%{MTA_ENQUEUE_TRANSINFO}|%{MTA_ENQUEUE_MULTI_OPTIONS}|%{MTA_ENQUEUE_DESTADDR}|%{MTA_ENQUEUE_ORIGADDR}|%{MTA_ENQUEUE_APPINFO_SHORT}

MTA (%{MTA_TRANSACTION_PREFIX})?\s*(?:%{MTA_TRANSACTION_CONNEXTION}|%{MTA_ENQUEUE_TRANSACTION}|%{MTA_RECEIVING}|%{MTA_MESSAGE_ID_LINE})


# -----------------------------------------------------------
#All patterns
SUN %{SYSLOG_ADDITIONAL_PREFIX} (?:%{IMAP}|%{MTA})
# Adding fields after syslog configuration for oracle
SYSLOG_ADDITIONAL_PREFIX %{SYSLOGTIMESTAMP} %{HOSTNAME:loghost} %{WORD:logtype}:
