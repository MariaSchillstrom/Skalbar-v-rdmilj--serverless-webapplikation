


**Note:** 
I Inlämning Del 1 byggde jag MVC-applikationen under tutorialens gång men "färdigställde" den (i praktiken en enkel Nginx välkomstsida) innan deploy-steget. I denna Del 2 utökar jag lösningen med serverless-komponenter (AWS Lambda och DynamoDB) för att hantera datahändelser event-drivet, vilket gör systemet mer skalbart och mindre beroende av kontinuerligt körande containrar.



# 1. Skapa en enkel Websida

Jag har valt att skapa en enkel websida, för svampentusiaster som man kan bygga vidare på. 

## 1.1 Skapa en indexfil

Börja med att skapa index.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Svampregistret</title>
</head>
<body>
    <h1>Välkommen till Svampregistret!</h1>
</body>
</html>

Verifiera i din browser

Kör start index.html i din terminal, välj vilken browser du vill använda,  output bör bli enligt nedan bild. 

https://i.imgur.com/b6uewSW.png


# 2. Sätt upp S3 för hosting 

Här har jag valt att först sätta upp det manuellt, för att sedan skapa en template via Iac generator. 

Därefter har jag tagit bort den manuella set-upen och skapat en ny via cloud formation.

Under inlämning 1 finns en instruction om hur du använder Iac generator. 


För att se hur jag satte upp S3 hosting manuellt, se [S3 hosting guide](S3%20hosting.md).




S3.yaml

---
Metadata:
  AWSToolsMetrics:
    IaC_Generator: "arn:aws:cloudformation:eu-west-1:542478884453:generatedTemplate/8ea5aa57-0f5d-4091-9453-db4cbbb25e43"
Resources:
  S3BucketSvampregistret:
    UpdateReplacePolicy: "Retain"
    Type: "AWS::S3::Bucket"
    DeletionPolicy: "Retain"
    Properties:
      WebsiteConfiguration:
        IndexDocument: "index.html"
      PublicAccessBlockConfiguration:
        RestrictPublicBuckets: false
        IgnorePublicAcls: false
        BlockPublicPolicy: false
        BlockPublicAcls: false
      BucketName: "svampregistret2"
      OwnershipControls:
        Rules:
        - ObjectOwnership: "BucketOwnerEnforced"
      BucketEncryption:
        ServerSideEncryptionConfiguration:
        - BucketKeyEnabled: true
          ServerSideEncryptionByDefault:
            SSEAlgorithm: "AES256"
            
  S3BucketPolicySvampregistret:
    UpdateReplacePolicy: "Retain"
    Type: "AWS::S3::BucketPolicy"
    DeletionPolicy: "Retain"
    Properties:
      Bucket:
        Ref: "S3BucketSvampregistret"
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
        - Resource: "arn:aws:s3:::svampregistret2/*"
          Action: "s3:GetObject"
          Effect: "Allow"
          Principal: "*"
          Sid: "PublicReadGetObject"


Kör i terminalen


cd Templates

```
aws cloudformation create-stack \
  --stack-name svampregistret-stack \
  --template-body file://s3-bucket.yaml \
  --region eu-west-1

```




