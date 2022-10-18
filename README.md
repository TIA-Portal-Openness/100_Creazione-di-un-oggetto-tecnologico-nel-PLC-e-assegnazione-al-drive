# Creazione-di-un-oggetto-tecnologico-nel-PLC-e-assegnazione-al-drive
Semplice esempio TIA Portal Openness che esegue le seguenti operazioni:

- Aggancio all'istanza TIA Portal e al progetto attualmente aperto
- Inserimento di una CPU S7-1513
- Assegnazione dell'indirizzo IP 192.168.0.10 al PLC
- Inserimento di un drive G120 integrato
- Assegnazione dell'indirizzo IP 192.168.0.11 al drive
- Creazione di una rete ethernet nel progetto
- Collegamento dei due dispositivi alla rete appena creata
- Creazione di una sottorete PROFINET assegnata al PLC S7-1513
- Assegnazione del drive alla sottorete appena creata e quindi al PLC
- Creazione di un oggetto tecnologico SpeedAxis
- Rilevamento dell'indirizzo di IO del telegramma del drive
- Assegnazione dell'oggetto tecnologico al drive
