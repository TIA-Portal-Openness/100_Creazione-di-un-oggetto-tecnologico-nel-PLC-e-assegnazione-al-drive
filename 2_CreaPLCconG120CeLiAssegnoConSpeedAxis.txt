using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Siemens.Engineering;
using Siemens.Engineering.Hmi;

using Siemens.Engineering.SW.Tags;
using Siemens.Engineering.HW;
using Siemens.Engineering.HW.Features;
using Siemens.Engineering.SW;
using Siemens.Engineering.SW.TechnologicalObjects;
using Siemens.Engineering.MC.Drives;
using Siemens.Engineering.SW.TechnologicalObjects.Motion;

namespace ConsoleAppOraz
{
    class Program
    {
        static void Main(string[] args)
        {
            //PREPARAZIONE DELL'HARDWARE
	    //Creazione progetto
            IList<TiaPortalProcess> mieiProcessiTIAAperti = TiaPortal.GetProcesses();
            TiaPortalProcess mioProcessoTIAPortal = mieiProcessiTIAAperti[0];
            TiaPortal mioTIAPortalAperto = mioProcessoTIAPortal.Attach();
            Project mioProgettoTIA = mioTIAPortalAperto.Projects[0];
            Console.WriteLine("Progetto creato");

            // Inserimento PLC di Linea
            Device miaStazionePLCLinea = mioProgettoTIA.Devices.CreateWithItem("OrderNumber:6ES7 513-1FL02-0AB0/V2.9", "PLC_Linea", "PLC_Linea");
            //Inserimento del G120C di linea
            Device miaStazioneDriveG120Linea = mioProgettoTIA.Devices.CreateWithItem("OrderNumber:6SL3210-1KE32-4UF1/4.7.13", "Drive_Linea", "Drive_Linea");
            Console.WriteLine("PLC e drive di linea inseriti");
            
	    //Impostazione dell'indirizzo IP del PLC
            DeviceItem mioPlcLinea = miaStazionePLCLinea.DeviceItems[1];
            DeviceItem mieInterfaccePLCLinea = mioPlcLinea.DeviceItems[3];
            NetworkInterface mieiDatiInterfaccePLCLinea = mieInterfaccePLCLinea.GetService<NetworkInterface>();
            Node mio_X1_PLCLinea = mieiDatiInterfaccePLCLinea.Nodes[0];
            mio_X1_PLCLinea.SetAttribute("Address", "192.168.0.10");
            Console.WriteLine("IP a PLC assegnato");

            //Modifica Indirizzo IP G120C
            DeviceItem mioDriveG120Linea = miaStazioneDriveG120Linea.DeviceItems[1];
            DeviceItem mieInterfacceG120Linea = mioDriveG120Linea.DeviceItems[1];
            NetworkInterface mieiDatiInterfacceG120Linea = mieInterfacceG120Linea.GetService<NetworkInterface>();
            Node mio_X1_G120Linea = mieiDatiInterfacceG120Linea.Nodes[0];
            mio_X1_G120Linea.SetAttribute("Address", "192.168.0.11");
            Console.WriteLine("IP a drive assegnato");
            
            //Creo rete PN su controller
            Subnet miaSottoretePN = mioProgettoTIA.Subnets.Create("System:Subnet.Ethernet", "PN/IE");

	    //Collego entrambi alla rete appena creata
            mieiDatiInterfaccePLCLinea.Nodes[0].ConnectToSubnet(miaSottoretePN);
	    mieiDatiInterfacceG120Linea.Nodes[0].ConnectToSubnet(miaSottoretePN);            

	    //Creo il sistema profinet per il controllore	
       	    IoSystem mioSistemaPNLinea = mieiDatiInterfaccePLCLinea.IoControllers.First().CreateIoSystem("PNIO");
            
	    //Connetto il drive alla sottorete PROFINET appena creata e quindi al PLC
            mieiDatiInterfacceG120Linea.IoConnectors[0].ConnectToIoSystem(mioSistemaPNLinea);
            Console.WriteLine("Drive agganciato a PLC");            

            //CREAZIONE E COLLEGAMENTO OGGETTO TECNOLOGICO
	    //Creo l'oggetto c# relativo al software per il PLC 
            SoftwareContainer mioSoftware = mioPlcLinea.GetService<SoftwareContainer>();
            PlcSoftware mioSoftwarePLC = mioSoftware.Software as PlcSoftware;
            
            //Identificazione dell'indirizzo di IO del telegramma del G120
            DriveObjectContainer mioSWDrive = mioDriveG120Linea.GetService<DriveObjectContainer>();
	    DriveObject mioOggettoDrive = mioSWDrive.DriveObjects[0];
	    Telegram mioTelegramma1_G120 = mioOggettoDrive.Telegrams[0];
            Address mioIndirizzoTelegramma1 = mioTelegramma1_G120.Addresses[0];
            int mioIndirizzoByteIniziale = mioIndirizzoTelegramma1.StartAddress;

            //Creazione oggetto tecnologico nel PLC
	    TechnologicalInstanceDBGroup miaCartellaOggettiTecnologici = mioSoftwarePLC.TechnologicalObjectGroup;
            TechnologicalInstanceDB mioOggettoTecnologicoNastroLinea = miaCartellaOggettiTecnologici.TechnologicalObjects.Create("NastroLinea", "TO_SpeedAxis", new Version("6.0"));
            
            //Preparazione del metodo di connessione di 2 oggetti
            AxisHardwareConnectionProvider miaConnessione = mioOggettoTecnologicoNastroLinea.GetService<AxisHardwareConnectionProvider>();

            //Collegamento di asse a TO: il numero di byte è moltiplicato per 8 dato che il metodo si aspetta il numero di bit
            miaConnessione.ActorInterface.Connect(mioIndirizzoByteIniziale * 8, mioIndirizzoByteIniziale * 8, ConnectOption.Default);

	    //Salva progetto
            mioProgettoTIA.Save();

        }
    }
}
