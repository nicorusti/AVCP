# AVCP-ANAC
Codice testato ed eseguito su python 3.4.2

Script per la conversione da xml a json dei file di contratti pubblici relativi alla legge 190/2012
La struttura dell'output in  json ricalca quella indicata  nelle [specifiche tecniche AVCP](http://www.anticorruzione.it/portal/rest/jcr/repository/collaboration/Digital%20Assets/pdf/AllCom27.05.13SpecificeTecnichev1.0.pdf )
![schema](https://cloud.githubusercontent.com/assets/11498717/7343336/afb74876-ecc0-11e4-8ca5-9fedcda4c178.png)
Il json creato si divide in tre sezioni:

	"data" 
	"metadata"
	"metrics"
* "data" contiene i dati dei contratti riutilizzando, ove possibile, la semantica dello schema AVCP
* "metadata" contiene i metadati del file xml, seguendo precisamente la semantica dello schema AVCP
* "metrics" contiene alcune statistiche sui dati contenuti nel file xml e sulla loro validità, come specificato nell'apposita sezione

#Correzione dei dati
* L'**aggiudicatario** di una gara viene aggiunto anche tra i partecipanti, qualora non fosse già presente. 
* Conversione in maiuscolo e pulizia da caratteri non alfanumerici di **cig** e **codici fiscali/p.iva**. Controllo della corrispondenza di cig e codice fiscale /p.iva alle specifiche (cig=10 char alfanumerici, p.iva=11 cifre, c.f.=lettere/numeri secondo specifiche). Controllo che cig, c.f./p.iva non siano valorizzati rispettivamente con "0000000000" e "00000000000". NON viene eseguito alcun check sulla correttezza del carattere di controllo di c.f. e P.Iva. 
* **Partecipanti/aggiudicatari** privi di dati validi ( intestazione e p.iva/c.f vuoti.) non vengono aggiunti al json 
* In un raggruppamento, se la stringa relativa al **ruolo** non corrisponde alle specifiche dello [schema XSD](http://dati.avcp.it/schema/TypesL190.xsd), viene inserita la stringa più simile tra quelle previste dallo schema. in questo caso, la stringa originale viene inserita comunque nel json con la chiave "ruoloOriginal". 
* Nello stesso modo viene processato il campo **sceltaContraente**: sotto la chiave "sceltaContraente" viene inserita la stringa più simile tra quelle previste dallo schema xsd, la chiave  "sceltaContraenteOriginal", contenente la stringa originale, viene aggiunta se essa non risponde alle specifiche del suddetto [schema XSD](http://dati.avcp.it/schema/TypesL190.xsd)
* Il campo **data** viene ripulito dai caratteri non numerici o non "/", inoltre viene ripossa la parte relativa al fuso orario, ove presente, di modo che la data sia conforme a xsd:date --> "yyyy-mm-dd"
* I campi contenenti un **importo** vengono ripuliti dai caratteri non numerici o non '.' e ',' e la ',' viene convertita in '.'
		
#Aggiunta di campi aggiuntivi in caso di campi errati/mancanti: 
+ Per ciascun partecipante/agiudicatario: campo **"type"**, valorizzato con  "partecipante" o  "raggruppamento", in modo da poter trattare i dati del partecipante o raggruppamento in modo diverso. 
+ Per ciascuna gara, è presente il campo **"cigValid"** valorizzato con true / false
+ Per ciascuna gara con CIG non valido o nullo (casi di gare per le quali non è prevista l'assegnazione del cig): campo **"cigHash"**  in forma di stringa esadecimale [sha1](http://en.wikipedia.org/wiki/SHA-1), costruito in base a: 
	* Codice Fiscale della struttura proponente
	* Importo Aggiudicazione
	* Scelta Contraente (tipo di procedura di aggiudicazione) 
	* Codice Fiscale(di seguito C.F.)/p.iva dell'aggiudicatario. Nel caso l'aggiudicatario sia un raggruppamento, l'hash è costruito in base ai C.F./p.iva in ordine alfabetico
+ Per ciascun raggruppamento: campo **"groupHash"** in forma di stringa esadecimale [sha1](http://en.wikipedia.org/wiki/SHA-1), costruito in base a: 
	* C.F./p.iva, in ordine alfabetico, dei membri del raggruppamento. 
	* cig o cigHash. Per ogni gara, quindi, un raggruppamento formato dalle stesse aziende, ottiene hash diversi. Questo affinché il raggruppamento corrisponde alla definizione giuridica di Associazione Temporanea di Impresa
* Aggiunta campi **"sceltaContraenteOriginal"** e  **"ruoloOriginal"** nei casi in cui questi campi nel xml siano valorizzati con stringhe non previste dallo  [schema XSD](http://dati.avcp.it/schema/TypesL190.xsd)
 
#Conteggi e metriche: 
Per ciascun file xml, sotto la chiave "metrics" sono disponibili delle metriche per i seguenti campi: 
* ciascuno dei possibili campi **sceltaContraente"** presenti nello  [schema XSD](http://dati.avcp.it/schema/TypesL190.xsd)
	esempio: 

		"01-PROCEDURA APERTA": {
            "nInvalid": 0,
            "nValid": 0,
            "totalAwardedPrice": 0,
            "totalPaidPrice": 0
            }
	* "nValid" indica il n. delle gare in cui il campo sceltaContrante è presente e risponce allo schema xsd
	* "nInvalid" indica le occorrenze in cui il campo sceltaContraente è simile ad uno dei campi nello schema xsd. In questo caso il campo scelto è quello con similitudine maggiore 
	* "totalAwardedPrice" indica la somma degli importi aggiudicati per singolo tipo di procedura di aggiudicazione
	* "totalPaidPrice" indica la somma degli importi liquidati per singolo tipo di procedura di aggiudicazione 
 Qualora il campo sceltaContraente sia assente, le seguenti misure sono riportate: 
	esempio:

		"unknownProcType": {
            "nValid": 0,
            "totalAwardedPrice": 0,
            "totalPaidPrice": 0
            }


#Istruzioni: 
Usare le seguenti funzioni presenti in main():

	getFileNameFromCommandLine()
	
> Converte singoli file xml, già presenti nella stessa cartella dove viene eseguito lo script, in formato json


	downloadAndParseEverything("http://url_file_indice.xml")
	
> Scarica e parsifica il file di indice e tutti i file da esso linkati.
Passare alla funzione l'url del file xml di indice. I file xml e json sono salvati nella cartella download\nome_istituzione\anno , dove nome_istituzione ed anno sono presi dal 	file di indice. 
Qualora esista già il percorso download\nome_istituzione\anno, i file presenti non sono sovrascritti ma salvati nella cartella download\nome_istituzione\anno_1 



