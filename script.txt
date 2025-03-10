let imageURL = ''; // Variabilă globală pentru URL-ul imaginii

// preluare imagine din JSON de la API-ul DOG
async function fetchDogImage() {
    try {
        const response = await fetch('https://dog.ceo/api/breeds/image/random');
        const data = await response.json();
        imageURL = data.message; // stocheaza URL-ul imaginii în variabila globală
        console.log("URL imagine: ", imageURL);
        return imageURL;
    } catch (error) {
        console.error("Eroare la obținerea imaginii: ", error);
    }
}

// transformare logaritmica a nivelului de gri
function logarithmicGrayLevelTransform(imageData) {
    // array-ul de pixeli al imaginii (toate datele de RGB și alpha)
    const pixels = imageData.data;
    console.log(`Dimensiune array pixeli: ${pixels.length / 4} pixeli`);
    for(let i = 0; i< pixels.length; i += 4){
        // calculez valoarea medie a nivelului de gri folosind coeficienți pentru R, G și B
        let gray = 0.2989 * pixels[i] + 0.587 * pixels[i + 1] + 0.114 * pixels[i + 2];
        // transformarea logaritmică la nivelul de gri, pentru a modifica intensitatea pe baza logaritmului
        gray = Math.log(1 + gray) * 255 / Math.log(256);
        gray = Math.min(255, Math.max(0, gray)); // asigurare că valoarea este între 0 și 255
        pixels[i] = pixels[i + 1] = pixels[i + 2] = gray;
    }

    return imageData;
}

// aplicare efect mirror
function mirrorEffect(imageData) {
    // dimensiunile imaginii
    const width = imageData.width;
    const height = imageData.height;
    console.log(`Dimensiune imagine: ${width} x ${height}`);
    // manipularea pixelilor
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width / 2; x++) {
            const index = (y * width + x) * 4; // Pixelul original
            const mirrorIndex = (y * width + (width - x - 1)) * 4; // Pixelul oglindit

            // schimbă valorile RGB
            for (let i = 0; i < 4; i++) {
                const temp = imageData.data[index + i];
                imageData.data[index + i] = imageData.data[mirrorIndex + i];
                imageData.data[mirrorIndex + i] = temp;
            }
        }
    }

    return imageData;
}

// funcția pentru generarea imaginii
async function generateImage() {
    // preluare URL-ului imaginii de la API
    const imageURL = await fetchDogImage();
    const originalImageElement = document.getElementById('originalImage');
    originalImageElement.src = imageURL; // afișează imaginea generată
    // setez crossOrigin pentru a permite manipularea imaginii pe canvas
    originalImageElement.crossOrigin = "Anonymous";
}

// împărțire procesare imagine în 4 etape asincrone
async function processImage() {
    // creare obiect imagine
    const img = document.getElementById('originalImage');
    if (!img.src) { // Verifică dacă imaginea a fost încărcată
        console.log("Nu există o imagine generată.");
        return;
    }

    const imageURL = img.src; //src-ul imaginii deja încărcate
    const startTime = performance.now();

    const canvas = document.createElement('canvas');
    const context = canvas.getContext('2d');
    canvas.width = img.width;
    canvas.height = img.height;
    context.drawImage(img,0,0); // desenare imagine în canvas   
    console.log(`Dimensiuni canvas: ${canvas.width} x ${canvas.height}`);

    let imageData = context.getImageData(0, 0, img.width, img.height);

    document.getElementById('startTime').innerText = '';
    document.getElementById('endTime').innerText = '';
    document.getElementById('executionTime').innerText = '';
    document.getElementById('jsonData').textContent = ''; // resetează timpii de procesare

    const timings = [];

    let currentStep = 0;
    const steps = [
        () => {
            const stepStart = performance.now();
            imageData = mirrorEffect(imageData);
            const stepEnd = performance.now();
            timings.push(`Etapa 1 (Mirror): ${(stepEnd - stepStart).toFixed(2)} ms`);
        },
        ()  => {
            const stepStart = performance.now();
            imageData = logarithmicGrayLevelTransform(imageData);
            const stepEnd = performance.now();
            timings.push(`Etapa 2 (Logaritmic): ${(stepEnd - stepStart).toFixed(2)} ms`);
        }
    ];

    // procesare cu delay
    const processStep = () => {
        if (currentStep < steps.length) {
            steps[currentStep](); // apelare funcția corespunzătoare
            currentStep++;
            setTimeout(processStep, 1000); // delay de 1 secundă între etape
        } else {
            // scrierea finală în canvas după terminarea procesării
            context.putImageData(imageData, 0, 0);
            const processedImageElement = document.getElementById('processedImage');
            processedImageElement.src = canvas.toDataURL(); // afisare imaginea procesată

            const endTime = performance.now();
            document.getElementById('startTime').innerText = `-Timp start = ${startTime}-`;
            document.getElementById('endTime').innerText = `-Timp final = ${endTime}-`;
            document.getElementById('executionTime').innerText = `-Timp total de execuție = ${(endTime - startTime).toFixed(2)} ms-`;

            // afișarea timpior pentru fiecare etapă
            const timingData = timings.join("\n");
            document.getElementById('jsonData').textContent += `\n\nTimpi de procesare:\n${timingData}`;
        }
    };

    // Începem procesarea asincronă
    processStep();

}

// Adăugare evenimente pentru butoane
document.addEventListener('DOMContentLoaded', function () {
    // Adăugare eveniment pentru generare imagine
    document.getElementById('generateImageButton').addEventListener('click', generateImage);

    // Adăugare eveniment pentru aplicare efecte
    document.getElementById('applyEffectsButton').addEventListener('click', processImage);
});