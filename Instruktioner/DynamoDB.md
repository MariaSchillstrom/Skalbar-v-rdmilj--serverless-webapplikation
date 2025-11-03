

**För denna delen har jag utgått från lärarens tutorial, men ändrat den enligt nuvarande gränssnitt samt mina preferenser**

# 6. Skapa DynamoDB-tabell för Svampar

---

## 6.1: Skapa tabellen

1. Öppna **DynamoDB** i AWS Console
2. Klicka **"Create table"** (orange knapp)
3. Fyll i:
   - **Table name:** `Svampar`
   - **Partition key:** `svampnamn` (Type: **String**)
4. Scrolla ner och klicka **"Create table"**
5. Vänta tills Status = **Active** (tar ~30 sekunder)

---

## 6.2: Lägg till svampdata

1. Klicka på din tabell **"Svampar"**
2. Klicka på fliken **"Explore table items"**
3. Klicka **"Create item"**

### Första svampen:

4. Du ser ett fält: `svampnamn` (String)
5. Skriv: `Kantarell`
6. Klicka **"Add new attribute"** (knapp)
   - Välj **String**
   - Attribute name: `beskrivning`
   - Value: `Gul, trattformad, god matsvamp`
7. Klicka **"Create item"**

### Andra svampen:

8. Klicka **"Create item"** igen
9. `svampnamn`: `Flugsvamp`
10. Lägg till attribut:
    - `beskrivning`: `Röd med vita prickar, giftig`
11. Klicka **"Create item"**

### Tredje svampen (valfri):

12. `svampnamn`: `Karl Johan`
13. `beskrivning`: `Brun hatt, tjock fot, god matsvamp`
14. Klicka **"Create item"**

---

## Steg 3: Verifiera

1. I **"Explore table items"**
2. Du ska se alla dina svampar listade
3. Klicka på en svamp för att se alla attribut

---

## Verifiering

[[Screenshot av DynamoDB-tabellen med svampdata]](https://i.imgur.com/PWpuzb6.png)

https://i.imgur.com/kmkOb60.png



---

**✅ Klart!** Nu har du en DynamoDB-tabell med svampdata som kan användas i din serverless applikation.

---

## Referens

Baserad på lärarens tutorial:  
https://cloud-developer.educ8.se/clo/3.-scalable-cloud-applications/1.-tutorials/6.-create-a-serverless-webapp-on-aws-greetings/index.html





# 7. Uppdatera Lambda för att läsa från DynamoDB

---

## Steg 7.1: Uppdatera Lambda-koden

**I AWS Console:**

1. Gå till **Lambda** → Din funktion (`Svampregister2`)
2. Scrolla ner till **Code source**
3. Ersätt koden med:

```python
import json
import boto3

# Initialize the DynamoDB client
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Svampar')

def lambda_handler(event, context):
    # Scan the DynamoDB table
    result = table.scan()
    items = result['Items']

    return {
        'statusCode': 200,
        'headers': {
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
            "Access-Control-Allow-Headers": "Content-Type"
        },
        'body': json.dumps(items)
    }
```

4. **Klicka "Deploy"** (orange knapp)

---

## Steg 7.2: Första testet (förväntat fel)

1. Öppna din S3 website URL i webbläsaren
2. Tryck **F12** → **Console**-fliken
3. Du kommer se ett **fel** i Console - Lambda har inte tillåtelse att läsa från DynamoDB än!

**Förväntat felmeddelande:**
```
User: arn:aws:sts::xxx:assumed-role/Svampregister2-role/Svampregister2 
is not authorized to perform: dynamodb:Scan on resource: table/Svampar
```

---

## Steg 7.3: Lägg till IAM-rättigheter för DynamoDB

1. I Lambda → Klicka på **Configuration**-fliken
2. Klicka **Permissions** i vänster meny
3. Under "Execution role" → **Klicka på roll-namnet** (öppnas i ny flik till IAM)
4. I IAM-konsolen → Klicka **"Add permissions"** → **"Attach policies"**
5. Sök efter: `AmazonDynamoDBReadOnlyAccess`
   - *(Använd ReadOnly istället för FullAccess - säkrare!)*
6. **Checka policyn**
7. Klicka **"Add permissions"**

**Policy tillagd:** Lambda kan nu läsa från DynamoDB! ✅

---

## Steg 7.4: Verifiera att DynamoDB-åtkomst fungerar

1. Gå tillbaka till din S3 website
2. Refresh sidan (Ctrl+Shift+R)
3. Öppna **F12** → **Console**-fliken
4. Felet ska vara borta!
5. Du ska se **JSON-data** från DynamoDB i Console (om du kollar Network-fliken)

**Förväntat svar från API:**
```json
[
  {
    "svampnamn": "Kantarell",
    "beskrivning": "Gul, trattformad, god matsvamp"
  },
  {
    "svampnamn": "Flugsvamp",
    "beskrivning": "Röd med vita prickar, giftig"
  },
  {
    "svampnamn": "Karl Johan",
    "beskrivning": "Brun hatt, tjock fot, god matsvamp"
  }
]
```

---

# 8. Visa svamparna som en lista på webbsidan

---

## Steg 8.1: Uppdatera index.html

Uppdatera din `index.html` för att visa svamparna i en lista:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Svampregistret</title>
</head>
<body>
    <h1>Välkommen till Svampregistret!</h1>

    <ul id="svampLista">
        <li>Loading...</li>
    </ul>
    
    <script>
        async function fetchSvampar() {
            const response = await fetch('https://09qqcalmx5.execute-api.eu-west-1.amazonaws.com/default/Svampregister');
            const svampar = await response.json();
            
            const lista = document.getElementById('svampLista');
            lista.innerHTML = ''; // Rensa "Loading..."
            
            svampar.forEach(svamp => {
                const listItem = document.createElement('li');
                listItem.textContent = `${svamp.svampnamn}: ${svamp.beskrivning}`;
                lista.appendChild(listItem);
            });
        }
        fetchSvampar();
    </script>
</body>
</html>
```

**Viktigt:** Byt ut API-URL:en mot din egen om den är annorlunda!

---

## Steg 8.2: Ladda upp till S3

1. **Spara** `index.html`
2. Gå till **S3 Console**
3. Öppna din bucket: **svampregistret2**
4. Klicka **"Upload"**
5. Välj `index.html`
6. Klicka **"Upload"** (Replace existing file)

---

## Steg 8.3: Verifiera att allt fungerar



**Fungerande webbsida med svamplista och bilder:**

https://i.imgur.com/HT7i63l.png


Applikationen visar nu dynamisk data från DynamoDB med:
- Svampnamn
- Beskrivningar
- Bilder (där tillgängligt)

Lambda läser från DynamoDB, IAM-rättigheter är konfigurerade, och CORS fungerar korrekt.

---

## Sammanfattning

**Vad vi byggt:**
- ✅ Lambda läser data från DynamoDB
- ✅ IAM-rättigheter konfigurerade
- ✅ Frontend visar dynamisk data från databasen
- ✅ Fullständig serverless stack: S3 → API Gateway → Lambda → DynamoDB

**Arkitektur:**
```
Browser → S3 (index.html) → API Gateway → Lambda → DynamoDB
```

---

## Slutsats

Din serverless webbapplikation är nu komplett! Du har:
- Static website hosting med S3
- REST API med API Gateway
- Serverless backend med Lambda
- NoSQL-databas med DynamoDB
- Allt säkrat med IAM-roller och CORS



---

## Referens

Baserad på lärarens tutorial:  
https://cloud-developer.educ8.se/clo/3.-scalable-cloud-applications/1.-tutorials/6.-create-a-serverless-webapp-on-aws-greetings/index.html
