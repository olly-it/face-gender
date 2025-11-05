## 🎯 Obiettivo

Fare un’app browser che:

1. Conta il numero di persone (volti)
2. Usa un modello TensorFlow.js (esportato da Teachable Machine)
3. Mostra “Maschio/Femmina” sopra ogni volto

---

## 🧠 1. Come creare il tuo modello gender su **Teachable Machine**

Vai su 👉 [https://teachablemachine.withgoogle.com/train/image](https://teachablemachine.withgoogle.com/train/image)

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

5. Scegli:

   * ✅ “Upload (Teachable Machine hosting)”
   * Aspetta il caricamento
   * Copia l’URL finale, che avrà la forma:

     ```
     https://teachablemachine.withgoogle.com/models/XXXXXXXXX/
     ```

6. Nella tua app useremo questo link per caricare il modello.

---

## 💻 2. Codice completo (funziona nel browser)

Salva questo file come `index.html`
*(ricordati di sostituire l’URL del modello con il tuo!)*

```html
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>Conteggio Persone + Gender</title>
  <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@4.20.0/dist/tf.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/face-api.js@0.22.2/dist/face-api.min.js"></script>
  <style>
    body { background:#111; color:#fff; text-align:center; font-family:sans-serif; }
    video, canvas { border-radius:12px; margin-top:10px; }
    #info { font-size:1.2rem; margin-top:15px; }
  </style>
</head>
<body>
  <h1>Conteggio Persone + Gender (TF.js + Teachable Machine)</h1>
  <video id="video" width="640" height="480" autoplay muted></video>
  <canvas id="overlay" width="640" height="480"></canvas>
  <div id="info">Caricamento...</div>

  <script>
    const MODEL_URL_FACE = 'https://justadudewhohacks.github.io/face-api.js/models/';
    const MODEL_URL_GENDER = 'https://teachablemachine.withgoogle.com/models/XXXXXXXXX/model.json'; // 👈 metti il tuo link qui

    const video = document.getElementById('video');
    const canvas = document.getElementById('overlay');
    const ctx = canvas.getContext('2d');
    const info = document.getElementById('info');

    let genderModel;

    async function loadModels() {
      await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL_FACE);
      genderModel = await tf.loadLayersModel(MODEL_URL_GENDER);
      info.textContent = "Modelli caricati ✅ — Avvio webcam...";
      startVideo();
    }

    function startVideo() {
      navigator.mediaDevices.getUserMedia({ video: true })
        .then(stream => { video.srcObject = stream; })
        .catch(err => console.error("Errore webcam:", err));
    }

    video.addEventListener('play', async () => {
      const opts = new faceapi.TinyFaceDetectorOptions({ inputSize: 224, scoreThreshold: 0.5 });
      info.textContent = "Analisi in corso...";

      setInterval(async () => {
        const detections = await faceapi.detectAllFaces(video, opts);
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        faceapi.matchDimensions(canvas, { width: video.width, height: video.height });
        const resized = faceapi.resizeResults(detections, { width: video.width, height: video.height });

        let male = 0, female = 0;

        for (const det of resized) {
          const { box } = det;
          const face = await faceapi.extractFaces(video, [det.detection]);
          if (face.length > 0) {
            const tfImg = tf.browser.fromPixels(face[0])
              .resizeNearestNeighbor([224, 224])
              .toFloat()
              .div(tf.scalar(255))
              .expandDims();

            const prediction = await genderModel.predict(tfImg).data();
            const classes = ['Male', 'Female']; // 👈 dipende da come hai nominato le classi
            const classIndex = prediction[0] > prediction[1] ? 0 : 1;
            const label = classes[classIndex];

            if (label === 'Male') male++; else female++;

            ctx.strokeStyle = "#00FF00";
            ctx.lineWidth = 2;
            ctx.strokeRect(box.x, box.y, box.width, box.height);
            ctx.fillStyle = "#fff";
            ctx.fillText(label, box.x + 5, box.y - 5);

            tfImg.dispose();
          }
        }

        info.textContent = `Persone: ${resized.length} | Maschi: ${male} | Femmine: ${female}`;
      }, 700);
    });

    loadModels();
  </script>
</body>
</html>
```

---

## 🧭 3. Come usarlo

1. Salva come `index.html`
2. Avvia un server locale:

   ```bash
   python3 -m http.server 8000
   ```
3. Apri: [http://localhost:8000](http://localhost:8000)
4. Permetti accesso alla webcam 🎥
5. Vedrai i riquadri con “Male” / “Female” sopra i volti
   e il conteggio totale in basso.

---

## ⚙️ 4. Riepilogo

| Funzione               | Libreria                              | Descrizione                |
| ---------------------- | ------------------------------------- | -------------------------- |
| Rilevamento volti      | `face-api.js`                         | TinyFaceDetector veloce    |
| Classificazione gender | `tfjs` + modello da Teachable Machine | 2 classi: Male / Female    |
| Conteggio              | JavaScript                            | somma dei volti per classe |

---

## 💡 Suggerimento pratico

* Se vuoi migliorare la **precisione**, allenalo con **immagini di volti ritagliati**, non foto a figura intera.
* Puoi anche aggiungere una terza classe (“Unknown”) per gestire casi incerti.
* Il modello Teachable Machine viene servito in hosting gratuito, ma puoi anche **scaricarlo** e ospitarlo localmente se lo vuoi offline.

---

Vuoi che ti mostri anche **come esportare il modello di Teachable Machine** e caricarlo localmente nel tuo progetto (senza dipendere dal link online)?
