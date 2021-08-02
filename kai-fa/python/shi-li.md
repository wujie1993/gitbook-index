# 示例

例子： 发送exchange电子邮件并携带附件

```text
# 首先安装exchangelib库
pip install exchangelib
```

```text
from exchangelib import DELEGATE, IMPERSONATION, Account, Credentials, Message, HTMLBody, Mailbox, FileAttachment

account = Account('<发送邮箱账号>',
                  credentials=Credentials(username='<发送邮箱账号>', password='<发送邮箱密码>'),
                  autodiscover=True)

m = Message(
        account=account,
        subject='测试带附件',
        body=HTMLBody('<h2>hello with attachment</h2>'),
        to_recipients=[Mailbox(email_address='<接收邮箱账号>')]
    )

m.attach(FileAttachment(name='<文件名>', content=open('<本地文件路径>','rb').read()))
m.send()

```

