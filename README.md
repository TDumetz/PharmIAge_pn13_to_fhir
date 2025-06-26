# convert_pn13_to_fhir
Script Python permettant de convertir un prescription médicale au standard PN13 (fichier XML) en une ressource MedicationRequest au standard FHIR (fichier JSON). Les fichiers en sortie ont été validés par un serveur HAPI FHIR v 8.0.


## Crédit

Le mapping de mon script est basé sur celui d'Intérop'Santé (mise à jour du 05/05/2025) :
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-medicationrequest-conceptmap.html
https://packages2.fhir.org/xig/hl7.fhir.fr.medication%7Ccurrent/ConceptMap/PN13-FHIR-prescmed-patient-id-seul-conceptmap
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-medicationnoncompound-conceptmap.html
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-practitioner-identite-conceptmap.html
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-dosageinstruction-conceptmap.html


## Fonctionnalités

- Lecture d'un fichiers PN13 en entrée
- Utilisation des mapping
- Transformation de plusieurs champs pour leur intégration dans un fichier FHIR.
- Pour être exécuté, l'environnement doit respecter celui du GitHub
- Le package contient un fichier Excel qui représente la table des médicaments déjà connus (équivalent simplifié des ressources Medication FHIR) et un fichiers CSV qui représente la table des voies d'administration. Ces derniers sont mis à jour si le code de la prescription n'est pas présent dans la table.
- La prescription créée contient un extrait des ressources Medication et Practitioner.
- Génération d'un fichier FHIR en sortie.


## Utilisation
- La version de votre python doit être supérieur ou égale à la 3.11.13
- Exécutez les commandes suivantes dans un terminal pour importer l'ensemble du contenu GitHub et exécuter le script :

### Installation de Git

conda install git #sous conda

winget install --id Git.Git -e --source winget #sous Windows

brew install git #sous macOS (via Homebrew)

sudo apt update
sudo apt install git #sous Linux (Debian/Ubuntu)

### Installation du package à partir de mon GitHub

pip install git+https://github.com/TDumetz/PharmIAge_pn13_to_fhir.git

### Se placer dans le bon répertoire

cd PharmIAge_pn13_to_fhir

### Ajout dynamique du dossier src au chemin de recherche des modules Python

set PYTHONPATH=src #sous Windows

export PYTHONPATH=src #sous Linux/macOS

### Exécute le script avec le fichier XML contenu sur le GitHub

python -m pn13_to_fhir.main input/2025051911285175_0000314_64166416_19924082.xml


## Autres utilisations
Pour convertir un autre fichier vous devez :
- placer votre fichier XML dans le dossier input
- adapter le script en remplaçant le nom du fichier XML par celui du nouveau (ligne 1365)
- adapter la commande utilisée pour exécuter le script avec le nouveau nom du fichier


## Traitement concret de chaque champs
Mapping réalisé :
-	Messages@Phast-id_message est mappé avec le champ MedicationRequest.groupIdentifier.value (pas testé) -> si le champ est vide alors on renvoi dans le FHIR les derniers numéro (après le dernier « _ ») 
-	Messages/M_prescription_médicaments/Prescription/Dh_prescription mappé avec MedicationRequest.authoredOn et est transformé dans le bon format si il respecte les conditions sinon il est renvoyé tel quel
-	Messages/M_prescription_médicaments/Prescription/Unité_hébergement mappé avec MedicationRequest.supportingInformation.identifier et ajout de l'extension
-	Messages/M_prescription_médicaments/Prescription/Unité_resp_médicale mappé avec MedicationRequest.supportingInformation.identifier et ajout de l'extension
-	Messages/M_prescription_médicaments/Prescription/Commentaire mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Commentaire_structuré/Identification_auteur/Identifiant mappé avec MedicationRequest.note.authorReference.identifier.value (pas testé)
-	Messages/M_prescription_médicaments/Prescription/Commentaire_structuré/Identification_auteur/Nom_usage mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Commentaire_structuré/Identification_auteur/Prénom_usage mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Commentaire_structuré/Identification_auteur/Titre mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Commentaire_structuré/Texte mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Rens_compl/Valeur_rens_compl mappé avec MedicationRequest.note.text et si il n'a pas de valeur on mappe le champ Messages/M_prescription_médicaments/Prescription/Rens_compl avec MedicationRequest.supportingInformation.reference
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Libellé_élément_prescr mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Urgent mappé avec MedicationRequest.priority et renvoie « routine » si Urgent n’est pas présent dans le PN13
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Fourniture mappé avec MedicationRequest.medication.extension.valueBoolean (inclue dans l'extrait de la ressource Medication) et fait apparaitre true lorsque sa valeur est 1, false lorsque sa valeur est 0 et le champs tel quel dans les autres cas et ajout de son extension
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Voie_administration mappé avec MedicationRequest.dosageInstruction.route.coding.code et renvoie le "system" http://standardterms.edqm.eu (grâce à la fonction add_system_if_code)
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Lieu_administration mappé avec MedicationRequest.dispenseRequest.performer.display
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Dispositif_associé mappé avec MedicationRequest.supportingInformation.display et la valeur "device" est ajoutée dans le champ supportingInformation.type si Dispositif_associé a une valeur grâce à la fonction add_device_if_dispositif
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Posologie mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Dh_début_prescrite mappé avec MedicationRequest.dosageInstruction.timing.repeat.boundsPeriod.start (format dateTime) seulement si Dh_début n’a pas de valeur (mapping conditionnel)
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Dh_début mappé avec MedicationRequest.dosageInstruction.timing.repeat.boundsPeriod.start au format dateTime
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Dh_fin_prescrite mappé avec MedicationRequest.dosageInstruction.timing.repeat.boundsPeriod.end (format dateTime) seulement si Dh_fin n’a pas de valeur (mapping conditionnel)
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Dh_fin mappé avec MedicationRequest.dosageInstruction.timing.repeat.boundsPeriod.end au format dateTime
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Indication mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire_structuré/Identification_auteur/Identifiant mappé avec MedicationRequest.note.authorReference.identifier.value (pas testé)
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire_structuré/Identification_auteur/Nom_usage mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire_structuré/Identification_auteur/Prénom_usage mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire_structuré/Identification_auteur/Titre mappé avec MedicationRequest.note.authorReference.display
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Commentaire_structuré/Texte mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/GoNogo mappé avec MedicationRequest.status et MedicationRequest.intent qui prend toujours la valeur "order" (static mapping)
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Motif_attente mappé avec MedicationRequest.statusReason.text
-	Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Conditions_application mappé avec MedicationRequest.note.text et ajout de l'extension et du préfixe
-	Tout le contenu de la balise Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Composant_prescrit mappé avec l'extrait de la ressource Medication. Création d'une ressource Medication pour chaque balise Composant_prescrit. L’id de chaque ressource est l’UCD du médicament prescrit. Les compasants sont :
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Composant_prescrit/Code_composant_1 mappé avec Medication.code.coding.code et ajout de son system
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Composant_prescrit/Libellé_composant mappé avec Medication.code.text
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Forme mappé avec Medication.form.coding.code
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie mappé avec MedicationRequest.dosageInstruction.timing.repeat.period et MedicationRequest.dosageInstruction.timing.repeat.periodUnit si Fréquence et Fréquence_structurée sont absents
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Fréquence mappé avec MedicationRequest.dosageInstruction.timing.frequency , MedicationRequest.dosageInstruction.timing.period et MedicationRequest.dosageInstruction.timing.periodUnit . Un mapping des valeurs possibles de la fréquence est à complété dans le script si besoin.
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Fréquence_structurée/Frq_échelle mappé avec MedicationRequest.dosageInstruction.timing.repeat.periodUnit
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Fréquence_structurée/Frq_durée mappé avec MedicationRequest.dosageInstruction.timing.repeat.period et égale à 1 si absent dans le PN13
- Elément_posologie/Fréquence_structurée/Frq_filtre/Frq_filtreVal_1_J mappé avec MedicationRequest.dosageInstruction.timing.repeat.dayOfWeek
- Elément_posologie/Fréquence_structurée/Frq_filtre/Frq_filtreVal_1_N , Frq_filtreVal_2	, Frq_filtreVal_3 , Frq_filtreVal_4 , Frq_filtreVal_5 , Frq_filtreVal_6 mappés avec MedicationRequest.dosageInstruction.timing.repeat grâce à la fonction map_structured_frequency (pas testé)
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Fréquence_structurée/Frq_multiplicité mappé avec MedicationRequest.dosageInstruction.timing.repeat.frequency (pas testé) et MedicationRequest.dosageInstruction.timing.repeat.frequency prend la valeur 1 par défaut si le champ n’est pas déjà présent dans le FHIR
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Fréquence_structurée/Frq_libellé mappé avec MedicationRequest.dosageInstruction.text (pas testé)
- Elément_posologie/Evénement_début mappé avec MedicationRequest.dosageInstruction.timing
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Elément_posologie/Int_temps_év_début mappé avec MedicationRequest.dosageInstruction.timing.repeat.offset . Lorsque le champs apparait, dans le PN13, sous la forme Int_temps_év_début/Nombre et Int_temps_év_début/Unité alors on transforme le résultat en minutes pour être inclue dans le champ offset (pas testés)
- Elément_posologie/Durée/Nombre mappé avec MedicationRequest.dosageInstruction.timing.repeat.duration et Elément_posologie/Durée/Unité mappé avec MedicationRequest.dosageInstruction.timing.repeat.durationUnit ont été modifiés pour que l’unité utilisée soit parmi celles de l’UCUM et que le nombre apparaisse sans les zéros inutiles grâce à la fonction parse_duration
- Elément_posologie/Quantité/Nombre mappé avec MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.value
- Elément_posologie/Quantité/Unité mappé avec MedicationRequest.dosageInstruction.doseAndRate.doseQuantity.code l’unité n’est pas renvoyée lorsqu’elle prend la valeur « dose » et ajout de son "system"
- Elément_posologie/Débit/Nombre mappé avec MedicationRequest.dosageInstruction.doseAndRate.rateQuantity.value
- Elément_posologie/Débit/Unité mappé avec MedicationRequest.dosageInstruction.doseAndRate.rateQuantity.code l’unité n’est pas renvoyée lorsqu’elle prend la valeur « dose » et ajout de son "system"
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Composant_prescrit/Non_substituable mappé avec MedicationRequest.substitution.allowedBoolean
- Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Composant_prescrit/Référent_poso mappé avec MedicationRequest.dosageInstruction.doseAndRate.extension.valueReference
- Messages/M_prescription_médicaments/Séjour/Id_séjour mappé avec MedicationRequest.encounter.identifier.value
- Messages/M_prescription_médicaments/Patient/Ipp mappé avec MedicationRequest.subject.identifier.value et ajout des champs MedicationRequest.subject.type et MedicationRequest.subject.identifier.use
- Messages/M_Prescription_médicaments/Patient/Nom_usuel mappé avec MedicationRequest.subject.display
- Messages/M_Prescription_médicaments/ Patient/ Prénoms mappé avec MedicationRequest.subject.display
- Tout le contenu de la balise Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Identification_prescripteur mappé avec l'extrait de la ressource Practictioner. L'id de la ressource est alléatoire. Le champ MedicationRequest.requester.reference est automatiquement créé avec un lien vers la ressource Practictioner. Les compasants sont :
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Identification_prescripteur/Identifiant mappé avec Practitioner.identifier.value
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Identification_prescripteur/Nom_usage mappé avec Practitioner.name.family
    - Messages/M_prescription_médicaments/Prescription/Elément_prescr_médic/Identification_prescripteur/Prénom_usage mappé avec Practitioner.name.given


Mapping incomplet : 
-	Id_élément_prescrit n’est pas accompagné d’un system
-	Forme n’est pas accompagné d’un system
-	Le script ne prévoit pas d’inclure le champ allowed dans FHIR si Non_substituable  n’est pas présent dans le PN13 alors que allowed est obligatoire dans un FHIR


Ajout supplémentaire :
-	Le script permet de rajouter le code et le libellé de un ou plusieurs médicaments à la fin d'une table Excel du médicament si le code n’est pas présent dans celle-ci
-	Le script permet de rajouter le code et le libellé de une voie d’administration à la fin d'une table CSV des voies d’administrations si le code n’est pas présent dans celle-ci  
-	Le script renvoie un message pour alerter lorsqu’un médicament ou une voie d’administration est rajouté dans les bases
