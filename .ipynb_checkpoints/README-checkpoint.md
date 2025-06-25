# package_convert_pn13_to_fhir
Script Python permettant de convertir un prescription médicale au standard PN13 (fichier XML) en une ressource MedicationRequest au standard FHIR (fichier JSON). Les fichiers en sortie ont été validés par un serveur HAPI FHIR v 8.0.


## Crédit

Le mapping de mon script est basé sur celui d'Intérop'Santé (mise à jour du 05/05/2025) :
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-medicationrequest-conceptmap.html
https://packages2.fhir.org/xig/hl7.fhir.fr.medication%7Ccurrent/ConceptMap/PN13-FHIR-prescmed-patient-id-seul-conceptmap
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-medicationnoncompound-conceptmap.html
https://build.fhir.org/ig/Interop-Sante/hl7.fhir.fr.medication/ConceptMap-PN13-FHIR-prescmed-practitioner-identite-conceptmap.html 


## 🧩 Fonctionnalités

- Lecture d'un fichiers PN13 en entrée
- Utilisation des mapping
- Transformation de plusieurs champs pour leur intégration dans un fichier FHIR.
- Pour être exécuté, l'environnement du script doit contenir le fichier Excel qui représente la table des médicaments déjà connus (équivalent simplifié des ressources Medication FHIR) et le fichiers CSV qui représente la table des voies d'administration. Ces derniers sont mis à jour si le code de la prescription n'est pas présent dans la table.
- Fait apparaitre un extrait des ressources Medication et Practitioner.
- Génération d'un fichier FHIR en sortie.



## 📦 Installation

### Cloner ce dépôt

```bash
git clone https://github.com/TDumetz/package_convert_pn13_to_fhir.git
cd package_convert_pn13_to_fhir
```

## Utilisation
- La version de votre python doit être supérieur ou égale à la 3.11.13
- L'environnement doit contenir le script, le fichier XML, le fichier Excel et le fichier CSV

## Traitement concret de chaque champs
Mapping réalisé :
-	Messages@Phast-id_message est mappé avec le champ indiqué (pas testé) -> si le champ est vide alors on renvoi dans le FHIR les derniers numéro (après le dernier « _ ») 
-	La date et l'heure sont transformées dans le bon format si elles respectent les conditions sinon elles sont renvoyées tel quel
-	Affiche l’extension et le préfix obligatoire pour le champ commentaire
-	Identifiant_auteur RAS (mais pas présent dans mes fichiers PN13)
-	Nom_usage, Prénom_usage et Titre sont dans le même champ
-	Affiche l’extension et le préfix obligatoire pour le champ Texte
-	Si le champ Valeur_rens_compl a une valeur on n’affiche pas le champ Rens_compl
-	Affiche l’extension et le préfix obligatoire pour le champ Libellé_élément_prescr
-	La valeur de priority (dans FHIR) prend une valeur différente en fonction de la valeur du champ Urgent et renvoie « routine » si Urgent n’est pas présent dans le PN13
-	Fourniture, inclue dans la ressource medication, fait apparaitre true lorsqu’il y a 1, false lorsqu’il y a 0 et le champs tel quel dans les autres cas et fait apparaitre son extension 
-	Voie d’administration renvoie le code de la voie et le system http://standardterms.edqm.eu grâce à une fonction (add_system_if_code)
-	Lieu_administration RAS
-	Dispositif_associé RAS + « device » est ajouté dans le champ ("supportingInformation", "type") si Dispositif_associé a une valeur grâce à la fonction add_device_if_dispositif
-	Affiche l’extension et le préfix obligatoire pour le champ Posologie
-	Dh_début_prescrite est affiché au bon format seulement si Dh_Début n’a pas de valeur (mapping conditionnel)
-	Dh_fin_prescrite est affiché au bon format seulement si Dh_fin n’a pas de valeur (mapping conditionnel)
-	Affiche l’extension et le préfix obligatoire pour le champ Indication
-	Affiche l’extension et le préfix obligatoire pour le champ Commentaire
-	Identifiant_auteur est dans la balise commentaire_structurée (pas testé)
-	Nom_usage, Prénom_usage et Titre sont dans le même champ
-	Affiche l’extension et le préfix obligatoire pour le champ Texte
-	La valeur de status (dans FHIR) prend une valeur différente en fonction de la valeur de GONOGO, si la valeur de GONOGO n’est pas parmi celles prévus on renvoi la valeur tel quel, s’il n’y a pas le champ GONOGO status = active, intent prend toujours la valeur order (static mapping)
-	Motif_attente RAS
-	Affiche l’extension et le préfix obligatoire pour le champ Conditions_application
-	La valeur de periodUnit ou period (dans FHIR) prend une valeur différente en fonction de la valeur de Frq_échelle, si la valeur de Frq_échelle n’est pas parmi celles prévus on renvoi la valeur tel quel,
-	Si Fréquence et Fréquence_structurée ne sont pas présent alors Dosage.timing.repeat.period = 1   et Dosage.timing.repeat.periodUnit = d
-	Frq_libellé RAS
-	Id_séjour RAS
-	Rajoute un préfixe à Ipp + les deux champs : type et use 
-	Nom_usuel et Prénoms sont inclus dans le même champ
-	Le script permet de rajouter le code et le libellé de 1 ou plusieurs médicaments à la fin de la table des médicaments si le code n’est pas présent dans celle-ci
-	Le script permet de rajouter le code et le libellé de 1 voie d’administration à la fin de la table des voies d’administrations si le code n’est pas présent dans celle-ci 
-	Le champ Elément_posologie/Fréquence permet de compléter les champs frequency, period et periodUnit selon un mapping à compléter pour chaque valeur du PN13 possible -> le résultat perd de l’information lorsqu’on a une prise par semaine puisque le jour n’est pas précisé dans les champs FHIR (sauf dans le texte)
-	Fréquence_structurée/Frq_durée a été inclue dans le simple mapping + le champ period prend la valeur 1 par défaut s’il n’est pas déjà présent dans le FHIR
-	Création de la fonction map_structured_frequency pour traiter tous les champs Frq_filtreVal_1_N Frq_filtreVal_2 Frq_filtreVal_3 Frq_filtreVal_4 Frq_filtreVal_5 Frq_filtreVal_6 en même temps (pas testé)
-	Frq_multiplicité : simple mapping (pas testé) + frequency prend la valeur 1 par défaut si le champ n’est pas déjà présent dans le FHIR
-	Frq_libellé : simple mapping (pas testé)
-	Int_temps_év_début est en simple mapping vers offset / lorsqu’il apparait sous la forme : Int_temps_év_début/Nombre et Int_temps_év_début/Unité sont transformé en minutes pour être inclue dans le champ offset (pas testés)
-	Durée/Nombre et Durée/Unité sont modifiés pour que l’unité soit parmi celles de l’UCUM et le nombre apparaisse sans les zéros (fonction parse_duration) -> tous les cas possibles n’ont pas été testés
-	Quantité/Nombre et Quantité/Unité sont intégrés en simple mapping, l’unité n’est pas renvoyée lorsqu’elle prend la valeur « dose » et le system « http://data.esante.gouv.fr/coe/standardterms » est ajouté
-	Débit/Nombre et Débit/Unité sont intégrés en simple mapping, l’unité n’est pas renvoyée lorsqu’elle prend la valeur « dose » et le system « http://unitsofmeasure.org » est ajouté
-	Création de la ressource Practioner ou j’intègre les éléments Identifiant, Nom_usage et Prénom_usage de Identification_prescripteur.
-	Le champ requester.reference est automatiquement créé avec un lien vers la ressource practitioner créé juste au-dessus
-	Le script renvoie un message pour alerter lorsqu’un médicament ou une voie d’administration est rajouté à la base Excel
-	Affiche l’extension pour Unité_hébergement 
-	Affiche l’extension pour Unité_resp_médicale 

Mapping incomplet : 
-	Id_élément_prescrit n’est pas accompagné d’un system
-	Forme n’est pas accompagné d’un system
-	Le script ne prévoit pas d’inclure le champ allowed dans FHIR si Non_substituable  n’est pas présent dans le PN13 alors que allowed est obligatoire dans un FHIR