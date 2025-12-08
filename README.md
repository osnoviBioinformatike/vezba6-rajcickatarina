# Vežba 6

## Struktura repozitorijuma

`README.md` (ovaj fajl) - sadrži uputstva potrebna za izradu vežbe, uključujući **početni deo teksta zadatka**.

`klebsiella_demonstracija_prosirena.md` - rešeni primer mini-projekta dobijen u okviru predavanja; sadrži **preostali deo teksta zadatka** (mini-projekta) zajedno sa primerima komandi za izvršavanje.

`za_vezbe_Klebsiella_sekvence.tar.gz` - koprimovana arhiva sa podacima/fajlovima za zadatak.

`Outputs` folder - sadrži `komande.md` Markdown fajl u koji je potrebno uneti tražene komande ili *output* komandi iskorišćenih za rešavanje zadatka. U samom fajlu je naznačeno šta tačno treba uneti. U folderu Outputs treba da se nađe samo ono što je konkretno traženo, odnosno što će biti pregledano (u slučaju ove vežbe, samo popunjen `komande.md` fajl). 

- Pre početka rada na zadacima, otvorite `komande.md` iz `Outputs` foldera pomoću MarkText editora.

---

## Zadatak 1 - *Klebsiella* mini-projekat

Terminologija

- <u>**cKp** - klasične *K. pneumoniae*</u>: najčešće povezane sa bolničkim infekcijama, često su multirezistentne
- <u>**hvKp** - hipervirulentne *K. pneumoniae*</u>: izazivaju teške, često prenosive infekcije kod inače zdravih osoba
- <u>Markeri hipervirulentnosti</u>: prisustvo gena ***iucA, iroB, peg-344, rmpA, rmpA2*** (barem dela ovog panela, a naročito svih pet), veoma pouzdano razlikuje hvKp od cKp
- <u>Alat **Kleborate**</u>: razvijen da, između ostalog, tipizira *K. pneumoniae* na osnovu prisustva tri glavna lokusa - *yersiniabactin* (*ybt*), *colibactin* (*clb*) i *aerobactin* (*iuc*); svakom izolatu dodeljuje **virulentni skor 0–5**
1. Proverite da li je na računaru prisutno conda okruženje *kleborate* i u njemu instaliran alat Kleborate:

```bash
conda activate kleborate
kleborate --version
kleborate --help | head
# Ako se pojavi Usage: i lista opcija – alat je spreman.
```

2. Ako nema okruženja/Kleborate nije instaliran - napravite okruženje i instalirajte ga:

```bash
conda update conda # prvo apdejtujte condu iz base okruženja

conda create -n kleborate -c conda-forge -c bioconda kleborate -y
conda activate kleborate # napravite okruženje i instalirajte Kleborate
```

3. Nakon instaliranja, ponovite komande pod (1) za testiranje okruženja.

4. Premestite se u koren repozitorijuma vežbe (*vezba-6-PeraPeric*, tj. folder sa istim nazivom kao na GitHub-u).

5. Raspakujte komprimovanu datoteku.

6. Premestite se u folder sa .fna (genomske sekvence u FASTA formatu za različite sojeve) i .gff datotekama (fajlovi sa anotacijama za svaki soj).

7. Odatle, jednom komandom napravite folder *data* u korenu repozitorijuma (*vezba-6-PeraPeric*), sa pod-folderima *genomes* i *annotations*, u koje ćete u sledećim koracima premestiti fajlove i dobiti strukturu direktorijuma šematski prikazanu ispod. Dakle, folder *data* treba da bude folder Git repozitorijuma *vezba-6-PeraPeric*, i treba da sadrži 2 foldera - *genomes* i *annotations*. Prilikom kreiranja foldera iskoristite opciju `-p` da zadate da se odjednom kreiraju navedeni folderi višeg i nižeg nivoa.

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
```

8. Premestite sve .fna fajlove u folder *genomes*, a .gff fajlove u folder *annotations*.

9. Vratite se u koren repozitorijuma vežbe (*vezba-6-PeraPeric*), pa kreirajte direktorijum *results* za čuvanje izlaznih fajlova alata Kleborate (dakle, *results* treba da bude folder Git repozitorijuma *vezba-6-PeraPeric*, kao što je i folder *data*).

10. Dalje nastavite analizu prema instrukcijama za pokazni primer sa predavanja: od pokretanja Kleborate-a, sve do kraja dokumenta.

---

## Zadatak 2 - preuzimanje genoma i integritet fajlova


1. Sve informacije o zadatom Klebsiella genomu se nalaze na URL="https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/021/057/265/GCF_021057265.1_ASM2105726v1/". Fajlovi od interesa su:

```bash
GCF_021057265.1_ASM2105726v1_genomic.fna.gz
GCF_021057265.1_ASM2105726v1_genomic.gff.gz 
md5checksums.txt
uncompressed_checksums.txt
```
2. Napravite folder *klebsiella_genome* u korenu repozitorije i udjite u taj folder.

3. Pomoću *wget* komande preuzmite 3 prethodno navedena fajla.

4. Raspakujte fajl *GCF_021057265.1_ASM2105726v1_genomic.fna.gz* i *GCF_021057265.1_ASM2105726v1_genomic.gff.gz* ali zadržite i neraspakovane verzije.

5. Izdvojte md5checksum za zadati genom iz ./md5checksums.txt i sačuvajte ga u odvojenom *genome_compressed.txt* fajlu.

6. Koristeći *md5sum* komandu proverite integritet za zadati genom (kompresovanu verziju). 


## Zadatak 3 - pravljenje arhive od *klebsiella_genome* foldera


1. U *klebsiella_genome* folderu napravite foldere *genomes* i *annotations*

2. Premestite raspakovane fajlove *GCF_021057265.1_ASM2105726v1_genomic.fna.gz* i *GCF_021057265.1_ASM2105726v1_genomic.gff.gz* redom u *genomes* i *annotations* foldere.

3. Vratite se u *root* folder repozitorijuma.

4. Kreirajte *klebsiella_archive.tar.gz* od *klebsiella_genome* foldera.

5. Proverite sadrzaj *klebsiella_archive.tar.gz* bez raspakivanja arhive.


## Zadatak 4 - automatizacija preuzimanja genoma i pravljenja arhive

1. Napravite folder *scripts* u korenu repozitorije i skript fajl *klebsiella_download_archive.sh* u *scripts*.

2. Udjite u scripts folder.

3. Skript treba da automatizuje sve korake iz prethodna dva zadatka (zadatak 2. i 3.) osim 2.5 i 2.6. Prethodno korišćene komande unesite u *klebsiella_download_archive.sh* fajl.

4. Skript *klebsiella_download_archive.sh* treba da se pokreće iz *skripts* foldera kako bi svi automatski kreirani folderi i fajlovi bili u okviru *scripts* foldera čime izbegavamo sukob sa već postojećim folderima i fajlovima iz zadataka 2 i 3.

5. Primer finalnog sadržaj arhive koju skript treba da kreira:

```bash
./klebsiella_genome/
./klebsiella_genome/annotations/
./klebsiella_genome/annotations/GCF_021057265.1_ASM2105726v1_genomic.gff
./klebsiella_genome/GCF_021057265.1_ASM2105726v1_genomic.fna.gz
./klebsiella_genome/GCF_021057265.1_ASM2105726v1_genomic.gff.gz
./klebsiella_genome/genomes/
./klebsiella_genome/genomes/GCF_021057265.1_ASM2105726v1_genomic.fna
```