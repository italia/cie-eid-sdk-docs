.. _sec-sdk-android:

SDK Android
===========

La versione del SDK per Android, \**CieID-android-sdk**, è disponibile
a
`questo link <https://github.com/italia/cieid-android-sdk>`__. É
costituita da una libreria software, realizzata in codice nativo Android
Kotlin, e integra una app di esempio che descrive le diverse modalitá di
integrazione dello schema «Entra con CIE»:

Requisiti di integrazione
~~~~~~~~~~~~~~~~~~~~~~~~~

L'utilizzo dell'SDK presuppone che il Service Provider sia correttamente
federato con l'Identity Provider e che abbia implementato almeno uno tra
i protocolli previsti dallo schema di autenticazione «Entra con CIE”.

Inoltre, è necessario che i seguenti requisiti siano soddisfatti:

-  versione Android 6.0 (API level 23) o successive;

-  utilizzo di un dispositivo mobile dotato di tecnologia NFC;

-  disponibilità di una connessione internet

Configurazione
~~~~~~~~~~~~~~

Dell'IdP sono messi a disposizione degli erogatori di servizi due
ambienti: uno di **preproduzione**, per gli sviluppi applicativi e i
test di federazione e l'altro di **produzione**, per il deploy in
esercizio. Difatti, per l'impiego di un'app terza con uno dei flussi
disponibili è necessaria una fase iniziale di configurazione, che
dipende dal tipo di flusso adottato.

Entrambi i flussi vengono avviati tramite l'utilizzo di una *Webview*: é
necessario caricare la URL del Service Provider che integra il pulsante
«Entra con CIE» come mostrato nell'esempio:

..  code-block:: java

	*//inserire url service provider*
	webView.loadUrl("URL del Service Provider")


**Flusso con reindirizzamento**

Nel caso di *flusso con reindirizzamento*, per far proseguire
correttamente il flusso, è necessario selezionare l'applica- zione
*«CieID»* a cui indirizzare le richieste di autenticazione. Ciò può
essere fatto modificando i commenti dalle righe di interesse, come
mostrato di seguito.

..  code-block:: java

	val appPackageName = "it.ipzs.cieid"
	*//COLLAUDO*
	*//val appPackageName = "it.ipzs.cieid.collaudo"*


**Flusso integrato**

Per quanto riguarda il *flusso integrato*, invece, la fase di
autenticazione viene gestita dalla libreria software. In questo caso é
necessario integrare il modulo «CieIDSdk».

L'SDK utilizza *Gradle* con strumento di build automatico. Per
configurare correttamente il flusso, é necessario selezionare l'ambiente
server dell'Identity Provider a cui indirizzare le richieste di
autenticazione. Ció puó essere fatto modificando il file *build.gradle*
modificando i commenti dalle righe di interesse, come mostrato di
seguito:

..  code-block::java

	*//AMBIENTI:*
	*//Ambiente di produzione*
	*//buildConfigField "String", "BASE_URL_IDP",
	"\"https://idserver.servizicie.interno.gov.it/idp/"\"*


	*//Ambiente di collaudo*
	*//buildConfigField "String", "BASE_URL_IDP",
	"\"https://preproduzione.idserver.servizicie.interno.gov.it/idp/"\"*



Modalità di integrazione
~~~~~~~~~~~~~~~~~~~~~~~~

L'SDK fornisce una app di esempio, con 2 activity, una per flusso, per
facilitare al Service Provider l'integrazione all'interno della propria
App. La gestione degli errori è demandata all'app integrante.

**Integrazione del flusso con reindirizzamento**

Per integrare nativamente le funzionalità dell'SDK é necessario, per
prima cosa, intercettare la URL contenente il valore «/OpenApp» ed
avviare l'App CieID integrando il codice seguente:

..  code-block:: java

	val intent = Intent()
	**try** {

		intent.setClassName(appPackageName, className)
		*//settare la url caricata dalla webview su /OpenApp*
		intent.data = Uri.parse(url)
		intent.action = Intent.ACTION_VIEW
		startActivityForResult(intent, 0)
	} **catch** (a : ActivityNotFoundException) {
		startActivity(

			Intent(

				Intent.ACTION_VIEW,

				Uri.parse("https://play.google.com/store/apps/details?id=$appPackageName")

			)

		)

	)

	**return true**


Una volta avviata correttamente l'App CieID, avviene l'autenticazione
tramite la CIE, e al termine viene restituita una nuova URL da ricarica
nella WebView precedente, come mostrato nell'esempio seguente:

..  code-block:: java

	override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {

		**super**.onActivityResult(requestCode, resultCode, data)
		val url = data?.getStringExtra(URL)

		webView.loadUrl(url)

	}



**Integrazione del flusso integrato**

Per integrare le funzionalità dell'SDK si utilizzano i seguenti metodi:

..  code-block:: java

	*//Configurazione iniziale*

	CieIDSdk.start(activity, callback)

	*//Avvio utilizzo NFC*

	CieIDSdk.startNFCListening(activity)

	*//Abilitare o disabilitare i log, da disattivare in produzione*

	CieIDSdk.enableLog = **true**

	*//Bisogna settare la url caricata dalla pagina web dell' SP dalla
	webview su /OpenApp*

	CieIDSdk.setUrl(url.toString())

	*//inserire il pin della CIE*

	CieIDSdk.pin = input.text.toString()

	*//Avviare NFC*

	startNFC()


É necessario, inoltre, realizzare le interfacce di Callback
implementando i seguenti metodi:

..  code-block:: java

	override fun onEvent(event: Event) {

	*//evento*

	}

	override fun onError(e: Throwable) {

	*//caso di errore*

	}

	override fun onSuccess(url: String) {

	*//caso di successo con url della pagina da caricare*

	}

