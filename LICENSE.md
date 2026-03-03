<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Echo der Guillotine | Visuelle Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@700&family=Quicksand:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --french-blue: #002395;
            --french-white: #ffffff;
            --french-red: #ed2939;
            --gold: #d4af37;
            --dark-bg: #121212;
            --overlay: rgba(0, 0, 0, 0.6);
        }

        body, html {
            margin: 0; padding: 0;
            height: 100%; font-family: 'Quicksand', sans-serif;
            background-color: var(--dark-bg); color: white;
            overflow: hidden;
        }

        /* Dynamischer Hintergrund: Das Bild der aktuellen Szene */
        #bg-image {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background-size: cover; background-position: center;
            z-index: -2;
            transition: background-image 0.8s ease-in-out; /* Weicher Übergang */
        }

        /* Dunkle Überlagerung, damit Text lesbar bleibt */
        .bg-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.7);
            z-index: -1;
        }

        .container {
            max-width: 900px; margin: 30px auto; padding: 30px;
            background: rgba(10, 10, 10, 0.9);
            border: 2px solid var(--gold);
            border-radius: 15px; box-shadow: 0 0 50px rgba(0, 0, 0, 0.8);
            text-align: center;
            position: relative;
            backdrop-filter: blur(5px); /* Moderner Blur-Effekt */
        }

        h1 { font-family: 'Cinzel', serif; color: var(--gold); font-size: 2.5rem; margin-top: 0; text-shadow: 2px 2px 4px black; }

        .stats-bar {
            display: flex; justify-content: space-around;
            margin-bottom: 25px; padding: 15px;
            border-bottom: 1px solid #444; font-size: 1.2rem;
            background: rgba(255,255,255,0.05); border-radius: 8px;
        }

        .stat { font-weight: bold; color: var(--french-white); }
        .stat span { color: var(--gold); }

        #story-text { 
            font-size: 1.3rem; line-height: 1.8; 
            margin-bottom: 40px; min-height: 160px; 
            text-align: left;
            padding: 0 20px;
        }

        .choices-container { display: flex; flex-direction: column; gap: 15px; padding: 0 20px; }

        .choice-btn {
            background: rgba(255, 255, 255, 0.03); 
            border: 1px solid rgba(255,255,255,0.3);
            color: white; padding: 20px; cursor: pointer;
            transition: 0.3s all ease; border-radius: 8px; font-size: 1.15rem;
            text-align: left;
            font-weight: bold;
        }

        .choice-btn:hover {
            background: var(--french-white); color: black;
            transform: scale(1.02);
            border-color: var(--french-white);
        }

        .progress-container {
            width: 100%; background-color: #333; height: 8px; margin-top: 25px; border-radius: 4px; overflow: hidden;
        }
        #progress-bar { width: 0%; height: 100%; background: linear-gradient(90deg, var(--french-blue), var(--french-red)); transition: 0.5s; }
    </style>
</head>
<body>

<div id="bg-image"></div>
<div class="bg-overlay"></div>

<div class="container">
    <div class="stats-bar">
        <div class="stat">Ruf: <span id="reputation">50</span>%</div>
        <div class="stat">Jahr: <span id="year">1789</span></div>
    </div>

    <h1 id="scene-title">Lade Revolution...</h1>
    <p id="story-text">Bitte warten...</p>

    <div class="choices-container" id="choices"></div>
    
    <div class="progress-container">
        <div id="progress-bar"></div>
    </div>
</div>

<script>
    const storyData = {
        start: {
            title: "Der Ballhausschwur",
            image: "bilder/ballhaus.jpg", // <--- PFAD ZUM BILD
            text: "Juni 1789: Die Generalstände sind blockiert. Der Dritte Stand, Vertreter des Volkes, erklärt sich zur Nationalversammlung. Ihr seid in der königlichen Tennishalle eingeschlossen. Die Wachen fordern euch auf, zu gehen. Was tust du?",
            choices: [
                { text: "„Wir weichen nur der Gewalt der Bajonette!“ Leiste den Schwur.", next: "bastille", stats: { rep: 25, year: 1789, prog: 15 } },
                { text: "Geh nach Hause. Es ist zu gefährlich, sich dem König zu widersetzen.", next: "prison", stats: { rep: -20, year: 1789, prog: 10 } }
            ]
        },
        bastille: {
            title: "Sturm auf die Bastille",
            image: "bilder/bastille.jpg",
            text: "14. Juli 1789: Der König zieht Truppen um Paris zusammen. Das Volk braucht Waffen. Die alte Festung Bastille lagert Schießpulver. Ein Funke genügt...",
            choices: [
                { text: "„A la Bastille!“ Schließe dich dem bewaffneten Aufstand an.", next: "declaration", stats: { rep: 30, year: 1789, prog: 30 } },
                { text: "Halte dich zurück. Gewalt führt nur zu Chaos.", next: "failed_negotiation", stats: { rep: -10, year: 1789, prog: 25 } }
            ]
        },
        declaration: {
            title: "Erklärung der Rechte",
            image: "bilder/rechte.jpg",
            text: "August 1789: Die Welt verändert sich. Die Nationalversammlung verabschiedet die 'Erklärung der Menschen- und Bürgerrechte'. Doch der König zögert. Die Frauen von Paris marschieren nach Versailles.",
            choices: [
                { text: "Marschiere mit den Frauen, um den König nach Paris zu holen.", next: "tuileries", stats: { rep: 20, year: 1789, prog: 45 } },
                { text: "Bleib in der Versammlung und arbeite an Gesetzen.", next: "tuileries", stats: { rep: 5, year: 1790, prog: 45 } }
            ]
        },
        tuileries: {
            title: "Die gescheiterte Flucht",
            image: "bilder/varennes.jpg",
            text: "Juni 1791: Der König und seine Familie wurden in Varennes gestoppt. Sie wollten ins Ausland fliehen, um eine Gegenrevolution zu starten. Das Vertrauen ist endgültig gebrochen.",
            choices: [
                { text: "Fordere die Republik! Der König hat sein Volk verraten.", next: "terror", stats: { rep: 40, year: 1793, prog: 60 } },
                { text: "Verteidige das Gesetz. Der König muss Staatsoberhaupt bleiben.", next: "guillotine_ending", stats: { rep: -40, year: 1792, prog: 55 } }
            ]
        },
        terror: {
            title: "Die Republik der Tugend",
            image: "bilder/terror.jpg",
            text: "1793-1794: Maximilien Robespierre regiert mit eiserner Hand. Der Wohlfahrtsausschuss schickt jeden Verdächtigen auf die Guillotine. Niemand ist sicher.",
            choices: [
                { text: "Unterstütze den radikalen Kurs für das Wohl der Nation.", next: "thermidor", stats: { rep: 10, year: 1794, prog: 80 } },
                { text: "Fordere ein Ende des Blutvergießens. Das ist Tyrannei!", next: "guillotine_ending", stats: { rep: 100, year: 1794, prog: 75 } }
            ]
        },
        thermidor: {
            title: "Der Aufstieg Bonapartes",
            image: "bilder/napoleon.jpg",
            text: "1799: Die Schreckensherrschaft ist vorbei, aber das Direktorium ist schwach. Ein junger General, Napoleon Bonaparte, kehrt siegreich aus Ägypten zurück. Er verspricht Ordnung.",
            choices: [
                { text: "Unterstütze seinen Staatsstreich. Frankreich braucht Führung.", next: "empire_ending", stats: { rep: 30, year: 1799, prog: 100 } },
                { text: "Verteidige die Republik gegen diesen neuen Caesar.", next: "exile_ending", stats: { rep: -20, year: 1799, prog: 100 } }
            ]
        },
        // Enden
        guillotine_ending: { title: "Das Ende unter dem Fallbeil", image: "bilder/ende_tod.jpg", text: "Deine Prinzipien waren edel, aber in Zeiten der Revolution tödlich. Dein Kopf fällt auf dem Place de la Révolution.", choices: [] },
        empire_ending: { title: "Ein neues Zeitalter", image: "bilder/ende_sieg.jpg", text: "Du hast überlebt. Napoleon krönt sich zum Kaiser. Die Ideale von 1789 sind tot, aber Frankreich ist mächtiger denn je.", choices: [] },
        exile_ending: { title: "Der vergessene Republikaner", image: "bilder/ende_tod.jpg", text: "Napoleon siegt. Du wirst als Staatsfeind deportiert und verbringst den Rest deines Lebens im Exil.", choices: [] },
        prison: { title: "Im Kerker vergessen", image: "bilder/ende_tod.jpg", text: "Der König hat gesiegt. Die Anführer der Nationalversammlung wurden hingerichtet, du verrottest im Bastille-Kerker.", choices: [] },
        failed_negotiation: { title: "Niedergemetzelt", image: "bilder/bastille.jpg", text: "Der Kommandant ließ das Feuer eröffnen. Du stirbst im Kugelhagel vor den Toren der Bastille.", choices: [] }
    };

    let currentRep = 50;

    function updateScene(sceneKey) {
        const scene = storyData[sceneKey];
        if (!scene) return; // Sicherheitscheck

        // 1. Titel und Text aktualisieren
        document.getElementById('scene-title').innerText = scene.title;
        document.getElementById('story-text').innerText = scene.text;
        
        // 2. Hintergrundbild aktualisieren (Hinzugefügt)
        if (scene.image) {
            document.getElementById('bg-image').style.backgroundImage = `url('${scene.image}')`;
        }

        // 3. Entscheidungen aktualisieren
        const choicesDiv = document.getElementById('choices');
        choicesDiv.innerHTML = '';

        scene.choices.forEach(choice => {
            const btn = document.createElement('button');
            btn.className = 'choice-btn';
            btn.innerText = choice.text;
            btn.onclick = () => {
                if(choice.stats) {
                    currentRep += choice.stats.rep;
                    document.getElementById('reputation').innerText = currentRep;
                    document.getElementById('year').innerText = choice.stats.year;
                    document.getElementById('progress-bar').style.width = choice.stats.prog + "%";
                }
                updateScene(choice.next);
            };
            choicesDiv.appendChild(btn);
        });

        // 4. Neustart-Button bei Enden
        if (scene.choices.length === 0) {
            const resetBtn = document.createElement('button');
            resetBtn.className = 'choice-btn';
            resetBtn.innerText = "Die Geschichte neu schreiben";
            resetBtn.style.borderColor = "var(--gold)";
            resetBtn.style.textAlign = "center";
            resetBtn.onclick = () => location.reload();
            choicesDiv.appendChild(resetBtn);
        }
    }

    // Spiel starten
    updateScene('start');
</script>

</body>
</html>
echo "# franzoesische-revolution.index.html" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/elisabethkorrmann-debug/franzoesische-revolution.index.html.git
git push -u origin main
