Projeto de Monitoramento de Ambiente com Arduino e Sensor DHT11

Descrição:
Este projeto tem como objetivo medir a temperatura e umidade do ambiente utilizando o sensor integrado DHT11, aproveitando a biblioteca implementada para o Arduino. Além disso, os valores de temperatura, umidade e luminosidade serão exibidos em um display LCD. A funcionalidade de alerta será estendida para incluir níveis críticos de temperatura e umidade, indicados por LEDs e um Buzzer.

Componentes Utilizados:
- Arduino Uno (ou compatível)
- Sensor de temperatura e umidade DHT11
- Display LCD 16x2
- LEDs (para indicar alertas de temperatura e umidade)
- Buzzer (para alertas sonoros)
- Resistores e fios de conexão

Instruções de Instalação:
1. Baixe e instale o [Arduino IDE](https://www.arduino.cc/en/software) em sua máquina.
2. Conecte o Arduino ao computador via cabo USB.
3. Baixe a biblioteca DHT11 para Arduino. Isso pode ser feito seguindo as instruções fornecidas pelo fabricante do sensor ou procurando por "DHT sensor library" no Library Manager do Arduino IDE.
4. Monte o circuito de acordo com o esquemático fornecido no arquivo "esquema_circuito.png".

Configuração:
1. Conecte os pinos do sensor DHT11 aos pinos correspondentes do Arduino de acordo com o esquemático.
2. Conecte o display LCD ao Arduino conforme indicado no esquemático.
3. Certifique-se de que os LEDs e o Buzzer estejam conectados corretamente para indicar alertas de temperatura e umidade.

Utilização:
1. Abra o código-fonte fornecido no Arduino IDE.
2. Compile e faça o upload do código para o Arduino.
3. O display LCD começará a exibir os valores de temperatura, umidade e luminosidade.
4. Os LEDs e o Buzzer serão acionados caso os níveis de temperatura e umidade atinjam valores críticos.
5. Monitore os valores exibidos e responda aos alertas conforme necessário.

(CÓDIGO FONTE)
#include <LiquidCrystal.h>

// Controle de Luz
int ESCURO   = 0;
int MEIA_LUZ = 1;
int CLARO    = 2;
int luz = ESCURO;

// Controle de temperatura
int TEMP_OK = 0;
int FRIO    = 1;
int QUENTE  = 2;
int temperaturaAmbiente = TEMP_OK;

// Controle de umidade
int UMID_OK    = 0;
int UMID_BAIXA = 1;
int UMID_ALTA  = 2;
int umidadeAmbiente = UMID_OK;

// Controle do display
int tempoMostrarDisplay = 1000;//5000;

int const MOSTRA_LUZ  = 0;
int const MOSTRA_TEMP = 1;
int const MOSTRA_UMID = 2;
int estadoDisplay = MOSTRA_LUZ;


int ledVermelhoPin = 11;  // LED vermelho no pino 9
int ledAmareloPin  = 12;  // LED amarelo no pino 10
int ledVerdePin    = 13;  // LED verde no pino 11
int ldrPin         = A1;  // LDR no pino A0
int buzzerPin      = 10;   // Buzzer no pino 7
int sensorTempPin  = A2;  // Sensor de temperatura no pino A2
int sensorUmidPin  = A0;  // Sensor de umidade no pino A1

LiquidCrystal LCD (9,8,7,6,5,4); // LCD nos pinos 13, 12, 5, 4, 3 e 2


int   valorLDR    = 0;
float temperatura = 0;
int   umidade     = 0;

int millisDisplayAgora = 0;
int millisDisplayAnterior = 0;

int   contadorMedia = 0;
float somaTemperatura = 0;
int   somaUmidade = 0;

float mediaTemperatura = 0;
int   mediaUmidade = 0;


void setup(){
	LCD.begin(16,2);
	pinMode (ledVermelhoPin, OUTPUT);  
	pinMode (ledAmareloPin, OUTPUT);
	pinMode (ledVerdePin, OUTPUT);
	pinMode (ldrPin, INPUT);
	pinMode (buzzerPin, OUTPUT);
	//pinMode (sensorTempPin, INPUT);
	pinMode (sensorUmidPin, INPUT);
	Serial.begin(9600);
}


void loop(){

	// A primeira coisa que precisamos fazer é ler os sensores
	ler_sensores();
	
	// A segunda coisa que precisamos fazer é verificar se as leituras estao OK
	verifica_sensores();
	
	// A terceira coisa é ativar os LEDs e o Buzzer
	aciona_alarmes();
	
	// Por fim, precisamos mostrar a mensagem no LCD
	mostra_mensagem();
}


void ler_sensores(){

	//Lê o valor do sensor ldr e armazena na variável valorLDR
	valorLDR = analogRead (ldrPin); 
	Serial.print("LDR: ");   
	Serial.println(valorLDR);
   
	// Le o valor da temperatura
	int valorSensorTemp = analogRead (sensorTempPin);
	
	// Converte o valor lido para tensao
	float tensao = valorSensorTemp * 5;
	tensao /= 1024;
	
	// Converte para Graus Celcius
	temperatura = (tensao - 0.5) * 100;

   	Serial.print("Temperatura: ");
	Serial.println(temperatura);
	
	// Le o valor de Umidade
	int valorSensorUmi = analogRead(sensorUmidPin); 
	
	//transforma o valor do potenciometro em uma escala até 100
	umidade = map(valorSensorUmi, 0, 1023, 0, 100); 
	Serial.print("Umidade: ");
	Serial.println(umidade);
}


void verifica_sensores(){

	// Primeiro vamos verificar se a iluminacao esta ok
	Serial.print("Ambiente esta ");
	if(valorLDR <= 700){
		luz = ESCURO;
		Serial.println("Escuro");
	} 
	else if(valorLDR > 700 && valorLDR <= 900){
		luz = MEIA_LUZ;
		Serial.println("Meia Luz");
	}
	else if(valorLDR > 900){
		luz = CLARO;
		Serial.println("Claro");
	}
	
	// Segundo, vamos verificar o sensor de temperatura
	Serial.print("Temperatura esta ");
	if(temperatura < 10){
		temperaturaAmbiente = FRIO;
		Serial.println("fria");
	}
	else if(temperatura > 15){
		temperaturaAmbiente = QUENTE;
		Serial.println("quente");
	}
	else {
		temperaturaAmbiente = TEMP_OK;
		Serial.println("OK");
	}
	
	// Por fim vamos verificar o sensor de umidade
	Serial.print("Umidade esta ");
	if(umidade < 50){
		umidadeAmbiente = UMID_BAIXA;
		Serial.println("baixa");
	}
	else if(umidade > 70){
		umidadeAmbiente = UMID_ALTA;
		Serial.println("alta");
	}
	else {
		umidadeAmbiente = UMID_OK;
		Serial.println("OK");
	}
}


void aciona_alarmes(){

	// Enquanto o ambiente estiver escuro, o LED Verde deve ficar aceso (Req. 1)
	if(luz == ESCURO){
		digitalWrite(ledVerdePin, HIGH);
	}
	else{
		digitalWrite(ledVerdePin, LOW);
	}
	
	// Enquanto o ambiente estiver a meia luz, o LED amarelo deve ficar aceso... (Req. 2)
	// Enquanto a temperatura estiver fora da faixa ideal, o LED Amarelo deve ficar aceso e o Buzzer deve ligar continuamente (Req. 8)
	if(luz == MEIA_LUZ || temperaturaAmbiente != TEMP_OK){
		digitalWrite(ledAmareloPin, HIGH);		
		if(temperaturaAmbiente != TEMP_OK){
			digitalWrite(buzzerPin, HIGH);
		}
	}
	else{
		digitalWrite(ledAmareloPin, LOW);
		
		// Verificando se o Buzzer precisa ficar ligado por causa da umidade
		if(umidadeAmbiente == UMID_OK){
			digitalWrite(buzzerPin, LOW);
		}		
	}
	
	// Enquanto o ambiente estiver totalmente iluminado, o LED vermelho deve ficar aceso... (Req. 3)
	// Enquanto a umidade estiver fora da faixa ideal, o LED Vermelho deve ficar aceso e o Buzzer deve ligar continuamente (Req. 11)
	if(luz == CLARO || umidadeAmbiente != UMID_OK){
		digitalWrite(ledVermelhoPin, HIGH);
		
		// Enquanto o ambiente estiver totalmente iluminado, o Buzzer deve ficar ligado continuamente (Req. 4)
		digitalWrite(buzzerPin, HIGH);
	}
	else{
		digitalWrite(ledVermelhoPin, LOW);
		
		// Verificando se o Buzzer precisa ficar ligado por causa da temperatura
		if(temperaturaAmbiente == TEMP_OK){
			digitalWrite(buzzerPin, LOW);
		}
	}
	
}


void mostra_mensagem(){
	
	// pegando quantos milissegundos o arduino esta ligado
	millisDisplayAgora = millis();
	
	// Para mostrar os valores, precisamos da média dos ultimos 5 segundos
	// Para isso, vamos primeiro fazer um somatorio dos dados lidos:
    somaTemperatura = somaTemperatura + temperatura;
    somaUmidade = somaUmidade + umidade;
	contadorMedia++;
	
	// Verifica se passou o tempo para poder atualizar a mensagem no display
	if(millisDisplayAgora - millisDisplayAnterior > tempoMostrarDisplay){
	
		// Preparando o LCD para mostrar a mensagem
	    LCD.clear();
		LCD.setCursor(0,0); 
	   
		switch(estadoDisplay){
			case (MOSTRA_LUZ):
				if(luz == ESCURO){
					LCD.print("Luz Ambiente OK");
				}
				else if(luz == MEIA_LUZ){
					LCD.print("Ambiente a");
					LCD.setCursor(0,1);
					LCD.print("meia luz");
				}
				else{
					LCD.print("Ambiente");
					LCD.setCursor(0,1);
					LCD.print("muito claro");
				}
				estadoDisplay = MOSTRA_TEMP;
			break;
			
			case(MOSTRA_TEMP):
				mediaTemperatura = somaTemperatura / contadorMedia;
				
				if(temperaturaAmbiente == TEMP_OK){
					LCD.print("Temperatura OK");
					LCD.setCursor(0,1);
					LCD.print("Temp. = "); LCD.print(mediaTemperatura); LCD.print("C");
				}
				else if(temperaturaAmbiente == FRIO){
					LCD.print("Temp. Baixa");
					LCD.setCursor(0,1);
					LCD.print("Temp. = "); LCD.print(mediaTemperatura); LCD.print("C");
				}
				else{
					LCD.print("Temp. Alta");
					LCD.setCursor(0,1);
					LCD.print("Temp. = "); LCD.print(mediaTemperatura); LCD.print("C");
				}
			estadoDisplay = MOSTRA_UMID;
			break;
			
			case(MOSTRA_UMID):
				mediaUmidade = somaUmidade / contadorMedia;
			
				if(umidadeAmbiente == UMID_OK){
					LCD.print("Umidade OK");
					LCD.setCursor(0,1);
					LCD.print("Umidade = "); LCD.print(mediaUmidade); LCD.print("%");
				}
				else if(umidadeAmbiente == UMID_BAIXA){
					LCD.print("Umidade Baixa");
					LCD.setCursor(0,1);
					LCD.print("Umidade = "); LCD.print(mediaUmidade); LCD.print("%");
				}
				else{
					LCD.print("Umidade Alta");
					LCD.setCursor(0,1);
					LCD.print("Umidade = "); LCD.print(mediaUmidade); LCD.print("%");
				}
			
			estadoDisplay = MOSTRA_LUZ;
			break;
			
			default:
				estadoDisplay = MOSTRA_LUZ;
			break;
		}
		// Zerando as vairaveis de média
		somaTemperatura = 0;
		somaUmidade = 0;
		contadorMedia = 0;
		
		// Atualizando o tempo para só entrar daqui a tempoMostrarDisplay milissegundos
		millisDisplayAnterior = millis();
	}
}


