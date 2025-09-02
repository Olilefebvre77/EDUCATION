<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Application de Dictée</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            color: #1a202c;
        }
        .container {
            max-width: 800px;
        }
        .message-box {
            padding: 1rem;
            border-radius: 0.5rem;
            margin-top: 1rem;
        }
        .correct {
            background-color: #d1fae5;
            color: #065f46;
        }
        .incorrect {
            background-color: #fee2e2;
            color: #991b1b;
        }
    </style>
</head>
<body class="p-4 sm:p-8 flex items-center justify-center min-h-screen">
    <div class="container bg-white p-6 sm:p-10 rounded-xl shadow-2xl space-y-8">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-center text-gray-900">Application de Dictée</h1>

        <!-- Menu de sélection des blocs -->
        <div id="block-selection-menu" class="space-y-4">
            <h2 class="text-xl sm:text-2xl font-bold text-gray-800 text-center">Choisissez un Bloc à Pratiquer</h2>
            <div id="block-list" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                <!-- Les boutons des blocs seront générés ici par JS -->
            </div>
        </div>

        <!-- Section de pratique -->
        <div id="practice-section" class="hidden space-y-6 text-center">
            <div id="current-word-title" class="text-lg font-semibold text-gray-600"></div>
            <button id="play-word" class="bg-blue-500 text-white font-bold py-3 px-6 rounded-full shadow-lg hover:bg-blue-600 transition duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-75">
                Écouter le mot
            </button>
            
            <div class="space-y-4">
                <input type="text" id="user-input" class="w-full p-3 text-center border-2 border-gray-300 rounded-lg focus:outline-none focus:border-blue-500 text-lg" placeholder="Écrivez le mot ici" autocomplete="off">
                <button id="check-word" class="w-full bg-green-500 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-green-600 transition duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-green-400 focus:ring-opacity-75">
                    Vérifier
                </button>
            </div>
            
            <div id="feedback" class="text-md font-medium text-center"></div>
            
            <div class="flex justify-center space-x-4">
                <button id="next-word" class="bg-indigo-500 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-indigo-600 transition duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-75 hidden">
                    Mot Suivant
                </button>
                <button id="restart-block" class="bg-red-500 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-red-600 transition duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-red-400 focus:ring-opacity-75">
                    Recommencer ce bloc
                </button>
                <button id="back-to-menu" class="bg-gray-400 text-white font-bold py-3 px-6 rounded-lg shadow-md hover:bg-gray-500 transition duration-300 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-gray-300 focus:ring-opacity-75">
                    Retour au menu
                </button>
            </div>
        </div>
    </div>

    <script>
        const speechSynthesis = window.speechSynthesis;
        let motsParBloc = {
            "Bloc 1": ["un angle", "déranger", "une planche", "une indépendance", "une balance", "un divan", "un sandwich", "intéressant", "une banlieue", "s'élancer", "intéressante", "une séance", "une banque", "franchir", "un mandat", "une tranche", "brillant/brillante", "une garantie", "une marchandise", "une croyance", "grandir", "un pansement"],
            "Bloc 2": ["un sifflement", "un équipement", "un parlement", "un aliment", "un déplacement", "ustensile", "une étendue", "ralentir", "un vendeur", "un remerciement", "fréquenter", "encourager", "intense", "enfoncer", "une vendeuse", "une rentrée", "repentir (se)", "entreprendre", "mentir", "venger", "vente", "une revendication", "un paiement", "environ"],
            "Bloc 3": ["un ambassadeur", "un emploi", "un pompier", "semblable", "une ambassadrice", "un emprunt", "une pompière", "une sympathie", "une importation", "un camp", "un rassemblement", "sympathique", "combattre", "un impôt", "une récompense", "une température", "un compliment", "un jambon", "une ressemblance", "trembler", "un membre", "un comportement"],
            "Bloc 4": ["absent", "absente", "second", "seconde", "doré", "dorée", "glacé", "glacée", "âgé", "âgée", "étroit", "étroite", "humain", "humaine", "un sergent", "une sergente", "sourd", "sourde", "un artisan", "une artisane", "évident", "évidente", "idiot", "idiote", "un candidat", "une candidate", "excellent", "excellente", "passé", "passée", "spécial", "spéciale", "un caporal", "une caporale", "expert", "experte", "principal", "principale", "puissant", "puissante"],
            "Bloc 5": ["adulte", "agréable", "artiste", "aveugle", "brave", "comique", "détective", "enfant", "fonctionnaire", "formidable", "fragile", "journaliste", "locataire", "photographe", "sévère", "souple", "propriétaire", "responsable", "scientifique", "scolaire", "spécialiste", "touriste", "véritable"],
            "Bloc 6 (Partie A)": ["professionnel", "professionnelle", "un technicien", "une technicienne", "un musicien", "une musicienne", "nul", "nulle"],
            "Bloc 6 (Partie B)": ["un acheteur", "une acheteuse", "un chauffeur", "une chauffeuse", "un écolier", "une écolière", "un portier", "une portière", "un acteur", "une actrice", "un conducteur", "une conductrice", "un électeur", "une électrice", "silencieux", "silencieuse", "un amateur", "une amatrice", "un cordonnier", "une cordonnière", "jumeau", "jumelle", "sportif", "sportive", "un auditeur", "une auditrice", "courageux", "courageuse", "menteur", "menteuse", "un traducteur", "une traductrice", "un berger", "une bergère", "un danseur", "une danseuse", "un nageur", "une nageuse", "un travailleur", "une travailleuse"],
            "Bloc 7": ["un sou", "des sous", "un bijou", "des bijoux", "un caillou", "des cailloux", "un chou", "des choux", "un genou", "des genoux", "un hibou", "des hiboux", "un joujou", "des joujoux", "un pou", "des poux", "un gouvernail", "des gouvernails", "bail", "baux", "un corail", "des coraux", "un travail", "des travaux", "un vitrail", "des vitraux", "un neveu", "des neveux", "adieu", "des adieux", "bleu", "bleus", "un pneu", "des pneus", "un signal", "des signaux", "un tribunal", "des tribunaux", "un bal", "des bals", "un carnaval", "des carnavals", "un festival", "des festivals", "un récital", "des récitals", "un régal", "des régals"],
            "Bloc 8": ["afin que", "clairement", "extrêmement", "soit", "autrefois", "d'accord", "là-haut", "tantôt", "dont", "autrement", "légèrement", "tranquillement", "droit", "ceci", "particulièrement", "uniquement", "certainement", "entièrement", "précisément", "vivement", "c'est-à-dire", "envers", "rarement"],
            "Bloc 9": ["un brin", "une indication", "une inspection", "orphelin", "orpheline", "une épingle", "une industrie", "une installation", "un saint", "une sainte", "convaincre", "une infirmité", "un syndicat", "un instinct", "un gamin", "une gamine", "une injure", "un intervalle", "vain", "vaine", "incapable", "un inconvénient", "inquiéter", "un inspecteur", "une inspectrice", "un inventeur", "une inventrice", "mince"],
            "Bloc 10": ["une division", "une représentation", "une opposition", "accuser", "anglais", "anglaise", "une précision", "un réseau", "excuser", "une blouse", "une explosion", "une supposition", "un rasoir", "la confusion", "une réalisation", "une liaison", "une utilisation", "réaliser", "un loisir", "utiliser", "croiser", "un gaz", "un zèle", "une zone"],
            "Bloc 11": ["abandonner", "une croyance", "un monument", "une récolte", "une clôture", "endormir", "un oreiller", "un torrent", "un contrôle", "un laboratoire", "une pommade", "un comité", "un moment", "un projectile", "aucun", "aucune", "une audace", "un auditeur", "une sauce", "un carreau", "un plateau", "un taureau", "un tonneau", "un marteau", "un seau"],
            "Bloc 12": ["un chiffon", "un fonctionnement", "fuir", "un tarif", "une coiffure", "fondre", "le futur", "vérifier", "un confort", "un fossé", "une profession", "une étoffe", "la foudre", "un refus", "étouffer", "frotter", "siffler", "un éléphant", "une éléphante", "la géographie", "une pharmacie", "un photographe"],
            "Bloc 13": ["accepter", "céder", "emmener", "promener", "accompagner", "conserver", "libérer", "réparer", "allumer", "demeurer", "livrer", "situer", "amener", "deviner", "murmurer", "soulever", "appuyer", "élever", "noyer", "taper"],
            "Bloc 14": ["une matinée", "une poignée", "la durée", "une actualité", "une extrémité", "une nationalité", "la solidarité", "une antiquité", "la facilité", "la propreté", "une communauté", "la lâcheté", "la pureté", "une spécialité", "la sûreté", "une éternité", "une minorité", "la saleté", "un clocher", "un évier"],
            "Bloc 15 (Partie A)": ["un carré", "une client", "une cliente", "une courbe", "une cuisse", "un canal", "des canaux", "un alcool", "cracher", "une écurie", "un cavalier", "un balcon", "un creux", "une excursion", "un local", "des locaux", "un colis", "décrire", "une sculpture", "un dictionnaire", "une structure"],
            "Bloc 15 (Partie B)": ["une barque", "fabriquer", "se moquer", "un orchestre", "une piqûre", "la psychologie"],
            "Bloc 16": ["un salaire", "une selle", "une soustraction", "surveiller", "le sel", "une semelle", "un astre", "une estime", "un pistolet", "un veston", "une bourse", "une illustration", "risquer", "une vis", "une consigne", "un mécanisme", "le tourisme", "une dispute", "un moustique", "vaste", "une distraction", "un pasteur", "une veste"],
            "Bloc 17": ["une boisson", "une lessive", "un tissage", "une paresse", "un établissement", "une pâtisserie", "une ressource", "une abondance", "un divorce", "un édifice", "un caprice", "une absence", "un domicile", "une conférence", "un pouce"],
            "Bloc 18": ["une addition", "une décoration", "une multiplication", "une station", "une alimentation", "une diminution", "une population", "une traduction", "la consommation", "une élection", "une précaution", "un stationnement", "une constitution", "une formation", "une réduction", "une construction", "une imitation", "une réparation", "une création", "une inondation", "une révolution"],
            "Bloc 19": ["une catégorie", "le golfe", "grasse", "gronder", "élégance", "gonfler", "un grenier", "la grosseur", "une glace", "goûter", "une griffe", "guérir", "une vague", "la vigueur", "un guide"],
            "Bloc 20": ["une agence", "un engagement", "la largeur", "un pourcentage", "un apprentissage", "un étalage", "la logique", "un réfrigérateur", "un barrage", "le feuillage", "un naufrage", "un rivage", "un changement", "un garage", "un nettoyage", "rougir", "une chirurgie", "gémir", "le partage", "un stage", "un élevage", "une hygiène", "un potager", "une tige"],
            "Bloc 21": ["un achat", "gras", "un parcours", "le sirop", "un accord", "un mât", "un passeport", "un sport", "un client", "un mont", "un placard", "tabac", "un deuil", "l'odorat", "un puits", "un égout", "un paquebot", "roux"],
            "Bloc 22": ["une barre", "un génie", "une marmite", "un poêle", "un beurre", "humide", "le mérite", "un reproche", "une brochure", "lâche", "une panne", "un rhume", "une brûlure", "le luxe", "un parachute", "un stade", "une chimie", "le marbre", "un pétrole", "une tragédie"],
            "Bloc 23 (Partie A)": ["admettre", "une charrette", "l'électricité", "une moyenne", "une assiette", "une délicatesse", "émettre", "une ouverture", "une averse", "un dessert", "examiner", "une politesse", "avertir", "dessiner", "une fermeture", "une vieillesse", "blesser", "effacer", "la modestie"],
            "Bloc 23 (Partie B)": ["beige", "un chêne", "prêter", "un prêtre", "un sifflet", "un robinet", "aimable", "la graisse", "un itinéraire", "parfait", "parfaite", "alimentaire", "une grammaire", "une librairie", "une dizaine", "une haie", "un marais", "taire", "une épaisseur", "un horaire", "un minerai", "un vocabulaire", "une artère", "célèbre", "un congrès", "un règne", "une cavalière", "une clientèle", "une nièce", "la sève"],
            "Bloc 25": ["j'agitais", "tu accrochais", "il/elle entourait", "ils/elles attaquaient", "j'ordonnais", "tu dressais", "il/elle se réfugiait", "ils/elles baissaient", "je rassurais", "tu nageais", "il/elle séparait", "ils/elles plongeaient", "je reculais", "tu séchais", "il/elle supportait", "ils/elles saluaient", "je sonnais", "tu soufflais", "ils/elles attiraient"],
            "Bloc 26": ["une aptitude", "une hache", "un oubli", "prévenir", "une armoire", "une honte", "une ouïe", "prévoir", "une blancheur", "une lenteur", "un pâté", "un torchon", "la bravoure", "une littérature", "le péril", "un vallon", "le charbon", "une mâchoire", "un poumon", "une villa", "une étape", "obtenir", "un pourboire"]
        };

        let motsCourants = [];
        let indexMotCourant = 0;
        let selectedBlock = "";

        const blockListDiv = document.getElementById('block-list');
        const blockSelectionMenu = document.getElementById('block-selection-menu');
        const practiceSection = document.getElementById('practice-section');
        const playWordBtn = document.getElementById('play-word');
        const checkWordBtn = document.getElementById('check-word');
        const userInput = document.getElementById('user-input');
        const feedbackDiv = document.getElementById('feedback');
        const nextWordBtn = document.getElementById('next-word');
        const restartBlockBtn = document.getElementById('restart-block');
        const backToMenuBtn = document.getElementById('back-to-menu');
        const currentWordTitle = document.getElementById('current-word-title');

        function generateBlockButtons() {
            Object.keys(motsParBloc).forEach(blocName => {
                const button = document.createElement('button');
                button.textContent = blocName;
                button.className = "bg-gray-200 text-gray-800 font-bold py-3 px-4 rounded-lg shadow hover:bg-gray-300 transition duration-300 focus:outline-none focus:ring-2 focus:ring-gray-400";
                button.addEventListener('click', () => startPractice(blocName));
                blockListDiv.appendChild(button);
            });
        }

        function shuffle(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        function startPractice(blocName) {
            selectedBlock = blocName;
            blockSelectionMenu.classList.add('hidden');
            practiceSection.classList.remove('hidden');
            
            // On retire les parties entre parenthèses pour la dictée
            let motsPourDictée = motsParBloc[selectedBlock].map(word => {
                const parts = word.split('/');
                return parts[0].trim();
            });

            motsCourants = shuffle(motsPourDictée);
            indexMotCourant = 0;
            displayCurrentWord();
        }

        function displayCurrentWord() {
            if (indexMotCourant < motsCourants.length) {
                currentWordTitle.textContent = `Mot ${indexMotCourant + 1} sur ${motsCourants.length}`;
                userInput.value = '';
                feedbackDiv.textContent = '';
                userInput.focus();
                nextWordBtn.classList.add('hidden');
                checkWordBtn.classList.remove('hidden');
            } else {
                currentWordTitle.textContent = "Félicitations ! Vous avez terminé ce bloc.";
                userInput.classList.add('hidden');
                playWordBtn.classList.add('hidden');
                checkWordBtn.classList.add('hidden');
                nextWordBtn.classList.add('hidden');
                restartBlockBtn.classList.remove('hidden');
            }
        }

        function speak(text) {
            const utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = 'fr-FR';
            speechSynthesis.speak(utterance);
        }

        playWordBtn.addEventListener('click', () => {
            if (motsCourants.length > 0) {
                speak(motsCourants[indexMotCourant]);
            }
        });

        checkWordBtn.addEventListener('click', () => {
            const reponseUtilisateur = userInput.value.trim().toLowerCase();
            const motCorrect = motsCourants[indexMotCourant].trim().toLowerCase();

            if (reponseUtilisateur === motCorrect) {
                feedbackDiv.className = 'message-box correct';
                feedbackDiv.textContent = "Correct! Bien joué.";
                nextWordBtn.classList.remove('hidden');
                checkWordBtn.classList.add('hidden');
                userInput.disabled = true;
            } else {
                feedbackDiv.className = 'message-box incorrect';
                feedbackDiv.innerHTML = `Incorrect. La bonne orthographe est : <span class="font-bold">${motsCourants[indexMotCourant]}</span>`;
                checkWordBtn.classList.add('hidden');
                nextWordBtn.classList.remove('hidden');
                userInput.disabled = true;
            }
        });

        nextWordBtn.addEventListener('click', () => {
            indexMotCourant++;
            userInput.disabled = false;
            displayCurrentWord();
        });

        restartBlockBtn.addEventListener('click', () => {
            startPractice(selectedBlock);
            userInput.classList.remove('hidden');
            playWordBtn.classList.remove('hidden');
        });

        backToMenuBtn.addEventListener('click', () => {
            practiceSection.classList.add('hidden');
            blockSelectionMenu.classList.remove('hidden');
            userInput.disabled = false;
            userInput.classList.remove('hidden');
            playWordBtn.classList.remove('hidden');
            nextWordBtn.classList.add('hidden');
            checkWordBtn.classList.remove('hidden');
        });

        document.addEventListener('DOMContentLoaded', generateBlockButtons);
    </script>
</body>
</html>
