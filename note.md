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


# Partie 3 : Mise à jour de documents (Update)

## Mettez à jour le titre d'un livre spécifique

db.livres.updateOne(
  { titre: "1984" },
  { $set: { titre: "1984 - Nouvelle Édition" } }
);


## Ajoutez un champ stock à tous les livres avec une valeur par défaut de 5

db.livres.updateMany(
  {},
  { $set: { stock: 5 } }
);


## Marquez un livre comme indisponible (disponible = false)

db.livres.updateOne(
  { titre: "Le Petit Prince" },
  { $set: { disponible: false } }
);


## Ajoutez un nouvel emprunt dans la liste livres_empruntes d'un utilisateur

db.utilisateurs.updateOne(
  { email: "marie.dupont@example.com" },
  { $push: { livres_empruntes: {
      livre_id: ObjectId("ID_DU_LIVRE"),
      titre: "Les Misérables",
      date_emprunt: new Date("2023-05-10"),
      date_retour_prevue: new Date("2023-06-10")
    }
  }}
);


## Changez l'adresse d'un utilisateur

db.utilisateurs.updateOne(
  { email: "marie.dupont@example.com" },
  { $set: { adresse: {
      rue: "456 Rue des Bouquinistes",
      ville: "Marseille",
      code_postal: "13001"
    }
  }}
);

## Ajoutez un nouveau tag à un utilisateur

db.utilisateurs.updateOne(
  { email: "marie.dupont@example.com" },
  { $push: { tags: "philosophie" } }
);


## Mettez à jour la note moyenne d'un livre

db.livres.updateOne(
  { titre: "Les Misérables" },
  { $set: { note_moyenne: 4.9 } }
);


# Partie 4 : Suppression de documents (Delete)

## Supprimez un livre spécifique par son titre

db.livres.deleteOne(
  { titre: "1984 - Nouvelle Édition" }
);


## Supprimez tous les livres d'un auteur spécifique

db.livres.deleteMany(
  { auteur: "George Orwell" }
);


## Supprimez un utilisateur par son email

db.utilisateurs.deleteOne(
  { email: "marie.dupont@example.com" }
);


# Partie 5 : Requêtes avancées et projection

## Listez tous les livres triés par note moyenne (ordre décroissant)

db.livres.find().sort({ note_moyenne: -1 });


## Trouvez les 3 livres les plus anciens

db.livres.find().sort({ annee_publication: 1 }).limit(3);


## Comptez le nombre de livres par auteur

db.livres.aggregate([
  { $group: { _id: "$auteur", total: { $sum: 1 } } }
]);


## Affichez uniquement le titre, l'auteur et la note moyenne des livres (sans l'id)

db.livres.find({}, { titre: 1, auteur: 1, note_moyenne: 1, _id: 0 });


## Trouvez les utilisateurs qui ont emprunté plus d'un livre

db.utilisateurs.find({ "livres_empruntes.1": { $exists: true } });


## Recherchez les livres dont le titre contient un mot spécifique (utilisez $regex)

db.livres.find({ titre: { $regex: "Harry", $options: "i" } });


## Trouvez les livres dont le prix est entre 10€ et 20€

db.livres.find({ prix: { $gte: 10, $lte: 20 } });
