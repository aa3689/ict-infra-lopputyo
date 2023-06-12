## ICT-infrastruktuuri pilvialustalla -kurssin lopputyö

Lopputyössä oli tehtävänä suunnitella ja toteuttaa pilviarkkitehtuuri AWS:n pilveen. Työssä piti luoda yksityiskohtainen arkkitehtuurikaavio ja CloudFormation-templaatti ympäristön pystyttämiseen.

### Arkkitehtuurikuva

<img src="/ict_infra_drawio.png">

- **VPC** – virtuaalinen yksityinen verkkoympäristö
- **Internet Gateway** – mahdollistaa kommunikaation VPC:n ja internetin välillä
- **Application Load Balancer** – jakaa liikennettä EC2-palvelinten välillä
- **EC2** - virtuaalipalvelin
- **NAT Gateway** – mahdollistaa private subnetissä oleville resursseille pääsyn VPC-verkon ulkopuolella oleviin palveluihin
- **Lambda Function** – serverlessinä suoritettava koodinpalanen
- **IAM Role** – annetaan käyttöoikeuksia resursseille
- **CloudWatch** - loggauspalvelu
- **DynamoDB** – NoSQL-tietokantapalvelu

### Templaatti

Templaatissa luodaan VPC-verkko sekä kaksi julkista aliverkkoa ja kaksi yksityistä aliverkkoa hajauttaen ne kahdelle availability zonelle. Aliverkkojen määrittäminen julkiseksi tai yksityiseksi tapahtuu luomalla reititystaulut ja assosioimalla nämä taulut kyseisiin aliverkkoihin. VPC:hen liitetty Internet Gateway mahdollistaa tiedon kaksisuuntaisen välityksen internetin ja julkisen aliverkon välillä (sekä NAT Gatewayn kautta yksityisestä aliverkosta ulos, mutta ei sisään).

Julkisella aliverkolla on yksi reititystaulu, mutta yksityisellä aliverkolla on kaksi koska eri AZ:hin luoduille aliverkoille on suositeltavaa määrittää omat NAT Gatewayt korkean saavutettavuuden (high availability) vuoksi. NAT Gatewayn avulla yksityisessä aliverkossa olevat resurssit (tässä tapauksessa lambda-funktiot) saavat yhteyden VPC-verkon ulkopuolella oleviin palveluihin (tässä tapauksessa DynamoDB ja CloudWatch). NAT Gatewayt luodaan julkiseen aliverkkoon ja lisäksi ne tarvitsevat Elastic IP:t toimiakseen.

Templaatissa luodaan myös kaksi EC2-instanssia (Linux-palvelin) julkisiin aliverkkoihin sekä yksi Application Load Balancer ja sen vaatimat listener ja target group. Lisäksi luodaan security groupit edellä mainituille palveluille, pari lambda-funktiota, DynamoDB-taulu ja IAM-roolit, jotka mahdollistavat DynamoDB:hen tallentamisen lambda-funktioiden avulla sekä DynamoDB:sta datan hakemisen EC2-instanssista.

Parameters-kohta templaatissa määrittää arvoja, joita käyttäjältä kysytään infraa
pystyttäessä: tässä tapauksessa key-pairia EC2:n luomiseen sekä nimeä ympäristölle, joka tagataan resursseihin. Parametreihin olisi voinut määrittää lisäksi esim. instanssityypin ja AZ:n valinnan, mutta yksinkertaistamisen vuoksi ne ovat kovakoodattuna.

Templaatti ei ole niin iso, että sitä olisi vielä helpomman tulkittavuuden vuoksi tarvinnut jakaa pienempiin kokonaisuuksiin (stackeiksi). Teoriassa sen olisi voinut jakaa mahdollisesti esim. seuraavanlaisesti:

- VPC ja subnetit (ja IAM-roolit)
- EC2 ja ALB
- Lambdat ja DynamoDB
