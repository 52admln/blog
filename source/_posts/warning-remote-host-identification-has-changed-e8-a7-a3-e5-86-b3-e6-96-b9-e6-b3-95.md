---
title: 'WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED解决方法'
tags:
  - ssh
id: 501
categories:
  - 纯粹折腾
date: 2015-07-03 10:08:23
---

当ssh 10.1.1.61
 时出现一下情况：
<pre lang="bash" escaped="true" >
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
d0:00:7c:bc:88:5c:dc:de:89:61:44:30:00:60:f9:b2.
Please contact your system administrator.
Add correct host key in /root/.ssh/known_hosts to get rid of this message.
Offending key in /root/.ssh/known_hosts:1
RSA host key for 192.168.4.222 has changed and you have requested strict checking.
Host key verification failed.
</pre>
<!--more-->

解决办法：
先cat一下家目录的.ssh/know_hosts
<pre lang="bash" escaped="true" >
cat ~/.ssh/known_hosts
10.1.1.66 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAs86YOUHdfYUMHkJUmSpFeJCc0ztFQiWGIKlyrnf4KVCz+Ece/yY59QXnVG7b0DWA/wyzlaGRdumWFexX4Y7VE3WunEeXVPMRjF0YZgG5qW6EDXNMEquZzI5k7Jg96VGq+5ZzhtsRhUqXH1aNrMYydRfMUFDXTh+a3jKcoQLx9IiifouUuh5JEelql9w9FRgmOgOqmm3CVbn33mblyHZa0UOa3GDpFGRxFjxyPVLuOD90rJIVc126CxIK3TmsFS0emO7qxpz4mrNG/1xpCqgKxNejBkrlUtxzLxGbwuod3HPX7OB28uk1RdGsXhcZtKsPph3a04i7Y5C5QZ1XDXFzQw==
10.1.1.61 ssh-rsa AAAAB3NzaC1yc2EAAAABJUHjuHGUsI5fLkoQayuhjMLXaE69VlxA7en/SmxXs+VDjgXLGLLTLdSOxki1cBDzuPm4FefmES4A3X3mfAB8L46rFnPJe45hca4U6uC/IbJMlO8GhrWs+fpIYVdMmOkabBQl8li0J0bclmKlsRfpnsuSfT/hm5nBUUlmQcoXzGqvoLHRgV7JESdgvMoxlHzCSGRj62aBtJXktv5dbh5vCxjeh4jFrn4FrNo7IkG3fA6NoGBqUs6tENAclxI8F1b+479ywAqQedy233n2gW+l5v6Ms1uD+1jxxCiHx8OtO1/V7/vWLfEQfEMU323y4zHu4uXFLv9WB1XGNMgqEBlELSBkNpC4Pw==
</pre>
进入此文件
<pre lang="bash" escaped="true" >
vim ~/.ssh/known_hosts
</pre>
删除10.1.1.61的相关rsa的信息即可.