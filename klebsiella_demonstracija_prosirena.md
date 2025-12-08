# Идентификација хипервирулентних сојева *Klebsiella pneumoniae* на командној линији

## 0. Циљ демонстрације

Циљ демонстрације је да, на примерима неколико генома *Klebsiella pneumoniae*, прикажемо како да:

1. аутоматски проценимо да ли је сој **хипервирулентан** или „класичан“ помоћу CLI алата **Kleborate**, и  
2. ручно проверимо присуство кључних гена за хипервирулентност у анотацијама генома, помоћу `grep`.

Фокус је истовремено на:

- концептуима хипервирулентности (*hvKp* vs. *cKp*),  
- радном току у Linux-у (директоријуми, `conda`, CLI алати, `grep`, `head`, `cut`, `column`),  
- разлици између „црне кутије“ (Kleborate) и здраворазумског, ручног проверавања.

---

## 1. Кратак увод: „класичне“ vs. хипервирулентне *Klebsiella pneumoniae*

### 1.1 Клинички и геномски критеријуми

- „**Класичне**“ *K. pneumoniae* („cKp“) најчешће су повезане са **болничким инфекцијама** и често су мултирезистентне; обично захтевају компромитован имунитет домаћина.  
- „**Хипервирулентне**“ *K. pneumoniae* („hvKp“) изазивају **тешке, често преносиве инфекције код иначе здравих особа** (нпр. апсцеси јетре, менингитис, ендофталмитис).  

Историјски, hvKp су се сматрале осетљивим на антибиотике, али све чешће видимо **конвергенцију резистентности и хипервирулентности** (нпр. carbapenem-резистентни hvKp).  

### 1.2 Генетичка основа хипервирулентности

Кључно откриће последње деценије је да се хипервирулентност често може објаснити присуством **великог вирулентног плазмида**, који носи специфичне гене:  

- сидерофорски локуси:  
  - *iuc* (аеробактин) – *iucA* као репрезентативни ген;  
  - *iro* (салмохелин) – *iroB* као репрезентативни ген;  
- регулатори хипермуковискозности/капсуле:  
  - *rmpA*, *rmpA2*;  
- транспортни ген *peg-344* (функционално мање јасан, али одличан маркер).  

У више студија показано је да присуство барем дела овог панела, а нарочито **свих пет гена *iucA, iroB, peg-344, rmpA, rmpA2***, веома поуздано разликује hvKp од cKp (дијагностичка тачност >0,95).  

### 1.3 Kleborate „вирулентни скор“ (0–5)

Алат **Kleborate** је развијен да, између осталог, типизира *K. pneumoniae* по вирулентним локусима и да сваком изолату додели **вирулентни скор 0–5** на основу присуства три главна локуса: *yersiniabactin* (*ybt*), *colibactin* (*clb*) и *aerobactin* (*iuc*).  

Схема:

- 0 – нема ни *ybt*, ни *clb*, ни *iuc*;  
- 1 – присутан само *ybt*;  
- 2 – *ybt* + *clb* (или само *clb*);  
- 3 – присутан *iuc* без *ybt/clb*;  
- 4 – *iuc* + *ybt* (без *clb*);  
- 5 – *iuc* + *ybt* + *clb*.  

Практично:

- **скор 0–1:** типично „класични“ или умерено вирулентни сојеви;  
- **скор ≥3:** индикација да је присутан *aerobactin* (*iuc*), што је снажан генетички маркер хипервирулентности;  
- **скор 4–5:** „класични“ профил високовирулентних плазмида hvKp.  

---

## 2. Подаци и структура директоријума

За демонстрацију претпостављамо:

- за сваки сој имате:
  - геномску FASTA датотеку (нпр. `GCF_000016305.fna`),  
  - анотацију у GFF3 формату (нпр. `GCF_000016305.gff`);  
- вeћи број сојева (конкретно 8) – неки хипервирулентни, неки не.

Предложена структура директоријума за минипројекат (демонстрацију):

```bash
data/
    genomes/
      strain1.fna
      strain2.fna
      strain3.fna
    annotations/
      strain1.gff
      strain2.gff
      strain3.gff

## 3. Инсталација Kleborate у засебном conda окружењу
### 3.1 Креирање окружења
Пример (на рачунарима где је conda већ инсталиран):

```bash
conda create -n kleborate -c conda-forge -c bioconda kleborate -y
conda activate kleborate
Проверите да је све успешно:
```

```bash
kleborate --version
kleborate --help | head
Ако се појави Usage: и листа опција – алат је спреман.
```

Напомена: Kleborate v3 користи модуларни интерфејс (опције -a, -o, -p, --trim_headers), и документација је доступна на ReadTheDocs.

## 4. Аутоматска процена хипервирулентности помоћу Kleborate-а

### 4.1 Покретање Kleborate-а на свим геномима

Претпоставимо да сте у корену репозиторијума курса, а FASTA датотеке су у data/klebsiella_demo/genomes/. Препоручујем да креирате директоријум за излаз:

```bash
mkdir results
Потом покрените Kleborate над свим FASTA-има одједном, са пресетом за K. pneumoniae species complex:
```

```bash
kleborate \
  -a data/genomes/*.fna \
  -o results \
  -p kpsc \
  --trim_headers
-a: листа асемблија (допуштен и .fasta.gz),
```

-o: директоријум за излаз;

-p kpsc: пресет за K. pneumoniae species complex,

--trim_headers: уклања назив модула из имена колона (чистији header-и).

Kleborate ће у results/kleborate/ направити излазни .txt фајл, нпр:

```bash
ls results
# очекује се нпр:
# klebsiella_pneumo_complex_output.txt
4.2 Брз преглед резултата на командној линији
bash
cd results
head -n 3 klebsiella_pneumo_complex_output.txt
Таб-делимитован фајл има велики број колона (MLST, серотип, QC, вирулентни локуси, AMR…). За потребе часа, фокусирамо се на:
```

идентификатор соја,

вирулентни скор,

присуство iuc, iro, rmpA, rmpA2.

Пример једноставног „извлачења“ релевантних колона (претпоставља се да се колоне зову strain, Virulence_score, iuc, iro, rmpA, rmpA2; у конкретној верзији Kleborate-а уверите се командом head и cut):

```bash
# Прикажи header (прва линија) као нумерисане колоне
head -n 1 klebsiella_pneumo_complex_output.txt | tr '\t' '\n' | nl -ba | head
Ово вам даје бројеве колона – рецимо да су:
```

strain → колона 1

Virulence_score → колона 10

iuc → колона 30

iro → колона 31

rmpA → колона 32

rmpA2 → колона 33

(бројеви су пример; на часу заједно налазимо тачне.)

Онда можете издвојити те колоне за све сојеве:

```bash
cut -f1,10,30-33 klebsiella_pneumo_complex_output.txt | column -t
Типичан излаз (илустрација):
```

text
strain    Virulence_score  iuc          iro         rmpA   rmpA2
strain1   4                iuc1         iro1        +      +
strain2   0                -            -           -      -
strain3   1                -            -           -      -

### 4.3 Тумачење резултата:

Који сојеви имају Virulence_score ≥ 3 (тј. поседују iuc локус)?

Да ли су ти сојеви уједно позитивни и на rmpA/rmpA2 (мукоидни фенотип)?

Који од сојева би сврстали у „класичне“ (вероватно non-hypervirulent), а који у кандидате за hvKp?

Кратак „rule-of-thumb“ за демонстрацију:

Класични сојеви (cKp): Virulence_score 0–1, iuc негативан, rmpA/rmpA2 негативни;

Хипервирулентни кандидати: Virulence_score ≥3 (присутан iuc), често и iro, rmpA, rmpA2.

### 4.4 Kleborate `resistance_score` (0–3) и мултирезистентни сојеви

До сада смо гледали само **Virulence_score** и појединачне вирулентне локусе (iuc, iro, rmpA, rmpA2). Kleborate паралелно израчунава и једноставан **резистентни скор** (`resistance_score`) који резимира присуство најважнијих β-лактамских механизама резистентности.

`resistance_score` је цео број од **0 до 3** и, поједностављено га можемо овако тумачити:

| resistance_score | Тумачење (поједностављено за час)                                           |
| ---------------- | --------------------------------------------------------------------------- |
| 0                | нема ESBL (разграђује већину "стандардних" антибиотика), нема карбапенемазе |
| 1                | присутан ESBL , али **без** карбапенемазе                                   |
| 2                | присутна карбапенемаза, али **без** колистин-резистентног генотипа          |
| 3                | карбапенемаза + колистин‑резистентни генотип                                |

Kолекцијe генома показују да изолати са **resistance_score > 0** скоро увек носе више стечених AMR гена из више класа антибиотика (типичан MDR профил). Зато можемо да користимо овај скор као грубу замену за „да/не“ процену мултирезистентности.

> **За демонстрацију на овом часу користићемо следеће правило:**
> 
> - „нерезистентни / ниско резистентни“: **resistance_score = 0**
> - „мултирезистентни (MDR)“: **resistance_score ≥ 1** (барем ESBL, а често и много више)
> - посебно забрињавајући: **resistance_score ≥ 2** (карбапенемаза, ± колистин-резистентни генотип)

---

### 4.4.1 Проналажење колоне `resistance_score` у Kleborate табели

Као и за Virulence_score, прво из header‑а нађите број колоне у којој се налази `resistance_score`:

```bash
cd results

# 1) Приказ header-а, једна колона по реду:
head -n 1 klebsiella_pneumo_complex_output.txt \
  | tr '\t' '\n' \
  | nl -ba \
  | grep 'resistance_score'
```

- `nl -ba` броји редове (1, 2, 3, …) → то су бројеви колона;
- `grep 'resistance_score'` вам каже *на ком реду/колони* је тачно тај назив.

Запишите број те колоне (нпр. може да испадне да је `resistance_score` у колони 14 – ово је само **пример**, ви користите број који заиста добијете из header‑а).

---

### 4.4.2 Заједнички преглед Virulence_score + `resistance_score`

Сада издвојте за све сојеве заједно:

- идентификатор соја (`strain`),
- Virulence_score (колону коју сте већ нашли у претходном делу),
- `resistance_score` (колону која је управо пронађена).

Пример (подесите бројеве колона према вашем header‑у):

```bash
# ПРИМЕР – бројеви колона су илустративни:
#   strain           → колона 1
#   Virulence_score  → колона 10
#   resistance_score → колона 14

cut -f1,10,14 klebsiella_pneumo_complex_output.txt | column -t
```

Типичан (измишљен) излаз може да изгледа овако:

```text
strain    Virulence_score  resistance_score
strain1   4                2
strain2   0                0
strain3   1                1
strain4   3                3
...
```

---

### 4.4.3 Питања за дискусију: ко је hvKp, а ко истовремено и MDR?

Комбинујемо све што смо до сада урадили:

1. На основу **Virulence_score** и iuc / rmpA / rmpA2 (део 4.3):
   
   - означите кандидате за **хипервирулентне сојеве** (hvKp).

2. На основу **resistance_score**:
   
   - излистајте све сојеве са **resistance_score = 0** (без ESBL/карбапенемазе);
   - излистајте оне са **resistance_score = 1** (ESBL, али без карбапенемазе);
   - излистајте оне са **resistance_score ≥ 2** (карбапенемаза ± колистин).

3. Сада укрстите ова два погледа:
   
   - који од hvKp кандидата имају **resistance_score ≥ 1**  
     → кандидати за *хипервирулентне, мултирезистентне* сојеве;
   - има ли hvKp кандидата са **resistance_score ≥ 2**  
     → кандидати за *carbapenem‑resistant hypervirulent* Kp (CR‑hvKp).

4. На крају, одаберите:
   
   - **један сој који је „само“ hvKp**  
     (висок Virulence_score, али `resistance_score = 0`),
   - **један сој који је и hvKp и MDR**  
     (Virulence_score висок, `resistance_score ≥ 1`, по могућству ≥ 2),
   
   и укратко образложите зашто су управо ти сојеви најкритичнији са клиничке тачке гледишта.

> Идеја: да видите како се у „једном реду табеле“ спајају
> 
> > (а) вирулентност (Virulence_score, iuc/rmpA/rmpA2) и  
> > (б) резистентност (`resistance_score`),  
> > и како Kleborate помаже да брзо препознате најопасније комбинације.

---

### 4.4.4 Потврда на NCBI Genomes: шта о овим сојевима кажу аутори?

До сада смо:

- из Kleborate табеле извукли **Virulence_score** и `resistance_score`,
- идентификовали **hvKp кандидате** (висок Virulence_score, iuc/rmpA/rmpA2 позитивни),
- одвојили **мултирезистентне** (resistance_score ≥ 1, посебно ≥ 2).

Следећи корак је да видите да ови сојеви нису „само бројеви у табели“, већ да су у NCBI бази заиста описани као:

- „*hypervirulent Klebsiella pneumoniae*“, и/или
- „*carbapenem-resistant hypervirulent Klebsiella pneumoniae*“ (CR‑hvKp).

Идеја је да повежете **геномски потпис**  
(iuc/iro/rmpA/rmpA2 + Virulence_score + `resistance_score`)  
са начином на који су исти ти изолати описани у оригиналним студијама и на NCBI страницама генома.

---

#### Практични кораци

1. **Изаберите два соја из сопствених резултата (део 4.4.3):**
   
   - један **хипервирулентан, али не и јако резистентан**  
     – Virulence_score висок (нпр. ≥ 3), iuc позитиван, rmpA/rmpA2 често +,  
     – `resistance_score = 0`;
   - један **хипервирулентан и мултирезистентан**  
     – Virulence_score висок (нпр. ≥ 3),  
     – `resistance_score ≥ 2` (кандидат за CR‑hvKp).

2. **Пронађите assembly accession за те сојеве у Kleborate излазу (почиње са `GCF_` или `GCA_`):**

3. **Отворите NCBI Genome / Datasets страницу за та два генома:**
   
   - у browser‑у идите на NCBI (Genomes / Datasets): [Genomes - NCBI](https://www.ncbi.nlm.nih.gov/home/genomes/)
   - у поље за претрагу налепите assembly accession (нпр. `GCF_012345678.1` – ово је само пример),
   - отворите страницу „Genome assembly“ за тај accession.

---

#### Шта да гледате на NCBI страници

На страници genome assembly‑ја (или повезане BioSample странице) потражите:

- **опис изолата** – у наслову / опису често стоје формулације типа  
  „*hypervirulent Klebsiella pneumoniae* strain …“ или скраћено „hvKp“;
- за CR‑hvKp кандидат: формулације као  
  „*carbapenem-resistant hypervirulent Klebsiella pneumoniae*“,  
  „CR‑hvKp“, „carbapenem-resistant serotype K1 hypervirulent…“ итд.;
- **линк на публикацију** (Publications / BioProject / BioSample) –  
  у апстракту/наслову често изричито стоји да је у питању hvKp или CR‑hvKp,  
  и да има хибридне плазмиде са генима за вирулентност и резистентност.

---

## 5. Ручна провера анотација помоћу grep (панел гена)

### 5.1 Панел гена за хипервирулентност

Из литературе: пет гена (iucA, iroB, peg-344, rmpA, rmpA2) показали су се као веома поуздани маркери за hvKp; присуство сва пет гена практично дефинише хипервирулентни генотип.

У овом делу:

користимо само анотационе фајлове (.gff),

и класични Unix алат grep за претрагу имена гена.

### 5.2 Пример за један геном (један GFF)

Претпоставимо да је strain1.gff анотација за први сој:

```bash
bash
cd data/annotations

grep -Ei "peg-344|iroB|iucA|rmpA|rmpA2" strain1.gff
-E омогућава regex са | (ИЛИ),

-i чини претрагу неосетљивом на велика/мала слова.

Типичан пример линија (илустрација):

... CDS ... gene=rmpA;product=transcriptional activator (mucoid phenotype regulator)
... CDS ... gene=rmpA2;product=transcriptional regulator (mucoid phenotype regulator 2)
... CDS ... gene=iucA;product=aerobactin synthase IucA
... CDS ... gene=iroB;product=salmochelin glycosyltransferase IroB
... CDS ... peg-344;product=hypothetical protein (peg-344)
```

Ако видите више оваквих погодака, то је јак сигнал да сој носи вирулентну плазмиду hvKp.

### 5.3 Аутоматизација за више генома

Ако у annotations/ имате више .gff датотека:

```bash
cd data/annotations
```

```bash
for file in *.gff; do
 echo "=== $file ==="
 grep -Ei "peg-344|iroB|iucA|rmpA|rmpA2" "$file" || echo " (ниједан маркер није пронађен)"
 echo
done
```

Могућ коментар резултата:

strain1.gff: свих пет маркера → јако вероватно hvKp;

strain2.gff: нема погодака → вероватно cKp;

strain3.gff: нпр. само iucA и rmpA2 → „непотпун панел“, али и даље јак кандидат (може бити „делимична“ или атипична плазмида).

Напomena: метод зависи од квалитета анотације. Ако се гени зову другачије (нпр. „aerobactin synthase“ без имена iucA), grep их неће наћи. Зашто нам је Kleborate (sequence-based) робуснији.

## 6. Поређење два приступа

### 6.1 Kleborate (аутоматизован, „high-level“)

Предности:

користи секвенце, није ограничен само на текст у анотацијама;

даје вирулентни скор и резиме свих локуса (ybt, clb, iuc, iro, rmpA/rmpA2) у једном кораку;

истовремено типизира и резистентност (AMR) и серотипове (K/O) – идеално за „стварни“ пројекат;

погодан за high-throughput (стотине/хиљаде генома).

### 6.2 grep + GFF (ручни, „low-level“)

Предности:

користи само стандардне Unix алате (grep, петље у bash-у);

потпуно транспарентан – директно се виде линије са iucA, iroB, rmpA, rmpA2, peg-344;

добар пример како просте командне линије могу да опонашају „прави“ биоинформатички алат.

Мане:

ослања се на квалитет анотације (ако је ген погрешно назван, биће пропуштен);

не даје глобални „скор“ нити контекст (нпр. да ли сој има и ybt/clb, K-тип, AMR профил).

## 8. Референце:

Kleborate – алат и документација

Lam MMC et al. A genomic surveillance framework and genotyping tool for Klebsiella pneumoniae and its related species complex. Nat Commun. 2021.

Kleborate GitHub репо: github.com/klebgenomics/Kleborate.

Kleborate v3 Usage (ReadTheDocs): опис опција -a, -o, -p kpsc, --trim_headers и излазних фајлова.

Вирулентни скор и генотиписање вирулентних локуса

Thorpe HA et al. A large-scale genomic snapshot of Klebsiella spp. isolates. Nat Microbiol. 2022 – опис скала за вирулентне и резистентне скореве.

Pathogenwatch – Kleborate virulence typing (опис локуса ybt, clb, iuc, iro, rmpA/rmpA2).

Дефиниција хипервирулентних K. pneumoniae и панел гена

Russo TA et al. Identification of Biomarkers for Differentiation of Hypervirulent and Classical Klebsiella pneumoniae. J Clin Microbiol. 2018.

Russo TA et al. Differentiation of hypervirulent and classical Klebsiella pneumoniae based on genotypic and phenotypic biomarkers. mBio / Emerg Infect Dis, 2023–2025 – потврда да панел iucA, iroB, peg-344, rmpA, rmpA2 има високу дијагностичку тачност (>0,95).

Stanton TD et al. What defines hypervirulent Klebsiella pneumoniae? 2024 – преглед вирулентних плазмида и hvKp концепта.

Прегледи о hvKp и конвергенцији резистентности и хипервирулентности

Chen T et al. Understanding carbapenem-resistant hypervirulent Klebsiella pneumoniae. 2024.

Hala S et al. The emergence of highly resistant and hypervirulent Klebsiella pneumoniae. Genome Med. 2024.
```