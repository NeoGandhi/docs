# Dokument für Exchange Entwickler
Dieses Dokument soll Exchange-Entwicklern bei der Einrichtung von NEO-Knoten auf dem Exchange-Server helfen und die zugehörige Programmentwicklung für Transaktionen mit NEO-Assets abschliessen. Vergewissern Sie sich dass Sie das [NEO White Paper](index.html) gelesen und das Hintergrund wissen über NEO verstanden haben, bevor Sie weiterlesen.

Im Allgemeinen muss ein Exchange folgende Schritte ausführen:

- [Bereitstellen einer NEO Node auf dem Server](#deploying-a-neo-node-on-server)
- [Eine Wallet und Adressen für Einzahlungen erstellen](#creating-a-wallet-and-deposit-addresses)
- [Umgang mit Global Asset Transaktionen](#dealing-with-global-assets-transactions)
- [Umgang mit NEP-5 Asset Transaktionen](#dealing-with-nep-5-assets-transactions)
- [(Optional) Verteilung von GAS an die User](#optional-distributing-gas-to-users)

## Bereitstellen einer NEO Node auf dem Server

Führen Sie die folgenden Schritte aus um eine NEO Node auf dem exchange Server bereitzustellen

1. Installieren Sie [.NET Core Runtime](https://www.microsoft.com/net/download/core#/runtime) auf dem Server, Version 1.0.1 oder neuer.
2. Laden Sie [Neo-CLI](https://github.com/neo-project/neo-cli/releases) von GitHub herunter und aktivieren Sie NEO Node

Mehr Informationen finden Sie unter [Installation und Bereitstellen einer NEO Node](node/setup.html).

## Eine Wallet und Deposit Adressen erstellen

### Über NEO-CLI

NEO-CLI ist ein Command-Line Client (Wallet) für Entwickler. Sie haben zwei Möglichkeiten damit zu interagieren:

- Verwenden von CLI (Command-Line Interface) Befehlen. Sie können z.B. eine Wallet erstellen, Adressen generieren, etc.
- Verwenden von Remote Procedure Call (RPC). Sie können z.B. Transfers auf eine angegebene Adresse machen, Block Informationen einer angegebenen Block-Höhe erlangen, Informationen eines angegebenen Trade erhalten, etc.

NEO-CLI stellt folgende Features zur Verfügung:

- Als Wallet, verwaltet es Assets über die Command-Line
  Geben Sie folgenden Befehl in der NEO-CLI directory ein um die Wallet zu aktivieren:
  
  ```
  dotnet neo-cli.dll
  ```
  
  Geben Sie folgenden Befehl ein um alle verfügbaren Befehle zu überprüfen:
  
  ```
  help
  ```
  
  Mehr Informationen finden sie unter [CLI Command Reference](node/cli.html).
  
- Stellt API's zur Verfügung um Blockchain Daten von den Nodes zu erhalten. Die Schnittstellen werden über [JSON-RPC](http://www.jsonrpc.org/specification) bereitgestellt, und die zugrunde liegende Kommunikation verwendet HTTP / HTTPS-Protokolle.

  Um eine Node zu starten, die den RPC-Dienst bereitstellt, geben Sie den folgenden Befehl in dem NEO-CLI Directory ein:
  
  ```
  dotnet neo-cli.dll /rpc
  ```
  Mehr Informationen zu API's finden sie unter [API Reference](node/api.html).
  
### Erstellen einer Wallet

Der Exchange muss eine Online Wallet erstellen um die Deposit Adressen der User zu verwalten. Eine Wallet dient dazu die Informationen von Accounts (Sowohl Public- als auch Private Keys) und Contracts zu verwalten. Es ist der wichtigste Beweis den ein User hält. User müssen die Wallet Files und die Wallet Passwörter sicher aufbewahren. Sie dürfen diese Daten weder verlieren noch offen legen.

> [!Anmerkung]
>
> Exchanges müssen nicht für jede Adresse eine Wallet erstellen. Ein Online Wallet hält normalerweise alle Deposit Adressen für User. Eine Cold Wallet (Offline Wallet) ist eine weitere Speicheroption, welche bessere Sicherheit gewährleistet.

Führen Sie folgende Schritte aus um eine Wallet zu erstellen:

1. Geben Sie `create wallet <path>` ein.
   <path> ist der Name des Pfad der Wallet und der Wallet-Datei.  Die Dateierweiterung kann einen beliebigen Typ aufweisen, z. B. create wallet mywallet.db3.
  
2. Legen Sie ein Passwort für die Wallet fest.

> [!Anmerkung]
>  
> Die Wallet muss ständig geöffnet bleiben, um auf Abhebungsanfragen von Usern zu antworten. Aus Sicherheitsgründen sollten die Wallets auf einem unabhängigen Server ausgeführt werden, auf dem die Firewall ordnungsgemäss konfiguriert ist (siehe unten).
  
  |                    | Mainnet | Testnet |
| ------------------ | ------- | ------- |
| JSON-RPC via HTTPS | 10331   | 20331   |
| JSON-RPC via HTTP  | 10332   | 20332   |
| P2P                | 10333   | 20333   |
| websocket          | 10334   | 20334   |

### Deposit-Adressen generieren

Eine Wallet kann mehrere Adressen speichern. Der Exchange muss für jeden User eine eigene Deposit-Adresse generieren.

Es gibt zwei Methoden umd Deposit-Adressen zu generieren:

- Wenn der User zum ersten Mal eine Einzahlung (NEO/NEO GAS) tätigt, generiert das Programm dynamisch eine neue Neo-Adresse. Der Vorteil besteht darin, dass keine Adressen in festen Zeitintervallen generiert werden müssen, während Sie den Nachteil haben, dass Sie kein Backup der Wallet machen können.

Verwenden Sie zum Entwickeln des Programms zum dynamischen Generieren von Adressen die NEO-CLI-API [getnewaddress-Methode] (node ​​/ api / getnewaddress.html). Die erstellte Adresse wird zurückgegeben.

-Der Exchange erstellt im Voraus eine Charge von NEO-Adressen. Wenn der Benutzer das erste Mal eine Einzahlung (NEO / NEO GAS) vornimmt, weist der Exchange ihm eine NEO-Adresse zu. Der Vorteil dieser Methode ist, dass Sie ein Backup der Wallet machen können, während der Nachteil ist, dass Sie NEO-Adressen manuell generieren müssen. Um eine Charge von Adressen zu generieren, führen Sie den NEO-CLI-Befehl `create address [n]` aus. Die Adressen werden automatisch in die Datei address.txt exportiert. [n] ist optional. Der Standardwert ist 1. Um zum Beispiel 100 Adressen zu erzeugen, geben Sie `create address 100` ein.

> [!Anmerkung]
>
> Bei beiden Methoden muss der Exchange die Adressen in die Datenbank importieren und an die User verteilen.

## Umgang mit Global Asset Transaktionen

### Programme entwickeln für Ein- und Auszahlungen von Usern

Für Global Assets muss der Exchange Programme entwickeln, um die folgenden Funktionen zu erfüllen:

1. Überwachen Sie neue Blöcke über die NEO-CLI-API ([getblock-Methode] (node ​​/ api / getblock2.html)).
2. Behandeln Sie die Einzahlungen von Usern gemäss den Transaktionsinformationen.
3. Speichern Sie die Transaktionsdatensätze für den Austausch.

#### Einzahlungen von Usern

In Bezug auf Einzahlungen von Usern muss der Exchange Folgendes beachten:

- Die Neo Blockchan hat nur eine Main-Chain, ohne Side-Chains, wird nicht geforkt werden und wird keine isolierten Blöcke haben.
- Eine Transaktion, die in der NEO-Blockchain aufgezeichnet wurde, kann nicht manipuliert werden, was bedeutet, dass eine Bestätigung einen Erfolg bei der Einzahlung darstellt.
- Im Allgemeinen ist der Saldo einer Einzahlungsadresse in der Börse nicht gleich dem Saldo, den der User auf dem Exchange hat. Das kann sein, weil:
  -


  
  
  
  
  
  
  
  
  
  
  




