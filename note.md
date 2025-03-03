# Requêtes MongoDB pour la gestion d'une bibliothèque

## Listez tous les livres disponibles

db.livres.find({ disponible: true });


## Trouvez les livres publiés après l'an 2000

db.livres.find({ annee_publication: { $gt: 2000 } });

## Trouvez les livres d'un auteur spécifique

db.livres.find({ auteur: "George Orwell" });


## Trouvez les livres qui ont une note moyenne supérieure à 4

db.livres.find({ note_moyenne: { $gt: 4 } });

## Listez tous les utilisateurs habitant dans une ville spécifique

db.utilisateurs.find({ "adresse.ville": "Lyon" });


## Trouvez les livres qui appartiennent à un genre spécifique

db.livres.find({ genre: "Science-fiction" });


## Trouvez les livres qui ont à la fois un prix inférieur à 15€ et une note moyenne supérieure à 4

db.livres.find({
  prix: { $lt: 15 },
  note_moyenne: { $gt: 4 }
});


## Trouvez les utilisateurs qui ont emprunté un livre spécifique (par titre)

db.utilisateurs.find({ "livres_empruntes.titre": "Le Petit Prince" });

