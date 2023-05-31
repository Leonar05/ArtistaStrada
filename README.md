# Esercizio dell'artista da strada
##### Leonardo Donno

Per risolvere questo esercizio per la sincronizzazione dei processi vengono usati dei  semafori e il codice è suddiviso in:
- Main
- Classe StreetArtist
- Classe Artist
- Classe Client

## Main

Il main viene usato solo per iniziare il programma, infatti istanzia l'oggetto "streetArtist" e avvia la simulazione.

``` java
public class Main {
	public static void main(String[] args) {
        StreetArtist streetArtist = new StreetArtist();
        streetArtist.startSimulation();
    }
}
```

## Classe StreetArtist
In questa classe infatti viene creato il processo della simulazione.
All'inizio viene creato il Thread dell'artista e successivamente in un ciclo infinto vengono creati i clienti che vengono generati ogni massimo 2 secondi, gestito da una variabile random che cambia ogni ciclo. Ogni cliente viene numerato in ordine.

``` java
public void startSimulation() {
		Thread artistThread = new Thread(new Artist());
		artistThread.start();
		Random random = new Random();
		int clientNumber = 1;
		
		while (true) {
			try {
				Thread.sleep(random.nextInt(2000)); // Tempo casuale di arrivo di un cliente (da 0 a 2 secondi)
				Thread clientThread = new Thread(new Client(clientNumber));
				clientThread.start();
				clientNumber++;
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
```
## Classe Artist
La classe Artist serve per simulare l'artista che dipinge un ritratto e che ogni volta che finisce un quadro viene liberata una sedia.
Anche qui il tempo che impiega l'artista a compiere il ritratto viene gestita da una variabile random e da un valore compreso tra 0 a 5 secondi.

``` java
public void run() {
            while (true) {
                try {
                    try {
						StreetArtist.getSeatsSemaphore().acquire();
					} catch (Exception e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					} // Occupa una sedia
                    System.out.println("L'artista sta facendo il ritratto...");
                    
                    // Simulazione del tempo impiegato per fare un ritratto
                    Thread.sleep(new Random().nextInt(5000));
                    
                    System.out.println("Il ritratto è pronto. Si è liberata una sedia.");
                    StreetArtist.seatsSemaphore.release(); // Rilascia la sedia
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
```

## Classe Client
Infine è presente la classe Client. Questa classe ha come attributo privato il numero di cliente. Quando il Thread del cliente viene eseguito, viene controllato se sono presenti sedie libere.
Se alcune sedie sono libere il cliente si siede altrimenti aspetta ma se un tempo casuale tra 0 a 3 secondi passa, esso rinuncia al ritratto e va via.
```java
public void run() {
            try {
                if (StreetArtist.getSeatsSemaphore().getQueueLength()==0) {
                    System.out.println("Il cliente " + clientNumber + " si è seduto.");
                    Thread.sleep(new Random().nextInt(3000)); // Tempo casuale di attesa per il ritratto (da 0 a 3 secondi)
                    System.out.println("Il cliente " + clientNumber + " ha ottenuto il suo ritratto va via.");
                    StreetArtist.seatsSemaphore.release(); // Rilascia la sedia
                } else {
                    System.out.println("Il cliente " + clientNumber + " ha rinunciato a farsi fare il ritratto.");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
```
