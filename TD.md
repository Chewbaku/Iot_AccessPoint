# Créer un point d'accès avec un ESP8266

Dans ce **TD** nous allons voir comment installer un point d'accès sans fil hébergé avec un **Esp8266** en Wifi.

# Pré-requis

Liste du matériel nécessaire pour ce TD
- Un **esp8622**
- Un **câble USB**
- Une **breadboard**
- Une **Led**
- Un ou plusieurs appareils avec une **connexion Wifi**

Il est nécessaire que votre IDE Arduino soit déjà paramétré pour un ESP8266
- **Inclure** les bibliothèques de l'esp8266 
- **Sélectionner** la carte avec le gestionnaire de carte Arduino


# Exercices

## Exercice 1 - Point d'Accès Basique

1. Pour commencer, nous allons **créer un nouveau projet Arduino** et inclure les bibliothèques de l'esp8266 et Wifi nécessaires à ce petit programme.

		#include <ESP8266WiFi.h>
		#include <WiFiClient.h>
		#include <ESP8266WebServer.h>

2. Ensuite nous allons choisir le **SSID** et le **mot de passe** de notre futur réseau et **démarrer** le serveur Wifi sur le port de votre choix (80 par défaut)

		const char *ssid = "**APTEST**";
		const char *password = "**12345678**";
		ESP8266WebServer server(**80**);

3. Maintenant attaquons nous à la méthode **setup** de notre programme pour **initialiser** nos variables.

		void setup{ 
			delay(1000);
			Serial.begin(115200);
			Serial.println();
			Serial.print("Configuring access point...");
			WiFi.softAP(ssid, password);
			IPAddress myIP = WiFi.softAPIP();
			Serial.print("AP IP address: ");
			Serial.println(myIP);
			server.on("/", handleRoot);
			server.begin();
			Serial.println("HTTP server started");
		}
		
4. Nous allons **ajouter** une fonction qui permet d'afficher un code html dès qu'un appareil se connecte à notre Access Point. Pour se faire, nous allons instancier la fonction **handleRoot** en prenant soin de la déclarer **avant** le setup.

		void handleRoot() {
			server.send(200, "text/html", "<h1>You are connected</h1>");
		}

5. Enfin nous déclarons la fonction **loop** de notre programme

		void loop() {
			server.handleClient()
		}
6. Téléverser votre programme, sélectionner votre point d'accès depuis un appareil en Wifi et tester votre réseau !
## Exercice 2 - Point d'Accès Avancé

Vous pouvez personnaliser votre point d'accès selon vos besoins à l'aide de la fonction suivante :

	WiFi.softAP(ssid, password, channel, hidden, max_connection);

1. Un point d'accès sur un ESP8266 permet d'avoir entre **0 et 8 connexions** simultanées. Par défaut cette valeur vaut 4 mais il est possible de la modifier.

	Pour cela, dans la boucle de **setup**, il suffit d'utiliser le constructeur complet de **WiFi.softAP()**, en changeant l'argument *max_connection*.

2. Il est possible de rendre point d'accès **invisible**, c'est à dire que le SSID n'est pas visible par les appareils à proximité mais si un appareil connait déjà le SSID du réseau, il peut s'y connecter sans problème.

	Pour se faire, il suffit d'initialiser l'argument *hidden* sur **true**, sa valeur par défaut étant **false**.

3. On peut aussi récupérer le nombre de connexion sur le point d'accès en temps réel. 
	Par exemple ici, on cherche juste à l'afficher dans la console à l'aide des lignes suivantes :

		void loop {
			Serial.printf("Stations connected = %d\n", WiFi.softAPgetStationNum());
			delay(1000);
		}

## Exercice 3 - Allumer une LED à distance

1. Déclarer la LED ainsi que les fonctions pour la contrôler au début de votre programme.

		const byte LED_PIN = 5; // Branchement de la led sur D1
	Allumer la LED
			
			void led_on(){
				Serial.println("Led on");
				digitalWrite(LED_PIN, HIGH);
			}
	
	Eteindre la LED

		void led_off(){
			Serial.println("Led off");
			digitalWrite(LED_PIN, LOW);
		}

2. Initialiser ensuite l'interface html pour les utilisateurs connectés qui affichera les contrôle de la LED.

		String homepage() {
			String s;
			s = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<!DOCTYPE HTML>\r\n<html><body>\r\n";
			s +="<script language='javascript' type='text/javascript'>\r\n";
			s +="<!--\r\n";
			s +="function on()  { var xhr = new XMLHttpRequest(); xhr.open('GET', '/on',  true); xhr.send();}\r\n";
			s +="function off() { var xhr = new XMLHttpRequest(); xhr.open('GET', '/off', true); xhr.send();}\r\n";
			s +="//-->\r\n";
			s +="</script>\r\n";
			s +="<table align='center' style='cursor: pointer;'><tr>\r\n";
			s +="<td height='100' width='100' bgcolor='#f45942' align='center'><font size='20'><div onclick='off();'>Off</div></font></td>\r\n";
			s +="<td height='100' width='100' bgcolor='#41f492' align='center'><font size='20'><div onclick='on();'>On </div></font></td>\r\n";
			s += "</tr></table></body></html>\r\n";
			return s;
		}

3. Initialiser le contrôle de la LED dans la boucle setup et ajouter les lignes suivantes :

		void setup{ 
		 	delay(1000);
		 	Serial.begin(115200);
		 	Serial.println();
		 	Serial.print("Configuring access point...");
		 	WiFi.softAP(ssid, password);
		 	IPAddress myIP = WiFi.softAPIP();
		 	Serial.print("AP IP address: ");
		 	Serial.println(myIP);
		 	server.begin();
		 	Serial.println("HTTP server started");
	
		 	pinMode(LED_PIN, OUTPUT);
			digitalWrite(LED_PIN, LOW);
		 }
		
	>La ligne *server.on("/", handleRoot);* 
	a été supprimée car cette fonction ne nous est plus utile ici.

4. Ajouter la fonction ***matchcommand()*** qui permet de comparer la demande des clients avec les fonctions ***led_on()*** et ***led_off()***

		boolean matchcommand(String req,String command){
			if (req.indexOf(command) != -1){
				return true;
			} else {
				return false;
			}
		}

5. Enfin, paramétrons la boucle loop. 

		void loop() {

			Serial.printf("Stations connected = %d\n",WiFi.softAPgetStationNum());
			delay(3000);

			// Vérifie si client se connecte
			WiFiClient client = server.available();
			if (!client) {
				return;
			}
	
			// Attends que le client envoie des données
			Serial.println("new client");
			while(!client.available()){
				delay(1);
			}

			// Affiche la page d'accueil 192.168.4.1
			client.print(homepage());

			// Lit la premiere ligne de la requete
			String req = client.readStringUntil('\r');
			Serial.println(req);
			client.flush();

			// requête 192.168.4.1/off --> éteins led
			if (matchcommand(req,"off")){
				led_off();
			}
			// requête 192.168.4.1/on --> allume led
			else if (matchcommand(req,"on")){
				led_on();
			}
			else {
				Serial.println("invalid request");
				client.stop();
				return;
			}
			client.flush();
		}

ENJOY !
