# RIHA API spetsifikatsioon

Versioon 1.4, 24.08.2016

Tellija: Riigi Infosüsteemi Amet; Täitja: Girf OÜ, Degeetia OÜ, Mindstone OÜ

![](http://www.struktuurifondid.ee/public/EL_Struktuuritoetus_horisontaal.jpg)

##Sisukord
- [Ülevaade](#Ülevaade)
- [Seotud dokumendid](#seotud-dokumendid)
- [Mõisted](#mõisted)
- [Protokolli üldistatud kirjeldus](#protokolli-üldistatud-kirjeldus)
- [Arhitektuuriline ülevaade](#arhitektuuriline-ülevaade)
  - [Klassikaline variant](#klassikaline-variant)
  - [Ühetaoline variant](#ühetaoline-variant)
  - [Eripäringud (ülevaade)](#eripäringud-ülevaade)
  - [Callback lisaparameeter](#callback-lisaparameeter)
- [Edastatavad sõnumid](#edastatavad-sonumid)
  - [Edukad päringuvastused](#edukad-päringuvastused)
  - [Päringute tüübid, lubatud käsud ja lisaparameetrite kodeerimine](#päringute-tüübid-lubatud-käsud-ja-lisaparameetrite-kodeerimine)
  - [Eripäringud](#eripäringud)
    - [Eripäringute loend](#eripäringute-loend)
  - [Andmete küsimine](#andmete-küsimine)
  - [Uute andmete lisamine](#uute-andmete-lisamine)
  - [Andmete muutmine](#andmete-muutmine)
  - [Andmete kustutamine](#andmete-kustutamine)
- [Sõnumite töötlemise reeglid](#sõnumite-töötlemise-reeglid)
- [Veateadete kirjeldused](#veateadete-kirjeldused)
- [Olekudiagrammide kirjeldused](#olekudiagrammide-kirjeldused)
- [Disaini kaalutlused](#disaini-kaalutlused)
- [Vastavusklausel](#vastavusklausel)
- [Vastavusmudeli kirjeldus](#vastavusmudeli-kirjeldus)
- [Muutelugu](#muutelugu)

##Ülevaade

Käesolev dokument kirjeldab RIHA sisemisi liideseid, mille abil nii kasutajaliides suhtleb andmehoidlaga kui ka teised komponendid omavahel suhtlevad.

Osa nendest liidestest võivad olla kasutatavad ka väljapoolt RIHA, osa on aga mõeldud sisemiseks kasutuseks.

Dokument ei esita konkreetseid liideseid, vaid nende struktuuri: mis kujul andmeid esitatakse ja kuidas neid saadetakse.

Dokumendi sihtgrupiks on RIHA süsteemi arendajad.

##Seotud dokumendid

Dokument ei vaja mõistmiseks täiendavaid dokumente.

##Mõisted

Dokumendis kasutatakse ainult tarkvara arenduses üldlevinud mõisteid, nagu:

* REST vt [http://en.wikipedia.org/wiki/Representational\_state\_transfer]
* json vt [http://en.wikipedia.org/wiki/JSON]
* http vt [http://en.wikipedia.org/wiki/Hypertext\_Transfer\_Protocol]
* http käsud ja päis vt eelmist viidet
* cgi vt [http://en.wikipedia.org/wiki/Common\_Gateway\_Interface]
* urlencoding vt [http://en.wikipedia.org/wiki/Percent-encoding]

##Protokolli üldistatud kirjeldus

Protokoll on mõeldud RIHA komponentide omavaheliseks suhtluseks, eeskätt aga brauseris töötava veebirakenduse suhtluseks RIHA serverirakendusega.

Kõik liidesed toimivad http või http kaudu REST põhimõtete alusel. Saadetud andmeid kodeeritakse reeglina json formaadis, erandina GET päringute puhul aga cgi nimi=urlencoded_väärtus paaridena. Json formaat võib omakorda sisaldada stringe, mis kodeerivad mistahes formaadis faile või tekste (.zip, .docx, .sql jne).

Ainult sisekomponentide vahel toimivad API-d piiratakse väliskeskkonnast üldjuhul IP aadressi põhiselt ning piiranguinfot päringus ei edastata.

Väliseks kasutuseks mõeldud päringud võivad olla kas piiramata või piiratud autentimistokeni abil, mis tuleb päringule kaasa panna kas ühe parameetri või http päises oleva väärtusena.

##Arhitektuuriline ülevaade

Päringute üldparameetreid võib esitada kahel alternatiivsel moel, kusjuures API peab ära tundma mõlemad ja kasutus on vaba:

###Klassikaline variant

* Tegevus esitatakse http käsuna `get` (päring), `post` (lisamine), `put` (muutmine) või `delete` (kustutamine)
* Objekt, mida päritakse, millele lisatakse uus väärtus, tehakse muudatus või kustutatakse, on antud URL  teel peale API baas-urli. Näites on baas-url `http://localhost/api/`

`http://localhost/api/db/mytable`

`http://localhost/api/db/mytable/123`

Teel on reeglina kõigepealt `db`, siis baasitabeli nimi ja seejärel tabelis oleva kirje id. Mitte-andmebaasi väärtuste korral võib tee alata mingi muu identifikaatoriga kui `db`, näiteks `files`. Keeruka andmestruktuuri pärimise korral võib path olla ka pikem.

* Ligipääsutoken (kui on vajalik) esitatakse http päises väljal `X-Auth-Token` näiteks nii:

```
X-Auth-Token: a8426a0-8eaf-4d22-8e13-7c1b16a9370c
```

###Ühetaoline variant

Kõik eelnimetatud parameetrid kodeeritakse kas cgi nimi=väärtus paaridena või jsoni objektis kujul {"nimi":"väärtus",…}, kasutades nimesid op, path, token, ning neid saadetakse api baas-urlile, milleks on näiteks [http://localhost/api]

* _op_ väärtus võib olla `get`, `post`, `put`, `delete` või hoopis mõni erioperatsiooni-nimi (ei ole piiratud), näiteks op=get või {"op": "get", …}
* _path_ on tabeli/objekti identifikaator, kasutades selleks variant 1 urli vastavat osa, näiteks

`path=/db/mytablename/12` või {"path":"/db/mytablename/12", …}

* * token* ligipääsuks (kui on vajalik) antakse kaasa kui token=aba... või {"token": "abab", …}

Näide:

[http://localhost/api?op=get&amp;path=db/mytable/123&amp;token=abca]

ehk samaväärselt

[http://localhost/api]urlile http postiga saadetud {"op":"get","path":"db/mytable/123","token":"abca"}

Kui päringus on korraga nii klassikalisel moel esitatud parameetreid kui ühetaolisel moel esitatud parameetreid, siis kehtivad ühetaolisel moel esitatud.

##Eripäringud

Päringute puhul, mille ülesanne klassikalise REST põhimõtte järgi ei ole selgelt määratud, tuleks anda POST meetodiga ja anda kaasa vastav "op":väärtus, näiteks "op":"specialtask", millele võivad lisanduda mistahes muud, mistahes struktuuriga parameetrid, näiteks:

[http://localhost/api]urlile http postiga saadetud

{"op":"addnums", "token":"abca", "param1":12.3, "param2": [2, 5]}

võib anda vastuse

{"result":19.3}

või isegi lihtsalt

19.3

Seejuures tuleb arvestada, et parameetrinimed token ja callback on reserveeritud nende standardkasutuseks ning nende sisuline tähendus peab olema sama, mis harilikel ülalkirjeldatud REST päringutel.

##Callback lisaparameeter

Igale päringule võib lisada callback parameetri kujul

callback=minufunktsioon

või

{"callback":"minufunktsioon",…}

mispeale pannakse tulemus-json (sh veateted) vastuses parameetriks funktsioonile minufunktsioon

Näide:

[http://localhost/api/db/mytable/123?callback=foo]

annab vastuse kujul

foo({ "value": 58.3788, "name": "lat"});

Callback on vajalik selleks, et kasutaja brauser saaks teha ajax päringuid domeenile, mis ei ole samas domeenis, kui veebileht. Eriti oluline on see arenduse ajal, kuid võib osutuda oluliseks ka lõppkasutuses.

##Edastatavad sõnumid

###Edukad päringuvastused

Ühe konkreetse kirje klassikalise päringu [http://localhost/api/db/mytable/123]  vastus on json objekt kujul

{ "value": 58.3788, "name": "lat"}

Mitme kirje päringu [https://localhost/api/db/mytable] või otsingupäringu   [https://localhost/api/db/mytable]?filter=...

vastus on json array kirjetest kujul

[{ "value": 58.3788, "name": "lat"},{ "value": 24.56, "name": "lng"}]}

Kui vastuses olev väli on omakorda json-objekt või json-array, siis esitatakse ta json kujul, mitte aga stringina, näiteks:

{ "value": 58.3788, "name": "lat", "address": {"city": "Tallinn", "street": "Gonsiori"}}

Kirjete lisamise päringuvastus on array edukalt lisatud kirjete (uutest) identifikaatoritest, näiteks

[1000]

või

[1000,1010,1011,1012]

Kirjete muutmise ja kustutamise päringuvastus on üldjuhul edukalt muudetud/kustutatud kirjete arv kujul

{"ok": N}

kus N on täisarv, mis võib olla ka 0.

Kui mingil eripäringul on keeruline või ebasoovitav anda konkreetset vastuste arvu, on edukas vastus üldjuhul selline:

{"ok": 1}

ja edutu (aga mitte veaga seotud) vastus selline:

{"ok": 0}

h3. Päringute tüübid, lubatud käsud ja lisaparameetrite kodeerimine

Üldjuhul jaotame päringu ülesanded CRUD põhimõtte järgi neljaks: andmete küsimine, uute andmete lisamine, andmete muutmine ja andmete kustutamine, pluss viienda kategooriana eripäringud. Iga päring annab alati ka mingi vastuse, sõltumatult päringutüübist.

Http get päringud ei tohi kunagi andmeid lisada ega muuta.

Päringuid võib kodeerida nii cgi formaadis nimi1=väärtus&amp;nimi2=väärtus… paaridena (väärtused sel juhul urlencoded) kui json formaadis objektidena {"nimi1":"väärtus", "nimi2":"väärtus", ...}

Päringul võivad olla lisaks käsule, objekti identifikaatorile ja tokenile ka muud parameetreid, näiteks väljastatavate kirjete maksimaalne arv, sorteering, filtrid, callback jms.

h3. Eripäringud

Lisaks CRUD päringutele on oluline kategooria eripäringud, mis ei kujuta otseselt mingite baasiandmete pärimist, muutmist või lisamist, või teostavad hulga selliseid operatsioone korraga. Eripäringud võivad teha arvutusi, statistikat, käivitada käske või protsesse vms.

Eripäringud saadetakse üldjuhul http post käsuga json objektina, kus *op* parameetri nimeks on eripäringu nimi. Kui eripäring mitte mingeid andmeid ei muuda, võib teda altenatiivina realiseerida ka http get käsuga, kasutades parameetite esitamiseks harilikku cgi nimi=väärtus kodeeringut.

Seejuures ei ole path ja token parameetrid üldjuhul kohustuslikud, kuigi neid võib kasutada. Konkreetne parameetrite hulk ja nende nimed võivad igal eripäringul olla erinevad, samuti ei ole mingeid piiranguid parameetrite tüübile/struktuurile. Näide hüpoteetilisest eripäringust:

[http://localhost/api]urlile http postiga saadetud

{"op":"specialop","param1": 23, "foo": {"lat": 12.4, "lng": 15.7}}

Eripäringute vastus peab jälgima siin dokumendis toodud veateadete põhimõtteid. Kui vastuseks on andmehulk, on soovitav jälgida siin dokumendis toodud "Edukad päringuvastused" esitatud põhimõtteid.

###Eripäringute loend

*COUNT* – päring kirjete arvu lugemiseks. Vastus on kujul {"ok":&lt;kirjete arv&gt;}

Näiteks: [http://192.168.50.106:8080/rest/api?op=count&amp;path=db/main_resource&amp;filter=name,=,prepareSignature&amp;token=testToken]

või POST päring [http://192.168.50.106:8080/rest/api]

{"op":"count","path":"db/main_resource","filter":[["service_code","=","aar.valdkonnad"]

*GETNAMES* – päring asutuste ja isikute nimede saamiseks vastavalt etteantud registri- või isikukoodile.

Vastus on kujul { "organizations": {&lt;registrikood&gt;: &lt;nimi&gt;, &lt;registrikood2&gt;: &lt;nimi2&gt;},"persons": {&lt;isikukood&gt;: &lt;nimi&gt;}}

Näiteks: POST päring [http://192.168.50.106:8080/rest/api]

{"op":"getnames", "organizations":["21345", "1234123"], "persons":["372115555", "3745555555"], "token":"testToken"}

*RESOURCE* – päring kõigi main_resource&#39;ga seotud kirjete saamiseks. Antud päring lisab vastusesse kõik Data_object&#39;id ja Document&#39;id, mille main_resource_Id võrdub päringus antud id-ga. Data_object&#39;id  lisatakse vastusesse välja, mille nimi võetakse Data_object&#39;i field_name väljast. Samamoodi toimitakse ka Document&#39;iga.

Näiteks: [http://192.168.50.106:8080/rest/api/resource/180349]

(HTTP päises peab olema X-Auth-Token määratud)

*FILE* – päring failide(dokumentide) allalaadimiseks. Parameetrina tuleb ette anda dokumendi identifikaator- /api/file/{document_id}?token={token}

Näiteks: [http://192.168.50.106:8080/rest/api/file/99567?token=testToken]

###Andmete küsimine

Võib kasutada nii http get kui post käsku.

Http get käsu puhul kodeeritakse päring cgi formaadis nimi=väärtus paaridena.

Näited:

[https://localhost/api/db/mytable/123]

või ühetaoliselt

[http://localhost/api?op=get&amp;path=db/mytable/123&amp;token=abca]

Http post käsu puhul eeldatakse parameetrite kodeeringut json formaadis kujul {"op":"get", "path":"….", …} juhul, kui päringu Content-type header sisaldab stringi "json". Vastasel korral eeldatakse parameetreid cgi formaadis nimi=väärtus paaridena. Näide eelmisega samaväärsest http post käsust andmete küsimiseks:

[http://localhost/api]urlile http postiga saadetud

{"op":"get","path":"/db/mytable/123","token":"abca"}

Lisaks pathile võib alati lisada järgmisi filter- ja sorteerimisparameetreid, kuid need ei ole kohustuslikud ja neil on vaikeväärtus:

* fields: väljade array, mida väljastada. Vaikimisi väljastatakse kõik.
* filter: array kolmik-arraydena [[field,op,value],...,[field,op,value]] mida interpreteeritakse kui and-iga seotud sql where lauset. Näide: [["lat","&gt;",53],["type","=","city"]]. Vaikimisi filtrit ei ole. Cgi formaadis antakse nii: filter=lat,&gt;,53,type,=,&#39;city&#39; kus kogu lat,&gt;,53,type,=,&#39;city&#39;  on urlencoded.

SQL like näide: [["type","like","%aa%"]]

* sort: väljanimi või väljanimi tema ees oleva -märgiga: {"sort":"lat", ...} või {"sort":"-lat", …}. Vaikimisi puudub. Cgi formaadis antakse nii: sort=lat või sort=-lat.
* offset: mitmendast kirjest hakatakse väljastama (offset kirjeid jäetakse vahele), vaikimisi 0
* limit:  maksimaalne arv väljastatavaid kirjeid. Kui puudub, eeldame, et on peal konfiguratsiooniga määratud vaikepiirang.

Näide url-encodingus ühteaoliselt esitatud päringust, kus %3E on urlencoded &gt;

[http://localhost/api?op=get&amp;path=db/mytable&amp;fields=id,value&amp;]

filter=id,%3E,1000&amp;sort=value&amp;offset=10&amp;limit=100&amp;token=test

Päringu vastus on klassikalise ühe objekti http get päringu puhul see objekt

{ "value": 58.3788, "name": "lat"}

ja mitme kirje päringu [https://localhost/api/db/mytable] või otsingupäringu   [https://localhost/api/db/mytable]

vastus on array kirjetest kujul

[{ "value": 58.3788, "name": "lat"},{ "value": 24.56, "name": "lng"}]}

###Uute andmete lisamine

Kasutada võib ainult post päringuid ja ainult json formaadis andme- ja lisaparameetreid.  Lisada võib ühe kirje json objektina või kirjete loendi json arrayna.

Näited:

[https://localhost/api/db/mytable/123] pathile klassikalisel viisil http post käsuga saadetud

{ "value": 58.3788, "name": "lat"}

või

[{ "value": 58.3788, "name": "lat"},{ "value": 24.56, "name": "lng"}]

või ühetaolisel viisil selliselt:

{"op":"post", "path": "/db/mytable", "data":{ "value": 58.3788, "name": "lat"}}

või selliselt:

{"op":"post", "path": "/db/mytable",  "data": [{ "value": 58.3788, "name": "lat"},{ "value": 24.56, "name": "lng"}]}

Kui lisatava välja väärtus on omakorda json array või json objekt, esitatakse ta json kujul, mitte stringina:

{ "value": 58.3788, "name": "lat", "address": {"city": "Tallinn", "street": "Gonsiori"}}

Päringu vastus on json array edukalt lisatud kirjete identifikaatoritest, näiteks [1000] või [1000,1002,1003]

###Andmete muutmine

Võib kasutada nii http put kui http post päringuid (viimasel juhul peab olema kasutusel ühetaoline variant, sh {"op":"put", "path":"….", ...} parameeter-väärtused) ja ainult json formaadis parameetreid. Http post võimaldab muuta mitut kirjet korraga.

Näited:

[https://localhost/api/db/mytable/123]/123 pathile klassikalisel viisil saadetud http put

{ "value": 58.3788, "name": "lat"}

või ühetaolisel viisil selliselt:

[https://localhost/api] urlile saadetud http post

{"op":"put", "path": "/db/mytable/123", "data":{ "value": 58.3788, "name": "lat"}}

Mitme kirje korraga muutmine toimub selliselt:

[https://localhost/api] urlile saadetud http post

{"op":"put", "path": "/db/mytable", "key":"id", "data":[{"id":123, "value": 58.3788, "name": "lat"},{"id":456, "value": 58.3788, "name": "lat"}]

Viimasel juhul esitab "key":"id" väljanime (näites id), mille järgi kirjeid muutmise jaoks identifitseeritakse. See väljanimi peab olema toodud järgnevates data kirjetes.

NB! Key väärtus ei pea olema  unikaalne identifikaator, seega võib üks kirje sisendis muuta mitut kirjet  baasis.

Oluline: andmetes esitatud väljad muudetakse, esitamata välju ei muudeta.

Päringu vastus on edukalt muudetud kirjete arv, näiteks {"ok": 2}. Kui kirjeid ei õnnestunud muuta, vastatakse lihtsalt {"ok": 0}

###Andmete kustutamine

Võib kasutada nii http delete kui http post päringuid (viimasel juhul peab olema antud op=delete parameeter-väärtus) ja ainult json formaadis lisaparameetreid. Http post võimaldab kustutada korra mitu kirjet.

Näited:

[https://localhost/api/db/mytable/123]/123 pathile klassikalisel viisil saadetud http delete kustutab antud kirje,

ning ühetaolisel viisil toimub ühe kirje kustutamine selliselt:

[https://localhost/api] urlile saadetud http post

{"op":"delete", "path": "/db/mytable/123"}

ja mitme kirje kustutamine selliselt:

[https://localhost/api] urlile saadetud http post

{"op":"delete", "path": "/db/mytable", "id":[123,456,777]}

kus "id" asemel kasutatakse konkreetset väljanime, millega antud tabeli kirjeid identifitseeritakse, ning selle väärtuseks on alati kustutatavate kirjete identifikaatorite array.

NB! Key väärtus ei pea olema  unikaalne identifikaator, seega võib üks kirje sisendis kustutada mitu kirjet  baasis.

Päringu vastus on edukalt kustutatud kirjete arv, näiteks {"ok": 2}. Kui kirjeid ei õnnestunud kustutada, vastatakse lihtsalt {"ok": 0}

##Sõnumite töötlemise reeglid

Sõnumite töötlemise üldised reeglid:

* http get käsuga saadetud sõnum ei tohi andmeid lisada, muuta ega kustutada, välja arvatud – potentsiaalselt – logikirjete lisamine.
* Iga sõnum moodustab ühe terviku, ning – kui välja arvata tema võimalik otsene efekt andmete muutmiseks, logi kirjutamiseks jms – ei tohi tekitada kõrvalefekte, mis võivad mõjutada järgmiste sõnumite tähendust ja nende töötlemist.

##Veateadete kirjeldused

Veateade esitatakse alati json objekti kujul vastusena, kus on alati vähemalt kaks välja:

* errcode, mis on üldistav vea iseloomustaja ning mida saab kasutata näiteks kasutajaliideses sobiva veateksti näitamiseks, ning
* errmsg, mis on üldjuhul arendajale arusaadav veateade ilma stack traceta

ning lisaks võib soovi korral kasutada kolmandat:

* errtrace, mis on tehniline stack trace debugimiseks

Näide:

{"errcode": 2, "errmsg": "arusaamatu parameeter foo"}

Seejuures errcode tähendus on järgmine:

Puhttehnilised vead, mis üldjuhul ei ole sisendiga seotud:

1: arusaamatu/klassifitseerimata viga

2: konfiguratsiooniviga

3: timeout

4: andmebaasi ühenduse loomise viga

…. varuks ….

Sisendi esitusega seotud vead:

10: tundmatu/vale http content-type

11: arusaamatu http käsk või op parameetri väärtus

12: süntaktiliselt vigane sisend

13: puuduvad vajalikud sisendparameetrid

14: tundmatud sisendparameetrid

15: sisendparameetril vale väärtustüüp

16: sisendparameetri pathi ei leitud

17: ligipääsuõigus puudub

18: andmebaasi päringu formaadi viga

… varuks muud universaalsed tehnilised vead ….

100: ja edasi:  spetsiifilised vead, mida rakendus võib kodeerida vabalt

Veateate puhul võib Http vastuskood olla seejuures kas

* 200 – OK - mida võib kasutada ka mittetehniliste vigade korral, kus ei ole päris sobivat järgnevat Http vastuskoodi, või mõni neist klassikalistest Http vastuskoodidest:
* 400 - Bad Request – mingi sisendiprobleem,
* 403 – Forbidden – ei ole õigust antud toimingut teha,
* 404 - Not Found – api pathi ei leitud,
* 500 - Internal Server Error – tehniline viga serveri pool

NB! json veateade tuleb esitada vastuskoodist sõltumatult.

##Olekudiagrammide kirjeldused

Kuivõrd kogu protokoll on http REST põhine, siis server ei hoia olekut, ning  andmete liiklus toimub alati päring – vastus paarina.

##Disaini kaalutlused

Peamised kaalutlused protokolli disainil:

* Lihtsus.
* Sobivus andmebaasioperatsioonide tegemiseks brauseri javascripti-rakendusest.
* Lähtumine REST liideste põhimõtetest.
* Võimalus laiendada traditsioonilisi REST get, post, put delete käske mistahes erikäskudega.
* Võimalus lisada, muuta ja kustutada korraga mitut kirjet.
* Võimalus lisada traditsioonilistele REST käskudele täiendavaid parameetreid.

##Vastavusklausel

Rakendus peab protokolliga kooskõlas olemiseks täitma kõiki ülaltoodud nõudeid. Seejuures tasub tähele panna, et eripäringute jaoks kehtestatud nõuded on väga nõrgad, st eripäringuid võib koostada küllalt vabalt.

##Vastavusmudeli kirjeldus

Spetsiaalsed profiilid puuduvad.

##Muutelugu
| Versioon | Kuupäev | Autor | Märkused |
|----------|---------|-------|----------|
| 1.3      | 18.04.2016 | T. Tammet | Esimene versioon |
| 1.4 | 24.08.2016 | M. Maarand, Marko Aid | Lisatud eripäringud COUNT, GETNAMES, RESOURCE, FILE. Täiendatud "like" tüüpi päringusüntaksit. Väiksemad veaparandused |