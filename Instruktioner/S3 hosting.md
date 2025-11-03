# 2. Sätt upp S3 för hosting 

Här har jag valt att först sätta upp det manuellt, för att sedan skapa en template via Iac generator. 

Därefter har jag tagit bort den manuella set-upen och skapat en ny via templaten.

Under inlämning 1 finns en instruction om hur du använder Iac generator. 


## 2.1 Skapa S3 bucket

- Navigera till S3 service i AWS Management Console.
- Create Bucket 
- Namnge bucket - unikt namn, inga versaler.
- Scrolla ner och create bucket 

https://i.imgur.com/yAoANhv.png


## 2.2 Ladda upp din index.html i bucket 

- Gå in på din bucket
- Välj uppload upp till höger 
- Add files - leta upp din index.html 
- Välj upload 

https://i.imgur.com/RJ6wKC4.png


## 2.3 Aktivera Static Website Hostings

- Gå till properties i din bucket
- Scrolla ner till "Static website hosting"
- Edit
- Enable
- Ange index.html som Index dokument
- Spara ändringar 


https://i.imgur.com/CCZNgah.png


## 2.4 Fixa publika rättigheter


- Gå till "Permissions"
- Under "Block public access" - avmarkera allt (Edit → uncheck)
- Gå till bucket Policy
- Add new statement - lägg in nedan json (med ditt bucketnamn)
- spara 

{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::svampregistret/*"
  }]
}

https://i.imgur.com/IncBInb.png


Verifiera

**Gå tillbaka till Static website hosting, och klicka på Bucket website endpoint


https://i.imgur.com/0AqEZXn.png


https://i.imgur.com/w3cWTkB.png






