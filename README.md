# GitHub Actions CI/CD Workflow för .NET Projekt

Den här filen förklarar hur du använder GitHub Actions för att skapa ett automatiserat CI/CD-arbetsflöde.

## Översikt av YAML-filens Struktur

Denna YAML-fil beskriver ett **CI/CD workflow** för ett .NET-projekt. Workflowet körs varje gång det görs en **push** till någon av följande tre brancher:
- `dev` (för utveckling)
- `qa` (för testmiljö)
- `main` (för produktionsklar kod)

Workflowet innehåller flera jobb (jobs) som körs i en viss ordning för att säkerställa att koden byggs, testas och deployas korrekt.

## Exempel på hur det funkar

### 1. **Kodning i dev-branchen**
Du börjar genom att arbeta i **dev**-branchen. Den här branchen är din dagliga arbetsyta, där du skapar nya funktioner, fixar buggar eller testar idéer.
När du gör en förändring i koden och är klar, gör du en **commit** och **push** till dev-branchen.

Efter att du har pushat din kod till **dev**-branchen kommer GitHub Actions automatiskt att trigga workflowet. 
Detta betyder att hela bygg-, test- och deploy-processen startar direkt utan att du behöver göra något manuellt.

### 2. **Bygg och test körs automatiskt**
Så snart din kod har pushats till dev, ser du att GitHub Actions har startat ett jobb för att bygga och testa applikationen.
Det betyder att du inte längre behöver oroa dig för att manuellt bygga projektet eller köra tester på din lokala maskin. Det sker direkt i bakgrunden.

- Först byggs koden på en **Ubuntu-maskin** som körs i GitHub Actions, vilket innebär att du kan vara säker på att bygget görs på en ren, standardiserad miljö.
- Sedan körs **unit tester** på koden för att säkerställa att ingen funktionalitet har brutits och att koden fortsätter att fungera som förväntat.
-  Om testerna misslyckas får du snabbt veta det, och då kan du fixa problemet innan det når nästa steg i processen.

### 3. **Feedback direkt via GitHub Actions**
Om bygg- eller testjobbet misslyckas, får du omedelbart en notifikation i GitHub om att workflowet inte gick igenom. På 
GitHub Actions-sidan kan du enkelt se vad som gick fel genom loggarna och åtgärda problemet direkt. Detta är mycket snabbare än att behöva
köra tester och byggen manuellt på din lokala utvecklingsmaskin.

### 4. **Push till QA-branch**
När koden i dev-branchen är klar och du känner att den är stabil, gör du en **pull request** (PR) till **qa**-branchen. Det här är
ett viktigt steg eftersom det innebär att koden är redo för mer omfattande testning och simulering i en staging-miljö.

- Om PR:en accepteras och koden mergeas till **qa**, triggas ett nytt workflow. GitHub Actions kommer automatiskt att deploya den
- senaste koden till QA-miljön. I detta exempel simuleras deploymenten, men i en riktig miljö skulle den verkligen deployas till en
-  testserver för ytterligare validering.
- Om något går fel i QA-miljön (t.ex. tester på QA-servern misslyckas eller deploymenten fallerar), får du feedback om det så att du kan rätta till problemet innan du går vidare till produktionsmiljön.

### 5. **Push till Main och Produktion**
När allting har testats och validerats i QA, är du redo att göra en **push till main**. Main-branchen representerar den stabila versionen
av applikationen, den som är redo för produktion.

- När du pushar till main, triggas ett workflow som kommer att köra tester igen, men nu är vi på väg att deploya koden till produktionsmiljön.
- I detta workflow simuleras deploymenten, men i en riktig produktionssituation skulle den distribueras till live-servrar som används av dina användare.
  
### 6. **Ingen mer manuell deployment**
Efter att koden har pushats till **main**, sker hela deploymenten automatiskt. Du behöver inte längre manuellt koppla upp dig på servrar, 
skapa byggfiler eller deploya applikationen till produktion. GitHub Actions gör detta åt dig.

### 7. **Snabbare releasecykler**
Eftersom hela processen är automatiserad får du snabba feedbackloopar. Det betyder att du kan fokusera på att utveckla funktionalitet
och rätta till buggar, medan GitHub Actions ser till att bygga, testa och deploya din kod korrekt genom alla miljöer.

### Sammanfattning av din upplevelse:
Som utvecklare innebär detta workflow att du slipper oroa dig för manuala steg i CI/CD-processen. Du arbetar med kod, gör ändringar i **dev**,
och när det är klart pushar du till **qa** och till slut till **main**. GitHub Actions tar hand om att bygga, testa och deploya din kod till rätt miljö,
och du får direkt feedback om något går fel. Detta sparar dig mycket tid och gör det enklare att hålla projektet i ett stabilt och fungerande tillstånd genom hela utvecklingsprocessen.

Automatiseringen gör också att du kan känna dig trygg i att din kod inte bryter funktionalitet eller orsakar problem i produktionsmiljön eftersom alla tester körs innan deploymenten sker.

## Yaml-filen 

```yaml
name: CI/CD Workflow

on:
  push:
    branches:
      - dev    # Kör workflow på push till dev
      - qa     # Kör workflow på push till qa
      - main   # Kör workflow på push till main

jobs:
  build:
    runs-on: ubuntu-latest  # Kör jobbet på en Ubuntu-maskin

    steps:
    - name: Checkout code
      uses: actions/checkout@v2  # Klona koden från repository

    - name: Setup .NET
      uses: actions/setup-dotnet@v3  # Installera .NET SDK
      with:
        dotnet-version: '8.0'  # Anpassa till din .NET-version

    - name: Restore dependencies
      run: dotnet restore  # Installera NuGet-beroenden

    - name: Build solution
      run: dotnet build --configuration Release  # Bygg projektet

  test:
    runs-on: ubuntu-latest  # Kör jobbet på en Ubuntu-maskin
    needs: build            # Testet körs efter att bygget är klart

    steps:
    - name: Checkout code
      uses: actions/checkout@v2  # Klona koden från repository

    - name: Run unit tests
      run: dotnet test  # Kör unit-tester

  deploy_to_qa:
    runs-on: ubuntu-latest  # Kör jobbet på en Ubuntu-maskin
    needs: test             # Kör deploy när tester är klara
    if: github.ref == 'refs/heads/qa'  # Körs bara om ändringar är på qa-branch

    steps:
    - name: Checkout code
      uses: actions/checkout@v2  # Klona koden från repository

    - name: Deploy to QA environment
      run: echo "Deploying to QA environment..."  # Simulera deployment till QA-miljö

  deploy_to_prod:
    runs-on: ubuntu-latest  # Kör jobbet på en Ubuntu-maskin
    needs: test             # Kör deploy när tester är klara
    if: github.ref == 'refs/heads/main'  # Körs bara om ändringar är på main-branch

    steps:
    - name: Checkout code
      uses: actions/checkout@v2  # Klona koden från repository

    - name: Deploy to Production environment
      run: echo "Deploying to Production environment..."  # Simulera deployment till produktionsmiljö

