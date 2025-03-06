## Exercice 1.1 :

Recherche par titre exact : db.livres.find({ titre: "Le Petit Prince" }).explain("executionStats")

Recherche par auteur : db.livres.find({ auteur: "Antoine de Saint-Exupéry" }).explain("executionStats")

Recherche par plage de prix (entre 10€ et 20€) et note minimale : db.livres.find({ 
    prix: { $gte: 10, $lte: 20 }, 
    note: { $gte: 4 }
}).explain("executionStats")

Recherche filtrée par genre et langue avec tri par note décroissante : db.livres.find({ 
    genre: "Fiction", 
    langue: "Français" 
}).sort({ note: -1 }).explain("executionStats")


## Exercice 1.2 :

db.livres.createIndex({ titre: 1 })
db.livres.createIndex({ auteur: 1 })
db.livres.createIndex({ prix: 1, note: 1 })
db.livres.createIndex({ genre: 1, langue: 1, note: -1 })


## Exercice 1.3 :

db.livres.createIndex({ titre: "text", description: "text" })
db.livres.find({ $text: { $search: "Prince" } })
db.sessions_utilisateurs.createIndex({ dateDerniereActivite: 1 }, { expireAfterSeconds: 1800 }) // 30 minutes


## Exercice 1.4 :

db.livres.createIndex({ auteur: 1, titre: 1 })
db.livres.find({ auteur: "Antoine de Saint-Exupéry" }, { titre: 1, _id: 0 }).explain("executionStats")
db.livres.createIndex({ ISBN: 1 }, { unique: true })
db.livres.insertOne({ titre: "Nouveau Livre", ISBN: "1234567890" })
db.livres.insertOne({ titre: "Autre Livre", ISBN: "1234567890" }) 
db.livres.createIndex({ disponible: 1 }, { partialFilterExpression: { disponible: true } })
db.setProfilingLevel(1, { slowms: 100 })
db.system.profile.find().sort({ millis: -1 }).limit(5)
db.livres.getIndexes()
db.livres.dropIndex("nom_de_l_index")


## Exercice 2.1 :

db.utilisateurs.insertMany([
  {
    nom: "Jean Dupont",
    adresse: { rue: "12 Rue de Paris", ville: "Paris", code_postal: "75001" },
    localisation: { type: "Point", coordinates: [2.3522, 48.8566] } // Paris
  },
  {
    nom: "Marie Curie",
    adresse: { rue: "8 Place Bellecour", ville: "Lyon", code_postal: "69002" },
    localisation: { type: "Point", coordinates: [4.8357, 45.7640] } // Lyon
  }
])

## Exercice 2.1 :

db.bibliotheques.insertMany([
  {
    nom: "Bibliothèque Nationale",
    adresse: { ville: "Paris", rue: "Quai François Mauriac", code_postal: "75013" },
    localisation: { type: "Point", coordinates: [2.3730, 48.8331] },
    zone_service: {
      type: "Polygon",
      coordinates: [[
        [2.35, 48.83], [2.38, 48.83], [2.38, 48.85], [2.35, 48.85], [2.35, 48.83]
      ]]
    }
  }
])

## Exercice 2.1 :

db.utilisateurs.createIndex({ localisation: "2dsphere" })
db.bibliotheques.createIndex({ localisation: "2dsphere" })
db.bibliotheques.createIndex({ zone_service: "2dsphere" })


## Exercice 2.2 : 

db.utilisateurs.find({
  localisation: {
    $near: {
      $geometry: { type: "Point", coordinates: [2.3522, 48.8566] }, // Paris
      $maxDistance: 5000 // 5 km
    }
  }
}).limit(5)

## Exercice 2.2 : 

let user = db.utilisateurs.findOne({ _id: ObjectId("65d3b2f4e4c1a4b78a23abcd") })
db.bibliotheques.find({
  localisation: {
    $near: {
      $geometry: user.localisation,
      $maxDistance: 10000 // 10 km
    }
  }
})


## Exercice 2.2 : 

db.bibliotheques.aggregate([
  {
    $geoNear: {
      near: { type: "Point", coordinates: [2.3522, 48.8566] }, // Paris
      distanceField: "distance_km",
      maxDistance: 10000, // 10 km
      spherical: true
    }
  }
])


## Exercice 2.3 :

db.utilisateurs.find({
  localisation: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [2.34, 48.83], [2.38, 48.83], [2.38, 48.85], [2.34, 48.85], [2.34, 48.83]
        ]]
      }
    }
  }
})


## Exerice 2.3 :

db.utilisateurs.find({
  localisation: {
    $geoWithin: {
      $geometry: db.bibliotheques.findOne({ nom: "Bibliothèque Nationale" }).zone_service
    }
  }
})


## Exercice 2.3 :

db.rues.insertOne({
  nom: "Rue Saint-Michel",
  trace: {
    type: "LineString",
    coordinates: [[2.34, 48.83], [2.37, 48.84]]
  }
})

db.bibliotheques.find({
  zone_service: {
    $geoIntersects: {
      $geometry: db.rues.findOne({ nom: "Rue Saint-Michel" }).trace
    }
  }
})


## Exercice 2.4 :

db.livraisons.insertOne({
  livre: ObjectId("65d3b2f4e4c1a4b78a23abcd"),
  utilisateur: ObjectId("65d3b2f4e4c1a4b78a23defg"),
  position_livreur: { type: "Point", coordinates: [2.35, 48.84] },
  itineraire: {
    type: "LineString",
    coordinates: [[2.35, 48.84], [2.36, 48.85], [2.37, 48.86]]
  }
})


db.livraisons.updateOne(
  { _id: ObjectId("65d3b2f4e4c1a4b78a23abcd") },
  { $set: { position_livreur: { type: "Point", coordinates: [2.36, 48.85] } } }
)

db.livraisons.find({
  position_livreur: {
    $near: {
      $geometry: { type: "Point", coordinates: [2.35, 48.84] },
      $maxDistance: 1000
    }
  }
})


## Exercice 3.1 :

db.livres.aggregate([
  {
    $group: {
      _id: "$genre",
      nombre_de_livres: { $sum: 1 },
      note_moyenne: { $avg: "$note" },
      prix_moyen: { $avg: "$prix" },
      prix_minimum: { $min: "$prix" },
      prix_maximum: { $max: "$prix" }
    }
  }
])

db.livres.aggregate([
  {
    $group: {
      _id: "$editeur",
      nombre_de_livres: { $sum: 1 },
      nombre_de_genres: { $addToSet: "$genre" },
      nombre_d_auteurs: { $addToSet: "$auteur" },
      note_moyenne: { $avg: "$note" }
    }
  }
])

db.emprunts.aggregate([
  {
    $lookup: {
      from: "utilisateurs",
      localField: "utilisateur_id",
      foreignField: "_id",
      as: "utilisateur"
    }
  },
  { $unwind: "$utilisateur" },
  {
    $group: {
      _id: "$utilisateur.adresse.ville",
      nombre_emprunts: { $sum: 1 }
    }
  }
])


## Exercice 3.2 : 
db.emprunts.aggregate([
  {
    $project: {
      duree: { $subtract: ["$date_retour", "$date_emprunt"] }
    }
  },
  {
    $group: {
      _id: null,
      duree_moyenne: { $avg: "$duree" },
      duree_min: { $min: "$duree" },
      duree_max: { $max: "$duree" },
      retard: { $sum: { $cond: [{ $gte: ["$duree", 14 * 24 * 60 * 60 * 1000] }, 1, 0] } }
    }
  }
])

db.emprunts.aggregate([
  {
    $group: {
      _id: "$livre_id",
      nombre_emprunts: { $sum: 1 }
    }
  },
  { $sort: { nombre_emprunts: -1 } },
  { $limit: 5 }
])

db.livres.aggregate([
  {
    $facet: {
      stats_generales: [
        {
          $group: {
            _id: null,
            total_livres: { $sum: 1 },
            prix_moyen: { $avg: "$prix" },
            note_moyenne: { $avg: "$note" }
          }
        }
      ],
      top_livres: [
        { $sort: { note: -1 } },
        { $limit: 5 },
        { $project: { _id: 0, titre: 1, note: 1 } }
      ],
      repartition_langue: [
        { $group: { _id: "$langue", nombre: { $sum: 1 } } }
      ],
      repartition_decennie: [
        {
          $project: {
            decennie: { $subtract: [{ $year: "$date_publication" }, { $mod: [{ $year: "$date_publication" }, 10] }] }
          }
        },
        { $group: { _id: "$decennie", nombre: { $sum: 1 } } }
      ]
    }
  }
])


# Exercice 3.3 : 
let genres_preferes = db.emprunts.aggregate([
  { $match: { utilisateur_id: ObjectId("65d3b2f4e4c1a4b78a23abcd") } },
  { $lookup: { from: "livres", localField: "livre_id", foreignField: "_id", as: "livre" } },
  { $unwind: "$livre" },
  { $group: { _id: "$livre.genre", nombre_emprunts: { $sum: 1 } } },
  { $sort: { nombre_emprunts: -1 } },
  { $limit: 1 }
]).toArray();

db.livres.find({ genre: genres_preferes[0]._id }).limit(5)

db.livres.aggregate([
  {
    $lookup: {
      from: "emprunts",
      localField: "_id",
      foreignField: "livre_id",
      as: "emprunts"
    }
  },
  { $match: { emprunts: { $size: 0 } } },
  { $project: { titre: 1, _id: 0 } }
])

db.emprunts.aggregate([
  {
    $group: {
      _id: { mois: { $month: "$date_emprunt" }, annee: { $year: "$date_emprunt" } },
      nombre_emprunts: { $sum: 1 }
    }
  },
  { $sort: { "_id.annee": 1, "_id.mois": 1 } }
])


## Exercice 3.4 : 

db.createView(
  "stats_livres",
  "livres",
  [
    { $group: { _id: "$genre", nombre_de_livres: { $sum: 1 }, note_moyenne: { $avg: "$note" } } }
  ]
)

db.livres.aggregate([
  { $match: { prix: { $gte: 10 } } },
  { $group: { _id: "$genre", moyenne_prix: { $avg: "$prix" } } }
]).explain("executionStats")

db.livres.aggregate(
  [
    { $group: { _id: "$genre", livres: { $push: "$$ROOT" } } }
  ],
  { allowDiskUse: true }
)
    