
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculatrice de Moyenne</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: white;
            margin: 0; /* optionnel : pour éviter les                 marges par défaut */
            height: 100vh; /* optionnel : s'assurer que le body prend toute la hauteur */
}
;
        }
        h1, h2, h3 {
            color: #333;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background-color: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: center;
        }
        th {
            background-color: #f2f2f2;
            font-weight: bold;
        }
        .module-header {
            background-color: #e0f0ff;
            font-weight: bold;
        }
        .semester-header {
            background-color: #d0e8ff;
            font-weight: bold;
        }
        .total-row {
            background-color: #ffffcc;
            font-weight: bold;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        input[type="number"] {
            width: 60px;
            padding: 5px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
        }
        button:hover {
            background-color: #45a049;
        }
        .fail {
            color: red;
            font-weight: bold;
        }
        .rattrapage {
            background-color: #ffebee;
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            border: 1px solid #ffcdd2;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1><center>Calculatrice de Moyenne</center></h1>
        
        <div id="app">
            <div id="results"></div>
            
            <div id="inputs">
                <h2><center>Entrez vos notes</center></h2>
                <div id="matieres-inputs"></div>
                <center><button onclick="calculerMoyennes()">Calculer les moyennes</button></center>
            </div>
        </div>
    </div>

    <script>
        // Structure des semestres, modules et matières
        const structure = {
            "Semestre 1": {
                "Module 1": {
                    "Langue pour scientifique": 2,
                    "Expression écrite et technique de rapport": 2
                },
                "Module 2": {
                    "Analyse": 3,
                    "Sciences des matériaux": 2,
                    "Électronique analogique": 3,
                    "Thermodynamique général": 2,
                    "Électrotechnique": 2,
                    "Mécanique des systèmes industriels": 2,
                    "Structure de la matière": 1.5
                },
                "Module 3": {
                    "Technologie de construction mécanique": 3,
                    "Dessin de construction mécanique": 3,
                    "Thermodynamique appliquée": 2,
                    "Transfert thermique": 2.5
                }
            },
            "Semestre 2": {
                "Module 4": {
                    "Algèbre linéaire et applications": 3,
                    "Analyse fonctionnelle et calcul opérationnel": 2,
                    "Probabilités et statistiques": 2.5,
                    "Informatique 1": 2,
                    "Dynamique des systèmes industriels": 1.5,
                    "Thermodynamique chimique": 1.5,
                    "Signaux et systèmes": 1.5
                },
                "Module 5": {
                    "Mécanique des fluides": 2,
                    "Système asservis linéaire": 2,
                    "Automatisme industrielle": 2,
                    "Métrologie": 2,
                    "Résistance des matériaux": 2,
                    "TP électronique électrotechnique et automatique": 1.5,
                    "TP signaux et systèmes et systèmes asservis linéaires": 1,
                    "TP métrologie et TP mécanique des fluides": 1
                }
            }
        };

        // Initialisation des champs de saisie
        function initialiserFormulaire() {
            const container = document.getElementById('matieres-inputs');
            
            for (const semestre in structure) {
                const semestreDiv = document.createElement('div');
                semestreDiv.innerHTML = `<h2><center>${semestre}</center></h2>`;
                
                for (const module in structure[semestre]) {
                    const moduleDiv = document.createElement('div');
                    moduleDiv.innerHTML = `<h3><center>${module}</center></h3>`;
                    
                    const table = document.createElement('table');
                    table.innerHTML = `
                        <tr>
                            <th>Matière</th>
                            <th>Crédits</th>
                            <th>CC</th>
                            <th>TPE</th>
                            <th>Synthèse</th>
                        </tr>
                    `;
                    
                    for (const matiere in structure[semestre][module]) {
                        const credits = structure[semestre][module][matiere];
                        const tr = document.createElement('tr');
                        tr.innerHTML = `
                            <td>${matiere}</td>
                            <td>${credits}</td>
                            <td><input type="number" min="0" max="20" step="0.25" id="cc_${semestre}_${module}_${matiere.replace(/ /g, '_')}"></td>
                            <td><input type="number" min="0" max="20" step="0.25" id="tpe_${semestre}_${module}_${matiere.replace(/ /g, '_')}"></td>
                            <td><input type="number" min="0" max="20" step="0.25" id="synth_${semestre}_${module}_${matiere.replace(/ /g, '_')}"></td>
                        `;
                        table.appendChild(tr);
                    }
                    
                    moduleDiv.appendChild(table);
                    semestreDiv.appendChild(moduleDiv);
                }
                
                container.appendChild(semestreDiv);
            }
        }

        // Calculer la note totale d'une matière
        function calculerNoteMatiere(cc, tpe, synthese) {
            if (cc === null || synthese === null) return null;
            
            // Si pas de TPE, 70% synthèse + 30% CC
            if (tpe === null) {
                return 0.7 * synthese + 0.3 * cc;
            }
            
            // Sinon 70% synthèse + 20% CC + 10% TPE
            return 0.7 * synthese + 0.2 * cc + 0.1 * tpe;
        }

        // Calculer les moyennes et afficher les résultats
        function calculerMoyennes() {
            const notes = {};
            const matieresSousMinimum = [];
            
            // Récupérer toutes les notes
            for (const semestre in structure) {
                notes[semestre] = {};
                
                for (const module in structure[semestre]) {
                    notes[semestre][module] = {};
                    
                    for (const matiere in structure[semestre][module]) {
                        const ccInput = document.getElementById(`cc_${semestre}_${module}_${matiere.replace(/ /g, '_')}`);
                        const tpeInput = document.getElementById(`tpe_${semestre}_${module}_${matiere.replace(/ /g, '_')}`);
                        const synthInput = document.getElementById(`synth_${semestre}_${module}_${matiere.replace(/ /g, '_')}`);
                        
                        const cc = ccInput.value !== "" ? parseFloat(ccInput.value) : null;
                        const tpe = tpeInput.value !== "" ? parseFloat(tpeInput.value) : null;
                        const synth = synthInput.value !== "" ? parseFloat(synthInput.value) : null;
                        
                        const noteFinale = calculerNoteMatiere(cc, tpe, synth);
                        const credits = structure[semestre][module][matiere];
                        
                        notes[semestre][module][matiere] = {
                            cc: cc,
                            tpe: tpe,
                            synth: synth,
                            final: noteFinale,
                            credits: credits
                        };
                        
                        // Vérifier si la matière est sous le minimum de 10
                        if (noteFinale !== null && noteFinale < 10) {
                            matieresSousMinimum.push({
                                semestre: semestre,
                                module: module,
                                matiere: matiere,
                                note: noteFinale.toFixed(2)
                            });
                        }
                    }
                }
            }
            
            // Calculer les moyennes par module, semestre et année
            const moyennes = calculerMoyennesParModule(notes);
            
            // Afficher les résultats
            afficherResultats(notes, moyennes, matieresSousMinimum);
        }

        // Calculer les moyennes par module, semestre et année
        function calculerMoyennesParModule(notes) {
            const moyennes = {
                modules: {},
                semestres: {},
                annee: 0
            };
            
            let totalCreditsAnnee = 0;
            let totalPointsAnnee = 0;
            
            for (const semestre in notes) {
                let totalCreditsSemestre = 0;
                let totalPointsSemestre = 0;
                
                for (const module in notes[semestre]) {
                    let totalCreditsModule = 0;
                    let totalPointsModule = 0;
                    let noteMinModule = 20;
                    
                    for (const matiere in notes[semestre][module]) {
                        const note = notes[semestre][module][matiere];
                        
                        if (note.final !== null) {
                            totalCreditsModule += note.credits;
                            totalPointsModule += note.final * note.credits;
                            
                            if (note.final < noteMinModule) {
                                noteMinModule = note.final;
                            }
                        }
                    }
                    
                    // Calculer la moyenne du module
                    let moyenneModule = 0;
                    if (totalCreditsModule > 0) {
                        moyenneModule = totalPointsModule / totalCreditsModule;
                    }
                    
                    // Vérifier si le module a une note < 8
                    if (noteMinModule < 8) {
                        moyenneModule = "EL";
                    } else if (moyenneModule < 10) {
                        moyenneModule = "EL";
                    } else {
                        totalCreditsSemestre += totalCreditsModule;
                        totalPointsSemestre += totalPointsModule;
                    }
                    
                    moyennes.modules[`${semestre}_${module}`] = {
                        moyenne: moyenneModule,
                        credits: totalCreditsModule
                    };
                }
                
                // Calculer la moyenne du semestre
                if (totalCreditsSemestre > 0) {
                    moyennes.semestres[semestre] = totalPointsSemestre / totalCreditsSemestre;
                    totalCreditsAnnee += totalCreditsSemestre;
                    totalPointsAnnee += totalPointsSemestre;
                } else {
                    moyennes.semestres[semestre] = 0;
                }
            }
            
            // Calculer la moyenne annuelle
            if (totalCreditsAnnee > 0) {
                moyennes.annee = totalPointsAnnee / totalCreditsAnnee;
            }
            
            return moyennes;
        }

        // Afficher les résultats dans un tableau
        function afficherResultats(notes, moyennes, matieresSousMinimum) {
            const resultsDiv = document.getElementById('results');
            resultsDiv.innerHTML = "";
            
            const table = document.createElement('table');
            
            // En-tête du tableau
            table.innerHTML = `
                <tr>
                    <th>Matière</th>
                    <th>Crédits</th>
                    <th>CC</th>
                    <th>TPE</th>
                    <th>Synthèse</th>
                    <th>Note Finale</th>
                </tr>
            `;
            
            // Parcourir les semestres et modules
            for (const semestre in structure) {
                // Ajouter l'en-tête du semestre
                const trSemestre = document.createElement('tr');
                trSemestre.className = 'semester-header';
                trSemestre.innerHTML = `<td colspan="6">${semestre}</td>`;
                table.appendChild(trSemestre);
                
                for (const module in structure[semestre]) {
                    // Ajouter l'en-tête du module
                    const trModule = document.createElement('tr');
                    trModule.className = 'module-header';
                    trModule.innerHTML = `<td colspan="6">${module}</td>`;
                    table.appendChild(trModule);
                    
                    // Ajouter les matières du module
                    for (const matiere in structure[semestre][module]) {
                        const noteMatiere = notes[semestre][module][matiere];
                        const tr = document.createElement('tr');
                        
                        let noteFinaleAffichage = noteMatiere.final !== null ? noteMatiere.final.toFixed(2) : "-";
                        let classeNote = "";
                        
                        if (noteMatiere.final !== null && noteMatiere.final < 10) {
                            classeNote = "fail";
                        }
                        
                        tr.innerHTML = `
                            <td>${matiere}</td>
                            <td>${noteMatiere.credits}</td>
                            <td>${noteMatiere.cc !== null ? noteMatiere.cc.toFixed(2) : "-"}</td>
                            <td>${noteMatiere.tpe !== null ? noteMatiere.tpe.toFixed(2) : "-"}</td>
                            <td>${noteMatiere.synth !== null ? noteMatiere.synth.toFixed(2) : "-"}</td>
                            <td class="${classeNote}">${noteFinaleAffichage}</td>
                        `;
                        
                        table.appendChild(tr);
                    }
                    
                    // Ajouter la moyenne du module
                    const moyenneModule = moyennes.modules[`${semestre}_${module}`];
                    const trMoyenneModule = document.createElement('tr');
                    trMoyenneModule.className = 'total-row';
                    
                    let moyenneModuleAffichage = typeof moyenneModule.moyenne === "number" ? moyenneModule.moyenne.toFixed(2) : moyenneModule.moyenne;
                    let classeModuleNote = "";
                    
                    if (moyenneModuleAffichage === "EL") {
                        classeModuleNote = "fail";
                    }
                    
                    trMoyenneModule.innerHTML = `
                        <td>Moyenne ${module}</td>
                        <td>${moyenneModule.credits}</td>
                        <td colspan="3"></td>
                        <td class="${classeModuleNote}">${moyenneModuleAffichage}</td>
                    `;
                    
                    table.appendChild(trMoyenneModule);
                }
                
                // Ajouter la moyenne du semestre
                const moyenneSemestre = moyennes.semestres[semestre];
                const trMoyenneSemestre = document.createElement('tr');
                trMoyenneSemestre.className = 'total-row';
                trMoyenneSemestre.innerHTML = `
                    <td colspan="5">Moyenne ${semestre}</td>
                    <td>${moyenneSemestre.toFixed(2)}</td>
                `;
                
                table.appendChild(trMoyenneSemestre);
            }
            
            // Ajouter la moyenne annuelle
            const trMoyenneAnnuelle = document.createElement('tr');
            trMoyenneAnnuelle.className = 'total-row';
            trMoyenneAnnuelle.innerHTML = `
                <td colspan="5">Moyenne Annuelle</td>
                <td>${moyennes.annee.toFixed(2)}</td>
            `;
            
            table.appendChild(trMoyenneAnnuelle);
            
            resultsDiv.appendChild(table);
            
            // Afficher les matières à rattraper
            if (matieresSousMinimum.length > 0) {
                const rattrapageDiv = document.createElement('div');
                rattrapageDiv.className = 'rattrapage';
                rattrapageDiv.innerHTML = `<h3>Matières à rattraper (note < 10)</h3>`;
                
                const tableRattrapage = document.createElement('table');
                tableRattrapage.innerHTML = `
                    <tr>
                        <th>Semestre</th>
                        <th>Module</th>
                        <th>Matière</th>
                        <th>Note</th>
                    </tr>
                `;
                
                matieresSousMinimum.forEach(matiere => {
                    const tr = document.createElement('tr');
                    tr.innerHTML = `
                        <td>${matiere.semestre}</td>
                        <td>${matiere.module}</td>
                        <td>${matiere.matiere}</td>
                        <td class="fail">${matiere.note}</td>
                    `;
                    tableRattrapage.appendChild(tr);
                });
                
                rattrapageDiv.appendChild(tableRattrapage);
                resultsDiv.appendChild(rattrapageDiv);
            }
        }

        // Initialiser le formulaire au chargement
        window.onload = initialiserFormulaire;
    </script> 
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Footer Professionnel</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap');
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Poppins', sans-serif;
        }
        
        .footer {
            background: linear-gradient(to right, #1a1a1a, #2d2d2d, #1a1a1a);
            color: #fff;
            padding: 40px 20px;
            border-top: 4px solid #3498db;
        }
        
        .footer-container {
     
