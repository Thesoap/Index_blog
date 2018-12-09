Postgresql og indexering
Abstract

Postgres databaser med store mængde data skaber langsomme database execute tider. Uden optimering vil postgres execute de givne statements på en ineffektiv måde som giver langsomme svartider. Med få enkelte implementeringer vil databaser med en n elementer kunne gå fra O(n) til O(log(n)). en million elementer vil kunne gå fra en million “tid” til seks “tid”.

Problemstilling
Vores problem opstod da vi byggede et projekt som skulle fungere som et forum med opslag og kommentare. Alle disse opslag og kommentare blev gemt i en PostgreSQL database. Over tid opnåede vi flere millioner opslag og kommentare, hvilket gjorde vores datadase opnåede response tider langt over hvad brugere kunne forvente. 

Introduktion
Databaser er en del af mange projekter rundt om i verden. Lige meget hvilket system der skal bygges vil der altid skulle bruges en database for at indeholde alt den data som bliver brugt. I starten indeholder disse databaser selvfølgelig små mængder af data men senere hen i projektet levetid vil databaserne ende med at indeholder “big data”. 
Databaser kan hente flere tusind elementer ud i sekundet men hvis du skal hente flere millioner eller bare søge mellem flere millioner elementer opnår vi response tider på flere sekunder som ikke er acceptabelt i dagens IT verden. 

Løsning
Da vi startede vores projekt kørte tingene i en fin hastighed men senere så vi nogle store response tider. Senere hen da vi opnåede flere hundrede tusinde elementer så vi response tider på flere sekunder som blev nød til at blive fixet.

Her ser vi en select statement som skal finde 1 element i vores tabel. Tabellen indeholder over 7 millioner elementer og er på ingen måde optimeret til at indeholde så store mængder af data.
Vi får altså en responstid på lidt over 44 sekunder. Ingen IT virksomheder eller brugere har tid til at vente 44 sekunder på et request. Uden optimering vil vi have en lineær search algoritme. Dette betyder at postgresql starter øverst og leder efter det element som har hanesst_id på 7 millioner. Dette kaldes også inde for data verden O(n). O(n) er slet ikke optimal nok når vi regner med så meget data som 7 millioner elementer.

For at fixe dette problem valgte vi at bruge indexes. Ved at bruge indexes kan vi opnå tidere som er flere tusind gange hurtigere uden at skulle skrive særlig meget ekstra kode.

Her bruger vi create index på vores tabel comments på den column der hedder hannest_id. Denne process tiger noget tid, i vores tilfælde omkring 1 minute. Denne process skal kun køres en gang og så er hele column indexeret. Som standard vil Postgresql bruge B-tree til at indexere som står for binary-tree. Ved det er binary vil hver gang postgresql have en search algoritme som opnår O(log n) i stedet for som er meget mere optimal for store mængder af data. Både O(n) og O(log n) regnes som worst case og ikke gennemsnittet eller medianen.

Her kan den samme statement ses men efter vi har oprettet et index på vores column. vi er gået fra 44000~ms til 2.3~ms. Dette er over 19000 gange hurtigere ved bare at skrive et statement og sætte et index på vores column.

Dog er indexes ikke kun positive. Ved at lave indexes vil ens write statements blive langsommere. Dette skyldes at hvis elementet skal også sættes ind i indexet og ikke kun ind i den normale tabel. Eftersom at indexet har en struktur vil den først skulle finde elementet placering i indexet. 
Et index er en ny tabel som bliver oprettet hvor den value som man har lavet indexet på bliver gemt. Så indexet er ligesom en indholdsfortegnelse for databasen omkring hvor den skal kigge.

I vores projekt skulle vi bruge et index på et ID. Eftersom dette ID er unique kunne vi bruge unique index. 
Dette index kan kun bruge b-tree indexering og kan kun bruges på tabeler hvor den valgte column er unique. Ved at gøre dette ved Postgresql at der ikke er flere elementer med samme værdi i indexet.


Denne indexering tager omkring 30 sekunder at sætte op. Dette er halvdelen af hvor en normal indexering tager. Dette er igen grundet af at Postgresql ved der ikke findes flere elementer med samme værdi. 

Med denne indexering får vi hurtigere tider end med vores normale indexering.Vi skær cirka ⅓ af tiden i forhold til en normal indexering bare ved at sige til postgresql at det er et unique index. Dette er omkring 28000~ gange hurtigere end ikke at sæt nogen indexering op på columnen.

Konklusion
Som det kan ses på resultaterne er index et yderst stærkt værktøj når databasen indeholder enorme mængder af data. postgresql er en super intelligent database men den kræver man selv sætter nogle ting op for at få det fulde udbytte af den. At få svar tider på flere sekunder vil aldrig kunne blive accepteret i dag når data skal flyttes rundt hele tiden og IT udvikler sig så hurtigt som det gør. Med en enkelt forståelse for hvordan databaser læser data og hvordan man kan vælge hvordan den skal gøre det kan man optimere sin database mange tusinde gange i forhold til hvor stor datamængde man arbejder med. 
Man skal have en forståelse for hvordan ens database og tabels er bygget op. Jo større forståelse man har desto mere præcist vil man kunne bygge sin optimering op for at opnår hurtigere og bedre performance på ens database.
Indexering fixer ikke alle problemer i en database men er helt klart noget man skal have sat op for at behandle kæmpe mængder af data.

Information
https://www.postgresql.org/docs/9.1/indexes-types.htm
https://severalnines.com/blog/postgresql-database-indexing-overview
https://www.postgresql.org/docs/9.4/indexes-unique.html
