# send-receive-email-net-core
## Install MailKit from Nuget

### Use below mentioned code to write the send and receive mail

```


using MailKit.Net.Smtp;
using MailKit.Security;
using MimeKit;
using System;
using System.Collections.Generic;
using System.Linq;

namespace nursing_api.Utilities
{


    public class OutlookMail
    {
        string _sender = "";
        string _password = "";
        public OutlookDotComMail(string sender, string password)
        {
            _sender = sender;
            _password = password;
        }

        public string SendMail(string recipient, string subject, string message)
        {
           try
            {
                var messageBuilder = new BodyBuilder();
                messageBuilder.HtmlBody = string.Format(@"<p> Hey Candidate,<br>
    <p> What are you up to this weekend?<br>
   <center>PMKJ</center> ");



                var mimeMessage = new MimeMessage();
                mimeMessage.From.Add(new MailboxAddress(_sender.Trim()));
                mimeMessage.To.Add(new MailboxAddress(recipient.Trim()));
                mimeMessage.Subject = subject + DateTime.Now;                
                mimeMessage.Body = messageBuilder.ToMessageBody();

                using (var client = new SmtpClient())
                {
                    client.Connect("smtp-mail.outlook.com", 587, false);
                    client.Authenticate(_sender, _password);
                    client.Send(mimeMessage);
                    client.Disconnect(true);
                }
                return "success";
            }
            catch(Exception e)
            {
                throw e;
            }
        }

        public List<EmailMessage> ReceiveEmail(int maxCount = 10)
        {
            try
            {
                using (var emailClient = new MailKit.Net.Pop3.Pop3Client())
                {
                    emailClient.Connect("outlook.office365.com", 995);
                    emailClient.Authenticate(_sender, _password);

                    List<EmailMessage> emails = new List<EmailMessage>();
                    for (int i = emailClient.Count - 1; i > (emailClient.Count - 5); i--)
                    {
                        var message = emailClient.GetMessage(i);
                        var emailMessage = new EmailMessage
                        {
                            Content = !string.IsNullOrEmpty(message.HtmlBody) ? message.HtmlBody : message.TextBody,
                            Subject = message.Subject,
                            Date = message.Date.DateTime,
                            MessageId = message.MessageId
                        };
                        emailMessage.ToAddresses.AddRange(message.To.Select(x => (MailboxAddress)x).Select(x => new EmailAddress { Address = x.Address, Name = x.Name }));
                        emailMessage.FromAddresses.AddRange(message.From.Select(x => (MailboxAddress)x).Select(x => new EmailAddress { Address = x.Address, Name = x.Name }));
                        emailMessage.ccAddresses.AddRange(message.Cc.Select(x => (MailboxAddress)x).Select(x => new EmailAddress { Address = x.Address, Name = x.Name }));
                        emails.Add(emailMessage);
                    }
                    emailClient.Disconnect(true);
                    return emails.OrderByDescending(p => p.Date).ToList();
                }
            }
            catch(Exception e)
            {
                throw e;
            }            
        }
    }

    public class EmailAddress
    {
        public string Name { get; set; }
        public string Address { get; set; }
    }
    public class EmailMessage
    {
        public EmailMessage()
        {
            ToAddresses = new List<EmailAddress>();
            FromAddresses = new List<EmailAddress>();
            ccAddresses = new List<EmailAddress>();
        }
        public string MessageId { get; set; }
        public DateTime Date { get; set; }
        public List<EmailAddress> ToAddresses { get; set; }
        public List<EmailAddress> FromAddresses { get; set; }
        public List<EmailAddress> ccAddresses { get; set; }
        public string Subject { get; set; }
        public string Content { get; set; }
    }

  
}


```

### To call the send and receive methods from controller

```
 [HttpGet]
        public IActionResult Get()
        {
            try
            {
                string mailUser = "your email id"; // you can use config file to mention these values
                string mailUserPwd = "your password"; 

                var outlook = new OutlookDotComMail(mailUser, mailUserPwd);
                 var data = outlook.ReceiveEmail();
                return Ok(data);
            }
            catch(Exception e)
            {
                return UnprocessableEntity(e.Message);
            }            
        }

        // GET api/values/5
        [HttpGet("{id}")]
        public IActionResult Get(int id)
        {
            try
            {
                string mailUser = "your email id";
                string mailUserPwd = "your password";

                var sender = new OutlookMail(mailUser, mailUserPwd);
                sender.SendMail("anirban.b2020@gmail.com", "PMKJ Mail", "PMKJ!");
                return Ok("Mail is send successfully");
            }
            catch(Exception e)
            {
                return UnprocessableEntity(e.Message);
            }
            
        }
```
