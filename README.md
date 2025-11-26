## Come creare il tuo modello gender su **Teachable Machine**

Vai su [https://teachablemachine.withgoogle.com/train/image](https://teachablemachine.withgoogle.com/train/image)

Poi segui questi passi:

1. **Crea due classi:**

   * “Male”
   * “Female”

2. **Aggiungi immagini di training**
   Puoi usare:

   * foto da dataset pubblici (es. immagini di volti da open datasets),  
   Es. dataset: https://www.kaggle.com/datasets?search=gender+age+faces
   * oppure farti foto da webcam (funziona anche così per un test rapido).
   


3. **Premi “Train Model”**

4. Quando ha finito, clicca **“Export Model” → “TensorFlow.js”**

5. Fai il download del modello (zip contenente model.json, metadata.json e weights.bin)

6. Nella tua app useremo questi file per caricare il modello.


## Codice
1. face-browser: contiene una versione che fa semplicemente il detect del volto (usando face-api)
2. test-model: fa un test stupido del modello generato con TableMachine
3. example_by_teachable_machine: codice recuperato direttamente dall'output di TeachableMachine, adattato
4. face-gender-browser: prova con chat-gpt a mettere insieme il modello per il detect del volto (face-api) e il modello del gender (generato con TeachableMachine). face-api e TeachableMachine hanno un po' di casini di compatibilità
5. mediapipe: mette insieme il modello per il face-detection usando MediaPipe (invece di face-api) e il modello del gender generato con TableMachine. Funziona!
6. example_face_gender_age: gira su browser, recupera gender e age delle persone collegate e le scrive in video. Di default la camera è spenta, possibilità di accendera. Funziona!


## start in locale
Per farlo partire dalla 8000 (con python3 installato):  
```
python3 -m http.server 8000
```


Olly
