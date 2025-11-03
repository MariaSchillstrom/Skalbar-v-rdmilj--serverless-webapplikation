
**Denna tutorial är baserad på lärarens, men har modifierats samt uppdaterats av mig**

**Efter denna manuella uppsättning skapade jag en CloudFormation template, raderade de manuella resurserna och skapade om dem via lambda-api-gateway.yaml**

# Steg 3-5: Lambda Function, API Gateway och CORS

*Uppdaterad tutorial baserad på AWS Console 2024/2025*



---

## Steg 3: Skapa en Lambda Function

### 3.1 Skapa funktionen

1. Navigera till **Lambda** i AWS Management Console
2. Klicka **"Create function"**
3. Välj **"Author from scratch"**
4. Konfigurera funktionen:
   - **Function name:** `Greetings` (eller `Svampregister`)
   - **Runtime:** Python 3.x (välj senaste versionen)
   - Expandera **"Change default execution role"** 
   - Notera namnet på rollen som skapas (t.ex. `Greetings-role-xxxxx`)
5. **Klicka "Create function"**

### 3.2 Lambda-kod

När funktionen är skapad kommer du till funktionens sida. Scrolla ner till **"Code source"**.

Standard-koden ser ut så här:

```python
import json

def lambda_handler(event, context):
    # TODO implement
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

*(Du behöver troligen inte ändra något i detta steg)*

### 3.3 Verifiera funktionen

1. Klicka på **"Test"**-knappen (uppe till höger i Code source-sektionen)
2. **Första gången:**
   - **Test event action:** Create new event
   - **Event name:** `Test`
   - Låt Event JSON vara som den är
   - Klicka **"Save"**
3. **Klicka "Test" igen**
4. **Verifiera responsen** under "Execution result":

```json
{
  "statusCode": 200,
  "body": "\"Hello from Lambda!\""
}
```

✅ Du ska se **"Executing function: succeeded"** (grön box)

---

## Steg 4: Skapa API Gateway som Trigger

### 4.1 Lägg till API Gateway trigger

1. **Scrolla upp till "Function Overview"** (diagrammet längst upp på Lambda-sidan)
2. Klicka **"+ Add trigger"**
3. **Välj "API Gateway"** från dropdown-listan
4. **Konfigurera:**
   - **Intent:** Create a new API (eller "Create an API")
   - **API type:** REST API *(Viktigt!)*
   - **Security:** Open *(Viktigt för detta projekt!)*
5. Klicka **"Add"**

### 4.2 Notera API Gateway URL

Efter att triggern skapats ser du en **API endpoint** URL i Function Overview, typ:

```
https://abc123xyz.execute-api.eu-west-1.amazonaws.com/default/FunktionensNamn
```

**Kopiera denna URL** - du behöver den i nästa steg!

### 4.3 Skapa metod i API Gateway (om den saknas)

**OBS:** Ibland skapas inte metoden automatiskt. Verifiera detta:

1. Gå till **API Gateway** i AWS Console
2. Klicka på din API (t.ex. "Svampregister-API")
3. Klicka **"Resources"** i vänster meny
4. **Om du ser "No methods defined":**
   - Klicka på din resurs (t.ex. `/Svampregister`)
   - Klicka **"Create method"**
   - **Method type:** GET
   - **Integration type:** Lambda function
   - **Lambda function:** Välj din funktion
   - Klicka **"Create method"**

### 4.4 Testa API Gateway (valfritt)

1. Öppna API Gateway URL:en i webbläsaren
2. Du ska se: `"Hello from Lambda!"`

---

## Steg 5: Uppdatera Webbsidan och Aktivera CORS

### 5.1 Uppdatera index.html

Uppdatera din `index.html` med följande kod:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Svampregistret</title>
</head>
<body>
    <h1>Välkommen till Svampregistret!</h1>
    
    <p id="greeting">Loading...</p>
    
    <script>
        async function fetchGreeting() {
            const response = await fetch('https://DIN-API-URL-HÄR');
            const responseBody = await response.text();
            document.getElementById('greeting').innerText = responseBody;
        }
        fetchGreeting();
    </script>
</body>
</html>
```

**Viktigt:** Byt ut `'https://DIN-API-URL-HÄR'` mot din faktiska API Gateway URL!

### 5.2 Ladda upp till S3

1. Spara `index.html`
2. Gå till **S3 Console**
3. Öppna din bucket
4. Klicka **"Upload"**
5. Välj `index.html`
6. Klicka **"Upload"** (Replace existing file)

### 5.3 Första testet (förväntat CORS-fel)

1. Öppna din **S3 Bucket website endpoint URL** i webbläsaren
2. Du ska se "Välkommen till Svampregistret!" och **"Loading..."** (står kvar)
3. Tryck **F12** → **Console**-fliken
4. Du kommer se ett **CORS-fel**:

```
Access to fetch at 'https://...' from origin 'http://...' 
has been blocked by CORS policy: No 'Access-Control-Allow-Origin' 
header is present on the requested resource.
```

**Detta är förväntat!** Nu måste vi fixa CORS.

### 5.4 Aktivera CORS - Del 1: Lambda Function

1. Gå tillbaka till **Lambda**-funktionen
2. Scrolla ner till **"Code source"**
3. **Uppdatera koden** till:

```python
import json

def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Origin': '*',
            'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
            'Access-Control-Allow-Headers': 'Content-Type'
        },
        'body': json.dumps('Hello from Lambda!')
    }
```

4. **Klicka "Deploy"** (orange knapp i Code source-sektionen)

### 5.5 Aktivera CORS - Del 2: API Gateway

1. Gå till **API Gateway** i AWS Console
2. Välj din API
3. Klicka **"Resources"** i vänster meny
4. Klicka på din resurs (t.ex. `/Svampregister`)
5. Klicka **"Enable CORS"** (knapp till höger under "Resource details")
6. I popup-rutan:
   - Låt default-värdena vara
   - Klicka **"Save"** eller **"Enable CORS and replace existing CORS headers"**
7. **VIKTIGT:** Klicka **"Deploy API"** (orange knapp uppe till höger)
   - **Deployment stage:** default
   - Klicka **"Deploy"**

### 5.6 Verifiera att allt fungerar

1. Gå tillbaka till din S3 website
2. Tryck **Ctrl+Shift+R** (hard refresh för att rensa cache)
3. **Nu ska du se:**
   - "Välkommen till Svampregistret!"
   - **"Hello from Lambda!"** (istället för "Loading...")

4. Tryck **F12** → **Console**
   - CORS-felet ska vara borta
   - Inga röda felmeddelanden

**✅ Klart!** Din serverless webbapplikation fungerar nu!


https://i.imgur.com/4OvqIoM.png

---

## Sammanfattning av vad vi skapade

- **S3 Bucket:** Hostar den statiska webbsidan
- **Lambda Function:** Backend-logik som körs on-demand
- **API Gateway:** REST API som triggar Lambda-funktionen
- **CORS-konfiguration:** Tillåter webbläsaren att anropa API:et från S3

---

## Vanliga problem och lösningar

### Problem: "Loading..." står kvar
- **Lösning:** CORS är inte korrekt konfigurerat. Se steg 5.4-5.5.

### Problem: "403 Forbidden"
- **Lösning:** API Gateway security är inte satt till "Open", eller så är inte metoden skapad korrekt.

### Problem: Ändringarna syns inte
- **Lösning:** 
  - Verifiera att du laddat upp den senaste versionen till S3
  - Gör hard refresh (Ctrl+Shift+R)
  - Testa i Incognito-läge
  - Kolla att du använder Bucket website endpoint URL (inte Object URL)

### Problem: Ingen metod i API Gateway
- **Lösning:** Skapa metoden manuellt enligt steg 4.3

## Verifiering - Manuell Setup

[Bild på webbsidan med "Hello from Lambda!" från manuella resurser]

---

## Infrastructure as Code - CloudFormation

Efter manuell verifiering skapades en CloudFormation template för att automatisera deploymentet.

### CloudFormation Template
Se [lambda-api-gateway.yaml](./lambda-api-gateway.yaml)

### Deployment via CloudFormation
```bash
aws cloudformation create-stack \
  --stack-name svampregistret2-lambda-api-stack \
  --template-body file://lambda-api-gateway.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region eu-west-1
```

### Verifiering - CloudFormation Deployment

**CloudFormation Stack Status:**

https://i.imgur.com/dpH9OWd.png

**Cloudformation Outputs**

https://i.imgur.com/XgZgXWG.png

**Fungerande applikation:**

https://i.imgur.com/4OvqIoM.png


**API Gateway URL från CloudFormation Outputs:**
```
https://09qqcalmx5.execute-api.eu-west-1.amazonaws.com/default/Svampregister
```





